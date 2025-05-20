---
title:  "Hadoop : query data from a self-hosted hadoop cluster"
teaser: "Hadoop is a very popular technology when it comes to big data storage and analysis. In this tutorial, we will set up a working hadoop cluster and use Hive -- a data exploration tool that can query and transform data stored in a hadoop cluster"
tags:
    - apache
    - superset
    - postgresql
    - docker
    - solution architect
    - hive
categories:
    - microservcies
math: true
comments: true
author: kanmeugne
---

Hadoop is an affordable, reliable and scalable platform for big data storage storage and analysis -- it runs on commodity hardware and it is open source. Technically speaking, the Hadoop platform is the answer to the unevitable question we face one day or another as we live in a data age -- which is : _how do we process tons of data efficiently ?_. It is not just about storage, but also, and even more, about implementing data processing models that can provide insights to decision makers in a competitive world -- where everything has to be fast and resilient.

![alt text](/images/hive.png){: width="50%" }
_Kanmeugne's Blog -- Hadoop : query data from a self-hosted hadoop cluster_

There are many important concepts to know in order to [understand the hadoop framework][1] -- in this tutorial we will focus on 3 of them : 
- **Map/Reduce model** : a programming model for data processing, inherently parallel, thus putting very large-scale data analysis into the hands of anyone with enough machines at their disposal. Map/Reduce program can be written in several popular languages -- Java, Python, Ruby etc. -- or wrapped using distributed tools, like [Apache Hive][2], built on top of the hadoop platform.
- **HDFS** : Hadoop comes with a distributed filesystem called HDFS, which stands for Hadoop Distributed Filesystem. _HDFS is a filesystem designed for storing very large files with streaming data access patterns, running on clusters of commodity hardware_
- **Apache YARN** (Yet Another Resource Negotiator) is Hadoop’s cluster resource management system. _YARN provides APIs for requesting and working with cluster resources, but these APIs are not typically used directly by user code. Instead, users write to higher-level APIs provided by distributed computing frameworks, which themselves are built on YARN and hide the resource management details from the user_

To learn more about hadoop platform, the interested reader could have a look at [this excellent book][1], published by Oreilly. 

## How To

Now let's jump to the hands on tutorial. I will mostly focus on high level operations -- data i/o and analysis -- for the interested users could easily find more specific tutorials on low-level operations. Here, we will rapidly set up a custom cluster using docker compose, add some data in the corresponding **HDFS** filesystem, and process the data using [**Hive**][2].



- **Clone and deploy the Hadoop Docker setup:**  
  ```bash
  $ git clone https://github.com/kanmeugne/modern-data-architectures.git
  $ cd modern-data-architectures/handzon-hadoop-hive 
  $ docker-compose up -d
  ```
  This launches namenodes, datanodes, and supporting services in containers. It also creates a hive server, to create and query data in a hdfs-compatible database.

- **Check running containers:**  
  ```bash
  $ docker ps
  ```
  All Hadoop containers (namenode, datanode(s), etc.) should be listed.
  
- **Check hdfs filesystem from inside the name node:**  
  ```bash
  $ docker exec <namenode> hdfs dfsadmin -report 
  # this command lists all live DataNodes connected to the cluster.
  Configured Capacity: *** (*** GB)
  Present Capacity: *** (*** GB)
  DFS Remaining: *** (*** GB)
  DFS Used: *** (*** MB)
  DFS Used%: 0.01%
  Replicated Blocks:
    Under replicated blocks: 6
    Blocks with corrupt replicas: 0
    Missing blocks: 0
    Missing blocks (with replication factor 1): 0
    Low redundancy blocks with highest priority to recover: 6
    Pending deletion blocks: 0
  Erasure Coded Block Groups: 
    Low redundancy block groups: 0
    Block groups with corrupt internal blocks: 0
    Missing block groups: 0
    Low redundancy blocks with highest priority to recover: 0
    Pending deletion blocks: 0
  -------------------------------------------------
  Live datanodes (1):

  Name: *** (datanode.handzon-hadoop-hive_hadoop_network)
  Hostname: e20decb5140e
  Decommission Status : Normal
  Configured Capacity: *** (*** GB)
  DFS Used: *** (*** MB)
  Non DFS Used: *** (*** GB)
  DFS Remaining: *** (*** GB)
  DFS Used%: 0.00%
  DFS Remaining%: 5.85%
  Configured Cache Capacity: 0 (0 B)
  Cache Used: 0 (0 B)
  Cache Remaining: 0 (0 B)
  Cache Used%: 100.00%
  Cache Remaining%: 0.00%
  Xceivers: 1
  Last contact: Fri May 09 20:23:16 UTC 2025
  Last Block Report: Fri May 09 20:17:40 UTC 2025
  Num of Blocks: 6

    ...
  ```

