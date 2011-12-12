---
layout: post
title: Deployment with RVM, nginx, and passenger for ruby on rails
categories:
- deploy
- RVM
- nginx
- passenger
---

**服务器系统:**   ubuntu server 11.10 64bit  
**案例服务器:**   Linode VPS 512系列 

租下vps, 选择所要的系统 ubuntu server 11.10 64bit, 我一直不喜欢直接用root来做部署, 所以我会创建一个具有sudo权限的用户来完成整个部署和操作, 下面以jerry为例:

    $  useradd jerry            # 添加用户'jerry'
    $  passwd jerry             # 为'jerry'设置密码
    $  gpasswd -a jerry sudo    # 将用户jerry添加进sudo组(sudo组是默认就存在的，所以不用创建)
    $  cd /home && mkdir jerry  # 为jerry创建工作目录
    $  chown jerry /home/jerry  # 设置权限

添加jerry后可能是不是bash环境, 如果发现不是bash环境, 可以融过root用户将jerry的环境改成bash

    $ usermod --shell=/bin/bash jerry

通过编辑`authorized_keys`, 添加public key到jerry用户目录下, 这样以后ssh远程登录服务器就无须输入密码了

    $  cd ~  
    $  mkdir .ssh  && cd .ssh  
    $  vim authorized_keys      # 贴入public key

接下来需要安装[RVM][1], 安装RVM前需要安装几个必要的packages

    $  sudo apt-get install curl git-core libtool

