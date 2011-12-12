---
layout: post
title: Distributed Revision Control with Git
categories:
- git
---

Git是一个开源的分布式版本控制系统, 用以有效、高速的处理从很小到非常大的项目版本管理.

下面来介绍如何在安装ubuntu server系统的服务器上管理你的项目, 以 [linode VPS][0] 为例.  

安装git-core和gitosis  

    $ sudo apt-get install git-core gitosis

线确认你本地机器中是否已经生成过public key, 查看 `~/.ssh/id_rsa.pub`, 如果还没有创建你的public key, 就需要通过ssh-keygen来生成:

    $ ssh-keygen
    
执行以上命令后会在 `.ssh` 目录下生成 `id_rsa` 和 `id_rsa.pub` 两个文件,

当然如果你本地机器中如果已经有了public key, 就不需要再生成一次了, 接下来要做的就是在服务器上进行git-server的配置了, 先把你本地的public key scp到服务器的 `/tmp` 目录下:  

    $ scp ~/.ssh/id_rsa.pub your_username_here@your_server_ip_here:/tmp

用刚scp上来的public key作为管理者来初始化gitosis:

    $ sudo -H -u gitosis gitosis-init < /tmp/id_rsa.pub

完成后，你应该会看到以下信息:

    Initialized empty Git repository in /srv/gitosis/repositories/gitosis-admin.git/
    Reinitialized existing Git repository in /srv/gitosis/repositories/gitosis-admin.git/
    
到这里, 刚才的public key已经没有用了, 可以删除

    rm /tmp/id_rsa.pub

gitosis的配置已经完成, 接下来就可以把gitosis的管理项目clone到本地进行项目的管理了，前提是在在服务器上必须有你的证书:

    git clone gitosis@your_server_ip_here:gitosis-admin.git

取到gitosis-admin这个管理项目后, 我们就可以任意增加项目成员和添加新的项目了, 可以通过编辑配置文件来完成:

    cd gitosis-admin
    vim gitosis.conf

添加一个名为foo的项目:  

    [gitosis]

    [group gitosis-admin]
    writable = gitosis-admin
    members = key

    [group foo]
    writable = foo
    members = key

保存该文件，然后提交改动:  

    git commit -a -m "add a new project"
    git push

接下来新建一个项目,来测试一下配置是否正确:  

    rails new foo
    cd foo
    git init
    git add .
    git commit -a -m "first commit"
    git remote add origin gitosis@your_server_ip_here:foo.git
    git push origin master

项目提交成功。foo这个项目已经可以协作开发了.

  [0]: http://www.llinode.com