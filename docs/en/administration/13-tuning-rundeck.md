% Tuning Rundeck

### File descriptors

The Rundeck server process opens a number of files during normal operation. These
include system and java libraries, logs, and sockets. 
Your system restricts the number of open file handles per process
but these limitations can be adjusted.

If your installation attempts to exceed the limit, you will see an error
like the one shown below in your [service.log](logs.html) file.

    Too many open files 


_On Linux nodes_

List the current limit with the [ulimit](http://ss64.com/bash/ulimit.html) command:

    ulimit -n

If the limit is low (eg ``1024``) it should be raised.

You can get the current number of open file descriptors used by the 
Rundeck server process with [lsof](http://linux.die.net/man/8/lsof):

    losf -p <rundeck pid> | wc -l

Increase the limit for a wide margin. 
Edit [/etc/security/limits.conf](http://ss64.com/bash/limits.conf.html) file
to raise the hard and soft limits. Here they are raised to ``65535`` for 
the "rundeck" system account:

    rundeck hard nofile 65535
    rundeck soft nofile 65535


The system file descriptor limit is set in /proc/sys/fs/file-max. 
The following command will increase the limit to 65535:

    echo 65535 > /proc/sys/fs/file-max

In a new shell, run the ulimit command to set the new level:

    ulimit -n 65535

The ulimit setting can be set in the [rundeckd](#rundeckd) 
startup script, or [profile](configuration.html#profile).

Restart Rundeck.

### Java heap size

The ``rundeckd`` startup script sets initial and maximum heap sizes
for the server process. For many installations it will be sufficient.

If the Rundeck JVM runs out of memory, the following error occurs:

    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space

Heap size is governed by the following startup parameters: 
``-Xms<initial heap size>`` and ``-Xmx<maximum heap size>``


You can increase these by updating the Rundeck [profile](configuration.html#profile). 
To see the current values, grep the ``profile`` for 
the Xmx and Xms patterns:

* Launcher installs:

        egrep '(Xmx|Xms)' $RDECK_BASE/etc/profile
   
* RPM installs:

        egrep '(Xmx|Xms)' /etc/rundeck/profile
   
The default settings initialized by the installer 
sets these to 1024 megabytes maximum
and 256 megabytes initial:

    export RDECK_JVM="$RDECK_JVM -Xmx1024m -Xms256m"

_Sizing advice_

Several factors drive memory usage in Rundeck:

* User sessions
* Concurrent threads
* Concurrent jobs
* Number of managed nodes

For example, if your installation has dozens of active users 
that manage a large environment (1000+ nodes), and has
sufficient system memory, the following sizings might be more suitable:

    export RDECK_JVM="$RDECK_JVM -Xmx4096m -Xms1024m -XX:MaxPermSize=256m"

### Quartz job threadCount

The maximum number of threads used by Rundeck for concurrent jobs 
is set in the ``quartz.properties`` file. By default, this is set to ``10``.

* RPM install: ``/var/lib/rundeck/server/exp/webapp/WEB-INF/classes/quartz.properties``
* Launcher install: ``$RDECK_BASE/server/exp/webapp/WEB-INF/classes/quartz.properties``

To change the maximum threadCount modify this file and alter the line:

    org.quartz.threadPool.threadCount = 20

Set the threadCount value to the max number of threads you want to run concurrently.

Please refer to the Quartz site for detailed information:
[Configuration - Thread Pool](http://www.quartz-scheduler.org/docs/1.x/configuration/ConfigThreadPool.html)

### JMX instrumentation

You may wish to monitor the internal operation of your Rundeck server via JMX.

JMX provides introspection on the JVM, the application server, 
and classes all through a consistent interface. 
These various components are exposed to the management console 
via JMX managed beans - MBeans for short.

_Note_: For more background information on JMX, see 
"[Java theory and practice: Instrumenting applications with JMX.](http://www.ibm.com/developerworks/library/j-jtp09196/)". 

Enable local JMX monitoring by adding the ``com.sun.management.jmxremote``
flag to the startup parameters in the [profile](configuration.html#profile).

    export RDECK_JVM="$RDECK_JVM -Dcom.sun.management.jmxremote"

You use a JMX client to monitor JMX agents. 
This can be a desktop GUI like JConsole run locally.

    jconsole <rundeck pid>
    
For instructions on remote JMX monitoring for Grails, Spring and log4j see:
[Grails in the enterprise](http://www.ibm.com/developerworks/java/library/j-grails12168/index.html).
