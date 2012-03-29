---
layout: post
title: How to Install JDK on Ubuntu 11.10
categories:
- linux
---

最近在玩solr, 需要用到java环境, 但是ubuntu官方源中却没有jdk安装包, 经过一些研究, 发现以下方案即可解决安装问题:

    > sudo apt-get install python-software-properties
    > sudo add-apt-repository ppa:ferramroberto/java
    > sudo apt-get update
    > sudo apt-get install sun-java6-jdk
    
Ubuntu中经常会用到添加ppa源的时候, 确实方便.