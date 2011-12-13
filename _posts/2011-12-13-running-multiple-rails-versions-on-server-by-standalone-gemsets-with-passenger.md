---
layout: post
title: Running Multiple Rails Versions by Standalone Gemsets with Passenger
categories:
- deploy
- RVM
- passenger
---

There was a case before that troubled me a lot, I have many projects to run on production server that each one has a different rails version, and each project should be run in a standalone gemset with RVM, I have implemented it before, and now, I will make a guide to show you guys hwo to run multiple rails versions with phusion passenger and RVM.

Just give a example to you guys, I have project 'foo' run with rails 3.0.10 and project 'bar' run with rails 3.1.3 which is the latest version for rails 3, As of Phusion Passenger 3 you can run all components as Phusion Passenger.

The setup that we currently recommend is to combine Phusion Passenger for Nginx, with Phusion Passenger Standalone. All applications that are to use a different gemset can be served separately through Phusion Passenger Standalone and hook into the main web server via a setup load paths configuration.

To run project 'foo' under gemset 'foo', you need to do several simple configurations below:

First you need to add a `.rvmrc` file to your rails root directory, I think every one may use this, if you don'y use rvmrc, never mind, just create it by yourself with

    $ rvm --rvmrc --create 1.9.3@foo
    
Then the `.rvmrc` file will be created, yeah, that's cool for the new functionanlity by RVM.

Create `config/setup_load_paths.rb` file in config directory, prefill with:

{% highlight ruby %}

    if ENV['MY_RUBY_HOME'] && ENV['MY_RUBY_HOME'].include?('rvm')
      begin
        rvm_path     = File.dirname(File.dirname(ENV['MY_RUBY_HOME']))
        rvm_lib_path = File.join(rvm_path, 'lib')
        $LOAD_PATH.unshift rvm_lib_path
        require 'rvm'
        RVM.gemset_use! 'foo'
      rescue LoadError
        # RVM is unavailable at this point.
        raise "RVM ruby lib is currently unavailable."
      end
    end

    # Pick the lines for your version of Bundler
    # If you're not using Bundler at all, remove all of them

    # Require Bundler 1.0 
    ENV['BUNDLE_GEMFILE'] = File.expand_path('../Gemfile', File.dirname(__FILE__))
    require 'bundler/setup'

    # Require Bundler 0/9
    # if File.exist?(".bundle/environment.rb")
    #   require '.bundle/environment'
    # else
    #   require 'rubygems'
    #   require 'bundler'
    #   Bundler.setup
    # end
{% endhighlight %}

The same as project `bar`, the only difference is set a difference gemset.

After these two configuration steps, you can deploy your project to server now,
if there is no gemset named 'foo' or 'bar on your server, you will got a error by running `cap deploy:setup`

    ERROR: Gemset 'foo' does not exist, rvm gemset create 'foo' first.
    
Follow the error message, you need to create a gemset named 'foo' on your server first.

    $ rvm use 1.9.3
    $ rvm gemset create foo
    
Then the foo gemset will be created, you can deploy your project to your server now,

    $ cap deploy:setup
    $ cap deploy
    
'bundle install' will automatically install all the gems under 'foo' gemset and run server with the right rails version.

Here is the deploy script:

{% highlight ruby %}

    $:.unshift(File.expand_path('./lib', ENV['rvm_path']))
    require "rvm/capistrano"
    ssh_options[:forward_agent] = true

    set :application, "foo"
    set :rvm_ruby_string, "1.9.3@#{application}"
    set :rvm_path, "/home/jerry/.rvm"
    set :rvm_bin_path, "/home/jerry/.rvm/bin"
    set :rvm_trust_rvmrcs_flag, 1

    set :scm, :git
    set :repository, "your_git_repo_url_here"
    set :branch, "master"

    set :deploy_to, "your_deploy_path_here/#{application}"
    set :user, "jerry"
    set :use_sudo, false
    server "your_server_domain_or_ip_here", :app, :web, :db, :primary => true
    set :deploy_via, :copy

    task :configure, :roles => :web do
      run "ln -s #{shared_path}/config/database.yml #{current_release}/config/database.yml"
      run "cd #{current_release}; bundle install --without development test"
    end

    task :trust_rvmrc, :roles => :web do
      run "rvm rvmrc trust #{current_release}"
    end

    after "deploy:update_code", :trust_rvmrc
    after 'deploy:update_code', :configure
    
{% endhighlight %}
    
Wish you happy with your deployment.
