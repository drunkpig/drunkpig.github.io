tags: 一个机器提交到多个github账号
    git



[TOC]



## 背景

有的时候为了搞点事情，搞啥不说了，需要从一台机器上向多个github账户提交代码。如何配置账户每个github账户关联一个ssh密钥，从而实现免密提交代码?



## 配置

###  ~/.ssh/config 文件

```yaml
Host bpmw
	HostName github.com
	User bpmw
	IdentityFile ~/.ssh/id_rsa_bpmw

Host jscrapy
	User jscrapy
	HostName github.com
	IdentityFile ~/.ssh/id_rsa
	
Host 123069959
	User yaofanzhennan
	HostName github.com
	IdentityFile ~/.ssh/id_rsa_123069959
	
```





### 生成ssh密钥

 秘钥可以用命令 `ssh-keygen -t rsa   -C "your@email.com"` 生成，注意第一步要你输入秘钥文件名字，（上例config中的 id_rsa_bpmw, id_rsa_123069959）否则会覆盖默认的id_rsa



### 让密钥生效

执行`ssh-add  your_id_rsa_xxx ` ，全部加进去。如果出现错误执行 `ssh-add bash`

执行以下两条命令让配置生效

```bash
ssh-add bash
ssh-add  your_id_rsa_xxx 
```



### 仓库提交地址变化

我有个仓库xconfig， clone下来的remote地址是 `git@github.com:bpmw/mytest.git`。

需要把这个地址根据config配置文件改为形式为  `git@<Host>:<User>/your-repo.git`的形式

```bash
pi@raspberrypi:~/workspace/xconfig $ git remote -v
origin	git@123069959:yaofanzhennan/xconfig.git (fetch)
origin	git@123069959:yaofanzhennan/xconfig.git (push)
```



然后就大功告成了