# JVM

## Extra Options for Java

option                         | equivalent                     | detail
------------------------------ | ------------------------------ | -------------------------------------------------------------
-Xms size                      | -XX:InitalHeapSize             | Sets the minimum and the initial size (in bytes) of the heap.
-Xmx size                      | -XX:MaxHeapSize                | Specifies the maximum size (in bytes) of the heap.
-Xmn size                      | (-XX:NewSize + -XX:MaxNewSize) | Sets the initial and maximum size (in bytes) of the heap for the young generation (nursery) in the generational collectors.
-XX:InitialSurvivorRatio=ratio |                                | Sets the initial survivor space ratio used by the throughput garbage collector.
-XX:+UseAdaptiveSizePolicy     |                                | Enables the use of adaptive generation sizing.
-XX:SurvivorRatio=ratio        |                                | Sets the ratio between eden space size and survivor space size.
-XX:MaxTenuringThreshold=threshold |                            | Sets the maximum tenuring threshold for use in adaptive GC sizing.
-Xlog:gc+age*=level
-Xlog:gc*
-XX:+ScavengeBeforeFullGC        |                              | Enables GC of the young generation before each full GC.