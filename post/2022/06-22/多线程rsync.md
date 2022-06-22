[TOC]



### rsync存在的问题



当我在国外一台VPS上下载了很多视频，想回传的时候，发现使用rsync网络速度非常慢。我的vps是200MBps,理论可以达到25MB/s，实际只有3-4MB/s的样子。

经过查资料发现`rsync`是一个单进程的程序，每次同步一个文件。由于单个TCP连接收到初始窗口和拥塞控制的影响导致无法充分压榨带宽。单线程为什么比多线程下载慢，可以参考这篇文章 https://www.zhihu.com/question/19914902。



### xargs方式



```bash
ssh  root@192.168.1.2 "cd /root/youtube/videos;find  . -type f -print0" | xargs -0 -I%  -P10 rsync -avuP  root@192.168.1.2:/root/youtube/videos/%   /my/local/dir/
```

-  `-print0`用NULL分割打印出来的内容
- `xargs -0` 代表也用NULL作为一个参数的分割
- `-P 10` 代表最大10个线程同时进行，也就是后面跟的`rsync`

这是拉的方式，如果是推送的方式不妨使用`inotifywait`

> 经过我的使用，发现带宽提高了3倍左右，接近10MB/s，已经非常不错。



### parelle方式

暂未使用。

```bash
ssh  root@192.168.1.2 "cd /root/youtube/videos;find  . -type f -print0" | xargs -0 -j 10 rsync -avuP  root@192.168.1.2:/root/youtube/videos/{}   /my/local/dir/
```







### 参考

- https://stackoverflow.com/questions/52900340/launch-multiple-simultaneous-rsync-processes-to-copy-one-file-in-every-sub-direc
- https://community.spiceworks.com/topic/2278315-rsync-to-use-multiple-threads-to-copy
- https://stackoverflow.com/questions/68920131/multi-thread-rsync