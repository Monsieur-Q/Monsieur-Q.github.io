[分享] Amlogic系列盒子彻底解决key丢失问题

MXQ Pro+ 2+16外贸盒子，千兆网卡用用还可以，就是刷机时出了问题导致mac地址不固定每次开机随机变很烦人；
网上查询了很多帖子，有很多网友有类似问题（例如N1...)，据说是因为刷机时key被弄丢了，主要解决方案如下：

1.最简单的解决办法是进uboot才能修改，无奈这盒子压根没有TTL接口，因此作罢；
2.有大神通过修改aml_autoscript脚本来改mac，但又要改文件又要换算CRC，流程繁琐不说还经常出错。

刷机大神VastStarGames，通过区区几条命令不拆机就轻松解决了mac地址不固定的问题，具体步骤如下：
1.先用开心盒子助手等工具ADB连接盒子（必须获得root权限），当然也可以直接TTL连接
2.敲入命令 cd /sys/class/unifykeys/
3.敲入list -l 命令读取list节点，可以获取当前的支持烧写那些key，如下支持一共14个key的烧写
********************************************************************************************
p212:/sys/class/unifykeys # cat list
14 keys installed
00: usid, normal, 7
01: mac, normal, 7
02: hdcp, secure, 7
03: secure_boot_set, efuse, 2
04: mac_bt, normal, 7
05: mac_wifi, normal, 7
06: hdcp2_tx, normal, 7
07: hdcp2_rx, normal, 7
08: widevinekeybox, secure, 7
09: deviceid, normal, 7
10: hdcp22_fw_private, secure, 7
11: PlayReadykeybox25, secure, 7
12: prpubkeybox, secure, 7
13: prprivkeybox, secure, 7
************************************************************************************************
4.敲入>name命令选择需要修改的key，例如需要修改mac地址就输入命令： echo mac > name
5.敲入>wirte命令修改指定的mac地址，例如 echo AA:BB:CC:DD:EE:FF > write
6.再敲入cat read命令就发现mac地址已经绑定

理论上该方法适合所有Amlogic盒子，例如需要修改设备序列号就执行echo usid > name
