---
layout: post
title: ONOS Application Tutorial - DHCP Application - Usage Information
categories: SDN, ONOS, DHCP
tags:
- SDN, ONOS, Tutorial
---

I have been acquainted to ONOS recently and trying to get my hands dirty with all the built-in apps that are shipped with ONOS. One such app is DHCP.

Unfortunately there is no clear documentation of that can be used. Also the source code of DHCP app doesn't contain suffecient comments either. So in this post, I will cover basics of DHCP and some aspects of how to use the DHCP app from ONOS.


### Traditional DHCP usecase

In a traditional networking world, end-hosts are allocated IP addresses dynamically from a DHCP server. Typically the access switches do have a DHCP agent that talks to remote DHCP server and assign the IP address to end-hosts.

In some sophisticated switches, the DHCP server runs internally to the switch or they have a static MAC address <-> IP address in place.

Below image gives a pictorial view of use-case

![ ](http://knowledgebase.cradlepoint.com/servlet/rtaImage?eid=ka0380000004kkk&feoid=00N50000002L6cE&refid=0EM50000000DKsU  "Example of DHCP")

[Source : Google Image search]

As you can see, the access switch / router acts as a DHCP agent / client to negotiate a dynamic IP address from a remote DHCP server.


### DHCP in SDN World

In green-field SDN world, there is no concept of switches & routers. All we have is the dumb forwading elements ( FE / FB ) that either talks in openflow / NETCONF standard protocols. If we have to move the DHCP usecase in SDN world, then it is pretty obvious that DHCP intelligence should be with controller.

This is how the use-case looks if we shift the DHCP to controller
<img src="{{ site.baseurl }}static/img_posts/onos-dhcp.jpg">


### DHCP in ONOS

ONOS ships with a default DHCP application and this can be used as follows.

* #### Activate DHCP app

DHCP app can be activated either via GUI or CLI. In-order to activate via CLI, fire the activate command as shown below.

```
onos> app activate org.onosproject.dhcp
```
Now you can goto Menu->Network->DHCP server to view the statistics of DHCP transactions. Obviously you won't see anything. 

* #### Init DHCP app

ONOS is having a roundabout way of initializing DHCP server using JSON config object. One such sample is already available in `tools/test/configs/office-dhcp.json`. You can make ONOS use this test config as follows.

```
$ onos-netcfg <ipaddress> tools/test/configs/office-dhcp.json
```
You can modify the fields as per your requirements.

Unfortunately there is no northbound REST available to push the configuration. This could be area of improvement in this app.

* #### Using mininet to test

Mininet by default assigns hard-coded `10.0.0.0/8` range of IP addresses to mininet hosts. So when the network comes up, the hosts would be already having IP addresses.

Still, you can dynamically assign the address via DHCP by issuing `dhclient <interface>` under respective hosts.

For Ex:
{% highlight text %}
$ sudo mn -c # clean up
$ sudo mn --controller remote,ip=127.0.0.1 # start default n/w
mininet> h1 dhclient h1-eth0 # fire a DHCP request
mininet> h2 dhclient h2-eth0 # fire a DHCP request
{% endhighlight %}

You can check for new IPs that got assigned by using `ifconfig` under respective hosts

```
mininet> h1 ifconfig -a
mininet> h2 ifconfig -a
```

* #### Statistics in ONOS DHCP app
	* ##### Via GUI
	It is also possible to check the working log of DHCP server (i.e List of IP addresses assigned, IP address to MAC address mapping etc ) in ONOS DHCP UI segment as follows.
	<img src="{{ site.baseurl }}static/img_posts/dhcp-app-gui-stats.jpg">
	
	* ##### Via ONOS-CLI
	DHCP app also extends ONOS-CLI shell and exposes lot of util commands

{% highlight text %}
onos> dhcp-<TAB>
dhcp-lease                    dhcp-list                     dhcp-remove-static-mapping    dhcp-set-static-mapping       
onos> dhcp-list 
MAC ID: 26:6B:8D:1B:B7:07/None -> IP ASSIGNED 10.1.11.56
onos> dhcp-lease
Lease Time: 300s
Renewal Time: 150s
Rebinding Time: 200s
onos> 
{% endhighlight %}

* #### Mininet Python script to test

While using `dhclient` after network is built is a rudimentry way, it is also possible to achieve the same via python mininet script as well. Below is one such approach

```
#!/usr/bin/python

from mininet.net import Mininet
from mininet.node import Controller, OVSKernelSwitch, RemoteController
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.clean import Cleanup

def emptyNet():

    net = Mininet(controller=RemoteController, switch=OVSKernelSwitch)

    c1 = net.addController('c1', controller=RemoteController, ip="127.0.0.1", port=6633)

    h1 = net.addHost( 'h1', ip='10.0.1.1' )
    h2 = net.addHost( 'h2', ip='10.0.1.2' )
    h3 = net.addHost( 'h3', ip='10.0.2.1' )
    h4 = net.addHost( 'h4', ip='10.0.2.2' )


    s1 = net.addSwitch( 's1' )
    s2 = net.addSwitch( 's2' )

    s1.linkTo( h1 )
    s1.linkTo( h2 )
    s2.linkTo( h3 )
    s2.linkTo( h4 )
    s1.linkTo( s2 )

    net.build()
    c1.start()
    s1.start([c1])
    s2.start([c1])

# dhclient to get dynamic IP
    intf = net.get('h1').defaultIntf()
    intf.cmd('dhclient h1-eth0')
    intf.updateIP()
  
    CLI( net )
    s1.stop()
    s2.stop()
    net.stop()
    Cleanup()

if __name__ == '__main__':
    setLogLevel( 'info' )
    Cleanup()
    emptyNet()

```

### Closing notes

I have started to get know a lot about ONOS. ONOS wiki is only in naive stages and there is no much information for a starter like to get going with ONOS apps. Hence I set out to write series of *how to use ONOS* like post to help fellow beginners like me. Watch out for more ONOS KB articles.

Cya!!!!
