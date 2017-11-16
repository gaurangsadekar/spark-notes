# Resilient Distributed Datasets (RDDs)
An RDD is an immutable distributed collection of objects. Each RDD is split into partitions, which can be computed on different nodes of the cluster. RDDs can contain any type of objects, including custom classes.

Everything in Spark is either creating, transforming or calling operations on an RDD.

RDDs are created by loading a dataset or distributing a collection of objects.
Datasets can be loaded from directories, compressed files and wildcards as well.

RDDs can be transformed or operated on by actions. RDDs are lazy, they are only evaluated if an action is called on them, and they are only evaluated enough to satisfy the action. All transformations are chained up until an action is called.

RDDs are recomputed by default each time an action is run on them. All transformations queued up so far are run every time an action is called.
To reuse RDDs in multiple actions, call `persist` on them. Spark persists RDDs in memory, can be done on disk too.
In practice use persist to load a subset of data into memory and query it repeatedly.

#### Difference between Transformation and Action:
Transformation returns an RDD type, action returns some other data type.

#### Transformations:
Each transformation returns a reference to a new RDD. The original RDD can still be reused in the program for some other operation. Transformations are computed lazily, only when RDD is used in actions.

Spark creates an RDD lineage graph to track dependencies between RDDs, to compute on demand.

#### Actions:
Return a final value to the driver program or write data to external storage. Actions force evaluation of transformations on the RDD. RDDs also have a `collect` action to retrieve the entire RDD into memory. The entire dataset must fit in memory on a single machine to use `collect`. All RDDs are lazily evaluated. In Spark there is no difference between writing a single complex map or chaining together many small maps.

Using methods and `vals` from a class requires sending the entire instance of the class to the cluster. To avoid sending large classes, extract the required fields into local variables.

### Understanding Closures
Deals with scope and life cycle of variables and methods that are outside the scope of the RDD operation.

Spark breaks up processing of RDD operations into tasks which are executed by an executor. Spark computes each task's **closure**, which is those variables and methods which must be visible to the executor to perform its computations on the RDD. The closure is serialized and sent to each executor. If there are variables in the closure which are outside the scope of the operation, they are copied over and serialized while sending to each executor. Closures should not be used to mutate global state.
Moral of the story: Keep operations as purely functional as possible. Side effects are bad.
If operating on global state, use `Accumulators` instead.

To deal with the entire RDD on one node, use `collect` to bring it to the driver and then execute side effecting code.
