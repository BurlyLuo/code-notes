# Docker限制容器可用的内存与可用的CPU

- [Docker限制容器可用的内存与可用的CPU](#docker限制容器可用的内存与可用的cpu)
  - [限制容器可用的内存](#限制容器可用的内存)
    - [为什么要限制容器对内存的使用](#为什么要限制容器对内存的使用)
  - [cgroup简介](#cgroup简介)
  - [内存限制](#内存限制)
    - [内存限制相关的参数](#内存限制相关的参数)
    - [用户内存限制](#用户内存限制)
    - [Memory reservation](#memory-reservation)
    - [OOM killer](#oom-killer)
    - [核心内存](#核心内存)
    - [Swappiness](#swappiness)
  - [CPU 限制](#cpu-限制)
    - [概述](#概述)
    - [CPU 限制相关参数](#cpu-限制相关参数)
    - [CPU 集](#cpu-集)
    - [CPU 资源的相对限制](#cpu-资源的相对限制)
    - [CPU 资源的绝对限制](#cpu-资源的绝对限制)
    - [正确的理解“绝对”](#正确的理解绝对)
  - [磁盘IO配额控制](#磁盘io配额控制)
    - [block IO 权重](#block-io-权重)
    - [限制 bps 和 iops](#限制-bps-和-iops)
  - [启动一个压力测试容器并限制资源](#启动一个压力测试容器并限制资源)
  - [参考](#参考)

## 限制容器可用的内存

默认情况下容器使用的资源是不受限制的。也就是可以使用主机内核调度器所允许的最大资源。但是在容器的使用过程中，经常需要对容器可以使用的主机资源进行限制，本文介绍如何限制容器可以使用的主机内存。

### 为什么要限制容器对内存的使用

限制容器不能过多的使用主机的内存是非常重要的。对于 linux 主机来说，一旦内核检测到没有足够的内存可以分配，就会扔出 OOME(Out Of Memmory Exception)，并开始杀死一些进程用于释放内存空间。糟糕的是任何进程都可能成为内核猎杀的对象，包括 docker daemon 和其它一些重要的程序。更危险的是如果某个支持系统运行的重要进程被干掉了，整个系统也就宕掉了！这里我们考虑一个比较常见的场景，大量的容器把主机的内存消耗殆尽，OOME 被触发后系统内核立即开始杀进程释放内存。如果内核杀死的第一个进程就是 docker daemon 会怎么样？结果是所有的容器都不工作了，这是不能接受的！

针对这个问题，docker 尝试通过调整 docker daemon 的 OOM 优先级来进行缓解。内核在选择要杀死的进程时会对所有的进程打分，直接杀死得分最高的进程，接着是下一个。当 docker daemon 的 OOM 优先级被降低后(注意容器进程的 OOM 优先级并没有被调整)，docker daemon 进程的得分不仅会低于容器进程的得分，还会低于其它一些进程的得分。这样 docker daemon 进程就安全多了。

我们可以通过下面的脚本直观的看一下当前系统中所有进程的得分情况：

```bash
#!/bin/bash
for proc in $(find /proc -maxdepth 1 -regex '/proc/[0-9]+'); do
    printf "%2d %5d %s\n" \
        "$(cat $proc/oom_score)" \
        "$(basename $proc)" \
        "$(cat $proc/cmdline | tr '\0' ' ' | head -c 50)"
done 2>/dev/null | sort -nr | head -n 40
```

此脚本输出得分最高的 40 个进程，并进行了排序：

![example](/Docker/IMG/014.png)

第一列显示进程的得分，红框中的是 docker daemon 进程，非常的靠后。

有了上面的机制后是否就可以高枕无忧了呢！不是的，docker 的官方文档中一直强调这只是一种缓解的方案，并且为我们提供了一些降低风险的建议：

- 通过测试掌握应用对内存的需求
- 保证运行容器的主机有充足的内存
- 限制容器可以使用的内存
- 为主机配置 swap

总结：**通过限制容器使用的内存上限，可以降低主机内存耗尽时带来的各种风险**。

## cgroup简介

docker 通过 cgroup 来控制容器使用的资源配额，包括 CPU、内存、磁盘三大方面，基本覆盖了常见的资源配额和使用量控制。

cgroup是Control Groups的缩写，是Linux 内核提供的一种可以限制、记录、隔离进程组所使用的物理资源(如 cpu、memory、磁盘IO等等) 的机制，被LXC、docker等很多项目用于实现进程资源控制。cgroup将任意进程进行分组化管理的 Linux 内核功能。cgroup本身是提供将进程进行分组化管理的功能和接口的基础结构，I/O 或内存的分配控制等具体的资源管理功能是通过这个功能来实现的。这些具体的资源管理功能称为cgroup子系统，有以下几大子系统实现：

- blkio：设置限制每个块设备的输入输出控制。例如:磁盘，光盘以及usb等等。
- cpu：使用调度程序为cgroup任务提供cpu的访问。
- cpuacct：产生cgroup任务的cpu资源报告。
- cpuset：如果是多核心的cpu，这个子系统会为cgroup任务分配单独的cpu和内存。
- devices：允许或拒绝cgroup任务对设备的访问。
- freezer：暂停和恢复cgroup任务。
- memory：设置每个cgroup的内存限制以及产生内存资源报告。
- net_cls：标记每个网络包以供cgroup方便使用。
- ns：命名空间子系统。
- perf_event：增加了对每group的监测跟踪的能力，即可以监测属于某个特定的group的所有线程以及运行在特定CPU上的线程。

目前docker只是用了其中一部分子系统，实现对资源配额和使用的控制。

## 内存限制

Docker 提供的内存限制功能有以下几点：

- 容器能使用的内存和交换分区大小。
- 容器的核心内存大小。
- 容器虚拟内存的交换行为。
- 容器内存的软性限制。
- 是否杀死占用过多内存的容器。
- 容器被杀死的优先级

一般情况下，达到内存限制的容器过段时间后就会被系统杀死。

### 内存限制相关的参数

执行docker run命令时能使用的和内存限制相关的所有选项如下。

| 选项                 | 描述                                                          |
| -------------------- | ------------------------------------------------------------- |
| -m,--memory          | 内存限制，格式是数字加单位，单位可以为 b,k,m,g。最小为 4M     |
| --memory-swap        | 内存+交换分区大小总限制。格式同上。必须必-m设置的大           |
| --memory-reservation | 内存的软性限制。格式同上                                      |
| --oom-kill-disable   | 是否阻止 OOM killer 杀死容器，默认没设置                      |
| --oom-score-adj      | 容器被 OOM killer 杀死的优先级，范围是[-1000, 1000]，默认为 0 |
| --memory-swappiness  | 用于设置容器的虚拟内存控制行为。值为 0~100 之间的整数         |
| --kernel-memory      | 核心内存限制。格式同上，最小为 4M                             |

### 用户内存限制

用户内存限制就是对容器能使用的内存和交换分区的大小作出限制。

使用时要遵循两条直观的规则：

- -m，--memory选项的参数最小为 4 M。
- --memory-swap不是交换分区，而是内存加交换分区的总大小，所以--memory-swap必须比-m,--memory大。

在这两条规则下，一般有四种设置方式。

```text
你可能在进行内存限制的实验时发现docker run命令报错：WARNING: Your kernel does not support swap limit capabilities, memory limited without swap.

这是因为宿主机内核的相关功能没有打开。按照下面的设置就行。

step 1：编辑/etc/default/grub文件，将GRUB_CMDLINE_LINUX一行改为GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"

step 2：更新 GRUB，即执行$ sudo update-grub

step 3: 重启系统。
```

**不设置**：

如果不设置-m,--memory和--memory-swap，容器默认可以用完宿舍机的所有内存和 swap 分区。不过注意，如果容器占用宿主机的所有内存和 swap 分区超过一段时间后，会被宿主机系统杀死（如果没有设置--00m-kill-disable=true的话）。

**设置-m,--memory，不设置--memory-swap**：

给-m或--memory设置一个不小于 4M 的值，假设为 a，不设置--memory-swap，或将--memory-swap设置为 0。这种情况下，容器能使用的内存大小为 a，能使用的交换分区大小也为 a。因为 Docker 默认容器交换分区的大小和内存相同。

如果在容器中运行一个一直不停申请内存的程序，你会观察到该程序最终能占用的内存大小为 2a。

比如`$ docker run -m 1G ubuntu:16.04`，该容器能使用的内存大小为 1G，能使用的 swap 分区大小也为 1G。容器内的进程能申请到的总内存大小为 2G。

**设置-m,--memory=a，--memory-swap=b，且b > a**：

给-m设置一个参数 a，给--memory-swap设置一个参数 b。a 是容器能使用的内存大小，b 是容器能使用的内存大小 + swap 分区大小。所以 b 必须大于 a。b - a 即为容器能使用的 swap 分区大小。

比如$ docker run -m 1G --memory-swap 3G ubuntu:16.04，该容器能使用的内存大小为 1G，能使用的 swap 分区大小为 2G。容器内的进程能申请到的总内存大小为 3G。

**设置-m,--memory=a，--memory-swap=-1**：

给-m参数设置一个正常值，而给--memory-swap设置成 -1。这种情况表示限制容器能使用的内存大小为 a，而不限制容器能使用的 swap 分区大小。

这时候，容器内进程能申请到的内存大小为 a + 宿主机的 swap 大小。

### Memory reservation

这种 memory reservation 机制不知道怎么翻译比较形象。Memory reservation 是一种软性限制，用于节制容器内存使用。给--memory-reservation设置一个比-m小的值后，虽然容器最多可以使用-m使用的内存大小，但在宿主机内存资源紧张时，在系统的下次内存回收时，系统会回收容器的部分内存页，强迫容器的内存占用回到--memory-reservation设置的值大小。

没有设置时（默认情况下）--memory-reservation的值和-m的限定的值相同。将它设置为 0 会设置的比-m的参数大 等同于没有设置。

Memory reservation 是一种软性机制，它不保证任何时刻容器使用的内存不会超过--memory-reservation限定的值，它只是确保容器不会长时间占用超过--memory-reservation限制的内存大小。

例如：

```bash
docker run -it -m 500M --memory-reservation 200M ubuntu:16.04 /bin/bash
```

如果容器使用了大于 200M 但小于 500M 内存时，下次系统的内存回收会尝试将容器的内存锁紧到 200M 以下。

例如：

```bash
docker run -it --memory-reservation 1G ubuntu:16.04 /bin/bash
```

容器可以使用尽可能多的内存。--memory-reservation确保容器不会长时间占用太多内存。

### OOM killer

默认情况下，在出现 out-of-memory(OOM) 错误时，系统会杀死容器内的进程来获取更多空闲内存。这个杀死进程来节省内存的进程，我们姑且叫它 OOM killer。我们可以通过设置--oom-kill-disable选项来禁止 OOM killer 杀死容器内进程。但请确保只有在使用了-m/--memory选项时才使用--oom-kill-disable禁用 OOM killer。如果没有设置-m选项，却禁用了 OOM-killer，可能会造成出现 out-of-memory 错误时，系统通过杀死宿主机进程或获取更改内存。

下面的例子限制了容器的内存为 100M 并禁止了 OOM killer：

```bash
docker run -it -m 100M --oom-kill-disable ubuntu:16.04 /bin/bash
```

是正确的使用方法。

而下面这个容器没设置内存限制，却禁用了 OOM killer 是非常危险的：

```bash
docker run -it --oom-kill-disable ubuntu:16.04 /bin/bash
```

容器没用内存限制，可能或导致系统无内存可用，并尝试时杀死系统进程来获取更多可用内存。

一般一个容器只有一个进程，这个唯一进程被杀死，容器也就被杀死了。我们可以通过--oom-score-adj选项来设置在系统内存不够时，容器被杀死的优先级。负值更教不可能被杀死，而正值更有可能被杀死。

### 核心内存

核心内存和用户内存不同的地方在于核心内存不能被交换出。不能交换出去的特性使得容器可以通过消耗太多内存来堵塞一些系统服务。核心内存包括：

- stack pages（栈页面）
- slab pages
- socket memory pressure
- tcp memory pressure

可以通过设置核心内存限制来约束这些内存。例如，每个进程都要消耗一些栈页面，通过限制核心内存，可以在核心内存使用过多时阻止新进程被创建。

核心内存和用户内存并不是独立的，必须在用户内存限制的上下文中限制核心内存。

假设用户内存的限制值为 U，核心内存的限制值为 K。有三种可能地限制核心内存的方式：

1. U != 0，不限制核心内存。这是默认的标准设置方式
1. K < U，核心内存时用户内存的子集。这种设置在部署时，每个 cgroup 的内存总量被过度使用。过度使用核心内存限制是绝不推荐的，因为系统还是会用完不能回收的内存。在这种情况下，你可以设置 K，这样 groups 的总数就不会超过总内存了。然后，根据系统服务的质量自有地设置 U。
1. K > U，因为核心内存的变化也会导致用户计数器的变化，容器核心内存和用户内存都会触发回收行为。这种配置可以让管理员以一种统一的视图看待内存。对想跟踪核心内存使用情况的用户也是有用的。

例如：

```bash
docker run -it -m 500M --kernel-memory 50M ubuntu:16.04 /bin/bash
```

容器中的进程最多能使用 500M 内存，在这 500M 中，最多只有 50M 核心内存。

```bash
docker run -it --kernel-memory 50M ubuntu:16.04 /bin/bash
```

没用设置用户内存限制，所以容器中的进程可以使用尽可能多的内存，但是最多能使用 50M 核心内存。

### Swappiness

默认情况下，容器的内核可以交换出一定比例的匿名页。--memory-swappiness就是用来设置这个比例的。--memory-swappiness可以设置为从 0 到 100。0 表示关闭匿名页面交换。100 表示所有的匿名页都可以交换。默认情况下，如果不适用--memory-swappiness，则该值从父进程继承而来。

例如：

```bash
docker run -it --memory-swappiness=0 ubuntu:16.04 /bin/bash
```

将--memory-swappiness设置为 0 可以保持容器的工作集，避免交换代理的性能损失。

示例：

```bash
docker run -tid —name mem1 —memory 128m ubuntu:16.04 /bin/bash
cat /sys/fs/cgroup/memory/docker/<容器的完整ID>/memory.limit_in_bytes
cat /sys/fs/cgroup/memory/docker/<容器的完整ID>/memory.memsw.limit_in_bytes
```

## CPU 限制

### 概述

Docker 的资源限制和隔离完全基于 Linux cgroups。对 CPU 资源的限制方式也和 cgroups 相同。Docker 提供的 CPU 资源限制选项可以在多核系统上限制容器能利用哪些 vCPU。而对容器最多能使用的 CPU 时间有两种限制方式：一是有多个 CPU 密集型的容器竞争 CPU 时，设置各个容器能使用的 CPU 时间相对比例。二是以绝对的方式设置容器在每个调度周期内最多能使用的 CPU 时间。

### CPU 限制相关参数

docker run命令和 CPU 限制相关的所有选项如下：

| 选项              | 描述                                                    |
| ----------------- | ------------------------------------------------------- |
| --cpuset-cpus=""  | 允许使用的 CPU 集，值可以为 0-3,0,1                     |
| -c,--cpu-shares=0 | CPU 共享权值（相对权重）                                |
| cpu-period=0      | 限制 CPU CFS 的周期，范围从 100ms~1s，即[1000, 1000000] |
| --cpu-quota=0     | 限制 CPU CFS 配额，必须不小于1ms，即 >= 1000            |
| --cpuset-mems=""  | 允许在上执行的内存节点（MEMs），只对 NUMA 系统有效      |

其中--cpuset-cpus用于设置容器可以使用的 vCPU 核。-c,--cpu-shares用于设置多个容器竞争 CPU 时，各个容器相对能分配到的 CPU 时间比例。--cpu-period和--cpu-quata用于绝对设置容器能使用 CPU 时间。

--cpuset-mems暂用不上，这里不谈。

### CPU 集

我们可以设置容器可以在哪些 CPU 核上运行。

例如：

```bash
docker run -it --cpuset-cpus="1,3" ubuntu:14.04 /bin/bash
```

表示容器中的进程可以在 cpu 1 和 cpu 3 上执行。

```bash
docker run -it --cpuset-cpus="0-2" ubuntu:14.04 /bin/bash
cat /sys/fs/cgroup/cpuset/docker/<容器的完整长ID>/cpuset.cpus
```

表示容器中的进程可以在 cpu 0、cpu 1 及 cpu 3 上执行。

在 NUMA 系统上，我们可以设置容器可以使用的内存节点。

例如：

```bash
docker run -it --cpuset-mems="1,3" ubuntu:14.04 /bin/bash
```

表示容器中的进程只能使用内存节点 1 和 3 上的内存。

```bash
docker run -it --cpuset-mems="0-2" ubuntu:14.04 /bin/bash
```

表示容器中的进程只能使用内存节点 0、1、2 上的内存。

### CPU 资源的相对限制

默认情况下，所有的容器得到同等比例的 CPU 周期。在有多个容器竞争 CPU 时我们可以设置每个容器能使用的 CPU 时间比例。这个比例叫作共享权值，通过-c或--cpu-shares设置。Docker 默认每个容器的权值为 1024。不设置或将其设置为 0，都将使用这个默认值。系统会根据每个容器的共享权值和所有容器共享权值和比例来给容器分配 CPU 时间。

假设有三个正在运行的容器，这三个容器中的任务都是 CPU 密集型的。第一个容器的 cpu 共享权值是 1024，其它两个容器的 cpu 共享权值是 512。第一个容器将得到 50% 的 CPU 时间，而其它两个容器就只能各得到 25% 的 CPU 时间了。如果再添加第四个 cpu 共享值为 1024 的容器，每个容器得到的 CPU 时间将重新计算。第一个容器的CPU 时间变为 33%，其它容器分得的 CPU 时间分别为 16.5%、16.5%、33%。

必须注意的是，这个比例只有在 CPU 密集型的任务执行时才有用。在四核的系统上，假设有四个单进程的容器，它们都能各自使用一个核的 100% CPU 时间，不管它们的 cpu 共享权值是多少。

在多核系统上，CPU 时间权值是在所有 CPU 核上计算的。即使某个容器的 CPU 时间限制少于 100%，它也能使用各个 CPU 核的 100% 时间。

例如，假设有一个不止三核的系统。用-c=512的选项启动容器{C0}，并且该容器只有一个进程，用-c=1024的启动选项为启动容器C2，并且该容器有两个进程。CPU 权值的分布可能是这样的：

```bash
PID    container    CPU CPU share
100    {C0}     0   100% of CPU0
101    {C1}     1   100% of CPU1
102    {C1}     2   100% of CPU2
```

示例：

```bash
docker run -it --cpu-shares=100 ubuntu:14.04 /bin/bash
cat /sys/fs/cgroup/cpu/docker/<容器的完整长ID>/cpu.shares
```

表示容器中的进程CPU份额值为100。

### CPU 资源的绝对限制

Linux 通过 CFS（Completely Fair Scheduler，完全公平调度器）来调度各个进程对 CPU 的使用。CFS 默认的调度周期是 100ms。

关于 CFS 的更多信息，参考[CFS documentation on bandwidth limiting](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt)

我们可以设置每个容器进程的调度周期，以及在这个周期内各个容器最多能使用多少 CPU 时间。使用--cpu-period即可设置调度周期，使用--cpu-quota即可设置在每个周期内容器能使用的 CPU 时间。两者一般配合使用。

例如：

```bash
docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:16.04 /bin/bash
```

将 CFS 调度的周期设为 50000，将容器在每个周期内的 CPU 配额设置为 25000，表示该容器每 50ms 可以得到 50% 的 CPU 运行时间。

```bash
docker run -it --cpu-period=10000 --cpu-quota=20000 ubuntu:16.04 /bin/bash
cat /sys/fs/cgroup/cpu/docker/<容器的完整长ID>/cpu.cfs_period_us
cat /sys/fs/cgroup/cpu/docker/<容器的完整长ID>/cpu.cfs_quota_us
```

将容器的 CPU 配额设置为 CFS 周期的两倍，CPU 使用时间怎么会比周期大呢？其实很好解释，给容器分配两个 vCPU 就可以了。该配置表示容器可以在每个周期内使用两个 vCPU 的 100% 时间。

CFS 周期的有效范围是 1ms~1s，对应的--cpu-period的数值范围是 1000~1000000。而容器的 CPU 配额必须不小于 1ms，即--cpu-quota的值必须 >= 1000。可以看出这两个选项的单位都是 us。

### 正确的理解“绝对”

注意前面我们用--cpu-quota设置容器在一个调度周期内能使用的 CPU 时间时实际上设置的是一个上限。并不是说容器一定会使用这么长的 CPU 时间。比如，我们先启动一个容器，将其绑定到 cpu 1 上执行。给其--cpu-quota和--cpu-period都设置为 50000。

```bash
docker run --rm --name test01 --cpu-cpus 1 --cpu-quota=50000 --cpu-period=50000 deadloop:busybox-1.25.1-glibc
```

调度周期为 50000，容器在每个周期内最多能使用 50000 cpu 时间。

再用docker stats test01可以观察到该容器对 CPU 的使用率在100%左右。然后，我们再以同样的参数启动另一个容器。

```bash
docker run --rm --name test02 --cpu-cpus 1 --cpu-quota=50000 --cpu-period=50000 deadloop:busybox-1.25.1-glibc
```

再用docker stats test01 test02可以观察到这两个容器，每个容器对 cpu 的使用率在 50% 左右。说明容器并没有在每个周期内使用 50000 的 cpu 时间。

使用docker stop test02命令结束第二个容器，再加一个参数-c 2048启动它：

```bash
docker run --rm --name test02 --cpu-cpus 1 --cpu-quota=50000 --cpu-period=50000 -c 2048 deadloop:busybox-1.25.1-glibc
```

再用docker stats test01命令可以观察到第一个容器的 CPU 使用率在 33% 左右，第二个容器的 CPU 使用率在 66% 左右。因为第二个容器的共享值是 2048，第一个容器的默认共享值是 1024，所以第二个容器在每个周期内能使用的 CPU 时间是第一个容器的两倍。

## 磁盘IO配额控制

Block IO 是另一种可以限制容器使用的资源。Block IO 指的是磁盘的读写，docker 可通过设置权重、限制 bps 和 iops 的方式控制容器读写磁盘的带宽。

相对于CPU和内存的配额控制，docker对磁盘IO的控制相对不成熟，大多数都必须在有宿主机设备的情况下使用。主要包括以下参数：

- –device-read-bps：限制此设备上的读速度（bytes per second），单位可以是kb、mb或者gb。
- –device-read-iops：通过每秒读IO次数来限制指定设备的读速度。
- –device-write-bps ：限制此设备上的写速度（bytes per second），单位可以是kb、mb或者gb。
- –device-write-iops：通过每秒写IO次数来限制指定设备的写速度。
- –blkio-weight：容器默认磁盘IO的加权值，有效值范围为10-100。
- –blkio-weight-device： 针对特定设备的IO加权控制。其格式为DEVICE_NAME:WEIGHT

存储配额控制的相关参数，可以参考[Red Hat文档 - blkio](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch-subsystems_and_tunable_parameters#sec-blkio)，了解它们的详细作用。

### block IO 权重

默认情况下，所有容器能平等地读写磁盘，可以通过设置--blkio-weight参数来改变容器 block IO 的优先级。

要使–blkio-weight生效，需要保证IO的调度算法为CFQ。可以使用下面的方式查看：

```bash
cat /sys/block/vda/queue/scheduler
```

这里`/sys/block/vda/queue/scheduler`是我云主机上的文件路径，不同的机器可能不一样。

--blkio-weight 与 --cpu-shares 类似，设置的是相对权重值，默认为 500。在下面的例子中，container_A 读写磁盘的带宽是 container_B 的两倍。

```bash
docker run -it --name container_A --blkio-weight 600 ubuntu
docker run -it --name container_B --blkio-weight 300 ubuntu
```

同样的，我们可以在 /sys/fs/cgroup/blkio/docker/<容器ID> 看到 block IO 的数值。

### 限制 bps 和 iops

bps 是 byte per second，每秒读写的数据量。

iops 是 io per second，每秒 IO 的次数。

可通过以下参数控制容器的 bps 和 iops：

- --device-read-bps，限制读某个设备的 bps。
- --device-write-bps，限制写某个设备的 bps。
- --device-read-iops，限制读某个设备的 iops。
- --device-write-iops，限制写某个设备的 iops。

下面这个例子限制容器写 /dev/vda 的速率为 30 MB/s

```bash
docker run -it --device-write-bps /dev/vda:30MB ubuntu
```

通过 dd 测试在容器中写磁盘的速度。

```bash
dd if=/dev/zero of=test.out bs=1M count=100 oflag=direct
```

因为/dev/zero是一个伪设备，它只产生空字符流，对它不会产生IO，所以，IO都会集中在of文件中，of文件只用于写，所以这个命令相当于测试磁盘的写能力。命令结尾添加oflag=direct将跳过内存缓存，添加oflag=sync将跳过hdd缓存。

![example](/Docker/IMG/017.png)

命令执行完成后查看，结果表明，峰值大概限制在 30 MB/s 左右。

作为对比测试，如果不限速：

```bash
docker run -it ubuntu
```

结果如下：

![example](/Docker/IMG/018.png)

## 启动一个压力测试容器并限制资源

1.启动一个stress镜像

创建 u-stress 镜像的 Dockerfile：

```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
        apt-get install stress
```

在Dockerfile的目录下，创建镜像u-stress：

```bash
docker build -t u-stress:latest .
```

*附*：apt-get update特别慢的话，可以参照 [Ubuntu--更换软件源](https://blog.csdn.net/Meteor_s/article/details/81301252) 更换软件源。

启动一个u-stress容器，限制其内存为300M ，可用cpu数为2

```bash
docker run -it -d -m 300M --cpus=2 --memory-swap -1 --name os1 u-stress
```

命令说明：

- -m 选项限制容器使用的内存上限为 300M。
- --cpus=2 选项限制容器可用cpu数为2
- memory-swap 值为 -1，它表示容器程序使用内存的受限，而可以使用的 swap 空间使用不受限制(宿主机有多少 swap 容器就可以使用多少)。

2.启动容器后，可以使用docker 的监控指令查看容器的运行状态

- docker top 容器名： 查看容器的进程
- docker stats 容器名：查看容器的CPU，内存，IO 等使用信息

```bash
docker stats os1
```

![example](/Docker/IMG/015.png)

3.stress压测

在容器使用stress指令进行负载压测，先进入容器中

```bash
docker exec -it os1 bash
```

模拟出1个繁忙的进程消耗cpu，然后使用-m 模拟进程最大使用的内存数500M，使用--vm 指定进程数

```bash
stress -m 500m --vm 1
```

再查看容器运行状态，可以os1容器的内存和cpu都得到了限制，即使给压测时超出了最大内存，也不会额外占用资源

![example](/Docker/IMG/016.png)

附：Stress参数说明

- -? 显示帮助信息
- -v 显示版本号
- -q 不显示运行信息
- -n，--dry-run 显示已经完成的指令执行情况
- -t --timeout N 指定运行N秒后停止
  - --backoff N 等待N微妙后开始运行
- -c --cpu 产生n个进程 每个进程都反复不停的计算随机数的平方根
- -i --io  产生n个进程 每个进程反复调用sync()，sync()用于将内存上的内容写到硬盘上
- -m --vm n 产生n个进程,每个进程不断调用内存分配malloc和内存释放free函数
  - --vm-bytes B 指定malloc时内存的字节数 (默认256MB)
  - --vm-hang N 指示每个消耗内存的进程在分配到内存后转入休眠状态，与正常的无限分配和释放内存的处理相反，这有利于模拟只有少量内存的机器
- -d --hadd n 产生n个执行write和unlink函数的进程
  - --hadd-bytes B 指定写的字节数，默认是1GB
  - --hadd-noclean 不要将写入随机ASCII数据的文件Unlink
- 时间单位可以为秒s，分m，小时h，天d，年y，文件大小单位可以为K，M，G

## 参考

- [Docker: 限制容器可用的内存](https://www.cnblogs.com/sparkdev/p/8032330.html)
- [Docker: 限制容器可用的 CPU](https://www.cnblogs.com/sparkdev/p/8052522.html)
- [Docker容器CPU、memory资源限制](https://www.cnblogs.com/zhuochong/p/9728383.html)
- [Docker 容器的资源限制](https://blog.51cto.com/wzlinux/2046566)
