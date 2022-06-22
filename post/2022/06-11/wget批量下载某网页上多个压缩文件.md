[TOC]







### 下载某页面上全部压缩文件

```shell
wget --spider --execute robots=off  -r -c -l1 -H -t1  -nd   -A zip,json,tar,gz,rar,xz  --no-check-certificate --content-disposition  https://computing.ece.vt.edu/~aish/vqacp/

# -l1 只下一层
# -r 递归
# -c 断点续传
# -H 如果连接到其他站点的文件也下载
# -t1 只重试1次
# -nd 不向url的上级目录扩张遍历
# -A 文件扩展名，也就是你要下载那种类型的文件，不在这个列表中的将会被忽略不下载

```

