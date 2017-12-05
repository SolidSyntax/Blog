title: Connect to JMX(MBeans) on Tomcat behind a firewall
tags:
  - Java
  - JMX
  - Monitoring
  - Tomcat
id: 32
categories:
  - Thoughts
date: 2013-10-14 20:40:20
---

I recently had to configure a Tomcat to allow remote access to the JMX service. It involved some configuration to get a connection through the firewall so I've made a configuration-guide for those interested.

<!-- more-->

Using tools like [VisualVM](https://visualvm.java.net/ "VisualVM") or [JConsole](http://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html "JConsole") can tell you a lot about how your application is performing. A typical use case could be monitoring the CPU and memory load or gather information about the running threads. Some more advanced features include reading and manipulating data by calling [MBeans ](http://docs.oracle.com/javase/tutorial/jmx/mbeans/).

The idea is to use the [JMX api ](http://www.oracle.com/technetwork/java/javase/tech/javamanagement-140525.html)to connect to MBeans on a remote [Tomcat](http://tomcat.apache.org/). Connecting to a local Tomcat takes [no effort at all](https://visualvm.java.net/gettingstarted.html). Just start your Tomcat, open VisualVM, and click on the local application you wish to monitor. Connecting to a remote Tomcat requires some additional configuration.

## What's the problem ?

JMX allows you to specify a server port by using the com.sun.management.jmxremote.port system-property. Access to this port allows the JMX-Client to connect to the [RMI](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136424.html)-registry where the client may find the location of the RMI-server. The problem is that the RMI-server is assigned a random port which will probably be blocked if you are behind a firewall !

Oracle [recommends ](http://docs.oracle.com/javase/6/docs/technotes/guides/management/faq.html#rmi1)to create your own RMI connector server programmatically in order to specify a specific port. Luckily the Tomcat team created a [JMX Remote LifeCycle](http://tomcat.apache.org/tomcat-7.0-doc/config/listeners.html#JMX_Remote_Lifecycle_Listener_-_org.apache.catalina.mbeans.JmxRemoteLifecycleListener) Listener to do the job for us.

## Step by step solution:

In the Tomcat binary distribution [download location](http://apache.belnet.be/tomcat/tomcat-7/) you will find a folder named (version)/bin/extras with the file catalina-jmx-remote.jar. Copy this jar to the lib folder of your Tomcat installation.

Edit the server.xml in the conf folder of the Tomcat installation to include the new Listener:
{% codeblock lang:xml %}
<Listener 
    className="org.apache.catalina.mbeans.JmxRemoteLifecycleListener" 
    rmiRegistryPortPlatform="9090" 
    rmiServerPortPlatform="9191" 
/>
{% endcodeblock %}
In this example the registry will be located on port 9090, the rmi-server will be accessible on 9191

Create 2 new files in the same conf directory:[ jmxremote.password and jmxremote.access ](http://docs.oracle.com/javase/7/docs/technotes/guides/management/agent.html)
file jmxremote.password, contains users and passwords
{% codeblock %} 
admin adminpassword{% endcodeblock %}
file jmxremote.access, specifies the authorizations for users
{% codeblock %} 
admin readwrite{% endcodeblock %}
Add the following system-properties to your Tomcat startup-script, or your environment:
{% codeblock %} 
-Dcom.sun.management.jmxremote.password.file=$TOMCAT_ROOT/conf/jmxremote.password
-Dcom.sun.management.jmxremote.access.file=$TOMCAT_ROOT/conf/jmxremote.access 
-Dcom.sun.management.jmxremote.ssl=false{% endcodeblock %}
And finally don't forget to open the two specified ports (in our example 9090 and 9191) on your firewall, in a Linux environment with Iptables you can use the following configuration:
{% codeblock %} 
-A RH-Firewall-1-INPUT -p tcp -m tcp --dport 9090 -m state --state NEW -j ACCEPT
-A RH-Firewall-1-INPUT -p tcp -m tcp --dport 9191 -m state --state NEW -j ACCEPT{% endcodeblock %}
Now you can connect with your favorite JMX client by using the url:
{% codeblock %} 
service:jmx:rmi://[SERVER_IP]:9191/jndi/rmi://[SERVER_IP]:9090/jmxrmi{% endcodeblock %}