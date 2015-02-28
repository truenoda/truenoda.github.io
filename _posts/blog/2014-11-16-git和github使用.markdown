---
layout: post
title:  "github和git使用 "
date:   2014-11-16
author: me
categories: 
- blog
- 网站
thumb: thumb03.png
---

安装git bash什么的教程不在此说明了。
git bash给我的感觉就是模拟一个linux的终端，提供了一些简单的linux命令，如果你有linux基础，上手so easy。
<!--more-->
<br><br>
ssh-keygen 生成文件 id_rsa
在.ssh下建立个config文件

	touch config 
	chmod 600 config
在其中添加下面的代码

	Host github.com
	HostName github.com
	User 1
	IdentityFile C:/Users/Administrator/.ssh/id_rsa
	[credential]   
	    helper = store     //第一个用户 存储密码 输入一次就记住
	Host github2  
	HostName github.com
	User 2
	IdentityFile C:/Users/Administrator/.ssh/id_rsa2
	[credential]   
	    helper = store

我在使用的时候，只有第一个才会成为默认的，否则会报错，所以你要使用的话，把第二个调整到第一个，才能默认出来。
输入的密码会存储在用户的.git-credentials中，如果换用户，可以手动更改。

	https://用户:密码@github.com


.gitconfig貌似对多用户的使用影响不大<br><br>

## tortoisegit的使用 ##
git bash的使用遇到过一些问题，解决起来费事，于是我选择了用tortoisegit这个GUI的方式。
并不是对CLI敏感，只不过遇到的问题使得CLI的确不如这样操作界面方便。<br><br>

我是这样使用的，在github创建仓库，手动建立一个readme文件，这个文件默认创建了一个master 的分支这个是主分支，于是git clone **或者使用tortoisegit 将项目git下来
本地修改后
tortoisegit到master分支，立即见效 

出现一次tortoisegit问题，不断的显示Disconnected: No supported authentication methods available (server sent: publickey)
试了改动settings里面的网络 client位置改成 Git/bin/ssh.exe  没有效果 于是删除了两个rsa文件（我是有两个账户的），然后重新生成key上传之后再去使用git和tortoisegit成功

**注意第一个显示的成功，不是真正成功，而是push了一个请求或者是给网络服务端一个文件列表  。选择推送，才是真正的传送文件。**
