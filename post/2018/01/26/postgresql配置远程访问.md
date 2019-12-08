### 安装
`#apt-get install postgresql postgresql-contrib`

### 配置访问
- 修改 /etc/postgresql/10/main/pg_hba.conf
- 重启/etc/init.d/postgresql restart, 或者 `/usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main -l /path/to/log.log start`