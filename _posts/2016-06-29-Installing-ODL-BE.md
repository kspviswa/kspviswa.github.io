---
layout: post
title: ODL Beryllium (Be) - Installation + Flow configuration
categories: SDN
tags:
- SDN
---

Beryllium (Be) is the fourth release of OpenDaylight (ODL), the leading open source platform for programmable, software-defined networks. Multivendor, traditional and greenfield, ODL is the industry’s de facto SDN platform, supporting a broad set of use cases and providing the foundation for networks of the future. 

Service providers and enterprises are using ODL to solve key network challenges related to Automated Service Delivery; Network Resource Optimization; Cloud and NFV; Research, Education and Goverment; and Visibility and Control. ODL Be strengthens its architecture based on the Model Driven Service Abstraction Layer (MD-SAL) to deliver higher scale and the ability to easily incorporate new applications and protocols.

Ever since ODL embraced the **MD-SAL** ( *Module Drive Service Activation Layer* ) approach, certain static configurations like creation of static flow, updation / deletion of flow from ODL UI etc have been replaced with much abstract approach.

Eventhough ODL's documentation is pretty vast and comprehensive, certain nitty gritty details on changed features aren't well documented yet. For a novice like me, watching **Youtube** videos of *yesteryear* ODL installations & App development doesn't make sense anymore.

This blog focuses on installation ODL Be and getting it up & running, along with steps to configure flows via RESTCONF interface. Obviously, the information contained in this blog is not something new and you can very well find more deeper info in ODL's pages. However this blog is an attempt to extract all the relevant info ( esp for Be release ), without your time wasted on un-necessary web links ( as the research is already completed  for you). These steps works perfectly in **Ubuntu** machine and it is safely assumed to be working in *JAVA* compatible machines too. 


`Oh Cut the crap. Take me to the steps directly!!`


## Installation

ODL-Be uses something known as `Karaf` ( better known as *Apache Karaf* ). *Karaf* can be equated something like a light weight java container, with all the dependencies well expressed. Assume something like both the execution context ( i.e `glibc` or `libstdc++`) and package management (i.e like `apt-get`) bundled together. In plain words, if you ever release a module / plugin to ODL, all you need is to express your dependencies and `karaf` will fetch the depenencies and run it `securely` for you in isolated fashion. i.e your plugin would still be logically running with ODL, however it won't affect ODL in anyother way.

* Download the *binary* version of ODL Be from [here](https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.4.2-Beryllium-SR2/distribution-karaf-0.4.2-Beryllium-SR2.tar.gz )


* Untar / Unzip and explode the contents `gunzip -dc <filename> | tar xvf -`

* Navigate to `<path to odl>~/distribution-karaf-0.4.2-Beryllium-SR2/bin$` and just start `./karaf`

* You would see a prompt like below . [ **Note : ODL will eat up RAM. So give it a minute to settle** ]

