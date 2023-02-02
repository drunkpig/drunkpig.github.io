[TOC]



# rsync server搭建



## 配置

配置文件一共涉及到以下3个

- /etc/rsyncd.conf  配置主要文件
- /etc/rsyncd.secrets  用户密码
- /etc/rsyncd.motd  登录时候客户端能看到一些欢迎信息，可以放这里。普通文本



### rsyncd.conf

```ini
#rsyncd.conf的配置项分全局参数和模块参数，全局参数只有少数几个，
#模块以[模块名]开头，后续参数仅作用于该模块
#卸载模块外的参数适用于所有模块
#rsyncd.conf文件的指令和值请参考 man rsyncd.conf
#般保持默认即可
#欢迎文件
motd file = /etc/rsyncd.motd
#用户和组id
uid = root
gid = root
port = 60200
#是否chroot，出于安全考患建议为yes
use chroot = no
#是否记录传输记录
transfer logging = no
#是否只读，值为true时客户端无法上传
read only = false
#是否只写，值为true时客户端无法下载
write only = false
#默认拒绝所有主机连接
#hosts deny = *
secrets file = /etc/rsyncd.secret

[backup]
# 模块说明
comment = backup directory
# 模块路径，请求改成自己的
path = /mnt/mydata
#允许的主机ip
hosts allow = 111.132.81.91
#允许的用户名
auth users = rsyncd
#是否允许列出该模块，建议为no
list = yes
```



### rsyncd.secrets 

```ini
rsyncd:pass1
user2:pass2
```

每行一个用户，在rsyncd.conf里每个路径都可以规定某个用户可以访问。这样方便管理权限



### rsyncd.motd

```
随便写点什么。
欢迎来的我的rsync服务
```





## 启动

```shell
systemctl start rsync
```

有的时候上面命令启动不会使用你的配置，这个时候就需要去修改一下rsync.service，

## 测试

```
echo "pass1" > rsync.pass
rsync -avrP --password-file=./rsync.pass  rsync://rsyncd@192.168.1.2:60200/backup/xxx.file   .

```





## 优势

1. 不走ssh协议，而是直接的sock连接，不受到ssh发送窗口最大值限制
2. 更改内核拥塞控制算法之后在高延迟、高丢包网络有显著改善10倍以上。



## 类rsync工具

- https://stackoverflow.com/questions/8575345/rsync-without-ssh-access

- [TCP内核参数优化](https://juejin.cn/post/6993124663878484005)
- [PPC拥塞控制算法](https://www.compiralabs.com/post/congestion-control-pcc-vs-cubic-bbr-hybla)
- [bbr, bbrv2拥塞控制算法](https://aws.amazon.com/cn/blogs/china/talking-about-network-optimization-from-the-flow-control-algorithm/)
- 