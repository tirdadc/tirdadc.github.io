---
title: Using JSONB attributes in Ruby on Rails
date: 2016-11-03 07:31:22 -0500
tags: Ruby on Rails, Postgres
---

As you probably already know by now, Postgres 9.4 introduced JSONB as a new column type, effectively allowing you to use it both as a relational database and a full-fledged indexed and queryable document store when needed instead of trying to shoehorn everything into the latter and then dealing with the inevitable aftermath. Ruby on Rails has provided support for JSONB since [4.2](http://guides.rubyonrails.org/4_2_release_notes.html).

There's already a [solid](https://blog.codeship.com/unleash-the-power-of-storing-json-in-postgres/) [amount](http://nandovieira.com/using-postgresql-and-jsonb-with-ruby-on-rails) of [blog](http://robertbeene.com/rails-4-2-and-postgresql-9-4/) posts out there on this topic, so this one aims to focus on a practical aspect of implementing it.

Start off by adding an indexed JSONB field to your table:

```ruby
class AddWebsitesToUser < ActiveRecord::Migration
  def change
    add_column :users, :websites, :jsonb, default: '{}'
    add_index  :users, :websites, using: :gin
  end
end
```

Now you can set whatever data you want inside of that field using keys:

```ruby
user.websites['primary_website'] = 'http://www.example.com'
```

Using a store accessor allows you to specify the keys you want to access directly without referencing the JSONB field:

```ruby
# user.rb
store_accessor :websites, :primary_website, :secondary_website, :optional_website
```

Which allows you to do this:

```ruby
user.primary_website
#=> 'http://wwww.example.com'
```

Your client `_form.html.erb` can then treat these as regular fields:

```erb
<div class="field">
  <%= f.label :primary_website %>
  <%= f.text_field :primary_website %>
</div>
<div class="field">
  <%= f.label :secondary_website %>
  <%= f.text_field :secondary_website %>
</div>
<div class="field">
  <%= f.label :optional_website %>
  <%= f.text_field :optional_website %>
</div>
```

Be sure to whitelist them as usual in the controller:

```ruby
# users_controller.rb
def user_params
  params.require(:user).permit(
    :primary_website,
    :secondary_website,
    :optional_website
  )
end
```
