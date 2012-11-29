---
layout: post
title: Setting up PPTP VPN on Linode VPS
categories:
- linux
---

[Linode][0] is a VPS hosting company built upon one simple premise: provide the best possible tools and services to those that what they need.

I have a VPS on Linode, my blog runs on it, you can do any thing you want to do. For example we need to set up a PPTP VPN, it was based on Ubuntu Server 11.10 64bit.

Fir of all, you should update your sources.

{% codeblock %}
$ sudo apt-get update
$ sudo apt-get upgrade
{% endcodeblock %}
    
Now we can start to install VPN server, It's very easy to be set up with PPTP which is a kind of VPN Protocol.

{% codeblock %}
$ sudo apt-get install pptpd
{% endcodeblock %}
    
The only thing left to do is that we should add some configurations for pptpd.

Modify `/etc/pptpd.conf`, find 'localip' and 'remoteip' and replace with

{% codeblock %}
localip 192.168.13.1
remoteip 192.168.13.50-100
{% endcodeblock %}

After add the ip scope, we can continue adding users VPN with modifying `/etc/ppp/chap-secrets`, for example

{% codeblock %}
jerry pptpd 123456 *
{% endcodeblock %}

BTW, here 'jerry' is your username, '123456' is your password

Well, It's better to set a DNS to make sure pptpd can resove domains successfully when connected to VPN. You can modify `/etc/ppp/options`, try to find `ms-dns` and replace with Google's public DNS

{% codeblock %}
ms-dns 8.8.8.8
ms-dns 8.8.4.4
{% endcodeblock %}

Then modify `/etc/sysctl.conf`, try to find `net.ipv4.ip_forward=1` and enable it.

How to make the settings work? try to run

{% codeblock %}
$ sudo sysctl -p
{% endcodeblock %}

you also should restart pptpd's server

{% codeblock %}
$ sudo /etc/init.d/pptpd restart
{% endcodeblock %}

Finally, we should transmit ips witl iptable

{% codeblock %}
$ sudo /sbin/iptables -t nat -A POSTROUTING -s 192.168.13.0/24 -o eth0 -j MASQUERADE
{% endcodeblock %}

But there is a problem here, when you restart your server, the transmitting will disappear, so we can add it into bootup script. Switch to your root user, create `transmition` in `/etc/init.d/` directory, and add the following script

    /sbin/iptables -t nat -A POSTROUTING -s 192.168.13.0/24 -o eth0 -j MASQUERADE

You also have to make `transmition` executable, using

    $ chmod +x transmition
    $ update-rc.d transmition defaults 

[0]: http://www.linode.com