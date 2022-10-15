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
$ kvm-ok
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

Preparing storage for vms

{% highlight shell %}
// cd anywhere where you want
$ truncate --size 30G juju-controller
{% endhighlight %}

Preparing xml file for juju controller
// some parameters should be adjusted to your env.

// juju controller needs at least 3584.0MB for memory
{% highlight xml %}
<domain type='kvm' id='76'>
  <name>juju-bootstrap</name>
  <memory unit='KiB'>4194304</memory>
  <currentMemory unit='KiB'>4194304</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <resource>
    <partition>/machine</partition>
  </resource>
  <os>
    <type arch='x86_64' machine='pc-i440fx-wily'>hvm</type>
    <boot dev='network'/>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/bin/kvm-spice</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='raw' cache='none'/>
      <source file='/home/xtrusia/juju-bootstrap'/>
      <backingStore/>
      <target dev='vda' bus='virtio'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </disk>
    <controller type='usb' index='0' model='ich9-ehci1'>
      <alias name='usb'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x7'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'>
      <alias name='pci.0'/>
    </controller>
    <interface type='network'>
      <mac address='52:54:00:97:24:01'/>
      <source network='maas-mgmt' bridge='virbr1'/>
      <target dev='vnet12'/>
      <model type='virtio'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0' multifunction='on'/>
    </interface>
    <interface type='network'>
      <mac address='52:54:01:97:24:01'/>
      <source network='maas-private' bridge='virbr2'/>
      <target dev='vnet13'/>
      <model type='virtio'/>
      <alias name='net1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x1'/>
    </interface>
    <serial type='pty'>
      <source path='/dev/pts/14'/>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
      <alias name='serial0'/>
    </serial>
    <console type='pty' tty='/dev/pts/14'>
      <source path='/dev/pts/14'/>
      <target type='serial' port='0'/>
      <alias name='serial0'/>
    </console>
    <input type='mouse' bus='ps2'>
      <alias name='input0'/>
          </input>
    <input type='keyboard' bus='ps2'>
      <alias name='input1'/>
    </input>
    <graphics type='vnc' port='5906' autoport='yes' listen='0.0.0.0'>
      <listen type='address' address='0.0.0.0'/>
    </graphics>
    <video>
      <model type='cirrus' vram='16384' heads='1' primary='yes'/>
      <alias name='video0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <memballoon model='virtio'>
      <alias name='balloon0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
    </memballoon>
  </devices>
</domain>
{% endhighlight %}

Install MAAS

![Install MAAS](https://maas.io/docs/how-to-install-maas)


Waiting on MAAS image synchronization.

Then you need to add this host as Chassis on MAAS.

![Addded machine for bootstrapping](/assets/images/capture1.png)

You also need to set DHCP for network subnet. Otherwise, you can't let machine boot by PXE

Then this machine needs Commission first.

![Commissioned machine](/assets/images/capture2.png)

Install Juju

![Install Juju](https://juju.is/docs/olm/installing-juju)
![How to use MAAS with Juju](https://juju.is/docs/olm/maas)

When you add credential, there is a part for maas-oauth. then you need to copy below key and paste it to there.

![Commissioned machine](/assets/images/capture2.png)

juju bootstrapping

{% highlight shell %}
xtrusia@xtrusia:~$ juju bootstrap xtrusia
Creating Juju controller "xtrusia-default" on xtrusia/default
Looking for packaged Juju agent version 2.9.35 for amd64
Located Juju agent version 2.9.35-ubuntu-amd64 at https://streams.canonical.com/juju/tools/agent/2.9.35/juju-2.9.35-linux-amd64.tgz
Launching controller instance(s) on xtrusia/default...
 - qgamxn (arch=amd64 mem=4G cores=2)  
Installing Juju agent on bootstrap instance
Fetching Juju Dashboard 0.8.1
Waiting for address
Attempting to connect to 10.0.0.254:22
Connected to 10.0.0.254
Running machine configuration script...
Bootstrap agent now started
Contacting Juju controller at 10.0.0.254 to verify accessibility...

Bootstrap complete, controller "xtrusia-default" is now available
Controller machines are in the "controller" model
Initial model "default" added

xtrusia@xtrusia:~$ juju status
Model    Controller       Cloud/Region     Version  SLA          Timestamp
default  xtrusia-default  xtrusia/default  2.9.35   unsupported  03:49:55Z

Model "admin/default" is empty.

{% endhighlight %}