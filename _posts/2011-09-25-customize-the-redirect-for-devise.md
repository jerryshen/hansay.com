---
layout: post
title: Customize the Redirect for Devise
categories:
- Rails Plugins
---

Devise has many default redirect, it depends on what you'v done with Devise.

Normally, after a user signed in, they are redirected to the root path.
But we should have Devise redirect to a custom route after signed in, such as a administrator.

If you are using the default routes with Devise, add the override methods in your `ApplicationController`

{% highlight ruby %}

    class ApplicationController < ActionController::Base

      protected

        def after_update_path_for(resource)
          if resource.is_admin?
            admin_root_path
          else
            super
          end
        end
    end

{% endhighlight %}

That's it, it very easy to customize your routes, we have other routes could be customized.

* after_sign_out_path_for(resource)
* after_update_path_for(resource)
* sign_out_and_redirect(resource)
* after_resending_confirmation_instructions_path_for(resource_name)
* after_confirmation_path_for(resource_name, resource)
* after_omniauth_failure_path_for(resource_name)
* after_sending_reset_password_instructions_path_for(resource_name)