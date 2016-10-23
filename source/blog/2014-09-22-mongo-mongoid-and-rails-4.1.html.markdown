---
title: "Mongo, Mongoid and Rails 4.1"
date: 2014-09-22 20:41:22 -0400
comments: true
tags: Ruby On Rails, Rails 4.1, Mongo, Mongoid
---

I typically forget to set the <code>--skip-active-record</code> when creating a new app, so this is the
little kabuki dance you need to perform to get [Mongoid](http://mongoid.org) to play nice with a freshly
created Rails 4.1 project.

1. Add mongoid to your Gemfile:<br>
<code>gem 'mongoid', '~> 4.0.0'</code>

2. Generate your `mongoid.yml` configuration file:<br>
<code>rails g mongoid:config</code>

3. Remove the sqlite gem now from your Gemfile:<br>
<code>gem 'sqlite3'</code>

4. Delete `database.yml`

5. Remove any lines that reference active_record from your environment files:

``` ruby
# development.rb
config.active_record.migration_error = :page_load

# production.rb
config.active_record.dump_schema_after_migration = false
```

<span>6.</span> Change this in your `application.rb`:

``` ruby
require 'rails/all'
```
to:
``` ruby
require "action_controller/railtie"
require "action_mailer/railtie"
require "sprockets/railtie"
```
