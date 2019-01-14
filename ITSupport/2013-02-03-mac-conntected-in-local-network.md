---
title: Mac机连接局域网文件方法
date: 2013-02-03 0:00:00
tags: [ "mac", "osx" ]
categories: [ "Mac" ]
---
如何在OS X访问其它主机

　　\&#8221;Finder-转到－连接到服务器\&#8221;或者直接按\&#8221;苹果+K\&#8221;。这时候会显示３个主要的工具组：

　　１、WORKGROUP：Windows工作组

　　２、Local：局域网内所有可访问的主机，包括Mac/Windows主机

　　３、MSHOME：Windows工作组（好象XP默认的都这个工作组）

　　如果你想访问的主机没有显示出来，你可以使用以下方法进行尝试：

　　１、Windows主机：smb://xxx.xxx.xxx.xxx

　　２、Mac主机：afp://xxx.xxx.xxx.xxx

　　其中xxx.xxx.xxx.xxx为该主机的局域网IP。类似于192.168.1.2

　　另外好象连接XP的主机其实不用输入用户名和密码。直接回车就行了。
