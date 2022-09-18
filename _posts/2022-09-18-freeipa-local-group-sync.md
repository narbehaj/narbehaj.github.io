---
layout: post
title: "Sync FreeIPA groups with local groups"
author: "Narbeh"
date: 2022-09-17 19:57:00 +0300
categories: freeipa openldap linux
---

While using the FreeIPA as a centralized authentication server, you might want to assign users a group that is not in FreeIPA and it's locally created, for example, `www-data`.

There is an option called [Group Merging](https://sourceware.org/glibc/wiki/Proposals/GroupMerging) which is in `glibc` library. With this option, you can simply force `nsswitch.conf` file to read groups 

What we need to do first, is to replace the `group` value in `nsswitch.conf` to:

```
group: files [SUCCESS=merge] sss
```

Second, we need to get the group id (GID) of the local group from `/etc/group` file:

```
$ grep www-data /etc/group
www-data:x:33:
```

Then we should create a group in FreeIPA with the same GID:

![](assets/img/freeipa-groups.png)

Finally, add the desired users to the group in FreeIPA. Immediately you should the users that now belong to the group:

```
$ getent group www-data
www-data:x:33:john,remote-user,narbeh
```

