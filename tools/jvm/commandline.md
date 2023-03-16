### Openshift interaction :

#### Openshift remote debug :

It is possible to debug your service/application remotely.

Add to the dc the following environment variable :

```yaml
- name: JAVA_OPTS_APPEND
  value: >-
    -agentlib:jdwp=transport=dt_socket,server=y,address=8000,suspend=n
```

Once you add it,  do the port forwarding to access the remote java application from your computer :

```bash
$ oc port-forward <POD> 8000
```

And finally, connect to the with your intellij or other debug tool.

<u>For intellij :</u>

<img src=".\assets\commandline.md" alt="image-20210210143318273" style="zoom: 67%;" />



Once you are connected, you can modify the source code and deploy the modification to the container.

#### Openshift remote jmx :

If you want to follow your java process from your local machine, you can connect to the remote process with this procedure.

Add to the dc the following environment variable :

```yaml
- name: JAVA_OPTS_APPEND
  value: >-
    -Dcom.sun.management.jmxremote=true
    -Dcom.sun.management.jmxremote.port=3000
    -Dcom.sun.management.jmxremote.rmi.port=3001
    -Djava.rmi.server.hostname=127.0.0.1
    -Dcom.sun.management.jmxremote.authenticate=false
    -Dcom.sun.management.jmxremote.ssl=false
```

Once you add it,  do the port forwarding to access the remote java application from your computer :

```bash
$ oc port-forward <POD> 3000 3001
```

And finally, you can connect your jvisualvm local tool to the remote process with this command :

```bash
$ visualvm --openjmx localhost:3000
```

### Java memory :

The java virtual machine memory is divided in multiple section :

<img src=".\assets\memory_areas.png" style="zoom:25%;" />

<img src=".\assets\metaspace_loop.png" />

The heap space holds object data, the method area holds class code,  and the native area holds references to the code and object data.

| Memory part           | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| Young Generation      | Young generation is the place where all the new  objects are created. When young generation is filled, garbage collection is performed. This garbage collection is called Minor GC. Young Generation is divided into three parts – **Eden Memory** and two **Survivor Memory spaces**. |
| Old Generation        | Old Generation memory contains the objects that  are long lived and survived after many rounds of Minor GC. Usually  garbage collection is performed in Old Generation memory when it’s full. Old Generation Garbage Collection is called Major GC and usually takes longer time. |
| Permanent Generation  | Permanent Generation or “Perm Gen” contains the  application metadata required by the JVM to describe the classes and  methods used in the application. Note that Perm Gen is not part of Java  Heap memory.  Perm Gen is populated by JVM at runtime based on the classes used by the application. Perm Gen also contains Java SE  library classes and methods. Perm Gen objects are garbage collected in a full garbage collection. |
| Method Area           | Method Area is part of space in the Perm Gen and used to store class structure (runtime constants and static variables)  and code for methods and constructors. |
| Memory Pool           | Memory Pools are created by JVM memory managers  to create a pool of immutable objects, if implementation supports it.  String Pool is a good example of this kind of memory pool. Memory Pool  can belong to Heap or Perm Gen, depending on the JVM memory manager  implementation. |
| Runtime Constant Pool | Runtime constant pool is per-class runtime  representation of constant pool in a class. It contains class runtime  constants and static methods. Runtime constant pool is the part of  method area. |
| Java Stack Memory     | Java Stack memory is used for execution of a  thread. They contain method specific values that are short-lived and  references to other objects in the heap that are getting referred from  the method. |

Java provides a lot of memory switches that we can  use to set the memory sizes and their ratios. Some of the commonly used  memory switches are                

| VM switch           | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| -Xmx                | Max heap size                                                |
| -Xms                | Initial heap size                                            |
| -XX:NewSize         | Initial young generation size                                |
| -XX:SurvivorRatio   | For providing ratio of Eden space and Survivor Space,  for example if Young Generation size is 10m and VM switch is -XX:SurvivorRatio=2 then 5m will be reserved for Eden Space and 2.5m each for both the Survivor spaces. The default value is 8. |
| -XX:NewRatio        | For providing ratio of old/new generation sizes. The default value is 2. |
| -Xss                | Thread’s stack size. The default value is 0 and lets the OS define the thread size. |
| -XX:+CompressedOops | This option allows pointer compression in 64bit JVM to reduce the heap. |

##### AOT vs JIT

The AOT compilation is execute when you compile your application.

The JIT compilation is done when the code is super used in the application. There is several level of JIT compilation, more the code is used more the compilation process is optimized.

