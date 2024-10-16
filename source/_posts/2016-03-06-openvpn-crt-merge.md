---
layout: post
title: OpenVPN客户端证书合并到配置文件中
categories: Tools
description: OpenVPN客户端证书合并到配置文件中
index_img: /img/post_def.png
date: 2016-03-06 09:09:09
tags: [OpenVPN, Tools, 证书合并]
---


生成的客户端证书和配置文件，客户端需要的一共有5个文件：ca.crt、client.crt、client.key、ta.key(如果不开启tls-auth，则无需该文件)、client.ovpn。为了避免文件太多，管理不便！需要将客户端证书合并到配置文件中。具体方法如下。

---

** 此方法在Linux、Windows、Mac OS下都通用！！


## 简单描述一下：

到我现在测试止，OpenVPN最新版本是2.3.6！！

** 我的环境：

- 客户端是win10，用的是OpenVPN2.3.6这个最新的版本！！

- 服务器端：CentOS6.3_x86-64  OpenVPN2.3.5

 

** 服务端与客户端的安装方法就不多说了，附上一个来自高手的文章，写的很好，赞一个！！！

【参考文章】<http://www.softown.cn/post/140.html>

** 在此附上各平台OpenVPN安装包的下载链接吧，免得同学们去找了：

- windows平台：<http://www.techspot.com/downloads/5182-openvpn.html>

- Linux平台：<http://fossies.org/linux/misc/openvpn-2.3.5.tar.gz>

- mac平台： <http://jaist.dl.sourceforge.net/project/tunnelblick/All%20files/Tunnelblick_3.5.0_build_4265.dmg>

 

## 正式进入主题，合并证书到配置文件中，很简单：

** 编辑client.ovpn客户端配置文件：
```Linux
vim client.ovpn
```

** 删除或者注释掉以下几行内容：
```Linux
#在这里我把它们注释掉：
ca ca.crt　　改为：#ca ca.crt
cert client.crt　　改为：#cert client.crt
key client.key　　改为：#key client.key
tls-auth ta.key 1　　改为：#tls-auth ta.key 1
```

** 在最后面添加以下内容：
```Linux
<ca> 
ca.crt文件内容
</ca>
<cert> 
client.crt文件内容
</cert>
<key> 
client.key文件内容
</key>
key-direction 1 
<tls-auth> 
ta.key文件内容
</tls-auth>
```

** 复制各文件里的内容到相应的位置即可！保存退出！！

** 然后可以删掉那4个证书文件了，只需要客户端的配置文件这一个即可，到此操作完毕！

** 可以去启动客户端加载配置文件了，一切正常！！
