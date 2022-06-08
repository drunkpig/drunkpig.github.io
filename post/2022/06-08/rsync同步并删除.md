[TOC]




### 同步之后删除已经同步成功的（PUSH方式）
```shell
rsync   -avuPr --remove-source-files  ./ root@8.218.8.207:/mnt/xuchaoo_dev/test
find . -depth -type d -empty -delete
```

### 同步远程目录/文件之后，删除（PULL方式）
```shell

rsync   -avuPr --remove-source-files   root@8.218.8.207:/mnt/xuchaoo_dev/test/  ./
rsync -av --delete `mktemp -d`/   root@8.218.8.207:/mnt/xuchaoo_dev/test/    # 思路是把一个空目录同步过去
```

### 两侧保持同步，可对已经同步的删除

下列命令可实现src/下文件/目录删除则 dst/也删除

```shell

rsync -avuP  --delete src/  dst/     # 这样的效果就是两边完全同步，如果src里有，后来被删除了，那么dst里已经被同步的也被删除
```



### 持续同步
```shell
while true
do  
  rsync -azuvP --exclude=DwnlData/ --no-owner --no-group ./* root@111.1.0.110:/mnt/dev/thunder/
  sleep 1m
done
```

