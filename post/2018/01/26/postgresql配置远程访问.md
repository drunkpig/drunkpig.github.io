### 安装
`#apt-get install postgresql postgresql-contrib`

### 配置可原创访问
1. 修改 /etc/postgresql/10/main/pg_hba.conf

   ```bash
   # DO NOT DISABLE!
   # If you change this first entry you will need to make sure that the
   # database superuser can access the database using some other method.
   # Noninteractive access to all databases is required during automatic
   # maintenance (custom daily cronjobs, replication, and similar tasks).
   #
   # Database administrative login by Unix domain socket
   local   all             postgres                                peer
   host    all             postgres        0.0.0.0/0                 md5
   # TYPE  DATABASE        USER            ADDRESS                 METHOD
   
   # "local" is for Unix domain socket connections only
   local   all             all                                     peer
   # IPv4 local connections:
   host    all             all             127.0.0.1/32            md5
   # IPv6 local connections:
   host    all             all             ::1/128                 md5
   # Allow replication connections from localhost, by a user with the
   # replication privilege.
   local   replication     all                                     peer
   host    replication     all             127.0.0.1/32            md5
   host    replication     all             ::1/128                 md5
   ```

   `host    all             postgres        0.0.0.0/0                 md5`这一行是自己添加的，代表让全部网络地址都可以以用户postgres来远程访问。

2. 修改postgresql.conf, 找到 `listen_addresses = 'localhost'` 改为`listen_addresses = '*'`,  如果处于被注释状态，请解注。

3. 修改密码。在本机登录psql , 执行 `ALTER USER postgres WITH PASSWORD 'postgres';`   一定注意末尾加分号。否则语句不会提交执行。

