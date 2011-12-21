---
layout: post
title: Method Missing Best Practice
categories:
- rails
---

Method missing allows access to the object attributes, which are held in the @attributes hash, as though they were first-class methods.

If you call a method that doesn’t exist, Ruby will call the method_missing method, and pass the name of the method and any arguments you supplied, which means you can dynamically handle the method.

Here we go, for example, we have a messages table:

{% highlight ruby %}

    create_table "messages", :force => true do |t|
      t.text     "content"
      t.boolean  "is_read",     :default => false
      t.boolean  "is_archived", :default => false
      t.integer  "post_id"
      t.integer  "user_id"
      t.datetime "created_at"
      t.datetime "updated_at"
    end

{% endhighlight %}

As a general way, many people will define some following methods

{% highlight ruby %}

    class Message < ActiveRecord::Base
      
      def is_read?
        !!is_read
      end
      
      def is_archived?
        !!is_archived
      end
      
      def mark_as_read!
        update_attribute(:is_read, true)
      end
      
      def mark_as_unread!
        update_attribute(:is_read, false)
      end
      
      def mark_as_archived!
        update_attribute(:is_archived, true)
      end
      
      def mark_as_unarchived!
        update_attributes(:is_archived, false)
      end
      
    end

{% endhighlight %}

Wow, this makes your code ugly, we can use `method missing` instead of these all methods

{% highlight ruby %}

    def method_missing(symbol, *args)
      case symbol
        when /^is_(un)?(.*)\?/
          eval "#{$1 ? '!' : '!!'}is_#{$2}"
        when /^mark_as_(un)?(.*)!/
          update_attribute :"is_#{$2}", ($1 ? false : true)
      else
        super
      end
    end

{% endhighlight %}

When you call `is_[string]` method or `mark_as_[string]` method, as you can see, there is definitely no `is_[string]` or `mark_as_[string]` method, yet Rails is smart enough to call the method with the correct parameters, What we have done is to run a regular expression against the name of the method, and if it matches the pattern `is_[string]` or `mark_as_[string]` then it calls the method with the [string] part as the first parameter and the first argument as the second parameter. Pretty powerful, don’t you think?