```
~/distribution-karaf-0.4.2-Beryllium-SR2/bin$ ./karaf
karaf: JAVA_HOME not set; results may vary

    ________                       ________                .__  .__       .__     __
    \_____  \ ______   ____   ____ \______ \ _____  ___.__.|  | |__| ____ |  |___/  |_
     /   |   \\____ \_/ __ \ /    \ |    |  \\__  \<   |  ||  | |  |/ ___\|  |  \   __\
    /    |    \  |_> >  ___/|   |  \|    `   \/ __ \\___  ||  |_|  / /_/  >   Y  \  |
    \_______  /   __/ \___  >___|  /_______  (____  / ____||____/__\___  /|___|  /__|
            \/|__|        \/     \/        \/     \/\/            /_____/      \/


Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown OpenDaylight.

opendaylight-user@root>

```

### Installing `bare-minimum` features for `openflow` experiments

ODL is a full blown controller that is capable of running production grade business application. Hence when you first install ODL, it will be a bare controller. Features can be added on demand. For openflow purposes, you will have to enable below features via `feature:install` command. Don't worry about the command nomenclature. `help` is good here.

`opendaylight-user@root>feature:install odl-restconf odl-mdsal-apidocs odl-l2switch-switch odl-dlux-all odl-openflowplugin-all`

- **dlux** is a web application providing YANG modelling, REST conf interface & Topology services.

- **l2switch** is well known mac learning switch

- **openflowplugin** is a OF plugin that takes care of interacting with south bound OF compatible switches.

- **restconf** is a restconf plugin, allowing ODL to export both conf / oper data of its own datastore, OVS data store and any other customer datastore to NETCONF modelling format (YANG ) that can be accessed via a *REST* interface.

---

## Launching web admin

You can now point to `http://<ip>:8181/index.html` with creds `admin\admin`to view the **dlux** interface.

Here you can view the `topology` or goto `YANG UI` to view the datastore that has been exposed by the controller.

---

## OpenFlow exercise

### Mininet & its tweaking

* Install `mininet` if not availble [ `sudo apt-get install mininet`]
* Disable default `reference openflow controller` if you are running ODL in same machine as that of `mininet`

``` 
sudo service openvswitch-controller stop
sudo update-rc.d openvswitch-controller disable
```

* Start `mininet` with appropriate Openflow protocol.

>>>>`~$ sudo mn --mac --topo single,3 --controller remote,ip=10.74.16.37 --switch ovsk,protocols=OpenFlow13`

* Open a separate term and enable `OF13` inside the switch manually. This is very important.

>>>> `~$ sudo ovs-vsctl set bridge s1 protocols=OpenFlow13`

* Once you enable `OpenFlow13`, your normal `ovs-ofctl ovs-vsctl` won't work. You would need a special output flag.

>>>> `~$ sudo ovs-ofctl -O OpenFlow13 dump-flows s1`

>>>> `~$ sudo ovs-ofctl -O OpenFlow13 del-flows s1`

**Note : ODL Openflow plugin is a OF13 plugin. All the packets will be serialized to OpenFlow1.3 format. Hence it is very important to run your switches in compatible mode. Else the communication between `odl-openflowplugin` and your OVS will `never` occur. If you monitor the `karaf` logs located in `~/distribution-karaf-0.4.2-Beryllium-SR2/data/log` , you will notice below error.**

```
2.Beryllium-SR2 | Node [Uri [_value=openflow:1]] does not support statistics request type : Group Features
2016-06-29 11:37:41,184 | WARN  | entLoopGroup-5-2 | OFEncoder                        | 247 - org.opendaylight.openflowjava.openflow-protocol-impl - 0.7.2.Beryllium-SR2 | Message serialization failed
java.lang.IllegalStateException: Serializer for key: msgVersion: 1 objectType: org.opendaylight.yang.gen.v1.urn.opendaylight.openflow.common.action.rev150203.actions.grouping.Action action type: org.opendaylight.yang.gen.v1.urn.opendaylight.openflow.common.action.rev150203.action.grouping.action.choice.DecMplsTtlCase experimenterID: null was not found - please verify that you are using correct message combination (e.g. OF v1.0 message to OF v1.0 device)

```
**One of the main consequence of this mis-configuration is that, eventhough when you provision a flow via RESTCONF ( with response of 200 OK), the provisioned flows will not reach the Openflow switches.**

---

### Flow provisioning

Flows can provisioned as follows

- By developing a ODL native plugin ( either in Python / JAVA ) and making it available in MD-SAL architecture.
- By developing a external app that just interacts with ODL's **REST** API. This includes even *RESTCONF* framework.


#### Config / Operation trees

Bydefault `dlux-ui` provides something known as **YANG UI** in the web interface. This page actually expose the controller's datastore in `operation` & `config` trees.

- **Config** - Data set / get in this tree will occur in configuration DB that is available to both MD-SAL & RESTCONF modules. Meaning, if you set / get any data, this request will eventually hit the corresponding MD-SAL broker module ( for eg openflow module ) and work accordingly ( for eg pushing a flow to OVS )

- **Operation** - Data get in this tree will be retrieved directly from the devices. For eg current Flow statistics, current states of interfaces etc cannot be configured. They reflect the real time data. Flows that are configured manually in the switch with `ovs-ofctl` etc can also be retrieved via same operational tree.

 
#### RESTCONF usage

RESTCONF is a beautiful way to manage a network element ( both mgmt & oper ) via single standard way. The flow management of any OpenFlow enabled devices can be management via OpenFlow. Basic configurations of the device can be done via NETCONF / OF-CONFIG. ODL has streamlined both of this into single RESTCONF framework, thus making the job of northbound applications way easier.


##### RESTCONF GET

|**URL**|`http://<IP>:8181/restconf/operational/opendaylight-inventory:nodes/node/openflow:1/table/0/flow/1`|
|-------|---------------------------------------------------------------------------------------------------|
|**HEADERS**| None|

Here the `openflow:1` is referring to first switch (often `s1` in mininet env ). This URI is trying to fetch the information of `flow id:1` under `table:0`, whose output will be something like this.

```
{
  "flow-node-inventory:flow": [
    {
      "id": "1",
      "flags": "",
      "priority": 2,
      "opendaylight-flow-statistics:flow-statistics": {
        "duration": {
          "second": 2013,
          "nanosecond": 768000000
        },
        "packet-count": 0,
        "byte-count": 0
      },
      "table_id": 0,
      "hard-timeout": 0,
      "instructions": {
        "instruction": [
          {
            "order": 0,
            "apply-actions": {
              "action": [
                {
                  "order": 0,
                  "dec-nw-ttl": {}
                }
              ]
            }
          }
        ]
      },
      "idle-timeout": 0,
      "match": {
        "ipv4-destination": "10.0.10.0/24",
        "ethernet-match": {
          "ethernet-type": {
            "type": 2048
          }
        }
      },
      "cookie": 0
    }
  ]
}
``` 

##### RESTCONF PUT ( Creating a flow )

|**URL**|`http://<IP>:8181/restconf/config/opendaylight-inventory:nodes/node/openflow:1/table/0/flow/5`|
|-------|---------------------------------------------------------------------------------------------------|
|**HEADERS**| Accept = application/xml  Content-Type = application/xml|


Here the `openflow:1` is referring to first switch (often `s1` in mininet env ). This URI is trying to create a flow with `flow id : 5`, resulting in a flowtable entry in switch as follows.

```
~$ sudo ovs-ofctl -O OpenFlow13 dump-flows s1
OFPST_FLOW reply (OF1.3) (xid=0x2):
 cookie=0x0, duration=142.415s, table=0, n_packets=0, n_bytes=0, priority=3,ip,nw_dst=10.0.0.0/24 actions=dec_ttl
 cookie=0xa, duration=2.601s, table=0, n_packets=0, n_bytes=0, priority=5,ip,nw_dst=40.0.0.0/24 actions=dec_ttl
 cookie=0x0, duration=103.187s, table=0, n_packets=0, n_bytes=0, priority=4,ip,nw_dst=40.0.0.0/24 actions=dec_ttl
 cookie=0x0, duration=284.485s, table=0, n_packets=0, n_bytes=0, priority=21,ip,nw_dst=10.0.10.0/24 actions=dec_ttl
 cookie=0x0, duration=233.044s, table=0, n_packets=0, n_bytes=0, priority=1,ip,nw_dst=10.0.10.0/24 actions=dec_ttl

```


This is just a start. RESTCONF APIs can be readily used in most of the applications. Technologies such as Python, ruby & JAVA provide readily available APIs to consume and parse the RESTCONF APIs.

Hope this guide helped a novice ( like me ) who just entered into ODL eco-system, struggling to complete a `hello-world` app . Feel free to contact me for any doubts / corrections in the commands.

Happy coding & exploring :)

