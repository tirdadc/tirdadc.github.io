---
title: "Upgrading to Ruby 2.2.1 with rbenv and RVM"
date: 2015-03-06 18:14:36 -0500
---

Another one of those brief utilitarian blog posts for myself.

rbenv:

``` sh
rbenv install -l
rbenv install 2.2.1
rbenv local 2.2.1
rbenv rehash
gem install bundler
bundle install
```

RVM:

``` sh
rvm install 2.2.1
rvm use 2.2.1
bundle install
```

Since Passenger is made aware of which Ruby to use, update the vhost entry too:

```
# /etc/apache2/sites-enabled/example.com
PassengerRuby /home/johndoe/.rvm/wrappers/ruby-2.2.1/ruby
```

and you then have to restart Apache:

```
sudo apache2ctl graceful
```