- **Copy your CSV file into the namenode container:**  
  ```bash
  $ curl -L -o movieratings.csv https://files.grouplens.org/datasets/movielens/ml-100k/u.data
  $ docker cp movieratings.csv <namenode>:/tmp/ # on the docker host
  ```
  The [dataset](https://grouplens.org/datasets/movielens/100k/ "MovieLens data sets were collected by the GroupLens Research Project at the University of Minnesota.") comes from [GroupLens](https://grouplens.org/about/what-is-grouplens/), a research lab in the Department of Computer Science and Engineering at the University of Minnesota, Twin Cities specializing in recommender systems, online communities, mobile and ubiquitous technologies, digital libraries, and local geographic information systems.

- **Load the CSV into an HDFS folder within the container:**  
  ```bash
  $ docker exec <namenode> hdfs dfs -mkdir -p /input
  $ docker exec <namenode> hdfs dfs -put /tmp/movieratings.csv /input/ # in the docker
  ```
- **Access the Hive service container**  
  ```bash
  $ docker exec -it <hive-server> bash # `<hive-server>` is the name of your hive server
  ```
- **Create an external table from the HDFS file:**
  ```bash
  # in the docker
  $ beeline -u jdbc:hive2://localhost:10000
  ...
  Connecting to jdbc:hive2://localhost:10000
  Connected to: Apache Hive (version 2.3.2)
  Driver: Hive JDBC (version 2.3.2)
  Transaction isolation: TRANSACTION_REPEATABLE_READ
  Beeline version 2.3.2 by Apache Hive
  ...
  ```
  ```bash
  0: jdbc:hive2://localhost:10000>
  # This tells Hive to use the CSV at `/input` in HDFS as the data source.
  CREATE EXTERNAL TABLE IF NOT EXISTS movieratins (
    user_id STRING,
    movie_id STRING,
    rating FLOAT,
    datation STRING
  ) ROW FORMAT
  DELIMITED FIELDS TERMINATED BY '\t'
  STORED AS TEXTFILE
  LOCATION '/input'; # hit enter
  ```
  ```bash
  # You should see this message after you hit `enter`
  No rows affected (1.629 seconds)
  ```
- **Query the created table**
  ```bash
  0: hive2://localhost:10000> select * from movieratings limit 10; # hit enter
  ```
  ```verbatim
  +----------------------+-----------------------+---------------------+-----------------------+
  | movierating.user_id  | movierating.movie_id  | movierating.rating  | movierating.datation  |
  +----------------------+-----------------------+---------------------+-----------------------+
  | 196                  | 242                   | 3.0                 | 881250949             |
  | 186                  | 302                   | 3.0                 | 891717742             |
  | 22                   | 377                   | 1.0                 | 878887116             |
  | 244                  | 51                    | 2.0                 | 880606923             |
  | 166                  | 346                   | 1.0                 | 886397596             |
  | 298                  | 474                   | 4.0                 | 884182806             |
  | 115                  | 265                   | 2.0                 | 881171488             |
  | 253                  | 465                   | 5.0                 | 891628467             |
  | 305                  | 451                   | 3.0                 | 886324817             |
  | 6                    | 86                    | 3.0                 | 883603013             |
  +----------------------+-----------------------+---------------------+-----------------------+
  ```
- **Do some analytics using sql queries on the hive table**
  ```bash
  # compute the average rating per movie
  0: jdbc:hive2://localhost:10000> 
  SELECT movie_id,
  AVG(rating) as rating
  FROM movierating
  GROU BY movie_id
  ORDER BY LENGTH(movie_id), movie_id
  LIMIT 10;
  ```
  ```bash
  # results
  +-----------+---------------------+
  | movie_id  |       rating        |
  +-----------+---------------------+
  | 1         | 3.8783185840707963  |
  | 2         | 3.2061068702290076  |
  | 3         | 3.033333333333333   |
  | 4         | 3.550239234449761   |
  | 5         | 3.302325581395349   |
  | 6         | 3.576923076923077   |
  | 7         | 3.798469387755102   |
  | 8         | 3.9954337899543377  |
  | 9         | 3.8963210702341136  |
  | 10        | 3.831460674157303   |
  +-----------+---------------------+
  10 rows selected (2.909 seconds)
  ```
  ```bash
  0: jdbc:hive2://localhost:10000> !quit
  ```

- **Voilà! You can add nodes and compare execution times**

Feel free to pull this repo and to send me your comments/remarks.


[1]: https://www.oreilly.com/library/view/hadoop-the-definitive/9781491901687/ "Hadoop: The Definitive Guide, 4th Edition, O'Reilly Media, Inc."

[2]: https://hive.apache.org/ "The Apache Hive ™ is a distributed, fault-tolerant data warehouse system that enables analytics at a massive scale and facilitates reading, writing, and managing petabytes of data residing in distributed storage using SQL."

