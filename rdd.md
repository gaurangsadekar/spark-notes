# Resilient Distributed Datasets (RDDs)
An RDD is an immutable distributed collection of objects. Each RDD is split into partitions, which can be computed on different nodes of the cluster. RDDs can contain any type of objects, including custom classes.

Everything in Spark is either creating, transforming or calling operations on an RDD.

RDDs are created by loading a dataset or distributing a collection of objects.

RDDs can be transformed or operated on by actions. RDDs are lazy, they are only evaluated if an action is called on them, and they are only evaluated enough to satisfy the action. All transformations are chained up until an action is called.

RDDs are recomputed by default each time an action is run on them. All transformations queued up so far are run every time an action is called.
To reuse RDDs in multiple actions, call persist on them. Spark persists RDDs in memory, can be done on disk too.
In practice use persist to load a subset of data into memory and query it repeatedly.

#### Difference between Transformation and Action:
Transformation returns an RDD type, action returns some other data type.
Each transformation returns a reference to a new RDD. The original RDD can still be reused in the program for some other operation.
