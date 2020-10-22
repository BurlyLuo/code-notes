# WSL2（Ubuntu）安装Docker

WSL2增加了对docker的支持。

## 确认windows版本

windows版本必须在预览版189XX以后。

## 开启“适用于Linux的Windows子系统”

打开**Windows PowerShell**，以管理员身份运行，输入：

```bash
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

重启。

## 安装WSL

进微软商店，搜WSL，这里我选择Ubuntu。

## 切换为WSL2

打开**Windows PowerShell**，以管理员身份运行，输入：

```bash
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

重启。默认使用WSL2：

```bash
wsl --set-default-version 2
```

查看是不是WSL2：

```bash
wsl -l -v
```

设置Ubuntu默认用户为root：

```bash
ubuntu config --default-user root
```

## 安装docker

打开Ubuntu，输入：

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo service docker start
```
