<!-- 
 (C) Copyright IBM Corp. 2015 - 2017

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Spark-Bench Built-In Workloads](#spark-bench-built-in-workloads)
  - [Common Parameters For All Workloads](#common-parameters-for-all-workloads)
  - [Data Generators](#data-generators)
  - [Common Parameters For All Data Generator Workloads](#common-parameters-for-all-data-generator-workloads)
    - [Kmeans Data Generator](#kmeans-data-generator)
    - [Linear Regression Data Generator](#linear-regression-data-generator)
    - [Graph Data Generator](#graph-data-generator)
  - [Exercise Workloads](#exercise-workloads)
    - [SparkPi](#sparkpi)
    - [Sleep](#sleep)
  - [Machine Learning and Statistics Workloads](#machine-learning-and-statistics-workloads)
    - [KMeans](#kmeans)
    - [LogisticRegression](#logisticregression)
  - [SQL](#sql)
    - [SQL Workload](#sql-workload)
- [Bring Your Own Workload](#bring-your-own-workload)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Spark-Bench Built-In Workloads

## Common Parameters For All Workloads

| Name        | Required (y/n)| Description  |
| ------------- |-------------| -----|
| name       | yes | The name/type of workload. For example, "kmeans", "sparkpi", "logisticregression"... |
| input      | yes, for some | The path (local, hdfs, S3, etc.) to the input dataset for the workload. Some workloads (ex: SparkPi) do not require input. |
| output | yes for data generators, otherwise no | If the user wishes to keep the output of the workload (ex: the value of pi generated by SparkPi), they can specify a path here. |

Remember, the benchmark output (e.g. how long it takes to run SparkPi) is not the same as the workload results (e.g. what is the approximate value of pi).
The benchmark output is collected from each run of each workload and output to a path specified by the suite.

## Data Generators

In spark-bench, data generators are just a different type of workload! 
The key difference is that `output` is required for all data generators. 
Otherwise, where would all that generated data go?

## Common Parameters For All Data Generator Workloads

| Name        | Required (y/n)| Description  |
| ------------- |-------------| -----|
| name       | yes | The name/type of workload. For example, "kmeans", "sparkpi", "logisticregression"... |
| rows      | yes, for most | number of rows to generate |
| cols     | yes, for most | number of columns to generate |
| output   | yes | If the user wishes to keep the output of the workload (ex: the value of pi generated by SparkPi), they can specify a path here. |


### Kmeans Data Generator

| Name    | Required (y/n)| Default  | Description |
| ------- |---------------| ---------| ------------|
| k      | no | 2 | number of clusters generated |
| scaling | no | 0.6 | scaling factor of the the dataset|
| partitions | no| 2 | number of partitions|

### Linear Regression Data Generator

| Name    | Required (y/n)| Default  |
| ------- |---------------| ---------|
| eps      | no | 2 |
| intercepts | no | 0.1 |
| partitions | no| 10 |

### Graph Data Generator

The graph data generator uses the GraphX logNormalGraph() generator.
This allows users to specify a certain number of vertices for the graph,
and the generator will generate edges between them according to the specified
distribution parameters.

From the [documentation](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.util.GraphGenerators$)

```text
Generate a graph whose vertex out degree distribution is log normal.

The default values for mu and sigma are taken from the Pregel paper:

Grzegorz Malewicz, Matthew H. Austern, Aart J.C Bik, James C. Dehnert, Ilan Horn, Naty Leiser, and Grzegorz Czajkowski. 2010. Pregel: a system for large-scale graph processing. SIGMOD '10.

If the seed is -1 (default), a random seed is chosen. Otherwise, use the user-specified seed.
```

**NOTE:** The Spark GraphLoader can only accept input in a very specific format.
Due to this limitation, the Graph Data Generator can only output to `.txt`.
All other output file formats will cause an error.

**Parameters**
| Name    | Required (y/n)| Default  | Description |
| ------- |---------------| ---------| ----------- |
| vertices      | yes | -- | Number of vertices in the graph |  
| mu | no | 4.0 | mean of out-degree distribution |  
| sigma | no| 1.3 | standard deviation of out-degree distribution |  
| seed | no | -1 | seed for random number generators, -1 causes a random seed to be chosen |  
| partitions | no | 0 | number of partitions |  

**Examples**
```hocon
// Generate a graph with 1,000,000 vertices using the default out degree parameters.
{
  name = "graph-data-generator"
  vertices = 1000000
  output = "hdfs:///one-million-vertex-graph.txt"
}
```
```hocon
// Generate a graph with 1,000,000 vertices but specify a random seed.
{
  name = "graph-data-generator"
  vertices = 1000000
  output = "hdfs:///one-million-vertex-graph.txt"
  seed = 104623
}
```

## Exercise Workloads

### SparkPi

Borrowing from the classic Spark example, this workload computes an approximation of pi.
From the Spark examples page: <https://spark.apache.org/examples.html>
```
Spark can also be used for compute-intensive tasks. This code estimates π
by "throwing darts" at a circle. We pick random points in the unit square
((0, 0) to (1,1)) and see how many fall in the unit circle. The fraction
should be π / 4, so we use this to get our estimate.
```

**Parameters**

| Name        | Required (y/n)| Default  | Description |
| ----------- |---------------| ---------| ------------|
| slices      | no | 2 | Number of partitions that will be spawned |


### Sleep

Sleeps the thread. Can be configured to sleep for
a specified number of milliseconds,
or for a random-not-to-exceed number of milliseconds,
or for a random  number of milliseconds less than 3600000L (one hour).

**Parameters**

The Sleep workload can work in two different ways. 

1. Specify a specific number of milliseconds for the thread to sleep
2. Draw from a distribution. Available distributions are uniform random, gaussian, and Poisson

_To Specify A Specific Number Of Milliseconds_  

| Name        | Required (y/n)| Default  | Description |
| ----------- |---------------| ---------| ------------|
| sleepMS     | no |  | specific number of milliseconds the thread should sleep |

_To Draw From A Distribution_

| Name        | Required (y/n)| Default  | Description |
| ----------- |---------------| ---------| ------------|
| distribution     | yes | -- | "uniform", "gaussian", or "poisson" |
| min     | no, optional for uniform | 0 | Minimum value of the uniform distribution |
| max     | yes for uniform | -- | Maximum number of milliseconds. Uniform distribution will choose a number \[(min or 0), max\] |
| mean    | yes for gaussian and poisson | -- | Mean of the gaussian or Poisson distribution |
| std     | yes for gaussian | -- | Standard deviation for the gaussian distribution |

**Examples**
```hocon
// Sleep the thread for exactly 60000 milliseconds
{
  name = "sleep"
  sleepMS = 60000
}
```
```hocon
// Sleep the thread for a number of milliseconds [0, max] chosen from a uniform distribution
{
  name = "sleep"
  distribution = "uniform"
  max = 60000
}
```
```hocon
// Sleep the thread for a number of milliseconds chosen from a Poisson distribution with a mean of 30000
{
  name = "sleep"
  distribution = "poisson"
  mean = 30000
}
```
```hocon
// Sleep the thread for a number of milliseconds chosen from a gaussian distribution with a mean of 30000 and standard deviation of 1000
{
  name = "sleep"
  distribution = "gaussian"
  mean = 30000
  std = 1000
}
```

## Machine Learning and Statistics Workloads

### KMeans

Runs the KMeans algorithm over the input dataset. `input` from the common parameters is required.

**Parameters**

| Name        | Required (y/n)| Default  | Description |
| ----------- |---------------| ---------| ------------|
| k     | no | 2 | number of clusters |
| seed  | no | 127L | initial values |
| maxiterations  | no | 2 | maximum number of times the algorithm should iterate |

### LogisticRegression

Runs LogisticRegression over the input datasets. `input` from the common parameters is required.

`input` in this case is the training dataset. The test dataset is specified by `testfile`.

**Parameters**

| Name        | Required (y/n)| Default  | Description |
| ----------- |---------------| ---------| ------------|
| testfile       | yes | --    | testing dataset |
| numpartitions  | no  | 32    | number of partitions |
| cacheenabled   | no  | false | whether or not the datasets are cached after being read from disk |


## SQL

### SQL Workload
Runs a SQL query over the input dataset. `input` from the common parameters is required.

The query string is required to use "input" as the name of the table. For example:
```sql
select * from input where SomeNumericField < 15
```

**Parameters**

| Name        | Required (y/n)| Default  | Description |
| ----------- |---------------| ---------| ------------|
| queryStr     | yes          | --    | the sql query to perform |
| cache        | no           | false | whether the dataset should be cached after being read from disk |

# Bring Your Own Workload

You can dynamically load a jar with your implementation of Workload.scala.

More documentation forthcoming.