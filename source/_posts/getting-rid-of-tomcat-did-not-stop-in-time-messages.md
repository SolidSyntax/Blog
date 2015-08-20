title: "Getting rid of 'Tomcat did not stop in time.' messages."
tags:
  - Tomcat
id: 104
categories:
  - Lessons learned
date: 2013-10-21 19:01:13
---

The larger your Servlet application becomes the more time it needs to start. The same thing happens with the time it takes to stop an application. 

By default a Tomcat server gives an application 5 seconds to terminate before the "Tomcat did not stop in time. PID file was not removed." messages is logged. 5 seconds isn't a lot of time, especially when the application needs to close a connection pool or perform some cleanup tasks.

<!-- more-->

The fix is rather easy and documented in the catalina command:

{% codeblock lang:bash %}
[user@Server bin]# ./catalina.sh
Using CATALINA_BASE:   /opt/apache-tomcat
Using CATALINA_HOME:   /opt/apache-tomcat
Using CATALINA_TMPDIR: /opt/apache-tomcat/temp
Using JRE_HOME:        /opt/jdk1.7.0_25
Using CLASSPATH:       /opt/apache-tomcat/bin/bootstrap.jar
Usage: catalina.sh ( commands ... )
commands:
  debug             Start Catalina in a debugger
  debug -security   Debug Catalina with a security manager
  jpda start        Start Catalina under JPDA debugger
  run               Start Catalina in the current window
  run -security     Start in the current window with security manager
  start             Start Catalina in a separate window
  start -security   Start in a separate window with security manager
  stop              Stop Catalina, waiting up to 5 seconds for the process to end
  stop n            Stop Catalina, waiting up to n seconds for the process to end
  stop -force       Stop Catalina, wait up to 5 seconds and then use kill -KILL if still running
  stop n -force     Stop Catalina, wait up to n seconds and then use kill -KILL if still running
  version           What version of tomcat are you running?
Note: Waiting for the process to end and use of the -force option require that $CATALINA_PID is defined
{% endcodeblock %}

Adding a timeout parameter to the shutdown command will give your application the time it needs to perform a clean shutdown. I've added the following line in the deamon script (/etc/init.d/tomcat). And the  "Tomcat did not stop in time" messages disappeared:
{% codeblock lang:bash %}
"$CATALINA_HOME"/bin/shutdown.sh 30 
{% endcodeblock %}