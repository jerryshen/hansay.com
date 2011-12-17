---
layout: post
title: Using Nested Form with Simple Form
categories:
- Rais Plugin
---

I would like to introduce you guys a Rails gem for conveniently manage multiple nested models in a single form.
It does so in an unobtrusive way through jQuery.

For now this gem only works with Rails 3.

Add it to your Gemfile then run `bundle` to install it.

{% highlight ruby %}

    gem 'nested_form', :git => 'git://github.com/ryanb/nested_form.git'

{% endhighlight %}

For Rails 3.1, the only thing you have to do is just require the js file in your `application.js`

{% highlight ruby %}

    # application.js
    //= require jquery
    //= require jquery_ujs
    //= require nested_form

{% endhighlight %}

So we can get started with nested form now, we can imagine that we have a Product model that `has_many :photos`.
To be able to use this function, you need to add some configuration to your Product model, ad we have Photo model
which is to store the photos.

{% highlight ruby %}

    class Photo
      include Mongoid::Document

      mount_uploader :data, PhotoUploader

      field :data, type: String

      embedded_in :product

    end

{% endhighlight %}

{% highlight ruby %}

    class Product
      include Mongoid::Document
      include Mongoid::Timestamps

      embedded_many :photos, cascade_callbacks: true

      accepts_nested_attributes_for :photos,
                                    reject_if: proc { |attributes| attributes['data'].blank? },
                                    allow_destroy: true
    end

{% endhighlight %}

If you don't have the `accepts_nested_attributes_for` settings for photos, you'll get a Missing Block Error.

We also use SimpleForm to make our form simpler, in this Git repo, It was implemented to use `simple_nested_form_for` for SimpleForm support. But this feature is not yet in a Gem release, so you can use this git repo.

Here is the form.

{% highlight ruby %}

    = simple_nested_form_for resource, :url => products_path do |f|
      = f.input :name
      
      - resource.photos.each do |photo|
        = f.simple_fields_for :photos, photo do |p|
          .photo
            = label_tag ''
            = image_tag photo.data
            %br/
            = label_tag ''
            = p.link_to_remove 'delete this image'
            = p.input :data_cache, :as => :hidden
            
      = f.simple_fields_for :photos, resource.photos.build do |p|
        = p.input :data, :as => :file
        = p.input :data_cache, :as => :hidden
        = p.link_to_remove 'remove'
      = f.link_to_add 'Add', :photos

      .actions
        = f.submit 'Save changes'
        = button_tag 'Reset', :type => 'reset'

{% endhighlight %}

This gem is produced by Ryanb, many thanks.


