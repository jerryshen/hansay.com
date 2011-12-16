---
layout: post
title: Setting a Capisrano variable to Limit Releases
categories:
- deploy
- capistrano
---

Capistrano is a awesome tool to deploy your projects, Usually when using capistrano, I will go and manually delete old releases from a deployed application long time before. I understand that you can run `cap deploy:cleanup` but that still leaves 5 releases. But what I want is just keep 3 releases or even less, It's very simple to cleanup old releases to just 3 previous deploy with a capistrano variable, add it to your deploy script.

{% highlight ruby %}

    set :keep_releases, 3
    after 'deploy:restart', 'deploy:cleanup'

{% endhighlight %}

Also there is option 2 to set the the capistrano variable from the command line:

    $ cap deploy:cleanup -s keep_releases=3
