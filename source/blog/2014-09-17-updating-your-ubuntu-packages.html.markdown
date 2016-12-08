---
title: "Updating your Ubuntu packages"
date: 2014-09-17 15:04:37 -0400
tags: Ubuntu, computer janitor
---

Update source list:

```
sudo apt-get update
```

Upgrade all installed packages:

```
sudo apt-get upgrade
```

Remove unused .deb files for packages that are no longer installed:

```
sudo apt-get autoclean
```

[Source](https://help.ubuntu.com/community/AptGet/Howto)
