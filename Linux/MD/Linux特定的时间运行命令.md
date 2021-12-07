# Linux特定的时间运行命令

目的：Linux 操作系统中在特定的时间运行一个命令，并且一旦超时就自动结束命令。

## 方法一：使用 timeout 命令

timeout 命令会有效地限制一个进程的绝对执行时间。

timeout 命令是 GNU coreutils 包的一部分，因此它预装在所有 GNU/Linux 系统中。

假设你只想运行一个命令 N 秒钟，然后杀死它

```bash
timeout <time-limit-interval> <command>
```

例如，以下命令将在 10 秒后终止

```bash
timeout 10s tail -f /var/log/pacman.log
```

也可以不用在秒数后加后缀`s`，以下命令与上面的相同

```bash
timeout 10 tail -f /var/log/pacman.log
```

可用的后缀有：

- s 代表秒。
- m 代表分钟。
- h 代表小时。
- d 代表天。

如果运行这个 `tail -f /var/log/pacman.log` 命令，它将继续运行，直到你按 CTRL+C 手动结束它。但是，如果你使用 timeout 命令运行它，它将在给定的时间间隔后自动终止。如果该命令在超时后仍在运行，则可以发送 kill 信号，如下所示。

```bash
#在这种情况下，如果 tail 命令在 10 秒后仍然运行，timeout 命令将在 20 秒后发送一个 kill 信号并结束。
timeout -k 20 10 tail -f /var/log/pacman.log
```

有关更多详细信息，请查看手册页。

```bash
man timeout
```

有时，某个特定程序可能需要很长时间才能完成并最终冻结你的系统。在这种情况下，你可以使用此技巧在特定时间后自动结束该进程。

## 方法二：使用 timelimit 程序

timelimit 使用提供的参数执行给定的命令，并在给定的时间后使用给定的信号终止进程。首先，它会发送警告信号，然后在超时后发送 kill 信号。

与 timeout 不同，timelimit 有更多选项。你可以传递参数数量，如 killsig、warnsig、killtime、warntime 等。

它存在于基于 Debian 的系统的默认仓库中。所以，你可以使用命令来安装它：

```bash
sudo apt-get install timelimit
```

对于基于 Arch 的系统，它在 AUR 中存在。因此，你可以使用任何 AUR 助手进行安装，例如 Pacaur 、 Packer 、 Yay 、 Yaourt 等。

对于其他发行版，请 在这里 下载源码并手动安装。安装 timelimit 后，运行下面的命令执行一段特定的时间，例如 10 秒钟：

```bash
timelimit -t10 tail -f /var/log/pacman.log
```

如果不带任何参数运行 timelimit，它将使用默认值：warntime=3600 秒、warnsig=15 秒、killtime=120 秒、killsig=9。有关更多详细信息，请参阅本指南最后给出的手册页和项目网站。

```bash
man timelimit
```

## 参考资料

- [如何在Linux中的特定时间运行命令](https://linux.cn/article-9794-1.html)
