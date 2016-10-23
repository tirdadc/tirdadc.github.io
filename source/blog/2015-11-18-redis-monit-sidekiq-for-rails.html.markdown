---
title: "Redis, Monit and Sidekiq 4 for Rails"
date: 2015-11-18 09:05:48 -0500
categories: [Redis, Monit, Sidekiq]
---

I recently upgraded my [Sidekiq](https://github.com/mperham/sidekiq) version to 4.0.0, which required upgrading my rather ancient version of Redis on my production environment. I had already set up Monit, but I'll cover the steps to get that up and running too.

# Monit

[Monit](https://mmonit.com/monit/) is a small Open Source utility for managing and monitoring Unix systems. We're going to use it to insure that Redis is always running, and restarted if the process goes down or the server gets rebooted.

## Install Monit

`$ sudo apt-get install monit`

## Configure Monit

Your main monit configuration file will be in `/etc/monit/monitrc`, and you're going to add a `redis.conf` file under `/etc/monit/conf.d/`.

```
# /etc/monit/conf.d/redis.conf
check process redis-server with pidfile "/var/run/redis/redis-server.pid"
  start program = "/usr/local/bin/redis-server /etc/redis/redis.conf"
  stop program = "/usr/local/bin/redis-cli shutdown"
  if failed host 127.0.0.1 port 6379 then restart
  if 5 restarts within 5 cycles then timeout
```

# Redis 3.0.5

[Installing Redis](http://redis.io/topics/quickstart) is pretty simple but you need to build it on Ubuntu to get the latest version:

```
$ wget http://download.redis.io/releases/redis-3.0.5.tar.gz
$ tar xzf redis-3.0.5.tar.gz
$ cd redis-3.0.5
$ make
```

you can also run `make test` to verify that everything checks out.

Once that is done, copy the files to `/usr/local/bin` and make sure you copy the new redis.conf to its appropriate location:

```
$ sudo cp src/redis-server /usr/local/bin/
$ sudo cp src/redis-cli /usr/local/bin/
$ sudo cp redis.conf /etc/redis/redis.conf
```

redis-cli is the command line tool, redis-server is what we'll be starting with Monit.

# Tying them together

You want monit to reload the configuration changes and to restart the redis-server:

```
$ sudo monit reload
$ sudo monit restart redis-server
```

You can see the output of the monit log at `/var/log/monit.log` to make sure everything is OK, or you can also use:

```
$ sudo monit status
```

I noticed these warnings:

```
15377:M 18 Nov 12:29:31.725 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
15377:M 18 Nov 12:29:31.725 # Redis can't set maximum open files to 10032 because of OS error: Operation not permitted.
15377:M 18 Nov 12:29:31.725 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
15377:M 18 Nov 12:29:31.726 # Warning: 32 bit instance detected but no memory limit set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.0.5 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 15377
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

15377:M 18 Nov 12:29:31.726 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
15377:M 18 Nov 12:29:31.727 # Server started, Redis version 3.0.5
15377:M 18 Nov 12:29:31.727 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
```
To fix these warnings:

```
$ sudo sysctl -w net.core.somaxconn=65535
$ sudo sysctl -w vm.overcommit_memory=1
$ sudo sh -c "ulimit -n 10032 && exec su $LOGNAME"
```

and edit your /etc/redis/redis.conf to set the following:

```
maxmemory 2mb
maxmemory-policy allkeys-lru
```
