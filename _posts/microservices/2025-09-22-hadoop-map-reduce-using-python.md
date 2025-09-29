---
title:  "Hadoop : Map/Reduce Using Python"
teaser: "Get started with big data processing using this hands-on tutorial, which guides you through running Hadoop and Python MapReduce jobs in a fully containerized environment. Hosted on GitHub, this repository provides a ready-to-use docker-compose.yml file to spin up a Hadoop cluster, along with sample data and Python scripts (mapper.py and reducer.py) for your first MapReduce workflow"
tags:
    - apache
    - docker
    - solution architect
    - hive
categories:
    - microservcies
math: true
comments: true
author: kanmeugne
---

Hadoop is an affordable, reliable and scalable platform for big data storage and analysis -- it runs on commodity hardware and it is open source. Technically speaking, the Hadoop platform is the answer to the unevitable question we face one day or another as we live in a data age -- which is : _how do we process tons of data efficiently ?_ It is not just about storage, but also, and even more, about implementing data processing models that can provide insights to decision makers in a competitive world -- where everything has to be fast and resilient.

There are many important concepts to know in order to [understand the hadoop framework][1] -- in this tutorial we will focus on 2 of them (the interested reader can check [this article][2] for an extensive review of hadoop concepts) : 
- **Map/Reduce model** : a programming model for data processing, inherently parallel, thus putting very large-scale data analysis into the hands of anyone with enough machines at their disposal. Map/Reduce program can be written in several popular languages -- Java, Python, Ruby etc. -- or wrapped using distributed tools, like [Apache Hive][1], built on top of the hadoop platform.
- **HDFS** : Hadoop comes with a distributed filesystem called HDFS, which stands for Hadoop Distributed Filesystem. _HDFS is a filesystem designed for storing very large files with streaming data access patterns, running on clusters of commodity hardware_

![alt text](/images/hive-hadoop.png){: width="50%" }
_Kanmeugne's Blog -- Hadoop : Map/Reduce Using Python_

In this tutorial, you'll learn how to process data in HDFS using Python, and then use Hive to query your results directly from the Hadoop cluster. Whether you're new to Hadoop or looking to experiment with distributed data processing and analytics, this tutorial offers a practical, reproducible starting point right from your local machine.

Ready to dive in? Just clone the repo, follow the step-by-step instructions, and start exploring big data with Hadoop, MapReduce, and Hive --- all powered by Docker.

## How to

### Build the application

The full code for this tutorial is available [from github][3], you just have to pull and run :

- **Clone and deploy the Hadoop Docker setup:**

```shell
  $ git clone https://github.com/kanmeugne/modern-data-architectures.git
  $ cd modern-data-architectures/handzon-hadoop-python-map-reduce 
  $ docker-compose up -d
  # [+] Building 0.0s ...
  # ...
```
This launches namenodes, datanodes, and supporting services in containers. It also creates a hive server, to create and explore an hdfs-compatible database.

### Add some data in the hdfs server

Let's add some data in the distributed filesystem:

- _Copy your CSV file into the namenode container_:

