---
title: Change the maximum number of open files in ubuntu 18.10
tags:
- linux
aliases:
- /2018/11/24/ubuntu-18-10-ulimits.html
---

The procedure changed on recent ubuntu compared to what I used to do. I used
[change
/etc/security/limits.conf](https://underyx.me/2015/05/18/raising-the-maximum-number-of-file-descriptors).

The procedure on ubuntu 18.10 that worked for me is this.

1. Go to `/etc/systemd/system.conf`
2. Uncomment `DefaultLimitNOFILE` and set your limit there, e.g.
   
   ```sh
   $ grep NOFILE /etc/systemd/system.conf
   DefaultLimitNOFILE=65535
   ```
3. Restart
4. Profit

Before:

```sh
$ ulimit -n
1024
$ ulimit -Sn
1024
$ ulimit -Hn
1048576
```

After:

```sh
$ ulimit -n
65535
$ ulimit -Sn
65535
$ ulimit -Hn
65535
```

The limits can be controlled by systemd and this is what we do here --- instruct
systemd to set it to 65k. Note that this setting will apply to all users, which
is OK for a laptop or a workstation. On a server you may need a finer grained
settings and you should look for a different method.
