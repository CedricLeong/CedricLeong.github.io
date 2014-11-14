---
published: false
---

## Simple Example of Method-Level Synchronization

A significant concern in distributed systems is conflicts that might arise from unsynchronized
requests to a single resource. A very simple example of this concern can be demonstrated by a
class that increments a counter through a method. If this class is instantiated by another class
that supports multi-threading, and increments the counter N times within its run() method one
can demonstrate that when multiple threads of this class are created there are situations when
the value of the total count is not what is expected. For example, let’s say that the multithreaded
counter object increments count 10 times and 3 threads of this object type are
launched. The expected value of count is 30 (10*3) but this will not be the case if the 3 threads
are not synchronized.
For this question create a Counter class with 2 methods increaseCount() and getCount() that correspondingly increase the value of a counter by 1 and gets the value of the counter. Now define a CountingThread class that instantiates a Counter object and implements the Runnable interface (i.e. supports multi-threading). In the run() method of CountingThread call increaseCount() several times. In the main method of CountingThread instantiate 3 threads of CountingThread. When these threads end get the value of the counter using getCount() and print out the result. Note: you might have to put the Counter object to sleep for a few milliseconds in the increaseCount() method to get significant synchronization issues. Code this and show that you have issues when you do not use method-level synchronization. Add the method-level synchronization to the code so that the expected counter sum is achieved.