### 定义
NFS是Network File System 的缩写，它可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。

NFS服务器可以让PC将网络中的NFS服务器共享的目录挂载到本地端的文件系统中，而在本地端的系统中来看，那个远程主机的目录就好像是自己的一个磁盘分区一样，在使用上相当便利；

NFS服务器我们一般是在ubuntu上搭建的。这里的客户端我们使用的是开发板，如下图所示。
![[NFS服务器与客户端.png]]

### 前置设置
- 由三部分组成：`window`主机、`Ubuntu`虚拟机、`Linux`开发板
- 首先确保`window`主机和`Ubuntu`虚拟机间通过**桥接模式**通信，即两者的网段一致（子网掩码和`IP`地址相与，一般是`IP`地址前三位相同）
- 开发板需通过网线或者`WiFi`连接电脑所连接的*路由器*，不可通过直接连接到电脑网口（网段不一致），并再次确认开发板的[[LAN与网段#子网（网段）|网段]]是否一致
- 确保三者都能互相`ping`通，并先关闭电脑*防火墙*，在服务搭建成功后可以尝试开启并确认是否有影响
### 搭建流程
#### NFS服务器-Ubuntu虚拟机
- 安装NFS
```shell
sudo apt-get install -y nfs-kernel-server nfs-common portmap
```

- 创建共享文件夹
```shell
sudo mkdir /home/nfs
```

- 打开配置文件添加NFS共享目录
> NFS配置文件为`/etc/exports`，需要正确设置共享目录和客户端访问权限。
> 权限设置包括读写（rw）、只读（ro）、同步写入（sync）、异步写入（async）等选项
> 用户身份映射选项，如`root_squash`和`all_squash`，控制客户端root用户访问共享目录时的权限
> 需要确保NFS服务和操作系统层面的权限设置一致，因为NFS配置的权限在文件本身的权限面前是不起作用的
```shell
sudo vi /etc/exports
```

- 在`/etc/exports`添加下列内容
> `home/nfs/` 是` nfs `服务器要共享的目录
> `rw`:是可读写权限
> `sync`:是资料同步写入内存和硬盘
> `no_root_squash`:当登录NFS主机使用共享目录的使用者是`root`时，其权限将被转换成为一名使用者，通常它的`UID`与`GID `都会变成 `nobody`身份。
```shell
/home/nfs *(rw,sync,no_root_squash)
```

- 重启`NFS`服务
```shell
/etc/init.d/nfs-kernel-server restart
```
显示`OK`就说明`NFS`服务器搭建成功
#### 客户端-Linux开发板
- 在客户端，创建一个挂载点并尝试挂载NFS共享
```shell
mkdir /mnt/nfs
sudo mount -t nfs -o nolock <服务器IP地址>:/home/nfs /mnt/nfs
```

- 检查挂载结果
```shell
df -h
```
如果挂载成功，在输出中看到`/mnt/nfs`

sudo mount -t nfs -o nolock 192.168.3.97:/home/nfs /mnt/nfs