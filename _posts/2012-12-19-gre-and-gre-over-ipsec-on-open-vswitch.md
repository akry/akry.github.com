---
layout: post
title: GRE over IPsec on Open vSwitch
---

As for GRE tunnel and GRE over IPsec tunnel on Open vSwitch,
almost no documents describe thoroughly especially GRE over IPsec.
(Surprisingly the official website also does not...)
A document what I have found so far talks about the setting for GRE tunnel in fine-grain,
and that is the only help for me to make it through to the communicating between
2 designated KVM instances via IPsec tunnel. Here is just a personal memorandom as well as a small clue
for the others who totally lost the way to go after strolling around the Internet to find the right settings for GRE over IPsec tunnel on Open vSwitch.

### Open vSwitch Installation

Building ovs packages for installation.

{% highlight bash %}
tar xzf <open-vswitch.tar.gz>
cd <ovs-dir>
dpkg-buildpackage
{% endhighlight %}

Install the following packages

* openvswitch-common
* openvswitch-datapath-source

Before going further instractions, compile and install ovs kernel modules.

{% highlight bash %}
module-assistant auto-install openvswitch-datapath
m-a a-i openvswitch-datapath
{% endhighlight %}

Now you have Open vSwitch kernel module, OVSDB server and Open vSwitch daemon.
After the installation of the following packages, you are ready to configure GRE over IPsec.

* openvswitch-ipsec
* openvswtch-brcompat (optional)

### Setting up tunnel

{% highlight bash %}
ovs-vsctl add-port br0 gre1
ovs-vsctl set interface gre1 type=ipsec_gre \
options:remote_ip=<REMOTE_IP_ADDRESS> \
options:pmtud=false \
options:psk=test \
options:certificate=cert.pem
{% endhighlight %}

### Configure Flow Table

Before configuring flow table directly,
you need to confirm that to which port the interfaces you have set are attached.

{% highlight bash %}
ovs-ofctl show br0
{% endhighlight %}

Suppose tap is attached to port 2 and gre1 is attached to port 3,
The following flows allow KVM instances to communicate to each other.

{% highlight bash %}
ovs-ofctl add-flow br0 'in_port=2, priority=100, actions=output:3'
ovs-ofctl add-flow br0 'in_port=3, priority=100, actions=output:2'
{% endhighlight %}

### Limitations

- ovs-monitor-ipsec works only on Debian/Ubuntu