Info on the process :

* Video : https://www.youtube.com/watch?v=sJVenujWGjs (10:01)
* Book : https://learning.oreilly.com/library/view/java-performance-2nd/9781492056102/ch04.html#JustInTimeCompilation

### Garbage collector :

The Java Garbage Collector is an automatic background process that manages the memory of the Java virtual machine. 

It mainly deals with the allocation and de-allocation of memory to allow its use in our program. The key to this is the ability to create new objects without having to worry, as is the case in C++ language for example, about having to manually manage the memory they occupy. 

To go further, it is necessary to introduce the concept of Heap and Stack, and how they are managed by the JVM. 

When executing a program or application, the JVM divides the memory into Heap and Stack. Each time new variables, objects or methods are declared, the JVM allocates space to these objects, either in Stack or in Heap. 

#### Stack & Garbage Collector

Stack is used for static allocation and process execution. The mechanism stores primitive values and maintains references to objects in the heap.  

As soon as a new method is invoked, a new block is created at the top of the stack, which contains values specific to that method, such as primitive variables and references to objects in Heap. 

When the method finishes execution, the relative frame of Stack is emptied, the flow returns to the calling method and space becomes available for the next method. 

#### Heap & Garbage collector

The management mechanism is used for dynamic memory allocation for Java objects and JRE classes at runtime. New objects are always created in Heap space and references to these objects are stored in Stack memory. These objects have global access and can be accessed from anywhere in the application. 

![](.\assets\gc\java-heap-stack-diagram.png)

Generally speaking, the GC has the task of finding and deleting unused (inaccessible) objects from memory but, in reality, it keeps track of every available object in the JVM space and deletes those that are not used. This phase is called "Stop the World" and may require pausing all threads in the application (this is called Pause the World).  

<img src=".\assets\gc\gc_phases.png" style="zoom:75%;" />

To do this, it uses two simple techniques called Mark and Sweep : 

- Mark identifies which objects in memory are used and which are not. 
- Sweep deletes the objects identified in the previous phase. 

During this cleaning process, all application threads will be suspended and no transactions will be performed. 

In this phase, all processor cycles will be used to clean memory. Once completed, all threads will be reassigned and the application can continue. 

Important: there is also a third phase (compacting phase) performed or not based on the chosen GC algorithm. This solves the problem of fragmented memory. Once all living objects are marked, they are moved to the beginning of the memory space, which avoids having a too fragmented memory. But compacting the heap is not free, quite the contrary! Copying objects and updating all references to them takes time and affects the duration of "Stop the world" . 

##### **The GC has four main implementations :** 

- Serial Garbage Collector 
- Parallel Garbage Collector 
- CMS Garbage Collector 
- G1 Garbage Collector 

##### Later versions of Java introduced other GC : 

- Epsilon Garbage Collector (v11) 
- Z Garbage Collector (v11) 
- Shenandoah Garbage Collector (v12) 

For each of these implementations, the functioning of the garbage collector is different :

<img src=".\assets\gc\main_gc.png" style="zoom:50%;" />

##### Here is a short description of these implementations:

###### Serial Garbage Collector

Simple implementation designed for a single thread or low memory environment. Freezes all application threads at runtime and uses a single thread for cleanup. Not recommended in multithreaded applications because it stops all application threads. 

###### **Parallel Garbage Collector** 

This is the default GC of the JVM (called Throughput Collector). Unlike the Serial GC, it uses a multiple thread for space management, but also blocks the other threads of the application when the GC is executed. If we use this GC, we can specify the maximum number of threads for object collection and pause time, throughput and Heap size. 

###### **CMS Garbage Collector** 

The CMS (Concurrent Mark Sweep) implementation uses multiple concurrent threads to scan the Heap ("mark") and to search for unused objects to clean up ("sweep"). Designed for applications that prefer shorter pauses and can afford to share CPU resources for cleaning while the application is running. Applications that use this type of GC respond slower on average, but do not stop responding during the cleanup phase. 

###### **G1 Garbage Collector** 

Or G1 (Garbage First). Available starting with JDK 7, it is designed for applications that run on multiprocessor machines with large memory space. Once the Heap is divided into sections of similar size, each with a contiguous range of virtual memory, it executes the "mark" phase simultaneously and, once completed, knows which sections are mostly empty. G1 focuses on these areas first, usually producing a significant amount of free space. It replaces the CMS because it is more efficient in terms of performance. 

###### **Epsilon Garbage Collector** 

