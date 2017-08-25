## Notes on IE JavaScript Engine: Chakra


#### Components

![components](./img/chakra-components.png)


#### Architecture

![components](./img/chakra-arch.png)

[Ref](https://github.com/Microsoft/ChakraCore/wiki/Architecture-Overview)


#### Video
[![1bfDB3YPHFI](https://img.youtube.com/vi/1bfDB3YPHFI/0.jpg)](https://www.youtube.com/watch?v=1bfDB3YPHFI)



###Chakra Core


Chakra Core has certain components of **Chakra** and is modified to report certain data differently.

 Instead of **COM-based** diagnostic APIs, Chakra Core provides **JSON-based diagnostic APIs** which allow more applications with JSON parsers to support Chakra Core.
 
 
 **JS to AST to bytecode**:

Chakra Core first reads through the Javascript code syntax and parses it to generate its AST. 

After the AST is generated, the code is passed to the **bytecode generator** and then the bytecode gets **profiled**. 

This is different from Chrome V8 engine, which has a decision process that decides whether a piece of code should be profiled and optimized or should be turned into bytecode.


During the profiling process, the interpreter attempts to send the code to parallel JIT compilers.

 There are two types of JIT compilers available: 
 
 1. simple JIT compiler (quick compilation without much optimization)
 2. full JIT compiler (takes more time but produces better machine code, in case of code out of it's profile, bailout to simple JIT compiler happens)


**Garbage Collection**

As for garbage collection, it is done in a generational **mark-and-sweep** manner. 

When garbage collection happens, a **foreground and background thread** will be spawned to carry out different executions.

The background garbage collector will do a mark, rescan and mark to find objects that should be cleaned up.



### How to use the Memory tool to diagnose memory issues that can impact the speed and stability of webpages

[Ref](https://docs.microsoft.com/en-us/microsoft-edge/f12-devtools-guide/memory)


#### Use case:

The webpage is performing well when it **first loads**, but gradually slows and sometimes crashes, the page's memory use might be the problem.

How to Diagnosing this memory issue?

After you've loaded your webpage in the browser, open the **F12** developer tools and open the Memory tool (CTRL + 6).

 You can start profiling from this screen and take heap snapshots during the profiling session.
 
If the problem is high memory usage from the start, but not memory growth, take one snapshot and examine it to find the bigger areas of memory use. 

If the memory use grows over time, you can compare snapshots to look at the areas where memory is growing.


![Session summary](https://docs.microsoft.com/en-us/microsoft-edge/f12-devtools-guide/media/edge_memory.png)


Note:

Total memory timeline shows the memory use by the **current browser tab's processes** as a graph over time.


Each snapshot shows a thumbnail of the webpage as it appeared at the **time** the snapshot was taken and specifics about the memory usage of your code.

Object Counts:

The number of new objects created and old objects removed are shown below the object count.

 If these numbers are high, you might have a **performance problem**. When the amount of memory you use isn't bad, but a lot of objects are being created and removed, you create work for the **garbage collector** process which frees up memory. 
 
**Garbase collector and performance impacts:**
 
 Garbage collecting uses (CPU) processing power, however, so it's best to find ways to reuse or recycle these objects and give the garbage collector a break.
 

**Signs for Memory Leak**

Another important factor to keep in mind is the difference between the number of objects being added and objects being removed.

 This is displayed in your second snapshot and beyond in the upper right in a "+x/-y" format.  Example: **+96/-9**
 
In the set of snapshots above, it shows that each time we ran the process we suspected of having a memory issue, we increased the number of objects by **96**, but only removed **9**. 

This is a strong clue that there's a **memory leak** in the process.

Memory leaks will keep expanding the memory use of your webpage during a user's session with it, causing slowness or even instability if the memory use grows too big.


#### Snapshot detailed view

![snapshot detailed view](https://docs.microsoft.com/en-us/microsoft-edge/f12-devtools-guide/media/edge_memory_details.png)

Note:

 If you see: **the DOM node is not attached to the DOM** : this can happen when the node has been removed from the DOM, but is referred to elsewhere.
 
 
 Circular references: 
 
 In its simplest form, a circular reference happens when one object refers to another object that refers back to it, creating a loop.
 

**3 View types**

The three view types

Types:

 view groups objects by their constructor function and gives a count of each type. This makes it easier to know how many arrays you have, how many strings, and so on. Those groups can be expanded to view individual objects.

Roots:

 view shows the major root objects that you can expand to see the child objects associated with them.

Dominators:

 view breaks out all individual HTML elements, DOM nodes, and JavaScript objects your code creates. This is the most detailed of the views.


#### Snapshot comparision view

Here we are comparing Snapshot #3 with Snapshot #2:

![Snapshot comparision view](https://docs.microsoft.com/en-us/microsoft-edge/f12-devtools-guide/media/edge_memory_compare.png)



## Octane 2.0 rating for the browsers


#### Chrome 60.0.3112.90
![Firefox 54.0.1](./img/octane2-score-chrome-60.0.3112.90.png)

#### Safari 10.1.2
![Firefox 54.0.1](./img/octane2-score-safari-10.1.2.png)

#### Firefox 54.0.1
![Firefox 54.0.1](./img/octane2-firefox-54.0.1.png)


### Comparsion chart
![Comparsion chart](https://adtmag.com/blogs/dev-watch/2016/06/~/media/ECG/adtmag/Images/2016/06/digitaltrends_javascript.png)

### Google Chrome V8

The V8 engine has an interpreter named **Ignition**. 

This interpreter is used for **interpreting and executing low level bytecode**. Bytecodes, although slower, are smaller than machine codes and requires **lesser** compilation time.


[v8-Ignition-Design-Doc.pdf](./doc/v8-Ignition-Design-Doc.pdf)

[V8-Ignition-online-doc](https://docs.google.com/document/d/11T2CRex9hXxoJwbYqVQ32yIPMh0uouUZLdyrtmMoL44/edit?ts=56f27d9d#heading=h.6jz9dj3bnr8t)


In order to compile Javascript to bytecode, the Javascript code has to be parsed to generate its **Abstract Syntax Tree** (AST). 

The interpreter has an accumulator register, which allows it reduce the size of the bytecode. The overall design makes Ignition a highly efficient interpreter.


Optimizing JIT compiler - **TurboFan** is better than CranShaft JIT:

TurboFan combines a cutting-edge intermediate representation with a multi-layered translation and optimization pipeline to **generate better quality machine code** than what was previously possible with the **CrankShaft JIT**. 

Optimizations in TurboFan are more numerous, more sophisticated, and more thoroughly applied than in CrankShaft, enabling fluid code motion, control flow optimizations, and precise numerical range analysis, all of which were more previously unattainable.

TurboFan is developed  with a layered architecture to allow the compiler to cope with new demands (due to new features) over time. 

A clearer separation between the source-level language (JavaScript), the VM's capabilities (V8), and the architecture's intricacies (from x86 to ARM to MIPS - around 7 target architectures) allows for cleaner and more robust code.

 Layering allows those working on the compiler to reason locally when implementing optimizations and features, as well as write more effective unit tests.


Each of the 7 target architectures supported by TurboFan requires fewer than 3,000 lines of platform-specific code, versus 13,000-16,000 in CrankShaft. 

This enabled engineers at ARM, Intel, MIPS, and IBM to contribute to TurboFan in a much more effective way.

 TurboFan is able to more easily support all of the coming features of ES6 because its flexible design separates the JavaScript frontend from the architecture-dependent backends.
 
 JavaScript enters the compiler pipeline in a mostly unoptimized form and is translated and optimized to progressively lower forms until machine code is generated. 
 
 The centerpiece of the design is a more relaxed sea-of-nodes internal representation (IR) of the code which allows more effective reordering and optimization.
 
 ![Example of Turbofan graph](https://lh6.googleusercontent.com/WuIFuWDNgOoc1Ag8RA6gQnL2gpr3pPRJbqnCurYSbjlvJMmBVmRWgi4iqVtz2t3Mvzly3d70SYWxsZr2mYNayO0c8EIa3GlyZmhFOcdI131nOvuanXdEoW5s_leuhEP-jDfaV4s)
 
 
 The graph-based IR allows most optimizations to be expressed as simple **local** reductions which are easier to write and test independently. 
 
 An optimization engine applies these local rules in a systematic and thorough way. 
 
 Transitioning out of the graphical representation involves an innovative scheduling algorithm that makes use of the reordering freedom to move code out of loops and into less frequently executed paths. 
 
 Finally, architecture-specific optimizations like complex instruction selection exploit features of each **target** platform for the best quality code.
 



[Digging into TurboFan](https://v8project.blogspot.de/2015/07/digging-into-turbofan-jit.html)

[Ref](https://developers.redhat.com/blog/2016/05/31/javascript-engine-performance-comparison-v8-charkra-chakra-core-2/)


**JS to AST to bytecode**:


TurboFan will profile the code and see if it is used **multiple times** throughout the entire Javascript execution. If it is, the code will be **dynamically optimized** immediately into machine code, without any intermediate binary code. 

If it is a one-time executed “non-hot” code, it will only be compiled into binary code.




### Garbage Collection:


Garbage collection is also done in a **stop-the-world**, generational way. This means that before the JavaScript engine does garbage collection, all processing of JavaScript **will be paused** and the garbage collector will find objects and data that are **no longer referenced** and collect them. This ensures that garbage collection is done in an accurate and efficient way


### SpiderMonkey


SpiderMonkey is Mozilla's JavaScript engine written in C and C++. It is used in various Mozilla products, including Firefox, and is available under the MPL2.

[SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey)










