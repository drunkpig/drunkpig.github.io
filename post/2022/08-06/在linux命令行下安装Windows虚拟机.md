
[TOC]


## 几个疑问
1. Windows10安装好了之后需要界面进去设置RDP远程访问。但是一般linux服务器完全的命令行方式岂不是陷入了鸡生蛋、蛋生鸡的困境？
> virtualbox 提供了一个叫vrde的方式，当你启动一个虚拟机（无论是否已经安装好os）的时候，你就可以使用rdp的任何客户端连接这个服务，然后就看到了界面，通过这个界面安装操作系统和在本地用USB的方式安装操作系统一样，点击下一步就可以了。


## 环境和软件准备
1. linux服务器一台，配置看个人，我的是完全的命令方式
2. 下载好windows10.iso
3. 下载 virtualbox https://download.virtualbox.org/virtualbox/6.1.36/VirtualBox-6.1-6.1.36_152435_el6-1.x86_64.rpm
4. 下载 virtualbox扩展包，有了这个才能再命令行模式下安装操作系统  https://download.virtualbox.org/virtualbox/6.1.36/Oracle_VM_VirtualBox_Extension_Pack-6.1.36-152435.vbox-extpack
5. 下载 virtualbox guest additions https://download.virtualbox.org/virtualbox/6.1.36/VBoxGuestAdditions_6.1.36.iso

> 关于virtualbox的不同版本，不同平台的安装都在这个连接 https://download.virtualbox.org/virtualbox/， 根据情况自己去看。

```bash
$ ls -l
-rw-r--r-- 1 xxxxxx root   11231896 Jul 20 22:08 Oracle_VM_VirtualBox_Extension_Pack-6.1.36a-152435.vbox-extpack
-rw-r--r-- 1 xxxxxx root   63803392 Jul 20 09:54 VBoxGuestAdditions_6.1.36.iso
-rw-r--r-- 1 xxxxxx root   97752792 Jul 20 10:42 VirtualBox-6.1-6.1.36_152435_el7-1.x86_64.rpm
drwx------ 3 xxxxxx root         48 Aug  4 10:49 win10
-rw-r--r-- 1 xxxxxx root 4682743808 Aug  4 15:09 Windows.iso

```

## 安装软件
```bash
$sudo yum install ./VirtualBox-6.1-6.1.36_152435_el7-1.x86_64.rpm  # 安装virtual box虚拟机
$sudo vboxmanage extpack install ./Oracle_VM_VirtualBox_Extension_Pack-6.1.36a-152435.vbox-extpack  # 安装virtual box 命令扩展包
$vboxmanage list extpacks # 检查一下扩展包是否成功，如果出现下面这个输出就是成功
Extension Packs: 1
Pack no. 0:   Oracle VM VirtualBox Extension Pack
Version:      6.1.36
Revision:     152435
Edition:      
Description:  Oracle Cloud Infrastructure integration, USB 2.0 and USB 3.0 Host Controller, Host Webcam, VirtualBox RDP, PXE ROM, Disk Encryption, NVMe.
VRDE Module:  VBoxVRDP
Usable:       true 
Why unusable: 

```


## 创建虚拟机

```bash
$VBoxManage createvm --name win10 --ostype Windows7_64 --basefolder ~/myvm/ --register # 创建虚拟机，名字为win10,保存再 ~/myvm/下
$VBoxManage createmedium --filename ~/myvm/win10.vdi --size 102400 #创建一个磁盘， 大小10GB
$VBoxManage storagectl win10 --name SATAController --add sata --controller IntelAHCI # 磁盘控制器
$VBoxManage storageattach win10 --storagectl SATAController --port 0 --device 0 --type hdd --medium ~/myvm/win10.vdi # 控制器和磁盘绑定
$VBoxManage storagectl win10 --name dvddrive --add ide # 加一个光驱
$VBoxManage storageattach win10 --storagectl dvddrive --type dvddrive --port 1 --device 0 --medium ~/win7.iso #插入操作系统光盘镜像
$VBosManage modifyvm win10 --memory 20480  # 配置虚拟机内存20GB
$VBosManage modifyvm win10 --cpus 8  # 配置虚拟机CPU个数

# 启动顺序设置
$VBoxManage modifyvm win10 --boot1 dvd 
$VBoxManage modifyvm win10 --boot1 disk

$VBoxManage modifyvm win10 --nic1 bridged --cableconnected1 on --bridgeadapter1 enp7s0 # 桥接模式上网，也就是虚拟机会从局域网得到一个IP，使用网卡为enp7s0, 网卡名字根据自己linux情况具体而定
$VBoxManage modifyvm win10 --vrde on
$VBoxManage modifyvm win10 --vrdeport 3900 # 这个端口是你用RDP连接Windows的端口
```

## 启动
```bash
$VBoxManage startvm win10 --type headless --vrdp on
```
0. 如果你的linux服务器有公网ip，那么啥都不用做。如果是通过的跳板机，那么还要做个ssh隧道，这个我就不详细说了，经常用的人都懂。把3900映射到本地来。
1. 此时可以通过rdp(就是Windows自带的那个远程桌面)远程访问了，输入`localhost:3900`（我是用的ssh隧道，如果你的linux有公网ip就用公网ip）第一次访问是安装操作系统，一直点击下一步就行了（这个时候鼠标重影还很严重）。
2. 安装好操作系统第二次进入稍作配置（此时依旧鼠标重影），然后关机
3. `$VBoxManage storageattach win10 --storagectl dvddrive --type dvddrive --port 1 --device 0 --medium ~/VBoxGuestAdditions_6.1.36.iso` 通过命令挂载VBoxGuestAdditions, 然后开机，点开我的电脑->光驱进行安装，之后重新启动，发现鼠标没有重影了。
4. 此时通过远程桌面连接，虚拟机的屏幕很小，怎么解决？ TODO

## 其他控制命令
```bash
# 关闭虚拟机
$VBoxManage controlvm win10 poweroff
$VBoxManage controlvm win10 acpipowerbutton
$VBoxManage list vms # 查看目前已经安装的虚拟机

```

## 异常

> VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005) with meanjs and vagrant \
> 这是因为在命令行下启动虚拟机，必须加上`headless` , 例如 `VBoxManage startvm "VM name" --type headless`

> VM in virtualbox is already locked for a session (or being unlocked) \
> 执行 `vboxmanage startvm <vm-uuid> --type emergencystop`


## 参考文章
- https://post.smzdm.com/p/a27n9lzn/?ivk_sa=1024320u
- https://www.linuxprobe.com/virtualbox-cmd-manage.html
