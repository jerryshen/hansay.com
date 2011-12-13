---
layout: post
title: Curl Problem when Installing Nginx Using Passenger Module
categories:
- nginx
---

I often got this kind of problem when installing nginx using passenger module,

    Checking for required software...

     * GNU C++ compiler... found at /usr/bin/g++
     * The 'make' tool... found at /usr/bin/make
     * A download tool like 'wget' or 'curl'... found at /usr/bin/wget
     * Ruby development headers... found
     * OpenSSL support for Ruby... found
     * RubyGems... found
     * Rake... found at /home/jerry/.rvm/wrappers/ruby-1.9.3-p0/rake
     * rack... found
     * Curl development headers with SSL support... not found
     * OpenSSL development headers... found
     * Zlib development headers... found

    Some required software is not installed.
    But don't worry, this installer will tell you how to install them.
    
Yes, that's easy to resolve, your system is lack of a package named `libcurl4-openssl-dev`,
You can install it with:

    $ sudo apt-get install libcurl4-openssl-dev
    
Then, run your `passenger-install-nginx-module`, it will pass all the required packages.