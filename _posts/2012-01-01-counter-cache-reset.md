---
layout: post
title: Counter Cache Reset for Counters
categories:
- rails
---

When you want to implement [counter cache][0], how to handle the counters for the old data?
Generally you will add a migration which runs a script to update all the counts, unfortunately you wil get an error,
because the column is readonly, you can't update this column in a normal way.

Model
    
{% highlight ruby %}

    class Post < ActiveRecord::Base
      belongs_to :category, :counter_cache => :resources_count
    end

{% endhighlight %}

Incorrect way
    
{% highlight ruby %}

    class InitializeResourcesCountForCategories < ActiveRecord::Migration
      def up
        Category.find_each do |cat|
          cat.update_attribute(:resources_count => cat.posts.count)
        end
      end
    end

{% endhighlight %}


You should be using `User.reset_counters` to do this

{% highlight ruby %}

    class InitializeResourcesCountForCategories < ActiveRecord::Migration
      def up
        Category.find_each do |cat|
          Category.reset_counters(cat.id, :posts)
        end
      end
    end

{% endhighlight %}


  [0]: http://api.rubyonrails.org/classes/ActiveRecord/CounterCache.html