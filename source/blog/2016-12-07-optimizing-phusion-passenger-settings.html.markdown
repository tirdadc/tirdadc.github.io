---
title: Optimizing Phusion Passenger Settings
date: 2016-12-07 09:04 EST
tags: Phusion Passenger, AWS, Ruby On Rails
---

[Proper configuration of Phusion Passenger](https://www.phusionpassenger.com/library/config/apache/optimization/) is an aspect of optimization that frequently gets missed on production deployments of Rails / Sinatra applications. With all of the legitimate emphasis on [preventing N+1 queries](https://github.com/flyerhzm/bullet), [not instantiating objects where a pluck will do](http://collectiveidea.com/blog/archives/2015/05/29/how-to-pluck-like-a-rails-pro) and other Rails-level performance improvements, server level fine-tuning is typically ignored and inefficient default settings end up getting used.

On a recent project we noticed a threshold of traffic during load testing at which the server would start returning 503's. After a lot of investigation, it eventually came down to the application server layer since everything else was barely strained or never solicited when the 503 responses starting pouring in.

By default, Phusion Passenger has a setting to control the max number of application processes that may simultaneously exist, and this [PassengerMaxPoolSize](https://www.phusionpassenger.com/library/config/apache/reference/#passengermaxpoolsize) is set to 6. This is fine when starting out, but once you start dealing with actual traffic you should definitely set it correctly.

The [documentation](https://www.phusionpassenger.com/library/config/apache/optimization/#tuning-the-application-process-and-thread-count) provides a rule of thumb to estimate what this value should be based on your average application instance's memory footprint and the total amount of memory you have.

```
max app processes = (total memory x 0.75) / application memory size
```

You can assess your application's memory footprint with [passenger-status](https://www.phusionpassenger.com/library/indepth/accurately_measuring_memory_usage.html#passenger-status):

```
$ passenger-status
Version : 5.0.21
Date    : 2016-12-07 09:54:11 -0500
Instance: REDACTED TO PROTECT THE INNOCENT

----------- General information -----------
Max pool size : 6
App groups    : 2
Processes     : 2
Requests in top-level queue : 0

----------- Application groups -----------
/var/www/prod/app1/current (production):
  App root: /var/www/prod/app1/current
  Requests in queue: 0
  * PID: 15920   Sessions: 0       Processed: 3255    Uptime: 3d 3h 26m 46s
    CPU: 0%      Memory  : 98M     Last used: 6m 4s

/var/www/prod/app2/current (production):
  App root: /var/www/prod/app2/current
  Requests in queue: 0
  * PID: 18842   Sessions: 0       Processed: 31      Uptime: 3d 1h 55m 0s
    CPU: 0%      Memory  : 64M     Last used: 1h 17m
```

For a small Rails project running on a [Linode](https://www.linode.com/) instance shared with another Rails application (and where I'm running a bunch of other things, hence the more conservative coefficient of 0.5), this works out to this estimate:

```
max app processes  = (4096 x 0.5) / (100 * 2) = 10.24
```

You can alter this setting in wherever you're configuring Phusion Passenger (in my case `/etc/apache2/apache2.conf`):

```
PassengerMaxPoolSize 10
```

Make sure you restart Apache (which I'm using here to run Passenger).

If you're using the standalone flavor of Passenger 5, you can commit a `Passengerfile.json` to the root folder of your Rails app:

```json
{
  "max_pool_size": 10
}
```

for 4.x, the file needs to be named `passenger-standalone.json`.

With this simple change, you can serve more concurrent users without any additional hardware upgrades or complex refactoring.
