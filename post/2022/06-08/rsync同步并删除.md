[TOC]




### 同步之后删除已经同步成功的（PUSH方式）
```shell
rsync   -avuPr --remove-source-files  ./ root@192.168.1.6:/mnt/test  # 把当前目录下文件和目录同步到远端机器的/mnt/test/下
find . -depth -type d -empty -delete  # 删除当前目录下的空目录，多这一步是因为rsync虽然会删除已经同步完成的文件，但是不会删除空目录
```

### 同步远程目录/文件之后，删除（PULL方式）
```shell

rsync   -avuPr --remove-source-files   root@192.168.1.6:/mnt/test/  ./
rsync -av --delete `mktemp -d`/   root@192.168.1.6:/mnt/test/    # 思路是把一个空目录同步过去
```

### 两侧保持同步，可对已经同步的删除

下列命令可实现src/下文件/目录删除则 dst/也删除，所谓的`--delete`参数表达的功能是让目标目录的文件和本地保持绝对一致，如果目标有，但是源方没有，那么目标方里多于源方的文件/目录就会被删除掉

```shell

rsync -avuP  --delete src/  dst/     # 这样的效果就是两边完全同步，如果src里有，后来被删除了，那么dst里已经被同步的也被删除
```



### 持续同步
```shell
while true
do  
  rsync -azuvP --exclude=DwnlData/ --no-owner --no-group ./* root@192.168.1.6:/mnt/dev/thunder/
  sleep 1m
done
```

