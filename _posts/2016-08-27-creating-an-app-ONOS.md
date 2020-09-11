---
layout: post
title: ONOS Application Tutorial - PortStatistics Application
categories: SDN
tags:
- SDN, ONOS, Tutorial
---

ONOS - Yet another SDN controller. This was the thought I had when I first heard about this opensource program. A lot has changed about my perception of ONOS, after getting a feel of what it is actually.

In this post, I'm going to concentrate on how to write an ONOS app.

### Basics

ONOS applications are having special extension - OAR ( ONOS Application aRchive format ). The main reason for this special kind of treatment goes to one of the special reason behind using ONOS - Distributed systems .

ONOS at its heart - a distributed system. ONOS unline ODL focus more on service provider use-case. Hence HA, scalablity & resiliency has been implicit requirements for ONOS.

In distributed system, multiple ONOS nodes cater to serve a single ONOS cluster. Deploying app in .OAR format ensures that, these apps gets installed in all ONOS nodes thereby achieving consistency across the ndoes.

Below diagram gives module level view of ONOS
![ ](http://sdnhub.org/wp-content/uploads/2015/01/onos-architecture.png  "ONOS arch")

### Getting Started

The [Template Application Tutorial](https://wiki.onosproject.org/display/ONOS/Template+Application+Tutorial) contains all the required content to get started. But it doesn't offer a full working template application. In this post, let me guide you to create a working application.

#### Create a sample app from template

When you setup a ONOS dev environment as per [this link](https://wiki.onosproject.org/display/ONOS/Development+Environment+Setup), things will become soooo easy.

When a new developer set-out to write a SDN app in ODL, the learning curve is way too steep. Even though ONOS also follows [maven archetypes](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html) , ONOS developers had made things simpler by providing a wrapper called **onos-create-app**.

I will let you glean-through the [Template Application Tutorial](https://wiki.onosproject.org/display/ONOS/Template+Application+Tutorial) to setup the skeleton app. Once you are done, continue reading through.


#### Scope of our application

Let us now create a simple application to get the **Port Statistics** of all the ports from all the devices ( possibly openflow devices ) that are attached to the controller.

So what exactly we mean about port statistics?? Well, it is basically the amount of packet, bytes that are sent to / recieved from a given port from a given device.

ONOS also maintain another special version of statistics known as **Port Delta Statistics** which actually provides the **delta** between 2 given probe intervals.

#### How to get our data from controller ?

ONOS exposes different services to poke into and get data. One such service is **DeviceService**. This particular service abstracts the south bound device irrespective of the south bound interface that is connected with.

So let us use this service to get the list of devices first. But before that, we need to get a reference of the interface class.

{% highlight java %}

@Reference(cardinality = ReferenceCardinality.MANDATORY_UNARY)
 protected DeviceService deviceService;
 
  Iterable<Device> devices = deviceService.getDevices();
  List<PortStatistics> portStatisticsList = deviceService.getPortDeltaStatistics(d.id());
                for (PortDeltaStatistics portdeltastat : portStatisticsList) {
                	 if(portdeltastat != null)
                    		log.info("portdeltastat bytes recieved" + portdeltastat.bytesReceived());
                	else
                    		log.info("Unable to read portDeltaStats");
                }
{% endhighlight %}

While this rudimentry, why not let us monitor every port of every device in autonomous way? Sounds good right??

All we need now is a worker for us to work in independant fashion. Well the solution is `TimerTask`.
So let us create a new class `portStatsReaderTask` extending `TimerTask` and fire an independant task operation as follows.

But before that, let us look how our `portStatsReaderTask` might look.

{% highlight java %}

public class portStatsReaderTask {

    class Task extends TimerTask {

        public Device getDevice() {
            return device;
        }
        public DeviceService getDeviceService() {
            return deviceService;
        }
        public long getDelay() {
            return delay;
        }

        @Override
        public void run() {
            while (!isExit()) {
                log.info("####### Into run() ");
                List<PortStatistics> portStatisticsList = getDeviceService().getPortDeltaStatistics(getDevice().id());
                for (PortStatistics portStats : portStatisticsList) {
                    log.info("########## port is " + portStats.port());
                    if (portStats.port() == getPort()) {
                        log.info("Port " + port + "has recieved " + portStats.bytesReceived() + "bytes");
                        try {
                            Thread.sleep((getDelay() * 1000));
                            break;
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }

    public void schedule() {
        this.getTimer().schedule(new Task(), 0, 1000);
    }
--snip--
{% endhighlight %}
 
 Hope you get the implementation. `portStatsReaderTask` contains an *inner class* named `Task` that extends `TimerTask`. This is necessary if we want to use `portStatsReaderTask` as a self contained component ( you will get more clarity in the place where we really use this class ).
 
 The `run()` & `schedule()` are really the heart & nerve of this whole logic. For the sake of this blog, I didn't make the full source available. I shall provide a link to github source at the end of this post.
 
 Having defined our worker, let us give some work to the worker.
 
#####  `activate()` & `deactivate` function :

The `AppComponent` class provides `activate()` & `deactivate()` functions which marks the start & end of every ONOS app's lifecycle. So we are going to place our logic in `activate()` function.

{% highlight java %}

@Activate
    protected void activate() {

        log.info("Started");
        Iterable<Device> devices = deviceService.getDevices();
        
        List<PortStatistics> portStatisticsList = deviceService.getPortDeltaStatistics(d.id());
                for (PortStatistics portStats : portStatisticsList) {
                    try {
                        int port = portStats.port();
                        log.info("#### Creating object for " + port);
                        portStatsReaderTask task = new portStatsReaderTask();
                        task.setDelay(3);
                        task.setExit(false);
                        task.setLog(log);
                        task.setPort(port);
                        task.setDeviceService(deviceService);
                        task.setDevice(d);
                        map.put(port, task);
                        task.schedule();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
      }
{% endhighlight %}
 
 First every device, we get `List<PortStatistics>` objects. And for every element in list, we prepare a map of `PortStatistics` with `port number` and spawn a new `portStatsReaderTask` for every element in list. Now you can understand the reason for create a wrapper as explained above.
 
<pre>
Note : As per ONOS's openflow driver (provider/of/device/impl/OpenFlowDeviceProvider.java), the polling frequency is set to 5 seconds.

Hence you might need to understand that the B/W measurement depends on this 5 seconds polling intervals. If incase you are trying to match the B/W data with external observer data ( like iperf for example ), you might need to fix the probing interval of that observer as well to 5 seconds.
</pre>

#### Github source

Please feel free to visit [myapps](https://github.com/kspviswa/myapps/tree/master/sample) page to view the complete source. Make sure you read the [Read.me](https://github.com/kspviswa/myapps/blob/master/README.md) before attempting to use the code .

#### Just before saying Cya...

I hope this post would have provided you enough inputs on getting started with your ONOS app development. The idea is to provide a complete walkthrough of a simple app and I hope I did justice to that intent. Feel free to fire in any questions / comments / clarifications and I would be happy to help.

Cya!!!