然后安装rvm 

    $  bash < <(curl -s https://rvm.beginrescueend.com/install/rvm)

在 `~/.bashrc` 中设置RVM的环境变量 

    if [[ -s "$HOME/.rvm/scripts/rvm" ]]  ; then source "$HOME/.rvm/scripts/rvm" ; fi

在 `~.bash_profile` 中引用 `.bashrc`  

    source ~/.bashrc

推出终端重新进入, 你会发现RVM已经生效了

    $  rvm notes  

安装必要的packages 

    $  sudo apt-get install bison build-essential zlib1g zlib1g-dev libssl-dev lib64readline-gplv2-dev libxml2-dev libxslt1-dev autoconf  

安装完以上的packages后, 正式用rvm编译ruby 1.9.3, rvm会将ruby自动编译到当前用户目录的 `~/.rvm` 下(非root用户). 

    $  rvm install 1.9.3

安装完ruby 1.9.3后, 我们可以把rvm ruby 1.9.3设置为默认

    $  rvm --default ruby-1.9.3-p0

接下来就可以查看ruby 版本了

    $  ruby -v

至此 Ruby 1.9.3编译已经完成

**安装 Postgresql 数据库**  

首先需要确认当前用户 env的LANG是否为UTF8,

    $  env

看显示信息是否有 LANG=en_US.UTF8，如果没有,在 `/etc/profile` 中加入环境变量:  
    
    export LANG=en_US.UTF8

之后退出终端再进入, 你的设置就生效了, 然后安装postgresql数据库, 以 ubuntu 11.10为例, 版本应该是 9.1

    $  sudo apt-get install postgresql

切换到root用户下, 初始化postgresql用户和密码  

    $  su postgres -c psql postgres
    $  ALTER USER postgres WITH PASSWORD 'postgres';
    $  \q  

其中 'postgres' 为要修改的密码

编辑 `/etc/postgresql/9.1/main/pg_hba.conf`

    # Database administrative login by UNIX sockets
    # local   all         postgres                        ident
    
    # TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD
    
    # "local" is for Unix domain socket connections only
    local   all         all                               md5 #ident
    # IPv4 local connections:
    host    all         all         127.0.0.1/32          md5
    # IPv6 local connections:
    host    all         all         ::1/128               md5  

如果不希望允许 postgres 使用密码登入的可以开启第2行 `# local all postgres ident` 的注释  

编辑 `/etc/postgresql/9.1/main/postgresql.conf`, 搜索 `# listen_addresses =` 将其修改为  

    listen_addresses = 'localhost'    # which IP address(es) to listen on;  

如果允许其他机器访问的请将 localhost 修改成 *  

重启数据库

    $  /etc/init.d/postgresql restart

**安装 Phusion Passenger**

ruby1.9.3安装好之后, 会自动带一个空的gemset, 切换到该gemset下并安装passenger

    $ rvm use 1.9.3@
    $ gem install -V passenger   

**安装 nginx 服务器**  

下载最新的stable version nginx

    $  mkdir -p /home/jerry/opt/src && cd /home/jerry/opt/src
    $  wget http://nginx.org/download/nginx-1.0.10.tar.gz
    $  tar xvf nginx-1.0.10.tar.gz

安装编译相关类库

    $  sudo apt-get install libpcre3-dev  

编译安装带有 passenger 模块的nginx  

使用 passenger 脚本 `passenger-install-nginx-module` 编译 nginx

选择 `2. No: I want to customize my Nginx installation. (for advanced users)`

输入 `src: /home/jerry/opt/src/nginx-1.0.10` 和 `prefix: /home/jerry/opt/nginx`

添加编译参数并编译

    $  --conf-path=/home/jerry/opt/etc/nginx/nginx.conf --with-http_gzip_static_module  

如果还要启动其他编译参数请自行添加  

另外如果不想使用passenger自带脚本编译nginx, 也可以手工编译nginx时加入以下参数, 来启动passenger模块     

    --add-module='/home/jerry//opt/passenger/ext/nginx 

**配置 nginx**

整理编译自动生成的配置文件  

    $  cd /home/jerry/opt/etc/nginx
    $  mkdir /home/jerry/opt/etc/nginx/default
    $  mv *.default default/
    $  mkdir conf.d
    $  mkdir sites-enabled

将 `/home/jerry/opt/etc/nginx/nginx.conf` 替换为  

    user  jerry jerry;
    worker_processes  1;

    events {
        worker_connections  1024;
    }


    http {
        include       mime.types;
        default_type  application/octet-stream;
        
        sendfile        on;
        #tcp_nopush     on;
        
        keepalive_timeout  65;
        gzip  on;
        
        include conf.d/*.conf;
        include sites-enabled/*;
    }  

添加 gzip_static 模块配置, 编辑 `/home/jerry/opt/etc/nginx/conf.d/gzip_static.conf`

    gzip_static on;
    gzip_types text/css application/x-javascript;

添加 Passneger 模块配置, 编辑 `/home/jerry/opt/etc/nginx/conf.d/passenger.conf`

    passenger_root /home/jerry/.rvm/gems/ruby-1.9.3-p0/gems/passenger-3.0.11;
    passenger_ruby /home/jerry/.rvm/bin/ruby-1.9.3-p0;

将nginx的server配置链接到sites-enabled下

    $  ln -s /home/jerry/apps/conf/nginx.conf /home/jerry/opt/etc/nginx/sites-enabled/default  

编辑 `/home/jerry/apps/conf/nginx.conf` 

    server{
      listen 80;
      server_name  demo.com www.demo.com;
      root /home/jerry/apps/demo/current/public;
      passenger_enabled on;

      location ~ ^/(images|javascripts|stylesheets)/  {
          root /home/jerry/apps/demo/current/public;
          expires 30d;
      }
    }

如果需要配置 详细的配置信息, 请参考 [Nginx 文档][1]  

添加启动脚本 `/home/jerry/opt/etc/init.d/nginx` 内容为  

    #! /bin/sh
    
    ### BEGIN INIT INFO
    # Provides:          nginx
    # Required-Start:    $all
    # Required-Stop:     $all
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: starts the nginx web server
    # Description:       starts nginx using start-stop-daemon
    ### END INIT INFO
    
    # PATH=/home/jerry/.rvm/gems/ruby-1.9.3-p0/bin:/bin:/home/jerry/.rvm/rubies/ruby-1.9.3-p0/bin:/home/jerry/.rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
    DAEMON=/home/jerry/opt/nginx/sbin/nginx
    NAME=nginx
    DESC=nginx
    PIDFILE=/home/jerry/opt/nginx/logs/$NAME.pid
    
    test -x $DAEMON || exit 0
    
    # Include nginx defaults if available
    if [ -f /home/jerry/opt/etc/default/nginx ] ; then
        . /home/jerry/opt/etc/default/nginx
    fi
    
    set -e
    
    . /lib/lsb/init-functions
    
    test_nginx_config() {
        if $DAEMON -t; then
            return 0
        else
            return $?
        fi
    }
    
    case "$1" in
        start)
            echo -n "Starting $DESC: "
            test_nginx_config
            start-stop-daemon --start --quiet --pidfile $PIDFILE \
                --exec $DAEMON -- $DAEMON_OPTS || true
            echo "$NAME."
        ;;
        stop)
            echo -n "Stopping $DESC: "
            start-stop-daemon --stop --quiet --pidfile $PIDFILE \
                --exec $DAEMON || true
            echo "$NAME."
        ;;
        restart|force-reload)
            echo -n "Restarting $DESC: "
            start-stop-daemon --stop --quiet --pidfile \
                $PIDFILE --exec $DAEMON || true
            sleep 1
            test_nginx_config
            start-stop-daemon --start --quiet --pidfile \
                $PIDFILE --exec $DAEMON -- $DAEMON_OPTS || true
            echo "$NAME."
        ;;
        reload)
            echo -n "Reloading $DESC configuration: "
            test_nginx_config
            start-stop-daemon --stop --signal HUP --quiet --pidfile $PIDFILE \
               --exec $DAEMON || true
            echo "$NAME."
        ;;
        configtest)
            echo -n "Testing $DESC configuration: "
            if test_nginx_config; then
                echo "$NAME."
            else
                exit $?
            fi
        ;;
        status)
            status_of_proc -p $PIDFILE "$DAEMON" nginx && exit 0 || exit $?
        ;;
        *)
            echo "Usage: $NAME {start|stop|restart|reload|force-reload|status|configtest}" >&2
            exit 1
        ;;
    esac
    
    exit 0

切换到root用户, 设置nginx的随机服务启动

    $  chmod +x /home/jerry/opt/etc/init.d/nginx
    $  ln -s /home/jerry/opt/etc/init.d/nginx /etc/init.d/nginx
    $  update-rc.d nginx defaults

到这里nginx的配置已经完成，接下来你就可以将项目发布到服务器了.
  
  [1]: http://beginrescueend.com/
  [2]: http://wiki.nginx.org/NginxFullExample