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
sudo apt install qemu-kvm qemu-system-x86 libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
{% endhighlight %}

Check if nested kvm is working.

{% highlight shell %}
\$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
{% endhighlight %}

Add root to libvirt and kvm

{% highlight shell %}
sudo usermod -aG libvirt root
sudo usermod -aG kvm root
{% endhighlight %}

Let's create network for virtual maas, in this case, it is maas-mgmt, maas-private

Stop default network

{% highlight shell %}
sudo virsh net-stop default
sudo virsh net-destroy default
sudo virsh net-undefine default
{% endhighlight %}

Save below file as maas-mgmt.xml

{% highlight xml %}
<network connections='13'>
  <name>maas-mgmt</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:c3:11:aa'/>
  <ip address='10.0.0.1' netmask='255.255.255.0'>
  </ip>
</network>
{% endhighlight %}

Then define it. and start.

{% highlight shell %}
sudo virsh net-define maas-mgmt.xml
sudo virsh net-start maas-mgmt
sudo virsh net-autostart maas-mgmt
{% endhighlight %}

Save below file as maas-private.xml

{% highlight xml %}
<network>
  <name>maas-private</name>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr2' stp='on' delay='0'/>
  <mac address='52:54:01:c3:11:aa'/>
  <ip address='10.1.0.1' netmask='255.255.255.0' />
</network>
{% endhighlight %}



TBD