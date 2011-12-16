---
layout: post
title: Allow Users to Login with Username or Email Using Devise
categories:
- Rails Plugins
---

Devise is a flexible authentication solution for Rails based on Warden,
I love it because it's very simple to use and easy to extend or override. Many people would say that
It's too huge to include all the devise's modules, but what I would say is that, take it easy, agile is the most important, and it's not so huge because of most of the modules are necessary for such a simple project.

Many websites are both enabled username and email to sign in, It will be very friendly to the users,
people will unhappy that every time they must type such a long email address. So Let me show you
how to allow users to login with username or email using Devise.

First of all, you must have a column named 'username' in your users table.

Then we can start, you should modify the User model and add username to attr_accessible.

{% highlight ruby %}

    attr_accessible :username
    
{% endhighlight %}

To support both username and email, you should create a login virtual attribute in Users and set the accessible

{% highlight ruby %}

    # Virtual attribute for authenticating by either username or email
    # This is in addition to a real persisted field like 'username'
    attr_accessor :login
    attr_accessible :login

{% endhighlight %}

As default, Devise use `:email` for authentication in the authentication_keys, so you should tell Devise to use
`:login` in the authentication_keys, modify `config/initializers/devise.rb` to have:

{% highlight ruby %}

    config.authentication_keys = [ :login ]

{% endhighlight %}

You should overwrite Devise’s `find_for_database_authentication` method in user model

{% highlight ruby %}

    # This is for ActiveRecord
    def self.find_for_database_authentication(warden_conditions)
       conditions = warden_conditions.dup
       login = conditions.delete(:login)
       where(conditions).where(["lower(username) = :value OR lower(email) = :value", { :value => login.strip.downcase }]).first
     end
     
     # This is for Mongoid:
     field :email
     
     def self.find_for_database_authentication(conditions)
       login = conditions.delete(:login)
       self.any_of({ :username => login }, { :email => login }).first
     end

{% endhighlight %}

Note: This code for Mongoid does some small things differently then the ActiveRecord code above. Would be great if someone could port the complete functionality of the ActiveRecord code over to Mongoid [basically you need to port the ‘where(conditions)’]. It is not required but will allow greater flexibility.

After these settings, make sure you have the Devise views in your project so that you can update `devise/sessions/new` view.

{% highlight ruby %}

    #  devise/sessions/new.html.erb
    -  <p><%= f.label :email %><br />
    -  <%= f.email_field :email %></p>
    +  <p><%= f.label :login %><br />
    +  <%= f.text_field :login %></p>

{% endhighlight %}

Add i18n translation for login field

{% highlight ruby %}

    en:
      activerecord:
        attributes:
          user:  
            login: "Username or email"

{% endhighlight %}

Yeah, you have done with the configuration to allow user sign in with username or email address, now there is another problem, you also need to allow users to recover their password using either username or email address, right? Let's do it.

Devise set the default reset password keys to `:email`, so you have to tell Devise to use `:loign`
in the `reset_password_keys`

{% highlight ruby %}
    
    # config/initializers/devise.rb
    config.reset_password_keys = [:login]

{% endhighlight %}

Devises's finder methods in user model should be overwritten to support both username and email address.

**This is for ActiveRecord**

{% highlight ruby %}

    protected

     # Attempt to find a user by it's email. If a record is found, send new
     # password instructions to it. If not user is found, returns a new user
     # with an email not found error.
     def self.send_reset_password_instructions(attributes={})
       recoverable = find_recoverable_or_initialize_with_errors(reset_password_keys, attributes, :not_found)
       recoverable.send_reset_password_instructions if recoverable.persisted?
       recoverable
     end 

     def self.find_recoverable_or_initialize_with_errors(required_attributes, attributes, error=:invalid)
       (case_insensitive_keys || []).each { |k| attributes[k].try(:downcase!) }

       attributes = attributes.slice(*required_attributes)
       attributes.delete_if { |key, value| value.blank? }

       if attributes.size == required_attributes.size
         if attributes.has_key?(:login)
            login = attributes.delete(:login)
            record = find_record(login)
         else  
           record = where(attributes).first
         end  
       end  

       unless record
         record = new

         required_attributes.each do |key|
           value = attributes[key]
           record.send("#{key}=", value)
           record.errors.add(key, value.present? ? error : :blank)
         end  
       end  
       record
     end

     def self.find_record(login)
       where(["username = :value OR email = :value", { :value => login }]).first
     end

{% endhighlight %}

**These two are all for Mongoid**

{% highlight ruby %}

    # as finder
    def self.find_record(login)
      found = where(:username => login).to_a
      found = where(:email => login).to_a if found.empty?
      found
    end

    # This can be optimized using a custom javascript function
    def self.find_record(login)
      where("function() {return this.username == '#{login}' || this.email == '#{login}'}")
    end

{% endhighlight %}

Don't forget to update your view and enjoy