Epsilon is a non-operational or passive GC. It allocates memory for the application, but does not collect unused objects. When the application runs out of Java Heap, the JVM shuts down. This means that Epsilon's Garbage Collector allows applications to run out of memory and fail. The purpose of this Garbage Collector is to measure and manage the performance of applications. 

###### **Z Garbage Collector** 

ZGC performs all expensive jobs simultaneously, without stopping the execution of application threads for more than 10 ms, which makes it suitable for applications that require low latency and/or use a very large Heap. ZGC will try to set the number of threads itself, and this is usually correct. But if ZGC has too many threads, your application will suffer. If it doesn't have enough, you will create inaccessible objects faster than the GC can collect them. 

###### **Shenandoah Garbage Collector** 

The functioning of Shenandoah garbage collector is very specific. It is a so-called ultra-low-pause Garbage Collector that reduces GC pause times by doing more work collecting inaccessible objects at the same time as the running Java program. Both CMS and G1 perform simultaneous live object marking. Shenandoah adds concurrent compaction. It uses memory areas to manage objects that are no longer in use, and those that are alive and ready for compaction. Shenandoah also adds a redirection pointer to each object in the Heap and uses it to control access to the object. Shenandoah must also collect the Heap faster than the application it serves allows. If the allocation pressure is too high and there is not enough room for new allocations, it will fail.  Shenandoah offers the same advantages as the ZGC with large piles but more tuning options. 

#### Heap et Roots objects

Objects are represented in memory through a tree. Each object tree must have one or more root objects. As long as the application can reach these roots, the whole tree is accessible. 

There are four types of GC roots in Java : 

- Local variables: kept alive by a Thread Stack 
- Active Threads, always considered as live and therefore GC root 
- Static variables, referenced by their classes 
- JNI References: Java objects created from native code as part of a JNI call 

![](.\assets\gc\les-racines-les-objets-accessibles-et-les-objets-inaccessibles_jpg.png)

#### stack vs heap

| Parameter               | Stack Memory                                                 | Heap Space                                                   |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Application             | Stack is used in parts, one at a time during execution of a thread | The entire application uses Heap space during runtime        |
| Size                    | Stack has size limits depending upon OS and is usually smaller then Heap | There is no size limit on Heap                               |
| Storage                 | Stores only primitive variables and references to objects that are created in Heap Space | All the newly created objects are stored here                |
| Order                   | It is accessed using Last-in First-out (LIFO) memory allocation system | This memory is accessed via complex memory management techniques  that include Young Generation, Old or Tenured Generation, and Permanent  Generation. |
| Life                    | Stack memory only exists as long as the current method is running | Heap space exists as long as the application runs            |
| Efficiency              | Comparatively much faster to allocate when compared to heap  | Slower to allocate when compared to stack                    |
| Allocation/Deallocation | This Memory is automatically allocated and deallocated when a method is called and returned respectively | Heap space is allocated when new objects are created and deallocated by Gargabe Collector when they are no longer referenced |

### Java commands :

There are multiple commands that exists to interact with the JVM, we will treat a part of them in this documentation.

#### Available diagnostic commands :

- jps 
- jinfo
- jcmd
- jmap
- jhat
- jstack
- jstat

##### jps command :

A simple command that lists all of the Java processes currently running on the machine. It returns the "JVMID," an identifier that is often used with other tools to uniquely identify this executing JVM. Most often, the JVMID is the exact same as the operating system's process identifier, or PID, but it can
include hostname and port, so that a tool can connect remotely to a running process, assuming the network facilities permit such communication. 

```bash
sh-4.2$ jps
487 jboss-modules.jar
12809 Jps

sh-4.2$ jps -m
487 jboss-modules.jar -mp /opt/eap/modules -jaxpmodule javax.xml.jaxp-provider org.jboss.as.standalone -Djboss.home.dir=/opt/eap -Djboss.server.base.dir=/opt/eap/standalone -c standalone-openshift.xml -P /opt/eap/standalone/configuration/env/base/cb-system-acc.properties -P /opt/eap/standalone/configuration/application-system-acc.properties -P /opt/eap/standalone/configuration/system.properties -b 10.126.152.105 -Djboss.node.name=oauth-v4-sic-36-4d26z -bmanagement=0.0.0.0
12845 Jps –m

sh-4.2$ jps -v
12930 Jps -Dapplication.home=/usr/lib/jvm/java-1.8.0-openjdk -Xms8m
487 jboss-modules.jar -D[Standalone] -XX:+UseCompressedOops -verbose:gc -Xloggc:/opt/eap/standalone/log/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms384m -Xmx1536m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.logmanager,jdk.nashorn.api,com.sun.crypto.provider -Djava.awt.headless=true -Djboss.modules.policy-permissions=true -javaagent:/opt/jolokia/jolokia.jar=config=/opt/jolokia/etc/jolokia.properties -Xbootclasspath/p:/opt/eap/jboss-modules.jar:/opt/eap/modules/system/layers/base/.overlays/layer-base-jboss-eap-6.4.20.CP/org/jboss/logmanager/main/jboss-logmanager-1.5.8.Final-redhat-1.jar:/opt/eap/modules/system/layers/base/org/jboss/logmanager/ext/main/jboss-logmanager-ext-1.0.0.Alpha2-redhat-1.jar -Djava.util.logging.manager=org.jboss.logmanager.LogManager -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+UseParallelOldGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRa
```

