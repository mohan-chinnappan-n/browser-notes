### Garbage collection in IE

Background:

JScript and VBScript both are automatic storage languages. Unlike, say, C++, the script developer does not have to worry about explicitly allocating and freeing each chunk of memory used by the program. The internal device in the engine which takes care of this task for the developer is called the garbage collector. This is very similar to Java Garbage Collection (GC):


Details about Automatic Garbage Collection:

Let us take Java GC to explain the concepts:

[Ref](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)

Step 1: Marking
![Java GC Marking](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide3.png)

Step 2: Normal Deletion

![Java GC Normal Deletion](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide1b.png)

Step 2a: Deletion with Compacting

![Java GC Deletion with Compaction](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide4.png)


Why Generational Garbage Collection?

Having to mark and compact all the objects in a Java Virtual Machine (JVM) is inefficient. As more and more objects are allocated, the list of objects grows and grows leading to longer and longer garbage collection time


You can find fewer objects remain allocated over time. In fact most objects have a very short life as shown by the higher values on the left side of the graph.
![Bytes Allocated vs Bytes Surviving](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/ObjectLifetime.gif)

###JVM Generations

The information learned from the object allocation behavior can be used to enhance the performance of the JVM. Therefore, the heap is broken up into smaller parts or **generations**. The heap parts are: Young Generation, Old or Tenured Generation, and Permanent Generation:

![JVM Generations](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide5.png)


**The Young Generation** is where all new objects are allocated and aged. 

When the young generation fills up, this causes a **minor garbage collection**. Minor collections can be optimized assuming a high object mortality rate. A young generation full of dead objects is collected very quickly. Some surviving objects are aged and eventually move to the old generation.

**Stop the World Event** 

 All minor garbage collections are "Stop the World" events. This means that all application threads are stopped until the operation completes. Minor garbage collections are always Stop the World events.

----

**Major GC**

The Old Generation is used to store long surviving objects. Typically, a threshold is set for young generation object and when that age is met, the object gets moved to the old generation. Eventually the old generation needs to be collected. This event is called a major garbage collection.


Major garbage collection are also Stop the World events. Often a major collection is much slower because it involves all live objects. So for Responsive applications, major garbage collections should be minimized. Also note, that the length of the Stop the World event for a major garbage collection is affected by the kind of garbage collector that is used for the old generation space.

-----
What is Permanent Generation?


The Permanent generation contains metadata required by the JVM to describe the classes and methods used in the application. The permanent generation is populated by the JVM at runtime based on classes in use by the application. In addition, Java SE library classes and methods may be stored here.






