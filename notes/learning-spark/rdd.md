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

Return a final value to the driver program or write data to external storage. Actions force evaluation of transformations on the RDD. RDDs also have a `collect` action to retrieve the entire RDD into memory. The entire dataset must fit in memory on a single machine to use `collect`. All RDDs are lazily evaluated. In Spark there is no difference between writing a single complex map or chaining together many small maps.
Using methods and `vals` from a class requires sending the entire instance of the class to the cluster. To avoid sending large classes, extract the required fields into local variables.

### Understanding Closures
Deals with scope and life cycle of variables and methods that are outside the scope of the RDD operation.

Spark breaks up processing of RDD operations into tasks which are executed by an executor. Spark computes each task's **closure**, which is those variables and methods which must be visible to the executor to perform its computations on the RDD. The closure is serialized and sent to each executor. If there are variables in the closure which are outside the scope of the operation, they are copied over and serialized while sending to each executor. Closures should not be used to mutate global state.
Moral of the story: Keep operations as purely functional as possible. Side effects are bad.
If operating on global state, use `Accumulators` instead.

To deal with the entire RDD on one node, use `collect` to bring it to the driver and then execute side effecting code.

## Working with Key Value Pairs:
Similar to Scalding, special operations like grouping and aggregation on RDDs of `[K, V]` type.

When using custom objects as keys in the KV pair operations, we require a custom `equals` and `hashCode` method (for serializing when computing closures)

`foreach` is an action, not a transformation because it doesn't return a reference to an RDD.

## Shuffle Operations
Shuffle means redistributing data to group differently across partitions. Shuffle is an all-to-all operation. Requires reading in all the partitions of the RDD and repartitioning the data.

### Performance Impact:
This is a very expensive operation, avoid unless absolutely necessary.
It involves disk I/O, data serialization and network I/O.

## Persistence:
When you persist or cache an RDD, each node stores its partitions of the RDD in memory and reuses them in other actions. This makes future actions much faster. RDDs can be persisted at different storage levels, configured by the `StorageLevel` object passed to persist. If RDDs fit easily in memory, use the default level `MEMORY_ONLY`. For space efficiency, use `MEMORY_ONLY_SER` and use a faster serialization library.

## Shared Variables:
Since Spark uses closures, variables are local to each executor by default.

### Broadcast Variables:
Broadcast variables allow you to keep a readonly variable cached on each machine rather than shipping a copy to each task.
Explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in deserialized form is important. Then reference the broadcast variable in any functions that run on the cluster.

### Accumulators:
Can be used to store counts across the cluster. They can be added to by associative and commutative operations. Only the driver can read the accumulator's value.


