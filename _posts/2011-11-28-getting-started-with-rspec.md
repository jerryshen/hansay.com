---
layout: post
title: Getting Started with RSpec
categories:
- rspec
---

One of the reasons that I love rails is the awesome test frameworks. Here’s an introduction for those of you interested in getting started with RSpec.

Everyone may know Behaviour Driven Development, It's an Agile development process that comprises aspects of Acceptance Test Driven Planning, Domain Driven Design and Test Driven Development. RSpec is a BDD tool aimed at TDD in the context of BDD.

Here is my gemfile.

{% highlight ruby %}

    #gemfile
    
    # mongo support
    gem 'mongo',      '~> 1.3.1'
    gem 'mongoid',    '~> 2.3.3'
    gem 'bson',       '~> 1.3.1'
    gem 'bson_ext',   '~> 1.3.1'
    
    group :development, :test do
      gem 'machinist',        '~> 2.0.0.beta2'
      gem 'machinist_mongo',  :git => 'https://github.com/nmerouze/machinist_mongo.git', :require => 'machinist/mongoid', :branch => 'machinist2'  # you need to add this if you are using mongoid
      gem 'database_cleaner', '~> 0.6.7'
      gem 'ffaker',           '~> 1.8.1'
      gem 'launchy'
      gem 'guard-rspec'
    end

    group :test do
      gem 'rspec',            '~> 2.7.0'
      gem 'rspec-rails',      '~> 2.7.0'
      gem 'mongoid-rspec',    '~> 1.4.4'
      gem 'spork',            '~> 0.8.5'
      gem 'cucumber-rails',   '~> 1.2.0'
      gem 'capybara',         '~> 1.1.1'
      gem 'email_spec'
      gem 'simplecov',        :require => false
      gem 'rb-fsevent',       :require => false
      gem 'growl_notify'
    end

{% endhighlight %}

In this gemfile, I add two gems 'machinist_mongo and 'mongoid-rspec' because of I'm using Mongoid in my project, if you are using activerecord, just remove these two gems.

#### Installation

Just Run `bundle` to install all the gems listed in gemfile

RSpec works great with other assistant plugins as I listed above, I will introduce you one by one.

---

#### RSpec

Someone would say that unit testing framework is good enough, but what I would say is that RSpec is a Domain Specific Language for describing the expected behaviour of a system with executable examples.

#### Configuration

Initialize Rspec

    $ rails g rspec:install
    
It will generate files and directories as below

    create  .rspec
    create  spec
    create  spec/spec_helper.rb

Let's modify `spec_helper.rb` to extend some awesome tools

{% highlight ruby %}

    require 'rubygems'
    require 'simplecov'
    require 'spork'
    SimpleCov.start 'rails'

    # This file is copied to spec/ when you run 'rails generate rspec:install'
    ENV["RAILS_ENV"] ||= 'test'
    require 'rails/mongoid'
    Spork.trap_class_method(Rails::Mongoid, :load_models)
    require File.expand_path('../../config/environment', __FILE__)
    require 'rspec/rails'

    # Requires supporting ruby files with custom matchers and macros, etc,
    # in spec/support/ and its subdirectories.
    Dir[Rails.root.join('spec/support/**/*.rb')].each {|f| require f}

    RSpec.configure do |config|
      config.mock_with :rspec
      config.include Mongoid::Matchers

      DatabaseCleaner.strategy = :truncation

      config.before do
        DatabaseCleaner.clean
      end

    end

{% endhighlight %}

Bye the way, I would like to suggest you change your rails generator in `config/application.rb`, add the following code:

{% highlight ruby %}

    config.generators do |g|
      g.orm                 :mongoid
      g.template_engine     :haml
      g.test_framework      :rspec, :fixture => false, :views => false
      g.fixture_replacement :machinist
    end

{% endhighlight %}

---

#### Guard

Guard watches files and runs a command after a file is modified. This allows you to automatically run tests in the background, restart your development server, reload the browser, and more.

##### Configuration

After you run `bundle` to install everything and, once that's done, run `guard init` rspec to set up Guard

    $ guard init
    Writing new Guardfile to (here is your path)
    rspec guard added to Guardfile, feel free to edit it

It will generate a Guardfile in your rails root directory, make it like:

{% highlight ruby %}

    # A sample Guardfile
    # More info at https://github.com/guard/guard#readme

    guard 'rspec', :version => 2 do
      watch(%r{^spec/.+_spec\.rb$})
      watch(%r{^lib/(.+)\.rb$})     { |m| "spec/lib/#{m[1]}_spec.rb" }
      watch('spec/spec_helper.rb')  { "spec/" }

      # Rails example
      watch(%r{^spec/.+_spec\.rb$})
      watch(%r{^app/(.+)\.rb$})                           { |m| "spec/#{m[1]}_spec.rb" }
      watch(%r{^lib/(.+)\.rb$})                           { |m| "spec/lib/#{m[1]}_spec.rb" }
      watch(%r{^app/controllers/(.+)_(controller)\.rb$})  { |m| ["spec/routing/#{m[1]}_routing_spec.rb", "spec/#{m[2]}s/#{m[1]}_#{m[2]}_spec.rb", "spec/acceptance/#{m[1]}_spec.rb"] }
      watch(%r{^spec/support/(.+)\.rb$})                  { "spec/" }
      watch('spec/spec_helper.rb')                        { "spec/" }
      watch('config/routes.rb')                           { "spec/routing" }
      watch('app/controllers/application_controller.rb')  { "spec/controllers" }
      # Capybara request specs
      watch(%r{^app/views/(.+)/.*\.(erb|haml)$})          { |m| "spec/requests/#{m[1]}_spec.rb" }
    end

{% endhighlight %}

Guard also has support for Growl notifications and these can be enabled by installing the growl gem, You can find `rb-fsevent` and `growl_notify` as I listed in Gemfile, but it only works via Mac OSX, remove these if you are using linux.

We can now run the `guard` command to start up the Guard server.

    $ guard
    Guard is now watching at '/Users/shenjerry/Workspace/demo'
    Guard::RSpec is running, with RSpec 2!
    Running all specs
    ...

    Finished in 1.02 seconds
    3 examples, 0 failures
    
If we make a breaking change to our code, for example remove a method in your user model, Guard will run immediately and we’ll see the broken test clearly.

---

#### Machinist

Machinist is awesome plugin which make it easy to create objects for use in tests, It generates data for the attributes you don't care about, and constructs any necessary associated objects, leaving you to specify only the fields you care about in your test.

##### Configuration

    $ rails g machinist:install
    
It will generate `blueprint.rb` file in your `spec/support` directory.

Here is a example to define a blueprint:

{% highlight ruby %}

    require 'machinist/mongoid'

    User.blueprint do
      email     { Faker::Internet.email }
      username  { "username_#{sn}" }
      password  { "password" }
      password_confirmation { object.password }
    end

{% endhighlight %}

If you want to create a user in your console, you need require the blueprints first

    >> require Rails.root.join('spec', 'support/blueprints')
    >> User.make!

Now you can find this user in database.

Another case that you want to define a admin blueprint as the same user model, if you supposed to add a whole blueprint again, that's ugly.
Here is the simple way to handle this:

{% highlight ruby %}

    require 'machinist/mongoid'

    User.blueprint(:admin) do
      is_admin  { true }
    end

{% endhighlight %}

You can use `User.make!(:admin)` to create a admin.

    >> require Rails.root.join('spec', 'support/blueprints')
    >> User.make!(:admin)

