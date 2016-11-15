---
title: Migrating from Ruby on Rails 4 to 5
date: 2016-11-15 08:47 EST
tags: Ruby On Rails
---

Start off by reading the [official edge guide](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-4-2-to-rails-5-0) as it covers the basics. This is meant mostly as a side note for 3rd party gems and other common concerns.

#### Gemfile changes:

Start off by setting the Rails version to 5 in your Gemfile:

``` diff
-gem 'rails', '~> 4.2.5'
+gem 'rails', '~> 5.0.0'
```

If you're using devise, switch to the github branch:

``` diff
-gem 'devise'
+gem 'devise', github: 'plataformatec/devise'
```

Same with Sinatra:

``` diff
-gem 'sinatra', require: nil
+gem 'sinatra', require: nil, github: 'sinatra'
```

If you have controller specs (who doesn't?), you're going to need [rails-controller-testing](https://github.com/rails/rails-controller-testing).

``` diff
+  gem 'rails-controller-testing'
```

``` diff
# spec/rails_helper.rb

-  config.include Devise::TestHelpers, type: :controller
+  config.include Devise::Test::ControllerHelpers, type: :controller
```

If you were using `transactional_fixtures = true` and `test_after_commit`, you can now get rid of the gem:

``` diff
-  gem 'test_after_commit'
```

If you're using quiet_assets, you can remove it and enable a new flag to have the same behavior:

``` diff
-gem 'quiet_assets'
```

``` ruby
# config/environments/development.rb

config.assets.quiet = true
```

#### Turbolinks
If you were using Turbolinks, you'll need to slightly alter the document event code:

``` js
// app/assets/javascripts/users.js
var ready;
ready = function() {
 // Your code here
}
$(document).ready(ready);
$(document).on('page:load', ready);
```

to:

``` js
var ready;
ready = function() {
 // Your code here
}
$(document).ready(ready);
$(document).on('turbolinks:load', ready);
```

The progress bar is enabled by default, so you can remove this:

``` diff
-Turbolinks.enableProgressBar();
```

[Turbolinks 5](https://github.com/turbolinks/turbolinks) is a completely different beast, so take some time to read up on it.

#### Specs

Fix some deprecation notices thrown during tests:

``` ruby
# config/environments/test.rb

-  config.serve_static_files  = true
-  config.static_cache_control = 'public, max-age=3600'
+  config.public_file_server.enabled = true
+  config.public_file_server.headers = { 'Cache-Control' => 'public, max-age=3600' }
```

You'll need to explicitly state params as a hash:

``` diff
describe "GET edit" do
  it "assigns the requested user as @user" do
    user = User.create! valid_attributes
-     get :edit, id: user.to_param
+     get :edit, params: { id: user.to_param }
    expect(assigns(:user)).to eq(user)
  end
end
