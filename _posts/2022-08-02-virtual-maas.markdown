---
layout: post
title:  "Setting up Virtual MAAS"
date:   2022-08-02 13:25:12 +0900
categories: jekyll update
---
Configuring Virtual MAAS in one machine makes that you are able to test some kind of openstack env or kubernetes env with juju & charm.

This post mentions that how to set up virtual maas in one machine.

* OS: Ubuntu 20.04 Focal

First, you need to install qemu-kvm, libvirt and related pkgs.

{% highlight shell %}
sudo apt install qemu-kvm qemu-system-x86 libvirt-bin libvirt-daemon libvirt-clients
{% endhighlight %}


TBD