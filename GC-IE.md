### Garbage collection in IE

[Ref](https://blogs.msdn.microsoft.com/ericlippert/2003/09/17/how-do-the-script-garbage-collectors-work/)

[Ref](http://milan.adamovsky.com/2012/02/javascript-memory-leaks-in-internet.html)

[Ref](https://msdn.microsoft.com/library/bb250448(v=vs.85).aspx)

[Ref](https://msdn.microsoft.com/en-us/library/bb250448.aspx)

[Ref](https://medium.com/@garthw/how-to-work-around-ie-memory-leaks-in-spas-b2f6bc7c9ae9)




Background:

JScript and VBScript both are automatic storage languages. Unlike, say, C++, the script developer does not have to worry about explicitly allocating and freeing each chunk of memory used by the program. The internal device in the engine which takes care of this task for the developer is called the garbage collector. This is very similar to Java Garbage Collection (GC):


Java GC is explained [here](java-GC.md)


JScript uses a nongenerational mark-and-sweep garbage collector. It works like this:

Scav list:

Every variable which is "in scope" is called a "scavenger".

 A scavenger may refer to a number, an object, a string, whatever. JScript engine maintains a list of scavengers -- variables are moved on to the scav list when they come into scope and off the scav list when they go out of scope.


Every now and then the garbage collector(GC) runs.

 First it puts a "mark" on every object, variable, string, etc – all the memory tracked by the GC. 

JScript uses the VARIANT data structure internally and there are plenty of extra unused bits in that structure, so we just set one of them.

Engine clears the mark on the scavengers and the transitive closure of scavenger references. So if a scavenger object references a nonscavenger object then we clear the bits on the nonscavenger, and on everything that it refers to.

At this point we know that all the memory still marked is allocated memory which cannot be reached by any path from any in-scope variable. All of those objects are instructed to tear themselves down, which destroys any circular references.


Manually forcing GC:

You can force the JScript garbage collector to run with the ```CollectGarbage()``` method,


Leaks:


When here is a reference of any kind created (such as an event handler) between the two engines, and let us say one of your Javascript function scope exits, there may still be a reference to a symbol within that function that is referenced in the DOM (e.g. via an event handler such as onload). This in turn confuses JScript garbage collector, which in turn causes for it not to clean up, which then in turn causes the memory leak. 


How to Help GC:
 We can help the garbage collector in JScript by marking the symbols with a **null** value that you want to be garbage collected.


Example to write leak free code:

```

 function createPerson(firstName, lastName) {
   var date = new Date(), counter = 0, registry = {}, output;

   // function code work here

   // cleanup time
   // locals cleanup after we produced output
   // The strategy is to dispose of as many symbols as possible

   date = counter = registry = null;

   // the return statement cleans up whatever symbol it returns 
   return output;
  }



```


### DOM references 

This is case of we are  crossing from one engine into the other:

```

function getElements(id) {
   // DOM method: getElementById(...). We are crossing engines

   var element = document.getElementById(id);
   var  i ;  
   // get all the div tags of tagLength in size 
   var  tags = document.getElementsByTagName("div"); 
   var tagsLength = tags.length;

   //  clear all references that you hold to a DOM element
   //  this is the main source for the memory leaks
   for (i = 0; i < tagsLength; i++) {
     // code goes here
     
     // help GC 
     tags[i] = null;
    }

   // help GC: clean up other locals
   element = i = tags = null;
}

``` 

How about Event Handlers:

Event handler is a link between the DOM engine and the JavaScript engine since the element references a JavaScript function

Solution:

maintain a registry of all event handlers (say: eventRegistry) that you've attached so that you can eventually detach them in window.unload()


```
//Pay particular attention to event subscription and DOM references, 
// and ensuring that each component has a working teardown operation that releases all DOM references. 

window.unload = function () {
  // clean up event by setting event registry items to null 

  // self
  window.unload = null;

   // force GC -  IE-specific function
   // it does not guarantee that it will be done on demand.
   //  we are merely asking for it to be done sooner than later.
   CollectGarbage();
 }


```

### Observations Single Page Applications (SPA):

An IE process functions normally until about 1.6GB RAM.

Somewhere between 1.6GB and 1.8GB the browser will recycle the process, which is almost transparent to the user, dropping RAM back down to 200MB.

The process can only be transparently recycled when the user navigates from one page to another or reloads the page (which may never happen in an SPA).

If the web app causes IE to consume RAM in a single operation so that it starts at < 1.6GB and completes at > 1.8GB the process will terminate without warning.



















