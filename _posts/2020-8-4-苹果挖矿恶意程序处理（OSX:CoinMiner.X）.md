---

layout: post
title:  "苹果挖矿恶意程序处理（OSX/CoinMiner.X）"
categories: 应急响应
tags: 挖矿 OSX/CoinMiner.X
author: 303Donatello

---


* content
{:toc}
# 背景
> 近期通过流量告警发现多起外连矿池的告警，均外连至43.249.204.231






**威胁情报信息如下：**
![](https://img2020.cnblogs.com/blog/2011235/202008/2011235-20200803142042738-537006647.png)
![](https://img2020.cnblogs.com/blog/2011235/202008/2011235-20200803142115199-932617052.png)
# 系统表象
1.通过`ps -ef|grep osascript`发现在`/library/LaunchAgents/`文件下均有恶意挖矿plist文件，主要为`/library/LaunchAgents/com.apple.00.plist`等形式，且plist已加密。有部分MAC还会存在一个下述所示的明文指向性plist文件，其在每次开机时通过osascript来执行AppleScript脚本来运行加密后的00.plist,kaspersky可以识别。
```
<dict>
	<key>Label</key>
	<string>com.apple.2KR</string>
	<key>Program</key>
	<string>/usr/bin/osascript</string>
	<key>ProgramArguments</key>
	<array>
		<string>osascript</string>
		<string>-e</string>
		<string>do shell script "osascript ~/Library/LaunchAgents/com.apple.00.plist"</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StartInterval</key>
	<integer>60</integer>
	<key>WatchPaths</key>
	<array/>
</dict>
```
加密plist文件中的门罗币挖矿程序（截图来自该恶意程序惯用命名：ssl3.plist）
![](https://img2020.cnblogs.com/blog/2011235/202008/2011235-20200803145659504-569060560.png)

2.活动监视器无法打开
3.如果挖矿程序运行机器CPU异常、电脑异常发热、卡顿
# 检查与处理
## 需要检查的目录
通过`ps -ef|grep osascript`确认恶意plist所在位置进行检查，同时还需要注意如下目录：
```
~/Library/LaunchAgents
~/Library/Safari
~/Library/Safari/openssl/lib/
/private/etc/mach_inlt (注意其与init混淆)
/Library/LaunchDaemons（目录下plist指向/etc/ncsys）
/etc/ncsys
/etc/evtconf
```
其中可能在目录中发现：
`不明*.plist`为挖矿主程序，一般都是加密后的集成性工具，主程序支持多种虚拟货币挖掘。
`pool.txt`为矿池配置文件
`Config.txt`用于配置超时时间与钱包地址 
`cpu.txt `用于记录CPU使用数
## 处理操作
1.杀掉相关进程
2.删除相关恶意文件（尤其是第三方dmg以及非官方APP）
3.必须重装系统
# IOC
```
43.249.204.231
budaybu.com
f1d274c8a995d8b1030c21710d1ea725  *ssl3.plist
ff1d7c07d0aa0a95e930009a38f17289 *com.apple.0V.plist
a0933446d65192ff1290f67e4dda169c *com.apple.2KR.plist
fbc709097d02187418a9709433e707b8 *com.apple.00.plist
c925e754140572c73e3fe5ca952f21f9 *com.microsoft.autorun.plist
```
# 参考
https://x.threatbook.cn/nodev4/vb4/article?threatInfoID=1240
http://www.mamicode.com/info-detail-2415920.html
https://sensorstechforum.com/osx-coinminer-detect-remove-macbook/
https://blog.csdn.net/weinierzui/article/details/98331203