```bash
$ curl -L -o movieratings.csv https://files.grouplens.org/datasets/movielens/ml-100k/u.data
$ docker cp movieratings.csv <namenode-container>:/tmp/ # on the docker host
```
> The [dataset](https://grouplens.org/datasets/movielens/100k/ "MovieLens data sets were collected by the GroupLens Research Project at the University of Minnesota.") comes from [GroupLens](https://grouplens.org/about/what-is-grouplens/), a research lab in the Department of Computer Science and Engineering at the University of Minnesota, Twin Cities specializing in recommender systems, online communities, mobile and ubiquitous technologies, digital libraries, and local geographic information systems.

- _Load the CSV into an HDFS folder within the container:_

```bash
$ docker exec <namenode-container> hdfs dfs -mkdir -p /input
$ docker exec <namenode-container> hdfs dfs -put /tmp/movieratings.csv /input/ # in the docker
```

### Test the mapper and reducer functions on the host

It is possible to test the python scripts using the console _pipe_. 

```bash
$ cat movieratings.csv | python mapper.py | python reducer.py
...
708     4.0
566     4.0
1010    4.0
50      5.0
134     5.0
...
```

- _Copy the `mapper.py` and `reducer.py` files in the namenode container_
```bash
$ docker cp mapper.py <namenode-container>:/tmp/
$ docker cp reducer.py <namenode-container>:/tmp/
```
> 
> - _Mapper_ (`mapper.py`):
  ```python
  #!/usr/bin/env python3
  import sys
  for line in sys.stdin:
    _, movie_id, rating, _ = line.strip().split('\t')
    print(movie_id+'\t'+rating)
  ```
  *Explanation:* For each line, output the movie ID as key and the rating as value.
> - _Reducer_ (`reducer.py`):
  ```python
  #!/usr/bin/env python3
  import sys
  current_movie = None
  ratings = []
  for line in sys.stdin:
      movie_id, rating = line.strip().split('\t')
      if current_movie and movie_id != current_movie:
          print(current_movie+'\t'+str(round(sum(ratings)/len(ratings), 2)))
          ratings = []
      current_movie = movie_id
      ratings.append(float(rating))
  if current_movie:
      print(current_movie+'\t'+str(round(sum(ratings)/len(ratings), 2)))
  ```
  *Explanation:* For each movie, compute and output the average of its ratings.

### Run the MapReduce Job

Now that we have tested the map/reduce python script on the terminal, we can now run the scripts on the hadoop nodes. 

```bash
  docker exec <namenode-container> \
   hadoop jar /opt/hadoop-3.2.1/share/hadoop/tools/lib/hadoop-streaming-3.2.1.jar \
  -file /tmp/mapper.py -mapper /tmp/mapper.py \
  -file /tmp/reducer.py -reducer /tmp/reducer.py \
  -input /input/movieratings.csv \
  -output /output
  
  # 2025-05-05 22:48:57,076 WARN streaming.StreamJob...
  # ...
  # 2025-05-05 22:49:05,815 INFO mapreduce.Job: Job job...
  # 2025-05-05 22:49:05,816 INFO mapreduce.Job:  map 0% reduce 0%
  # 2025-05-05 22:49:11,892 INFO mapreduce.Job:  map 50% reduce 0%
  # 2025-05-05 22:49:12,901 INFO mapreduce.Job:  map 100% reduce 0%
  # 2025-05-05 22:49:16,927 INFO mapreduce.Job:  map 100% reduce 100%
  # 2025-05-05 22:49:16,936 INFO mapreduce.Job: Job .... completed successfully
  # 2025-05-05 22:49:17,032 INFO mapreduce.Job: Counters: 54
  # ...
  # 2025-05-05 22:49:17,032 INFO streaming.StreamJob: Output directory: /output
```

### Use a hive server

You can query the data from a hive server that runs on the hadoop cluster nodes :

- _Start the Hive server (from any cluster node)_:

```bash
  $ docker exec -it <hive-server-container> bash 
  root@xxx:/opt/ beeline -u jdbc:hive2://localhost:10000
  # SLF4J: Class path contains multiple SLF4J bindings.
  # SLF4J: Found binding in ...
  # SLF4J: Found binding in ...
  # SLF4J: See ...
  # SLF4J: Actual binding is of type ...
  # Connecting to jdbc:hive2://localhost:10000
  # Connected to: Apache Hive (version 2.3.2)
  # ...
  # Beeline version 2.3.2 by Apache Hive
  0: jdbc:hive2://localhost:10000>
```

- _Create an external table for results_:

```sql
  beeline> 
  CREATE EXTERNAL TABLE movie_avg_rating (
    movie_id STRING,
    avg_rating FLOAT
  )
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t'
  LOCATION '/output';
```

- _Query results_:

```sql
  SELECT * FROM movie_avg_rating ORDER BY avg_rating DESC LIMIT 4;
```
```verbatim
+----------+-----------+---------+-----------+
| user_id  | movie_id  | rating  | datation  |
+----------+-----------+---------+-----------+
| 196      | 242       | 3.0     | 881250949 |
| 186      | 302       | 3.0     | 891717742 |
| 305      | 451       | 3.0     | 886324817 |
| 6        | 86        | 3.0     | 883603013 |
+----------+-----------+---------+-----------+
```
## Compare job durations as you increase the number of DataNodes.

You can increase the number of nodes and confirm the following speed-ups.

| Nodes | Example Time (s) | Notes                    |
|-------|------------------|--------------------------|
| 1     | 120              | Single DataNode          |
| 2     | 75               | Parallel processing      |
| 3     | 55               | Further speedup          |

## Conclusion

Thank you for your attention. Feel free to share this tutorial and to send your comments. 

[1]: https://www.oreilly.com/library/view/hadoop-the-definitive/9781491901687/ "Hadoop: The Definitive Guide, 4th Edition, O'Reilly Media, Inc."
[2]: /posts/hadoop-and-hive/ "Kanmeugne's Blog : Hadoop - query data from a self-hosted hadoop cluster"
[3]: https://github.com/kanmeugne/modern-data-architectures.git "Kanmeugne's Blog (github) : Tutorials on modern data architectures"