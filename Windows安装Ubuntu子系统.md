## Windows安装Ubuntu子系统

```shell
ls | grep .ssh # error
# 判断是否安装ssh服务
ps -e|grep ssh

ssh-keygen -t rsa -C "2951121599@qq.com"

# Linux:
ssh-keygen -t rsa -C "2951121599@qq.com"
cd /home/kiral/.ssh/
cat id_rsa.pub

或者
explorer.exe .
打开这个目录 \\wsl$\Ubuntu\home\kiral\.ssh
```

```shell
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD0mh6M3yuvKUDK8kYrkFQ6zvmS/zktZvs0p6Vz5R0QGWcELdqj706Vx37fOWjYyTlKjuS3waWnHv8D0+2sAMbylkawv8Zkyco7llwsLpsil91A/gKDIe8icUZaVo6wIJogv5cI3RYnbWETE83xQeQjWSbHFmGUPV4HebloB26ZxwU5GsHONZxz7thrNdbTHfzxA2FS6Em3ETOyZjBWRFSxxW40sdpoqcIiwiEm+qqjKBOFufR+j08O7uBKzPG4rz+xxw/eGtXRNj0DjaR2JS/VcSjL11ZBlB/lNUpbcXHuq1VwxkPa1XQ6uPDnXnhjsq35sHkPwiMy6xLs/V8++0FcfLephDODiBW5yECQfIyGWrDcuQf1wXOSYGKokBULDgKvKG9cq4VpglEsHBH67FXRFgfeB/hxKmx2IU4xB2Ma71iqLwykhI4pztkiBDBZd/1UdR+OHNv/CW5hKFNnXskLpMHl50m4bIjb+0CA7mMJybaI8s0cfgYiJrGyAzjrBec= 2951121599@qq.com
```

github 新建ssh链接 添加pub公钥

### 添加文件

```shell
git add .
```

### 设置账号邮箱

```shell
git config --global user.name "2951121599"
git config --global user.email "2951121599@qq.com"
```

### 合并两次提交

```shell
# 修正
git  commit --amend
```

### 查看日志

```shell
git log
```



### Windows安装WSL 2 (Linux子系统)

电脑规格要求 20H2

参考链接: https://docs.microsoft.com/zh-cn/windows/wsl/install-win10

步骤:

power shell 以管理员方式打开

```shell
# 部署映像服务和管理工具
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```



### 安装 zshell

```shell
sudo apt install zsh
```

### Linux换源

```shell
# 首先备份源列表
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup

# 打开sources.list文件
sudo vim /etc/apt/sources.list
(dd命令删除原来的源)
```

```she
// 阿里源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

```shell
# 更新源
sudo apt update
```

```shell
# 查看更新列表
sudo apt list --upgradable

# 执行更新
sudo apt upgrade
```



```shell
# 任务管理器
sudo apt install htop
```

