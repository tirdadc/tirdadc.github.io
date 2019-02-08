---
title: Trouble installing mysql gem with Homebrew MySQL
date: 2019-02-08 10:37 EST
tags: bundler, mysql, homebrew
---

If the mysql gem fails to install during a `bundle install` and you have installed MySQL with Homebrew, try this (but swap out 8.0.13 for whatever version you're on):

```
gem install mysql2 -- --with-mysql-config=/usr/local/Cellar/mysql/8.0.13/bin/mysql_config
```
