### Garbage collection Java Virtual Machine 


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

Classes may get collected (unloaded) if the JVM finds they are no longer needed and space may be needed for other classes. The permanent generation is included in a full garbage collection.

----

Garbage Collection


Younger Generation:

First, any new objects are allocated to the eden space. Both survivor spaces start out empty:

![YG GC](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide13.png)


When the eden space fills up, a minor garbage collection is triggered.
![YG GC trigger](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide14.png)


Referenced objects are moved to the first survivor space. Unreferenced objects are deleted when the eden space is cleare
![YG move to first survivor space](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide6.png)

At the next minor GC, the same thing happens for the eden space. Unreferenced objects are deleted and referenced objects are moved to a survivor space. However, in this case, they are moved to the second survivor space (S1). In addition, objects from the last minor GC on the first survivor space (S0) have their age incremented and get moved to S1. Once all surviving objects have been moved to S1, both S0 and eden are cleared. Notice we now have differently aged object in the survivor space.

![YG move to S1](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide8.png)


At the next minor GC, the same process repeats. However this time the survivor spaces switch. Referenced objects are moved to S0. Surviving objects are aged. Eden and S1 are cleared.

![YG s1-s0 switch](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide9.png)

 After a minor GC, when aged objects reach a certain age threshold (8 in this example) they are promoted from young generation to old generation.

![YG promotion](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide7.png)

![YG promotion2](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide10.png)

![GC general](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/images/gcslides/Slide11.png)