##### jinfo command :

An "information dump" utility. When given a private static String message = JMVID, it will connect to the target JVM and "dump"
a large amount of information about the process's environment, including all of its system properties, the command-line used to launch the JVM, and the nonstandard JVM options used (meaning the “-XX” flags)  

```bash
sh-4.2$ jinfo -help
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server) 

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message 

sh-4.2$ jinfo -flags 487
Attaching to process ID 487, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09
Non-default VM flags: -XX:AdaptiveSizePolicyWeight=90 -XX:CICompilerCount=2 -XX:CompressedClassSpaceSize=260046848 -XX:+ExitOnOutOfMemoryError -XX:GCLogFileSize=3145728 -XX:GCTimeRatio=4 -XX:InitialHeapSize=1610612736 -XX:MaxHeapFreeRatio=20 -XX:MaxHeapSize=1610612736 -XX:MaxMetaspaceSize=268435456 -XX:MaxNewSize=536870912 -XX:MinHeapDeltaBytes=524288 -XX:MinHeapFreeRatio=10 -XX:NewSize=536870912 -XX:NumberOfGCLogFiles=5 -XX:OldSize=1073741824 -XX:ParallelGCThreads=2 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:-TraceClassUnloading -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseGCLogFileRotation -XX:+UseParallelOldGC
Command line:  -D[Standalone] -XX:+UseCompressedOops -verbose:gc -Xloggc:/opt/eap/standalone/log/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms384m -Xmx1536m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.logmanager,jdk.nashorn.api,com.sun.crypto.provider -Djava.awt.headless=true -Djboss.modules.policy-permissions=true -javaagent:/opt/jolokia/jolokia.jar=config=/opt/jolokia/etc/jolokia.properties -Xbootclasspath/p:/opt/eap/jboss-modules.jar:/opt/eap/modules/system/layers/base/.overlays/layer-base-jboss-eap-6.4.20.CP/org/jboss/logmanager/main/jboss-logmanager-1.5.8.Final-redhat-1.jar:/opt/eap/modules/system/layers/base/org/jboss/logmanager/ext/main/jboss-logmanager-ext-1.0.0.Alpha2-redhat-1.jar -Djava.util.logging.manager=org.jboss.logmanager.LogManager -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+UseParallelOldGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:MaxMetaspaceSize=256m -XX:ParallelGCThreads=2 -Djava.util.concurrent.ForkJoinPool.common.parallelism=2 -XX:CICompilerCount=2 -XX:+ExitOnOutOfMemoryError -Djava.security.egd=file:/dev/./urandom -Xms1536m -Xmx1536m -Dcommonbuild.properties.path=/opt/eap/standalone/configuration/env/base/cb-acc.properties -Dfile.encoding=UTF-8 -Dorg.jboss.boot.log.file=/opt/eap/standalone/log/server.log -Dlogging.configuration=file:/opt/eap/standalone/configuration/logging.properties

sh-4.2$ jinfo -flag NewSize 487
-XX:NewSize=536870912
```

##### jcmd command :

A "command" utility that can issue a number of different debugging/monitoring commands to the target JVM.
Use "jcmd <pid> help" to see a list of commands that can be sent.
One of the most useful will be GC.heap_dump to take a snapshot of the entire JVM heap, for offline analysis. 

