---
title: OpenVPN and DNSes on Ubuntu
tags:
- linux
aliases:
- /2019/01/09/openvpn-dns-ubuntu.html
---

One of the VPNs I connect to sets the DNS server for the link. It is an OpenVPN with `DNS` option. The problem is that is doesn't really work out of the box in Ubuntu (at least 18.04 and 18.10) &#x2014; the VPN DNS is not consulted.

At the time of writing, I'm on 18.10, and I write it from that perspective. Ubuntu uses [systemd-resolved](https://www.freedesktop.org/wiki/Software/systemd/resolved/) service to provide a local DNS resolution. We need to "push" the VPN DNS address to this service. We can do it using [jonathanio/update-systemd-resolved](https://github.com/jonathanio/update-systemd-resolved) helper script.

# systemd-resolved for DNS resolution<a id="sec-1"></a>

Consult its readme for details, but the steps are these:

```sh
git clone https://github.com/jonathanio/update-systemd-resolved.git
cd update-systemd-resolved
sudo make  # By default it installs the script to /etc/openvpn/scripts/update-systemd-resolved, so you need root permissions
```

Make sure the systemd-resolved is up and running (and autostarts).

```sh
systemctl enable systemd-resolved.service
systemctl start systemd-resolved.service
```

And that you have `resolve` somewhere in the `hosts` section of `/etc/nssswitch.conf`:

```sh
$ cat /etc/nsswitch.conf  | grep hosts
hosts:          files resolve mdns4_minimal [NOTFOUND=return] dns
```

**Note** This may not be needed as ubuntu moved to use [`/etc/resolv.conf` and DNS stub directly](https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1685045); this may also result in a weird "bug" if your 18.10 is updated from a previous version. See the last section

## Interlude: Why is this helper script needed?<a id="sec-1-1"></a>

OpenVPN (at least on Ubuntu 18.10), comes with its own helper script `/etc/openvpn/update-resolv-conf`, but the problem is that the script relies on `resolvconf` service which is replaced with `systemd-resolved`. This is why we need the helper.

# .ovpn client configuration<a id="sec-2"></a>

Now that we have the helper script setup, we need to make the `*.ovpn` profiles us it. We can either edit the files directly and add the lines below to the `*.ovpn`:

```sh
script-security 2
up /etc/openvpn/scripts/update-systemd-resolved
down /etc/openvpn/scripts/update-systemd-resolved
down-pre
```

Or, if you don't want to edit it, you can pass them to the `openvpn` call:

```sh
# Something like:
sudo openvpn --config ${NAME}.ovpn \
     --script-security 2 \
     --up /etc/openvpn/scripts/update-systemd-resolved \
     --down /etc/openvpn/scripts/update-systemd-resolved \
     --down-pre
```

# Troubleshooting<a id="sec-3"></a>

You've followed all the steps, but the DNS is not being consulted.

## Conflict between resolvconf and systemd-resolved<a id="sec-3-1"></a>

This is common if your Ubuntu 18.10 is migrated from the older version, or if you've installed `resolvconf` package manually.

If the `systemd-resolved.service` log reports something along the lines of

```text
Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying
```

You can check it like that:

```sh
$ systemctl status systemd-resolved.service 
● systemd-resolved.service - Network Name Resolution
   Loaded: loaded (/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2019-01-08 10:05:11 CET; 1 day 4h ago
     Docs: man:systemd-resolved.service(8)
           https://www.freedesktop.org/wiki/Software/systemd/resolved
           https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
           https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
 Main PID: 1087 (systemd-resolve)
   Status: "Processing requests..."
    Tasks: 1 (limit: 4915)
   Memory: 6.5M
   CGroup: /system.slice/systemd-resolved.service
           └─1087 /lib/systemd/systemd-resolved

Jan 09 13:13:27 p51 systemd-resolved[1087]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying 
Jan 09 13:13:55 p51 systemd-resolved[1087]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying 
# (...)
```

It may indicate a conflict between `resolvconf` and `systemd-resolvd`. Make sure that `resolv.conf` points to the systemd version:

```sh
$ ll /etc/resolv.conf 
lrwxrwxrwx 1 root root 32 Jan  9 13:40 /etc/resolv.conf -> /run/systemd/resolve/resolv.conf
```

If there is some other target of the symlink, then:

```sh
cd /etc
sudo rm resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf
```

Will do the trick.

See [this ask ubuntu post](https://askubuntu.com/a/1091556/870446) which put me on the right track.
