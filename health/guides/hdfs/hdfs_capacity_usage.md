# hdfs_capacity_usage

**Storage | HDFS**

_The Hadoop distributed file system (HDFS) is a distributed, scalable, and portable file system
written in Java for the Hadoop framework. Some consider it to instead be a data store due to its
lack of POSIX compliance, but it does provide shell commands and Java application programming
interface (API) methods that are similar to other file
systems._<sup>[1](https://en.wikipedia.org/wiki/Apache_Hadoop) </sup>

The Netdata Agent calculates the percentage of used space capacity across all DataNodes. Receiving
this alert indicates high DataNodes space capacity utilization.

This alert is triggered into warning when the percentage of used space capacity across all DataNodes
is between 70-80% and in critical when it is between 80-90%.

<details>
<summary>See more about Hadoop</summary>

Wikipedia provides a great explanation of
HDFS<sup>[1](https://en.wikipedia.org/wiki/Apache_Hadoop) </sup>. Here are the main takeaways:

HDFS provides a software framework for distributed storage and processing of big data using the
`MapReduce` programming model. HDFS is used for storing the data and `MapReduce` is used for
processing data. It achieves reliability by replicating the data across multiple hosts, and hence
theoretically does not require redundant array of independent disks (RAID) storage on hosts. With
the default replication value, 3, data is stored on three nodes, two on the same rack, and one on a
different rack. DataNodes can talk to each other to rebalance data, to move copies around, and to
keep the replication of data high.

HDFS has five services as follows:

1. Name Node
2. Secondary Name Node
3. Job tracker
4. Data Node
5. Task Tracker

Top three are master Services/Daemons/Nodes and bottom two are slave Services. Master Services can
communicate with each other and in the same way slave services can communicate with each other.
NameNode is a master node and DataNode(s) is its corresponding slave(s) node(s) and can talk with
each other.

- NameNode: HDFS consists of only one NameNode that is called the master node. The master node can
  track files, manage the file system and has the metadata of all the stored data within it. Some
  information the NameNode keep track of are:

    - details (metadata) of blocks
    - in which DataNode each block lives, and its location
    - replication metadata of each block

  The NameNode is the gateway that a client uses to manage the HDFS cluster.

- DataNode: A DataNode stores data in it as blocks. This is also known as the Slave node and it
  stores the actual data into HDFS which is responsible for the client to read and write. These are
  slave daemons. Every DataNode sends a Heartbeat message to the NameNode every 3 seconds and
  conveys that it is alive. In this way when NameNode does not receive a heartbeat from a DataNode
  for 2 minutes, it will take that DataNode as dead and starts the process of block replications on
  some other DataNode.

- Secondary NameNode: This is only to take care of the checkpoints of the file system metadata which
  is in the NameNode. This is also known as the checkpoint node. It is the helper node for the
  NameNode. The secondary NameNode instructs the NameNode to create and send an `fsimage` and
  `editlog` file. The secondary NameNode create a compacted `fsimage` file using these inputs.

- Job Tracker: Job Tracker receives the requests for `MapReduce` execution from the client. Job
  tracker talks to the NameNode to know about the location of the data that will be used in
  processing. The NameNode responds with the metadata of the required processing data.

- Task Tracker: It is the slave node for the Job Tracker and, it will take the task from the Job
  Tracker. It also receives code from the Job Tracker. Task Tracker will take the code and apply on
  the file. The process of applying that code on the file is known as Mapper.

Some more useful information/concepts about HDFS from the official
website <sup>[2](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) </sup> :

- The File System Namespace: HDFS supports a traditional hierarchical file organization. A user or
  an application can create directories and store files inside these directories. The file system
  namespace hierarchy is similar to most other existing file systems; one can create and remove
  files, move a file from one directory to another, or rename a file. HDFS does not yet implement
  user quotas. HDFS does not support hard links or soft links. However, the HDFS architecture does
  not preclude implementing these features.

  The NameNode maintains the file system namespace. Any change to the file system namespace or its
  properties is recorded by the NameNode. An application can specify the number of replicas of a
  file that should be maintained by HDFS. The number of copies of a file is called the replication
  factor of that file. This information is stored by the NameNode.

- Data Blocks: HDFS is designed to support very large files. Applications that are compatible with
  HDFS are those that deal with large data sets. These applications write their data only once but
  they read it one or more times and require these reads to be satisfied at streaming speeds. HDFS
  supports write-once-read-many semantics on files. A typical block size used by HDFS is 64 MB.
  Thus, an HDFS file is chopped up into 64 MB chunks, and if possible, each chunk will reside on a
  different DataNode.

- Cluster Rebalancing: The HDFS architecture is compatible with data rebalancing schemes. A scheme
  might automatically move data from one DataNode to another if the free space on a DataNode falls
  below a certain threshold. In the event of a sudden high demand for a particular file, a scheme
  might dynamically create additional replicas and rebalance other data in the cluster. These types
  of data rebalancing schemes are not yet implemented.

</details>

<details>
<summary>References and sources</summary>

1. [Apache Hadoop on wikipedia](https://en.wikipedia.org/wiki/Apache_Hadoop)
2. [HDFS architecture](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)

</details>

### Troubleshooting section

Data is priceless. Before you perform any action, make sure that you have taken any necessary backup
steps. Netdata is not liable for any loss or corruption of any data, database, or software.

<details> 
<summary>Check your Disk Usage across the cluster </summary>

1. Inspect the Disk Usage for each DataNode

    ```
    root@netdata #  hadoop dfsadmin -report
    ```

   If all the DataNodes are in Disk pressure, you should consider adding more disk space. Otherwise,
   you can perform a balance of data between the DataNodes.

2. Perform a balance

    ```
    root@netdata # hdfs balancer –threshold 15
    ```

   This means that the balancer will balance data by moving blocks from over-utilized to
   under-utilized nodes, until each DataNode’s disk usage differs by no more than plus or minus 15
   percent.

</details>