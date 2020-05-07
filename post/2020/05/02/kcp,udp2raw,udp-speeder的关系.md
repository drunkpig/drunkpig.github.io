tags:  kcp
    kcptun
    udp2raw
    upd2raw-tun
	udpspeeder
    finalspeeder


[TOC]


### [KCP](https://github.com/xtaci/kcp-go)
kcp 是一个协议，能在尽量降低延迟的情况下，加大传输流量。底层使用UDP，也可以是tcp

### [udp2raw](https://github.com/wangyu-/udp2raw-multiplatform)
udp2raw是一个把udp协议伪装成tcp的底层库，因为udp流量太大某些运营商会QoS，对tcp则不。例如运营商限制了你的网络出口udp和tcp流量比例为1:9 

### [kcptun](https://github.com/xtaci/kcptun)
kcptun, udp2raw-tun是利用上述两种协议分别实现的传输应用

### [udp2raw-tunnel](https://github.com/wangyu-/udp2raw-tunnel)
A Tunnel which turns UDP Traffic into Encrypted FakeTCP/UDP/ICMP Traffic by using Raw Socket

### [udpspeeder](https://github.com/wangyu-/UDPspeeder)
udpspeeder  底层基于udp2raw开发的udp加速，原理和kcptun差不多，不过多了一些混淆。

### [finalspeeder](https://github.com/malaohu/finalspeed_bak)
 finalspeeder 加速tcp的库，java写的

 ### [pingtunnel](https://github.com/esrrhs/pingtunnel)
 
 把tpc/udp, socks转为icmp报文，绕过wifi网络验证和加速传输 