```bash
sh-4.2$ jcmd
13317 sun.tools.jcmd.JCmd
487 /opt/eap/jboss-modules.jar -mp /opt/eap/modules -jaxpmodule javax.xml.jaxp-provider org.jboss.as.standalone -Djboss.home.dir=/opt/eap -Djboss.server.base.dir=/opt/eap/standalone -c standalone-openshift.xml -P /opt/eap/standalone/configuration/env/base/cb-system-acc.properties -P /opt/eap/standalone/configuration/application-system-acc.properties -P /opt/eap/standalone/configuration/system.properties -b 10.126.152.105 -Djboss.node.name=oauth-v4-sic-36-4d26z -bmanagement=0.0.0.0 

sh-4.2$ jcmd 487 help
487:
The following commands are available:
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
VM.classloader_stats
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.finalizer_info
GC.heap_info
GC.run_finalization
GC.run
VM.uptime
VM.dynlibs
VM.flags
VM.system_properties
VM.command_line
VM.version
help

For more information about a specific command use 'help <command>'.

sh-4.2$ jcmd 487 VM.flags
487:
-XX:AdaptiveSizePolicyWeight=90 -XX:CICompilerCount=2 -XX:CompressedClassSpaceSize=260046848 -XX:+ExitOnOutOfMemoryError -XX:GCLogFileSize=3145728 -XX:GCTimeRatio=4 -XX:InitialHeapSize=1610612736 -XX:MaxHeapFreeRatio=20 -XX:MaxHeapSize=1610612736 -XX:MaxMetaspaceSize=268435456 -XX:MaxNewSize=536870912 -XX:MinHeapDeltaBytes=524288 -XX:MinHeapFreeRatio=10 -XX:NewSize=536870912 -XX:NumberOfGCLogFiles=5 -XX:OldSize=1073741824 -XX:ParallelGCThreads=2 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:-TraceClassUnloading -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseGCLogFileRotation -XX:+UseParallelOldGC
```

##### jmap command :

Another "heap dump" utility that can not only dump the JVM heap into the same format that jcmd uses, but can also track and dump ClassLoader statistics, which can be helpful to discover ClassLoader class leaks.

```bash
sh-4.2$ jmap -help
Usage: 
    jmap [option] <pid>
        (to connect to running process)    
    jmap [option] <executable <core>      
        (to connect to a core file)  
    jmap [option] [server_id@]<remote server IP or hostname>  
        (to connect to remote debug server)                                                                                  

where <option> is one of:            
    <none>               to print same info as Solaris pmap    
    -heap                to print java heap summary  
    -histo[:live]        to print histogram of java object heap; if the "live"    
                         suboption is specified, only count live objects   
    -clstats             to print class loader statistics             
    -finalizerinfo       to print information on objects awaiting finalization  
    -dump:<dump-options> to dump java heap in hprof binary format    
                         dump-options:                       
                           live         dump only live objects; if not specified, 
                                        all objects in the heap are dumped.    
                           format=b     binary format 
                           file=<file>  dump heap to <file> 
                         Example: jmap -dump:live,format=b,file=heap.bin <pid> 
    -F                   force. Use with -dump:<dump-options> <pid> or -histo  
                         to force a heap dump or histogram when <pid> does not  
                         respond. The "live" suboption is not supported 
                         in this mode.                 
    -h | -help           to print this help message      
    -J<flag>             to pass <flag> directly to the runtime system                                                     

sh-4.2$ jmap -heap 487 
Attaching to process ID 487, please wait...  
Debugger attached successfully.   
Server compiler detected.          
JVM version is 25.201-b09           

using thread-local object allocation.  
Parallel GC with 2 thread(s)                                                                                                    

Heap Configuration:   
   MinHeapFreeRatio         = 10   
   MaxHeapFreeRatio         = 20   
   MaxHeapSize              = 1610612736 (1536.0MB)  
   NewSize                  = 536870912 (512.0MB)    
   MaxNewSize               = 536870912 (512.0MB)    
   OldSize                  = 1073741824 (1024.0MB) 
   NewRatio                 = 2               
   SurvivorRatio            = 8              
   MetaspaceSize            = 21807104 (20.796875MB)  
   CompressedClassSpaceSize = 260046848 (248.0MB)     
   MaxMetaspaceSize         = 268435456 (256.0MB)    
   G1HeapRegionSize         = 0 (0.0MB)          
   
Heap Usage:        
PS Young Generation  
Eden Space:            
   capacity = 521666560 (497.5MB)    
   used     = 19502248 (18.598793029785156MB)  
   free     = 502164312 (478.90120697021484MB)  
   3.738450860258323% used                      
From Space:              
   capacity = 7864320 (7.5MB)  
   used     = 0 (0.0MB)       
   free     = 7864320 (7.5MB)  
   0.0% used                   
To Space:            
   capacity = 7340032 (7.0MB)  
   used     = 0 (0.0MB) 
   free     = 7340032 (7.0MB)  
   0.0% used                 
PS Old Generation           
   capacity = 1073741824 (1024.0MB)   
   used     = 91425096 (87.18976593017578MB)   
   free     = 982316728 (936.8102340698242MB)  
   8.514625579118729% used              
   
43905 interned Strings occupying 5036320 bytes. 
```

