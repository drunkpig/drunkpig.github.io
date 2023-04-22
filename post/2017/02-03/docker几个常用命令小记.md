## 安装
安装docker : https://wiki.deepin.org/index.php?title=Docker

+root 用户组避免每次都输入sudo: http://www.cnblogs.com/franson-2016/p/6412971.html

## PG数据库
```shell
sudo docker run -d --restart=always -p5432:5432 -v/home/cxu/_data/pg/data/:/var/lib/postgresql/data/ postgres
```
## PG-admin
```shell
docker run -d --restart=always -p 80:80 -e "PGADMIN_DEFAULT_EMAIL=xuchaoo@gmail.com" -e "PGADMIN_DEFAULT_PASSWORD=toor" \
           dpage/pgadmin4
```


## mongodb
```shell
docker run -d --restart=always -p27017:27017 -v/home/cxu/_data/_mongo_qikan/:/data/db mongo:3.5
```
## kafka 单机
```shell
docker run -d  --restart=always   -v/home/cxu/_data/kafka/:/data/kafka/ --name=kafka \
           --net=host -e HOSTNAME=localhost  xuchaoo/kafka-standalone-docker
```


