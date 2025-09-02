# 配置WSL

## WSL安装Rocky

本次配置的是rockylinux-10.0，可从官网或者镜像网站https://mirrors.aliyun.com/rockylinux/10.0/images/下载wsl文件

可参考：[将 Rocky Linux 导入 WSL 或 WSL2 ](https://docs.rockylinux.org/guides/interoperability/import_rocky_to_wsl/)

WSL安装后，可尝试将下载的`.wsl`文件右键打开即可进行安装，或者通过下面的命令进行安装：

```powershell
wsl --install --from-file <path-to/Rocky-10-WSL-Base.latest.x86_64.wsl> <machine-name>
```

下面是安装界面：

```sh
正在安装: D:\Download\Rocky-10-WSL-Base.latest.x86_64.wsl
已成功安装分发。可以通过 “wsl.exe -d rocky” 启动它=========]
正在启动 rocky...
Please create a default user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: spider
Your user has been created, is included in the wheel group, and can use sudo without a password.
To set a password for your user, run 'sudo passwd spider'
[spider@spider Spider]$ sudo passwd spider
......
[spider@spider Spider]$ sudo passwd root
......
```

修改用户信息：

```sh
$ id root
uid=0(root) gid=0(root) groups=0(root)
$ id spider
uid=1000(spider) gid=1000(spider) groups=1000(spider),10(wheel)
$ sudo usermod -a -G root spider
$ id spider
uid=1000(spider) gid=1000(spider) groups=1000(spider),0(root),10(wheel)
```

查看版本信息：

```sh
$ uname -a
Linux spider 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun  5 18:30:46 UTC 2025 x86_64 GNU/Linux
$ cat /etc/redhat-release
Rocky Linux release 10.0 (Red Quartz)
```

ifconfig安装：

```sh
sudo dnf install net-tools
```

安装passwd：

```sh
sudo dnf install passwd
```

安装防火墙：

```sh
sudo dnf install firewalld
```

ssh安装：

```sh
sudo dnf install openssh-server
service sshd restart
#查看是否启动22端口
sudo netstat -antp | grep sshd
```

执行以下命令替换默认源：

```sh
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    -i.bak \
    /etc/yum.repos.d/rocky-*.repo

dnf makecache
```

