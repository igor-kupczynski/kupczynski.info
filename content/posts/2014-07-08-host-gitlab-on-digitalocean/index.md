---
title: Host Gitlab on DigitalOcean
tags: []
aliases:
- /2014/07/08/host-gitlab-on-digitalocean.html
---
Building own github in the cloud.

I recently migrated my private repos to digital ocean and gitlab. This
post is a walk through on how to do it and summarizes my experience.


Intro
========

[Digital Ocean][doref] is a VPS provider. They have their data centers
in NY, SF, Amsterdam and Singapore. Their servers are provisioned with
SSD drives and have a very competitive pricing. They are a new kid in
the block (launched in 2013), but they gained a lot of traction over
the internet and there are a lot of success stories posted by
different people. I think this is because they offer a good service, a
simple UI and a competitive pricing.

[Gitlab][gl] is a project witch aims at giving a github experience,
but its open sourced and you can install it on-premise (although they
offered a hosted solutions as well).

[gl]: https://about.gitlab.com/

Without further ado, lets start the tutorial.

Droplet Setup
=============

I went for the 10$/mo droplet, with 1024 mb memory. Gitlub recommends
more, though I found that for a few projects with few users it is
enough. After you create the droplet, you need to do the following.


Point your dnses to gitlab ip
-----------------------------

1. Get the IP

	Droplets -> Droplet -> Settings -> Networking -> IP Address

2. Set up your dns to point from `gitlab.<domain>` to the IP address
   of your droplet.

This is provider dependent, but most probably you need to set up an A -record from `gitlab.youdomain.com` to the droplet ip address using the UI provided by the company you've bought your domain from.


Add a non-root user
-------------------

It is not recommended to log in to your system using a root account. You should add a new user with sudo privileges.

	$ ssh root@gitlab.<yourdomain>

	$ adduser your-user-name

	$ visudo
	# your-user-name ALL=(ALL) NOPASSWD: ALL


SSH
---

Its advised not to run the ssh at port 22. This will make it a bit
harder for a potential.

	$ vim /etc/ssh/sshd_config

	Port 2143
	PermitRootLogin no
	AllowUsers your-user-name git

	$ service ssh restart


SWAP
----

If we went for a droplet with 1024 mb of memory then probably we
should add some swap space.
	
	$ sudo fallocate -l 1024M /mnt/swap.img
	$ sudo mkswap /mnt/swap.img
	$ sudo swapon /mnt/swap.img
	$ sudo vim /etc/fstab


HTTPS
=====

I have a habit of not entering my password to any service over an
insecure connection. Hence, the need to set up an https
connection. You can buy the certificates for your domain or you can
create a self signed pair (e.b. follow one of my previous posts and
[Be Your Own CA][own-ca]). Upload the pair to `/home/your-user-name`.

[own-ca]: https://kupczynski.info/2013/04/21/creating-your-own-certificates.html

Make the certificates available to nginx
---------------------------

	$ sudo mkdir -p /etc/nginx/ssl
	$ sudo cd /etc/nginx/ssl
	$ sudo cp ~/<file>.crt .
	$ sudo cp ~/<file>.key .
	$ sudo chown root:root *


Gitlab
======


Backend conf
------------

Setup basic options for gitlab, like ssh and https details.

	$ sudo vim /etc/gitlab/gitlab.rb

	external_url "https://gitlab.<yourdomain>/"
	gitlab_rails['gitlab_email_from'] = "gitlab@<yourdomain>"
	gitlab_rails['gitlab_support_email'] = "gitlab-support@<yourdomain>"
	gitlab_rails['gitlab_shell_ssh_port'] = "2143"
	nginx['redirect_http_to_https'] = true
	nginx['ssl_certificate'] = "/etc/nginx/ssl/<filename>.crt"
	nginx['ssl_certificate_key'] = "/etc/nginx/ssl/<filename>.key"


	$ sudo gitlab-ctl reconfigure

	$ vim config/gitlab.yml

Admin password
--------------

Go to `http://gitlab.<yourdomain/` and log in using the credentials:

* username: `admin@local.host`
* password: `5iveL!fe`

Set up new admin password and create a user which you'll use later


Import existing repos
---------------------

First copy your existing repos (from your current server) to your `~`. Then

	$ sudo cd /var/opt/gitlab/git-data/repositories/
	$ sudo mv ~i/<repo.git> .
	$ sudo chown -R git:git <repo.git>
    $ sudo chmod u+rw -R <repo.git>
    $ sudo chmod g+rw -R <repo.git>
	$ sudo gitlab-rake gitlab:import:repos

That's all. Enjoy your own github :-)


Sources
=======

There are some online sources documenting the basic droplet and gitlab setup. Please see this:

* [Initial setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)

* [Gitlab setup](https://www.digitalocean.com/community/tutorials/how-to-use-the-gitlab-one-click-install-image-to-manage-git-repositories)

* [Importing existing repos](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/raketasks/import.md)



### Note

I used a referral code in the [Digital Ocean][doref] link.

[doref]: https://www.digitalocean.com/?refcode=649b18bb5e59
