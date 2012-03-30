---
layout: post
title: How to Install the Latest Version of Mongodb and Repair it
categories:
- linux
---

前几天在自己的服务器上安装并部署了mongodb数据库, 不过Ubuntu官方并没有最新的apt源, 所以要自己添加一个10gen的源.

10gen这个安装包包含了最新的Mongodb版本, 需要添加到 `/etc/apt/sources.list` 的最后,

    $ sudo vim /etc/apt/sources.list

就像下面这样的配置:

    ## Uncomment the following two lines to add software from Ubuntu's
    ## 'extras' repository.
    ## This software is not part of Ubuntu, but is offered by third-party
    ## developers who want to ship their latest software.
    # deb http://extras.ubuntu.com/ubuntu natty main
    # deb-src http://extras.ubuntu.com/ubuntu natty main

    # other sources
    deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen
    
10gen这个安装包需要用到GPG的key, 不然是无法更新的, 所以要添加GPG key

    $ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10
    
导入了GPG key之后, 就可以更新你的源了

    $ sudo apt-get update

然后即可安装最新版本的Mongodb了

    $ sudo apt-get install mongodb-10gen

可以在命令行输入 `mongo` 测试一下是否可以连接成功

    $ mongo
    MongoDB shell version: 2.0.4
    connecting to: test
    >
    
安装过程其实很假单, 但是如果遇到以下情况就需要, 你就要想以下是否mongodb的lock文件已经损坏了

    Mongo::ConnectionFailure (Failed to connect to a master node at localhost:27017)
    
这个问题其实是由于不正常关闭数据库导致的, 比如服务器崩溃导致不正常启动, 可能会引起服务器重启后无法启动mongodb的情况

下面就来看下我是如何修复这个问题的

首先删除mongodb的lock文件

    $ sudo rm /var/lib/mongodb/mongod.lock

进入修复

    $ sudo -u mongodb mongod -f /etc/mongodb.conf --repair
    
接下来就能启动mongodb了

    $ sudo service start mongodb

希望能对大家有所帮助