##### jhat command :

The "Java heap analyzer tool". It takes a heap dump generated by any of the other utilities and provides a tiny HTTP-navigable server to examine
the contents. While the user interface isn’t amazing and clearly hasn’t been updated in years, jhat has one unique facility to it that is quite useful: the ability to write OQL queries to examine the contents of the heap without having to manually click through every object.

```bash
sh-4.2$ jhat myheap.hprof
Reading from myheap.hprof... 
Dump file created Thu Feb 11 15:04:47 CET 2021  
Snapshot read, resolving...          
Resolving 1269319 objects...       
Chasing references, expect 253 dots.....................................................................................
Eliminating duplicate references............................................ 
Snapshot resolved.                                              
Started HTTP server on port 7000           
Server is ready. 
```

##### jstack command :

Does a Java thread stack dump of any running JVM process. A critical first step to diagnosing any thread-deadlock or high-contention errors in a
running JVM, even if that JVM is in production.

```bash
sh-4.2$ jstack -help
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server) 

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```

##### jstat command :

A "Java statistics" utility. Run "jstat -options" to see the full list of commands that can be passed to the target JVM, most of which are GC-related.  

```bash
sh-4.2$ jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]] 

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.

sh-4.2$ jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```

































#### Deeper in JVM :

This collection of SIC related training resources is based on videos (globally available on YouTube for easy access), online articles
and books (on Safari books), or parts thereof. 

* The videos can be considered as a "fast track" into a subject.
* The books are typically used to obtain more information about the subject and/or as reference material. 

While following along with these training resources you will learn a lot on the internals of the JVM.
This will help you assess and understand the health of your application and fix issues. 

You will also learn how to test your applications to detect instability, performance problems, ...

##### The Java Virtual Machine 

###### Memory 

* Garbage collection (GC)
    * Video : https://www.youtube.com/watch?v=CdAmS9H93q4 (42:24)  
* Generational garbage collection
    * Video : https://www.youtube.com/watch?v=XrNgF2sbhGQ (26:45)
    * Books :
                *** https://learning.oreilly.com/library/view/java-performance-2nd/9781492056102/ch05.html#GC
                *** https://learning.oreilly.com/library/view/java-performance-2nd/9781492056102/ch06.html#Collectors 
* Hunting for memory leaks
    * Video : https://www.youtube.com/watch?v=spDb2pUKs7Q (26:02)
* Memory leaks
    * Videos :
                   *** OutOfMemoryError in Java heap space : https://www.youtube.com/watch?v=kQpkjCUQvEc (13:51)
                   *** Solving Java Memory Leaks : https://www.youtube.com/watch?v=E2KYTXKUsT4 (45:05)

###### Heap dumps

* JVM Heap Dump Analysis
    * Video : OpenJPA memory leak : https://www.youtube.com/watch?v=5joejuE2rEM (12:25)
    * Book : https://learning.oreilly.com/library/view/java-performance-2nd/9781492056102/ch07.html#Memory

###### JVM tools

* JVisualVM
    * Video : https://www.youtube.com/watch?v=qrvDKp8iTIQ (08:26)
    * Book : https://learning.oreilly.com/library/view/java-ee-8/9781788473064/264f0062-c0a2-4077-80da-e02fa465a305.xhtml
* Tips :
               *** security problem :
               If you have problems to detect local applications, you probably need to create this folder:
               `C:\Users\%username%\AppData\Local\Temp\hsperfdata_%username%`
               with security permissions that give everyone full control.

###### JVM flags

* Website : https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html
  * Book : https://learning.oreilly.com/library/view/java-performance-2nd/9781492056102/app01.html#FlagAppendix

 





























\- name: JAVA_OPTS_APPEND

​       value: >-

​        -Xms2g -Xmx2g -XX:FlightRecorderOptions=stackdepth=200

​        -XX:+UnlockCommercialFeatures -XX:+FlightRecorder



 

 

​        Article :

https://dzone.com/articles/remote-debugging-of-java-applications-on-openshift

 

 

