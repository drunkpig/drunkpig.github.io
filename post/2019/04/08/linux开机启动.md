1. /etc/init.d/ 中添加一个脚本，例如runsth.sh
2. 添加到启动项  sudo update-rc.d runsth.sh defaults 90   # 90越大，启动时间越晚
3. 如果删除执行  sudo update-rc.d -f runsth.sh remove
4. supervisord使用 https://www.restran.net/2015/10/04/supervisord-tutorial/





> 很可惜使用脚本开机启动已经被systemd替代了。
>
> 不过supervisord在监控应用程序方面还是非常好用。