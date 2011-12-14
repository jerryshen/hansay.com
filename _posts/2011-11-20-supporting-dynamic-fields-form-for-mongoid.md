---
layout: post
title: Supporting Dynamic Fields Form for Mongoid
categories:
- mongoid
---

Mongoid currently supports the dynamic fields configuration option, which provided in the mongoid.yml,

    allow_dynamic_fields: true
    
When attributes are not defined as fields but added to an object, they will get fields added for them dynamically and will get persisted. If set to false an error will get raised when attempting to set a value that has no field defined.

But there is a problem with a dynamic fields form, you will get raised when attempting to add a undefined atribute in your form, so question comes, how to enable form to support dynamic fields that could be added to an object? The only thing you should do is that override the `method_missing` method in mongoid module. 

{% highlight ruby %}

    module Mongoid 
      module Attributes
        alias_method :old_method_missing, :method_missing
        def method_missing(name, *args)
          old_method_missing(name, *args)
        rescue
          nil
        end
      end
    end 

{% endhighlight %}

You can directly put this file in `config/initializer` directory, or `lib` and require this file, whatever you like.