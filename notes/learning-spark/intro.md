# Spark Main Concepts
## Spark Architecture

The underlying system that manages all the computation is called **Spark Core**. Spark Core defines the RDDs, the main programming abstraction of Spark.

## Programming Concepts
Every program has a driver, that submits tasks to the cluster. Each task executes on a particular partition of the RDD.
Driver accesses Spark through a `SparkContext`, which represents a connection to a computer cluster. In the shell, `SparkContext` is created under the variable `sc`.
To run these tasks, the driver uses multiple executors.

Like scalding, Spark runs stuff like map and filter in parallel on the cluster (under the hood). This will most likely be a pain in the ass to debug.
