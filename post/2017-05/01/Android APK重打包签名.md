[TOC]

## apk签名的作用

1. 无缝省级。签名作为每个app的唯一id, 和package name绑定。具有相同package name的app在系统中对应了唯一的签名，先入为主。不同签名相同package name的app在后来安装则会被拒绝。

2. 安全考虑。运行时刻代码中动态校验自己打包的签名和代码中签名是否一致。

3. 应用模块化。Android允许**多个有同一个签名的应用程序**在一个进程内运行。这样应用可以以模块方式独立省级，而应用也可以以模块方式独立部署。

4. 相同签名的APP可以共享数据。

   

## 签名的步骤

### 整体步骤

1. 使用java自带的keytool创建key
2. 使用1中创建的key对apk签名
3. 使用zipalign 对签名之后的apk进行优化

### 用到的工具

使用到的3个工具为：

1. **keytool** 位于`$JDK_HOME/jre/bin`

2. **jarsigner** 位于 $JDK_HOME/bin下。也可以用Android SDK自带的**apksign.jar**。另外一个比较优秀的商业化公司开发的[**Apksigner**](http://apk.aq.163.com/apkpack.do#download)

3. **zipalign** 需要另外安装

   ### 具体操作

4. `keytool -genkey -alias demo.keystore -keyalg RSA -validity 40000 -keystore demo.keystore`

   说明：

   -genkey 产生芈月

   -alias demo.keystore 设置别名

   -keyalg RSA 使用RSA算法对签名加密

   -validity 40000 有效期40000天

   -keystore demo.keystore 产生文件名字

5. `jarsigner -verbose -keystore demo.keystore -signedjar demo_signed.apk demo.apk demo.keystore`

   说明：

   -verbose 输出过程日志

   -keystore demo.keystore 密钥库路径

   -signedjar demo_signed.apk demo.apk demo.keystore 三个参数中依次为签名后产生的文件demo_signed，要签名的文件demo.apk和密钥库demo.keystore

6. `zipalign -v 4 demo_signed.apk final.apk`

   zipalign让apk文件未压缩的数据在4个字节的边界上对其，这样Android就能够用nmap()啊含糊在读取文件上获取更高的性能。

## 再来说签名

假如你有一个宏伟的战略工程，想在生活，服务，游戏，系统各个领域都做一个独立的APP，谷歌建议你把你所有的APP都使用同一个签名证书。
　　 使用你自己的同一个签名证书，就没有人能够覆盖你的应用程序，即使包名相同，所以影响有：

1. **App升级**。 使用相同签名的升级软件可以正常覆盖老版本的软件，否则系统比较发现新版本的签名证书和老版本的签名证书不一致，不会允许新版本安装成功的。
2. **App模块化**。android系统允许具有相同的App运行在同一个进程中，如果运行在同一个进程中，则他们相当于同一个App，但是你可以单独对他们升级更新，这是一种App级别的模块化思路。
3. **允许代码和数据共享**。android中提供了一个基于签名的Permission标签。通过允许的设置，我们可以实现对不同App之间的访问和共享。

> **基于签名的Permission标签**
>
> 其中protectionLevel标签有4种值：normal(缺省值),dangerous, signature,signatureOrSystem。简单来说，normal是低风险的，所有的App不能访问和共享此App。dangerous是高风险的，所有的App都能访问和共享此App。signature是指具有相同签名的App可以访问和共享此App。signatureOrSystem是指系统image中App和具有相同签名的App可以访问和共享此App，谷歌建议不要使用这个选项，因为签名就足够了，一般这个许可会被用在在一个image中需要共享一些特定的功能的情况下。

### 常见错误

------

```
无法对 jar 进行签名: java.util.zip.ZipException: invalid entry compressed size (expected 1368 but got 1379 bytes)
```

尝试删除apk里的META-INF