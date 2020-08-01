
[TOC]

### 删除sendmail相关服务（如果有）

```bash
service sendmail stop
apt remove sendmail
```


### 安装postfix
```bash
sudo apt-get install libsasl2-modules postfix sasl2-bin

```


### 配置postfix

```ini
# /etc/postfix/main.cf
myhostname = mail.xxx.com # 系统主机名字,指向邮件服务器的域名地址
mydomain = xxx.com   # 这个将会成为Email地址 @符号后面的部分
myorigin = $mydomain  
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain  # 可接收邮件的主机和域名
inet_interfaces = all   # 接收来自所有网络的请求
inet_protocols = ipv4
relay_domains = $mydomain # 只有mydomain的邮件才能转发，避免其他人用来发垃圾邮件
home_mailbox = Maildir/ # 存放用户邮件的目录，每个用户一个文件夹，每个邮件一个单独文件。还有一种Mailbox方式，同一个用户全部邮件内容为单个文件， 默认保存在/var/spool/mail/这个目录下面
mynetworks = 127.0.0.0/8 # 可转发哪些网络的邮件
smtpd_banner = $myhostname ESMTP 
# 规定邮件最大尺寸为10M 
message_size_limit = 10485760 
# 规定收件箱最大容量为1G 
mailbox_size_limit = 1073741824 

```

### 配置stmp账户

#### /etc/postfix/main.cf 继续配置

```ini
#指定可以向postfix发起SMTP连接的客户端的主机名或ip地址,此处permit_sasl_authenticated意思是允许通过了sasl认证的所有用户
smtpd_client_restrictions = permit_sasl_authenticated

smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, 

#指定postfix使用sasl验证 通俗的将就是启用smtp并要求进行账号、密码校验
smtpd_sasl_auth_enable = yes

#取消smtp的匿名登录 此项默认值为noanonymous 此项请务必指定为noanonymous
smtpd_sasl_security_options = noanonymous

```

#### 编辑通过sasl启用smtp账号密码效验的配置

首先安装`saslauthd`

然后编辑 `/usr/lib/sasl2/smtpd.conf`

```ini
pwcheck_method: saslauthd
#auxprop_plugin: sasldb
mech_list: PLAIN LOGIN

```

#### 创建smtp 账号

执行命令 ``saslpasswd2 -c -u `postconf -h mydomain` test`` 或者 `saslpasswd2 -c -u applesa.cn test`
输入密码即可创建账号 test@applesa.cn 这个账号。

使用`sasldblistusers2 ` 可以查看目前所有的smtp账号。

添加完账号之后重启一下postfix`systemctl restart postfix`


### 启动
```bash

systemctl  restart  postfix
systemctl  enable  postfix
```

### 测试
`echo "content" | mail -s "title" xxx@foxmail.com`

### SPF, DKIM, DMARC， RDNS配置
配置要看具体操作系统的： https://wiki.debian.org/PostfixAndSASL
几个概念比较详细 https://www.smartertools.com/blog/2019/04/09-understanding-spf-dkim-dmarc
看[这一篇](https://wiki.zimbra.com/wiki/Best_Practices_on_Email_Protection:_SPF,_DKIM_and_DMARC)就够了 

https://www.smartertools.com/blog/2019/04/09-understanding-spf-dkim-dmarc

- SPF (Sender Policy Framework)
- DKIM (Domain Keys Identified Mail) 私钥发邮件时签个名，公钥放在dns记录里。
- DMARC(Doamin-based Message Authentication, Reporting & Conformance)
- rDNS(reverse DNS) ip到域名的映射

- DKIM Generator https://tools.socketlabs.com/dkim/generator
- DMARC Generator https://tools.socketlabs.com/dmarc/generator