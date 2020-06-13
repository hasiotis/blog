---
title: "My Virtualisation HomeLab"
date: 2020-06-11
draft: true
summary: "Spawning VMs fast and easy"
keepImageRatio: true
summaryImage: "summary.jpg"
tags: ["tech", "virtualisation"]
---

Here how I setup a virtualised environment at home to be able to easily spawn VMs and try out new tools, or perfect my ansible roles. First thing
first we need some hardware. I bought some years ago a [shuttle xpc nano](http://global.shuttle.com/products/productsDetail?productId=2073). I
made sure to have as much RAM as possible and also attached my old ssd disk.

# The hypervisor

I used to run a free ESXi on it, but unfortunately there is not much you can do with the API without vsphere. So at some point I decided to switch
to [proxmox](https://www.proxmox.com/en/). The first thing was to go through the [installation](https://pve.proxmox.com/wiki/Installation) of proxmox.
The installation is straightforward and completed without any issues.

# The DNS setup

For the DNS server I used [PowerDNS](https://www.powerdns.com/) mostly due to the availability of a [rest API](https://doc.powerdns.com/authoritative/http-api/index.html).
Since I use Debian 10 and already had pdns-recursor in place I had to install the server:

{{< code language="shell" line-numbers="false">}}
$ apt-get install pdns-server pdns-backend-sqlite3
{{< /code >}}

Below is part of the */etc/powerdns/pdns.conf* where we make sure PowerDNS it available on port 5300. This way we still use pdns-recursor for our DNS requests, but only for
the *pve.hasiotis.dev* we send requests to pdns.

{{< code language="shell" line-numbers="false">}}
# /etc/powerdns/pdns.conf
local-address=127.0.0.1
local-port=5300

master=yes
{{< /code >}}

and the */etc/powerdns/recursor.conf* part:

{{< code language="shell" line-numbers="false">}}
# /etc/powerdns/recursor.conf
forward-zones=pve.hasiotis.dev=127.0.0.1:5300
{{< /code >}}

Also we make sure to enable the api server and make it accessible from our lan:

{{< code language="shell" line-numbers="false">}}
# /etc/powerdns/pdns.conf
api=yes
api-key=SUPERLONGSECRET

webserver=yes
webserver-address=0.0.0.0
webserver-allow-from=192.168.0.0/24
webserver-port=8081
{{< /code >}}

And create a zone specific to my virtualisation lab

{{< codeWide language="shell" line-numbers="false">}}
$ sqlite3 /var/lib/powerdns/pdns.sqlite3 < /usr/share/pdns-backend-sqlite3/schema/schema.sqlite3.sql
$ pdnsutil create-zone pve.hasiotis.dev
$ pdnsutil list-zone pve.hasiotis.dev
{{< /codeWide >}}

# The IPAM setup

One thing that you might miss when not operating in the cloud, is an easy way to assign IP addresses to your VMs. For this an IP management solution is quite handy. So here
is [phpIPAM](https://phpipam.net/) to the rescue. After we [install phpIPAM](https://phpipam.net/documents/installation/) we need to do at least two things. First setup our
subnet, and second declare an api user:

{{< figure src="phpipam-api.png" width="784" >}}

# Packer to create OS templates

Thankfully there is a [proxmox builder](https://www.packer.io/docs/builders/proxmox/) available for [packer](https://www.packer.io/).

# Setup ZBuilder

{{< code language="shell" line-numbers="false">}}
$ pip3 install --user zbuilder
{{< /code >}}
