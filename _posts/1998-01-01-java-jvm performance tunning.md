---
layout: post
title:  "JVM performance tunning"
date:   2018-01-02 13:15:42 -0500
categories: tech-java-spring
---

## heap

Heap memory = younger generation + older generation + permanent generation  
Younger generation = Eden area + two Survivor areas (From area, To area)  

**SITE** (a unique stack trace of a fixed depth)


## HPROF -agentlib:hprof (JVM heap/cpu profiling agent)

https://docs.oracle.com/javase/8/docs/technotes/samples/hprof.html

HPROF can be used to track down and isolate **performance problems involving memory usage and inefficient code**

### introduction

HPROF agent can be used to generate a wide variety of CPU and HEAP profiles. make sure you have a large enough sampling to know that your profile data makes sense.

HPROF is actually a JVM native agent library (JNI) which is dynamically loaded through a command line option, at JVM startup, and becomes part of the JVM process. By supplying HPROF options at startup, users can request various types of heap and/or cpu profiling features from HPROF. 

The data generated can be in textual or binary format. The binary format file from HPROF can be used with tools such as jhat to browse the allocated objects in the heap.

	java -agentlib:hprof[=options] ToBeProfiledClass		// run time JVM params
	java -Xrunprof[:options] ToBeProfiledClass				// run time JVM params, deprecated. use java - agentlib:hprof instead.
	javac -J-agentlib:hprof[=options] ToBeProfiledClass		// compile time javac params. Its implementation uses bytecode injection technology
	

hprof usage: java -agentlib:hprof=[help]|[<option>=<value>, ...]

	Option Name and Value  Description                    Default
	---------------------  -----------                    -------
	heap=dump|sites|all    heap profiling                 all
	cpu=samples|times|old  CPU usage                      off
	monitor=y|n            monitor contention             n
	format=a|b             text(txt) or binary output     a
	file=<file>            write data to file             java.hprof[.txt]
	force=y|n              force output to <file>         y
	
	// CPU Usage Sampling Profiling(cpu=samples). The generated profile file name is java.hprof.txt, in the current directory. 
	java -agentlib:hprof=cpu=samples,interval=20,depth=3 Hello
	// CPU Usage Times Profiling(cpu=times) can obtain finer-grained CPU consumption information than CPU Usage Sampling Profile
	java -agentlib:hprof=cpu=times Hello.java
	// Heap Allocation Profiling(heap=sites)
	java -agentlib:hprof=heap=sites Hello.java
	// Heap Dump(heap=dump) can generate more detailed Heap Dump information than the previous Haip Allocation Profiling
	java -agentlib:hprof=heap=dump Hello.java
	
By default, heap profiling information (sites and dump) is written out to **java.hprof.txt** (ascii).  
Normally the default (force=y) will clobber any existing information in the output file, so if you have multiple VMs running with HPROF enabled, you should use force=n, which will append additional characters to the output filename as needed.
	
Although adding "--agentlib:hprof=heap=site" parameter to JVM startup parameter can generate CPU/Heap Profile file, it has a great impact on JVM performance and is not recommended for online server environment.	

### Heap Allocation Profiles (heap=sites)

heap allocation profile generated by running the Java compiler (javac) :
	java -agentlib:hprof=heap=site Hello.class

A crucial piece of information in heap profile is the amount of allocation that occurs in various parts of the program.  

A good way to relate allocation sites to the source code is to record the dynamic stack traces that led to the heap allocation. Each frame in the stack trace contains class name, method name, source file name, and the line number. The user can set the maximum number of frames collected by the HPROF agent (depth option). The default depth is 4. Stack traces reveal not only which methods performed heap allocation, but also which methods were ultimately responsible for making calls that resulted in memory allocation. 	

Normally you want to watch out for large accumulations of objects, allocated at the same location, that seem excessive.  

Don't expect the above information to reproduce on identical runs with applications that are highly multi-threaded.  

