tags: linux



[TOC]



# linux增加root用户



## 添加用户

```bash
sudo adduser arootuser
sudo passwd arootuser  #设置这个用户的密码
sudo usermod -a -G root arootuser # 把用户加入到root用户组
```



## 让用户可执行 sudo命令

### 修改sudoers文件为可读

```bash
sudo chmod -v u+w /etc/sudoers
```

### sudoers文件里新增一行

使用vim打开 /etc/sudoers文件，然后添加一行

```bash
arootuser ALL=(ALL)  ALL
```

### 恢复sudoers文件为只读模式

```bash
sudo chmod -v u-w /etc/sudoers
```





最后你可以在arootuser用户下使用 `sudo xxx`来执行特权指令了。