# Memory-leak-using-jfr
Debug a Memory Leak Using Java Flight Recorder

# Java Flight Recorder

* Java Flight Recorder (JFR) is a monitoring tool that collects information about the events in a Java Virtual Machine (JVM) during the execution of a Java application. JFR is part of the JDK distribution, and it's integrated into the JVM.
* JFR is designed to affect the performance of a running application as little as possible.

``
These three components — JFR, jcmd and JMC — form a complete suite for collecting low-level runtime information of a running Java program. We may find this information very useful when optimizing our program, or when diagnosing it when something goes wrong.
``

# Detect a Memory Leak

Heap Statistics generates an accurate live set information. If you suspect a rather quick memory leak, take a profiling recording that runs over, for example, an hour. Click Memory tab and select Heap Usage.


![Inkedimage (4)_LI](https://user-images.githubusercontent.com/30730414/118960827-4ab18600-b981-11eb-9b8a-497d80facd26.jpg)

* It will show how heap memory reduce overtime there is some memory which GC fail to free  due to which heap memory reduce so there is definitely a memory leak in system.


``
Next select the Garbage Collections tab to inspect the first and the last old collections as shown below there is difference between heap usage when GC run first time and last run.
Select the first old collection as shown in Figure 3-1 to look at the heap data and heap usage after GC. In this recording, it is 34.10 MB. Now look at the same data from the last old collection in the list and see if the live set has grown.
``

* Heap Usage after first GC run is 19MB. 


![image (5)](https://user-images.githubusercontent.com/30730414/118962655-3a9aa600-b983-11eb-9a27-8e368ab8a20b.png) 



* Heap Usage after latest GC run is 43MB.


![image (3)](https://user-images.githubusercontent.com/30730414/118962765-5900a180-b983-11eb-8793-dacfd73095e2.png)

# Find the Leaking Class
After you have your recording showing the leak, you can look at Object Statistics. Look at one long recording, then look at which classes grew the most in heap usage over the recording. If you took several recordings at intervals, then compare the heap contents section and see what object types have increased the most between the recordings, as shown in Figure 3-2.

![Alt text](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/img/jfr-memory-leak.png)


Especially, watch the classes that are not part of the standard library. For example, you will often see Char arrays as one of the top growers. This is due to many Strings being allocated; therefore, watch out for objects that keep these Strings alive. If you have a class that has 10 Strings as members, then the object itself will not use too much heap. The heap will be used by the Strings, which mostly contains pointers to the Char arrays. Therefore, it is good to sort on the number of instances and not the size of the objects. If one of your application class has many instances, then it may very well be those objects that keep other objects alive.

``
Note: if one particular class grew fast than you can restrict the object creation of that class using Static/by using static factory method in place of a constructor or by using singleton construction.
``


# Find Bottlenecks
Different applications have different bottlenecks. For some applications, a bottleneck may be waiting for I/O or networking, it may be synchronization between threads, or it may be actual CPU usage. For others, a bottleneck may be garbage collection times. It is possible that an application has more than one bottleneck.

One way to find out the application bottlenecks is to look at the Events tab. This is an advanced tab, and there are a few things to do. First, click the Events tab, which opens the Event Types tab on the left side of the JFR window. 

The Graph tab may be hard to grasp at first. Each row is a thread, and each thread can have several lines. In Figure 4-2, each thread has a line, which represents the Java Application events that were enabled in the Event Types tab for this recording. The selected Java Application events all have the important property that they are all thread-stalling events. Thread stalling indicates that the thread was not running your application during the event, and they are all duration events. The duration event measures the duration the application was not running.

From the Event Types tab, look at the color of each event. For example, yellow represents Java Monitor Wait events. The yellow part is when threads are waiting for an object. This often means that the thread is idle, perhaps waiting for a task. Red represents the Java Monitor Blocked events or synchronization events. If your Java application's important threads spend a lot of time being blocked, then that means that a critical section of the application is single threaded, which is a bottleneck. Blue represents the Socket Reads and Socket Writes events. Again, if the Java application spends a lot of time waiting for sockets, then the main bottleneck may be in the network or with the other machines that the application communicates.

Green represents parts that don't have any events. The green part means that the thread is not sleeping, waiting, reading to or from a socket, or not being blocked. In general, this is where the application code is run. If your Java application's important threads are spending a lot of time without generating any application events, then the bottleneck in the application is the time spent executing code or the CPU itself.

![image (1)](https://user-images.githubusercontent.com/30730414/118965741-9fa3cb00-b986-11eb-9949-e992ad2b7d5c.png)


# Code Execution Performance

When there are not a lot of Java Application events, it could be that the main bottleneck of your application is the running code.Select the Code tab group and look at the Hot Threads tab in case your application is using a lot of CPU time. This tab shows the threads that use the most CPU time. However, this information is based on method sampling, so it may not be 100% accurate if the sample count is low. When a JFR is running, the JVM samples the threads. By default, a continuous recording does only some method sampling, while a profiling recording does as much as possible. The method sampling gathers data from only those threads running code. The threads waiting for I/O, sleeping, waiting for locks, and so on are not sampled. Therefore, threads with a lot of method samples are the ones using the most CPU time; however, how much CPU is used by each thread is not known.

![image (2)](https://user-images.githubusercontent.com/30730414/118965787-ae8a7d80-b986-11eb-9550-53a64f5f8c0f.png)

# Related links:
* [Enable JFR for Oracle JDK 8](https://www.jetbrains.com/help/idea/java-flight-recorder.html) 
* [JFR Oracle Docs](https://docs.oracle.com/javase/10/troubleshoot/troubleshoot-performance-issues-using-jfr.htm#JSTGD311)
