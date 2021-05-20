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

# Find the Leaking Class
After you have your recording showing the leak, you can look at Object Statistics. Look at one long recording, then look at which classes grew the most in heap usage over the recording. If you took several recordings at intervals, then compare the heap contents section and see what object types have increased the most between the recordings, as shown in Figure 3-2.

![Alt text](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/img/jfr-memory-leak.png)

Especially, watch the classes that are not part of the standard library. For example, you will often see Char arrays as one of the top growers. This is due to many Strings being allocated; therefore, watch out for objects that keep these Strings alive. If you have a class that has 10 Strings as members, then the object itself will not use too much heap. The heap will be used by the Strings, which mostly contains pointers to the Char arrays. Therefore, it is good to sort on the number of instances and not the size of the objects. If one of your application class has many instances, then it may very well be those objects that keep other objects alive.

