---
layout: post
title: ONOS Application Tutorial - Create an app using BUCK
categories: SDN
tags:
- SDN, ONOS, Tutorial
---

ONOS project started using [BUCK](https://buckbuild.com/) from 1.7.0 humming bird release. **Buck** is a build tool developed by Facebook,
and supports highly parallel and incremental builds. To build ONOS, we simply need to run the `buck build onos` command.

![ ](https://buckbuild.com/presentations/droidcon-nyc-2014/images/logo.png)


Ever since ONOS migrated to buck, there has been lot of requests on how to utilize *BUCK* to build ONOS apps.
If you have already written apps on top of ONOS, you might be aware of `onos-create-app` utility, which utilizes [maven archetypes](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html) to generate `pom.xml` and other related files.

For more information on how to write ONOS app based on old maven process, please refer my earlier post [ONOS Application Tutorial - PortStatistics Application](http://kspviswa.github.io/creating-an-app-ONOS.html)

In this post, I'm going to explain how to write an ONOS app based on **BUCK** build system.

## Move your apps inside ONOS source tree

`onos-create-app` utility, lets you place your app's source in any path. This is because, `onos-create-app` follows `mvn` based approach, where all the dependencies will be expressed as mvn targets. These can be either fetched from internet or can be fetched from your `~/.m2` repo.

Eventhough ONOS has migrated to BUCK, `onos-create-app` still uses old mvn style targets. BUCK support isn't added yet.
In-order to create apps based on **buck**, you will have to place your apps under `onos/apps` subtree for the build script to locate your source.

## Sample CLI application

In-order to get our hands dirty with buck, let create a sample CLI application. As explained above, we need to place our app inside `onos/apps` tree. Lets name our app as `failedintentsanalyze` and the path now looks like `onos/apps/failedintentsanalyze`.

Most JAVA projects will be organized as `main` and `test` dirs, where actual logic will be under `main` and UT / IT logic will be placed under `test`. Java files will be placed under `main\java`, property files, XML files etc will be placed under `main\resources` and web\gui related files will be placed under `main\web`.

If you are looking for detailed steps on how to write a CLI app on top of ONOS, I suggest reading through [CLI and Service Tutorial](https://wiki.onosproject.org/display/test/CLI+and+Service+Tutorial) from ONOS wiki. Once done, read-on...

Ok so in-order to have a custom command in ONOS, all we need to do, is to extend `AbstractShellCommand` class and override `execute()` method. So lets just have a sample command implemented like below

{% highlight java %}
package org.onosproject.failedintentsanalyzer;

import org.apache.karaf.shell.commands.Command;
import org.onosproject.cli.AbstractShellCommand;
import org.apache.karaf.shell.commands.Option;
/**
 * Created by kspviswa on 2/17/17.
 */
@Command(scope = "onos", name = "analyze-failed-intents",
         description = "Command to print Analysis on failed intents")
public class FailedIntentsAnalyzerCommand extends AbstractShellCommand{

    @Option(name = "--all", aliases = "-a",
            description = "Include all Events (default behavior)",
            required = false)
    private boolean all = false;

    @Override
    protected void execute() {
    print("Hey there.... I'm working :-) ");
    }
}
{% endhighlight %}

Next important step, is to let **karaf** know that, this command exists. This is done via XML config in a filed named `shell-config.xml` as follows.

{% highlight java %}
<!--
  ~ Copyright 2016-present Open Networking Laboratory
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0">

    <command-bundle xmlns="http://karaf.apache.org/xmlns/shell/v1.1.0">
        <command>
            <action class="org.onosproject.failedintentsanalyzer.FailedIntentsAnalyzerCommand"/>
        </command>
    </command-bundle>
</blueprint>
{% endhighlight %}

**This `shell-config.xml` should be placed under `onos/apps/failedintentsanalyzer/src/main/resources/OSGI-INF/blueprint` path.**

<hr>

## Let cook the app using `buck`

Okay now are raedy to cook the app using `buck`. Just like how `ant` has `Makefile`, `maven` has `pom.xml`, `buck` uses **BUCK** files to understand what needs to be cooked. 

### `BUCK` file

So let us start writing a very simple `BUCK` file as follows.

{% highlight java %}
COMPILE_DEPS = [
    '//lib:CORE_DEPS',
    '//lib:org.apache.karaf.shell.console',
    '//cli:onos-cli',
]

osgi_jar (
    deps = COMPILE_DEPS,
)

onos_app (
    title = 'Failed Intents Analyzer',
    category = 'Utility',
    url = 'http://onosproject.org',
    description = 'App to analyze failed intents',
)
{% endhighlight %}

Best way to read `BUCK` file is to read from bottom. At the bottom, we just create an onos app with a `bucklet` named `onos_app()`. We are naming the app, category, url, description etc. This is analogus to following in `pom.xml`.

{% highlight java %}
    <groupId>org.test</groupId>
    <artifactId>sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>bundle</packaging>

    <description>ONOS OSGi bundle archetype</description>
    <url>http://onosproject.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <onos.version>1.7.0-SNAPSHOT</onos.version>
        <!-- Uncomment to generate ONOS app from this module.-->
        <onos.app.name>org.test.sample</onos.app.name>
        <onos.app.title>Sample App</onos.app.title>
        <onos.app.origin>Sample</onos.app.origin>
        <onos.app.category>Demo</onos.app.category>
        <onos.app.url>http://onosproject.org</onos.app.url>
        <onos.app.readme>ONOS OSGi bundle archetype.</onos.app.readme>
        <!---->
    </properties>
{% endhighlight %}

*Note : For curious people like, you can take look at all available `bucklets` under `~/onos/bucklets/onos.bucklet`.*

ONOS apps are essentially `karaf` features, which are installed / un-installed / activated / deactivated in karaf runtime. All *karaf* features are essentially `osgi-jars`. When you build a `jar` using `javac`, someone has to convert normal `jar` to `osgi-jar`. In case of `mvn` based builds, `maven-bundle-plugin` takes the responsiblity of packaging `OAR` to `osgi-jar`. Incase of `buck jars`, there exists `osgi_jar` bucklet, which mimics the functionality of `maven-bundle-plugin`. 

As an application developer, all you need to do, is to notify **buck** to use `osgi_jar()` bucklet. If you want to include test targets, then feel free to use another similar bucklet `osgi_jar_with_tests`.

## Dependencies

If you look at the args, that we have passed for `osgi_jar` bucklets, `deps = COMPILE_DEPS` instructs `buck` to use all the targets specified in `COMPILE_DEPS` during compile time. Obviously, we will be needing core libs, cli deps ( since our app is essentially extending ONOS' cli ) and karaf console dependencies.

If your app needs to do something with GUI / REST, all you need to do is to just add `'//lib:javax.ws.rs-api', '//utils/rest:onlab-rest',` to list of deps and then you are done. Yeah... it is as simple as that.

<hr>

## Transitive dependencies

One of the important feature of **buck** is its ability to understand JAVA. It also enforce you to specify any **transitive dependency** explicitly. Though this is kind of extra work ( as opposed to automatic dependency injection techniques ), it saves lot of unknowns.

Assume a scenario, where-in we are adding a runtime dependency to our app. For example, let us add `ConnectPointCompleter` bean to our command as follows

{% highlight java %}
<blueprint xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0">

    <command-bundle xmlns="http://karaf.apache.org/xmlns/shell/v1.1.0">
        <command>
            <action class="org.onosproject.failedintentsanalyzer.FailedIntentsAnalyzerCommand"/>
            <completers>
                <ref component-id="connectPointCompleter"/>
            </completers>
        </command>
    </command-bundle>

    <bean id="connectPointCompleter" class="org.onosproject.cli.net.ConnectPointCompleter"/>
</blueprint>
{% endhighlight %}

The `ConnectPointCompleter` object is part of `onos-cli` library and a reference to this object will be made during **runtime**, when our app is activated. Incase of `mvn`, bundleplugin will be able to scan through the `shell-config.xml` file to determine if there are dependencies in there that the bundle needs to import, but with `buck` it is not looking at the XML files. It is now the responsiblity of the developer to explicitly specify run-time dependencies. With ONOS bucklets, this is rather simple one-line change.

The `ConnectPointCompleter` class is available in `org.onosproject.cli.net` package. So just go-ahead and add that as `import_packages` directive in your `BUCK` file as follows.

{% highlight java %}
COMPILE_DEPS = [
    '//lib:CORE_DEPS',
    '//lib:org.apache.karaf.shell.console',
    '//cli:onos-cli',
]

osgi_jar (
    deps = COMPILE_DEPS,
    import_packages = '*,org.onosproject.cli.net',
)

onos_app (
    title = 'Failed Intents Analyzer',
    category = 'Utility',
    url = 'http://onosproject.org',
    description = 'App to analyze failed intents',
)
{% endhighlight %}

*Note : The default `*` simply gives you all of your compile time dependencies, by adding the cli one you are telling the OSGI runtime to load it even though there is no explicit dependency.*

<hr>

## Tying everything together

We are almost there. We have assembled all that are needed to cook our app under `buck`. Last step is to tie our app, with master list of all `buck` targets, so that oru apps gets built, installed automatically when building ONOS. This step is *super-easy*.

All we need to do, is to locate `modules.defs` under ONOS root and add an entry to our app under `ONOS_APPS` section as follows

{% highlight java %}
ONOS_APPS = [
    # Apps
    '//apps/dhcp:onos-apps-dhcp-oar',
    '//apps/dhcprelay:onos-apps-dhcprelay-oar',
    '//apps/fwd:onos-apps-fwd-oar',
    '//apps/acl:onos-apps-acl-oar',
    '//apps/bgprouter:onos-apps-bgprouter-oar',
    '//apps/cip:onos-apps-cip-oar',
    '//apps/drivermatrix:onos-apps-drivermatrix-oar',
    '//apps/events:onos-apps-events-oar',
    '//apps/failedintentsanalyzer:onos-apps-failedintentsanalyzer-oar',
{% endhighlight %}

**Note : Pay attention to `onos-apps-failedintentsanalyzer-oar` naming convention. The pattern is `onos-apps-<appname>-oar`.**
**We need to ensure that our app is located in `onos\apps\<appname>\`**

<hr>

## Lock n' Load

Alright.. we are ready to go. Just hit `tools/build/onos-buck run onos-local -- clean` which would instruct `buck` to build entire ONOS source tree, create a package and run it instantly. Since our app is already inside onos source tree, as a bonus, our app will be automatically installed in ONOS. There is no need for an extra step of `onos-app localhost install app.oar`.

You are free to just **activate** your app using `app activate <package>.<classname>`

If you are wondering where buck outputs are maintained, take a look at `onos/buck-out/bin/apps` path.

<hr>

### Just before saying Cya…
I hope this post would have provided you enough inputs on getting started with your ONOS app development using **BUCK**.
Feel free to fire in any questions / comments / clarifications and I would be happy to help.

Cya!!!