This option can impact the application performance due to the data gathering (stack traces) on object allocation and garbage collection.  

### Heap Dump (heap=dump)

A complete dump of the current **live objects** in the heap can be obtained with:
	java -agentlib:hprof=heap=dump Hello.class
	
This is a very large output file, but can be viewed and searched in any editor. But a better way to look at this kind of detail is with HAT. All the information of the above heap=sites option is included, plus the specific details on every object allocated and the references to all objects.

This option causes the greatest amount of memory to be used because it stores details on every object allocated, it can also impact the application performance due to the data gathering (stack traces) on object allocation and garbage collection.	

### CPU Usage Sampling Profiles (cpu=samples)

The HPROF agent periodically samples the stack of all running threads to record **the most frequently active stack traces**. The count field above indicates how many times a **particular stack trace** was found to be active (not how many times a method was called). These stack traces correspond to the **CPU usage hot spots** in the application. 

cpu=samples option does not require BCI or modifications of the classes loaded. Of all hprof options, cpu=samples option causes the least disturbance of the application being profiled.

The interval option can be used to adjust the sampling time or the time that the sampling thread sleeps between samples.

Don't expect the above information to reproduce on identical runs with highly multi-threaded applications,

### CPU Usage Times Profile (cpu=times)

HPROF can collect CPU usage information by injecting code into every method entry and exit, keeping track of exact method call counts and the time spent in each method. This uses Byte Code Injection (BCI) and runs considerably slower than cpu=samples.

java -agentlib:hprof=cpu=times Hello.class

Here the count represents the true count of the times this method was entered, and the percentages represent a measure of thread time spent in those method.

## Using HAT with HPROF

$JAVA_HOME/bin/jhat

HAT (Heap Analysis Tool) is a browser based tool that uses the HPROF binary format to construct web pages so you can browse all the objects in the heap, and see all the references to and from objects.


## How Does HPROF Work?

HPROF is a dynamically-linked native library that uses JVM TI and writes out profiling information either to a file descriptor or to a socket in ascii or binary format. This information can be further processed by a profiler front-end tool or dumped to a file. 

It generates this information through calls to JVM TI, event callbacks from JVM TI, and through Byte Code Insertion (BCI) on all class file images loaded into the VM. JVM TI has an event called **JVMTI_EVENT_CLASS_FILE_LOAD_HOOK** which gives HPROF access to the class file image and an opportunity to modify that class file image before the VM actually loads it. In the case of HPROF the BCI operations only instrument and don't change the behavior of the bytecodes. Use of JVM TI was pretty critical here for HPROF to do BCI because we needed to do BCI on ALL the classes, including early classes like java.lang.Object. Of course, the instrumentation code needs to be made inoperable until the VM has reached a stage where this inserted code can be executed, normally the event JVMTI_EVENT_VM_INIT.

The amount of BCI that HPROF does depends on the options supplied, cpu=times triggers insertions into all method entries and exits, and the heap options trigger BCI on the <init> method of java.lang.object and any 'newarray' opcodes seen in any method. 

Currently HPROF injects calls to static Java methods which in turn call a native method that is in the HPROF agent library itself. This was an early design choice to limit the extra Java code introduced during profiling.

The cpu=samples option doesn't use BCI, HPROF just spawns a separate thread that sleeps for a fixed number of micro seconds, and wakes up and samples all the running thread stacks using JVM TI.

The cpu=times option attempts to track the running stack of all threads, and keep accurate CPU time usage on all methods. This option probably places the greatest strain on the VM, where every method entry and method exit is tracked. Applications that make many method calls will be impacted more than others.

The heap=sites and heap=dump options are the ones that need to track object allocations. These options can be memory intensive (less so with hprof=sites) and applications that allocate many objects or allocate and free many objects will be impacted more with these options. On each object allocation, the stack must be sampled so we know where the object was allocated, and that stack information must be saved. HPROF has a series of tables allocated in the C or malloc() heap that track all it's information. HPROF currently does not allocate any Java objects.



