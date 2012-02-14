---
layout: post
title: Using APN Sender to Send Push Notifications to Your IOS Devices
categories:
- IOS
---

Happy Valentine's Day.

Recently, I have a project based on both ruby on rails and iPhone applications, there is a requirement that is pushing notifications from server side to iPhone client, and I built a server component of an iPhone application in Ruby with [apn_sender][1].

I want to send background notifications through Apple Push Notification servers, which I was stumbled at first, to setup the plugin is very simple, but I couldn't receive any notifications after sending them, it's very weird that everything seemed right. After further testing and trying, I found the problem is format of IOS Device Token, Because I encoded and stored it in mongodb as a Base64 string, So I have to decode it to hexadecimal.

APN Sender is based on Resque and Redis, so I wrote a script to skip the Resque Queue to make development easier.

{% highlight ruby %}

    require 'base64'

    ios_device_token = User.where(:username => 'Jerry').first.ios_device_token
    token = Base64.decode64(ios_device_token).unpack('H*').first.scan(/\w{8}/).join(' ')

    notification = APN::Notification.new(token, { :alert => 'Hey, how are you?', :sound => true, :badge => 9 })

    sender = APN::Sender.new(verbose: true, environment: 'production')
    sender.send_to_apple(notification)

{% endhighlight %}

The app installed on my iPhone is based on Ad Hoc, it's very important that you should set the environment to production when you initialize the sender object, otherwise you would receive nothing.


  [1]: https://github.com/kdonovan/apn_sender
