### Cassandra (kə'sændrə)
目前值得关注，在性能上比较好的 NoSQL 数据库只有三个， MongoDB， HBase， Cassandra ，MongoDB目前我们已经在用着，所以要考察的是 Cassandra 和 HBase。

Cassandra 和 HBase， 国内外的使用情况貌似相反，国外似乎是 Cassandra 更流行一些，而国内则是一大波上 HBase。在 DB-Engines 的排名上， Cassandra 排在前十， 而 HBase 则在十多名。之所以优先考察 Cassandra， 一则是因为微博上关注的几个搞数据库的人的推荐，一则是因为 HBase 的搭建还需要 Hadoop， ZooKeeper 等一系列复杂的基础服务。

#### **Cassandra 的历史**：

 1. 最初由 Facebook 的两名员工（一个是 亚马逊 Dynamo 数据库的开发者之一， 一个貌似参与过微软的 SQL Server）开发的，用于 Facebook 的邮箱搜索服务。 也因此， Cassandra 比较接近 Dynamo， 也有一些 SQL Server 的特征。
 2. 2008 年 Facebook 将 Cassandra 开源， 2009 年成为 Apache 孵化器项目， 2010 年成为 顶级项目
 3.  2011 年增加 CQL 查询语言，与 Thrift API 并存，但是到 Cassandra 3.0, Thrift API 将会被抛弃，完全用 CQL 进行数据的增删改查
 4. 2013 年增加集群， vnode 等特性，支持轻量级事务
 5. 2014 年发布 2.1 版本

目前 Cassandra 社区主要由 DataStax 这家公司主导，其核心产品 DataStax Enterprise，包括 Cassandra（优化过的），Opscenter (集群、数据库管理工具，免费版功能较少）， Devcenter  （CQL 开发工具）， Enterprise 版还包括对 Spark， Hadoop， Hive， Solr 等工具的集成。EnterPrise 版采取订阅收费，分为三个版本， Standard，Pro 和 Max。可以下载来用于开发，但是生产环境的使用是限制的，具体价格官方未给出。

正是由于有一家企业主导并进行商业化支持， Cassandra 的进化才能比 HBase 快不少， HBase 到现在都没有发布 1.0 版本。目前 Cassandra 有至少五百家企业用户， 有 25 家是财富 100 强。比较不可思议的是苹果公司，在 2014 年 Cassandra summit 上透露他们有 75k+ 的 Cassandra 节点。 貌似现在苹果有三个 Cassandra commitor。Hulu, ebay, Netflix， LinkIn 等公司都是重度使用 Cassandra， Facebook 也重新使用回 Cassandra 为其一部分产品服务。

####  **Cassandra  的性能：**

关于 Cassandra 与其他几个主要的 NoSQL 数据库的对比，主要有三份报告，一个是由 DataStax 提供的 Cassandra、HBase 和 MongoDB 的性能测试对比, 详情可看 [http://vdisk.weibo.com/s/ILoNseLL5eF](http://vdisk.weibo.com/s/ILoNseLL5eF) 。以及 [SequoiaDB vs. MongoDB vs. Cassandra vs. HBase](http://www.csdn.net/article/2014-09-16/2821707-benchmark-test-of-MongoDB-SequoiaDB-HBase-Cassandra/2]), 以及 [NoSQL性能测试白皮书](http://www.infoq.com/cn/articles/nosql-performance-test) 。后面这两篇性能测试都说采用了雅虎的 YCSB 测试规则进行的测试，不过我觉得应该都是 SequoiaDB 的软广。

通过上述的性能测试指标来看， Cassandra 总体上是优于 HBase的，与 MongoDB 相比， 写入性能更高，读取性能也与 MongoDB 较为接近。

##### **Cassandra的特性：**
 

 1. 去中心化， 基于一致性哈希的完全P2P架构，每行数据通过哈希来决定应该存在哪个或哪些节点中[11]。集群没有master的概念，所有节点都是同样的角色，彻底避免了整个系统的单点问题导致的不稳定性，集群间的状态同步通过Gossip协议来进行P2P的通信。每个节点都把存储数据在本地，每个节点都可以接受来自客户端的请求。每次客户端随机选择集群中的一个节点来请求数据，对应接受请求的节点将对应的key在一致性哈希的环上定位是哪些节点应该存储这个数据，将请求转发到对应的节点上，并将对应若干节点的查询反馈返回给客户端。
 2. 支持多数据中心的 Replication， replication 策略是可配置的
 3. 伸展性好，可以通过加新机器获取到线性的吞吐能力提高
 4. 容错性好， 数据自动被复制到多个节点以进行容错
 5. 可配置的一致性策略
 6. 支持MapReduce，可以与 Hadoop， Hive 和 Pig 集成（要钱买）
 7. 使用 CQL 查询语言，与 SQL 类型

CAP 理论下各种数据库所属的位置如下图：

![CAP](http://e.picphotos.baidu.com/album/s%3D550%3Bq%3D90%3Bc%3Dxiangce%2C100%2C100/sign=2212f2703987e9504617f3692003227e/3bf33a87e950352a524ed8bb5043fbf2b2118b52.jpg?referer=2d2a53d34cc2d562ab1fe5ddf530&x=.jpg)

Log 数据分析对一致性的要求比较低， Cassandra 这样注重 AP 的会更合适一些吧。

##### **CQL 和 Cassandra 存储结构：**
Cassandra 存储的数据，有行有列， 同时也有数据类型。数据是以列存储的，在一些概念上，与关系型数据库有些许相似之处，如下图：

![RMDBS](http://www.ebaytechblog.com/wp-content/uploads/2012/07/analogy.png)

在使用 CQL 进行数据的读写之前，先要进入到一个 keyspace 里面， 如下的 CQL 语句创建了一个 keyspace：

    CREATE KEYSPACE demodb WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3 };

官方的推荐是一个 keyspace 对应一个 Application。 keyspace 关乎数据的分布式复制策略。

然后使用 use demodb 进入到该 keyspace 。

在 Cassandra 里使用 Column family 一词表示一个 table 这样的数据集合， 不过 CQL 里面则直接使用 table ，如下语句创建了一个 Column family，也是一个 table：

      CREATE TABLE users (
      user_name varchar,
      password varchar,
      gender varchar,
      session_token varchar,
      state varchar,
      birth_year bigint,
      PRIMARY KEY (user_name));

CQL 目前支持的数据类型如下：

![CQL](http://b.picphotos.baidu.com/album/s%3D550%3Bq%3D90%3Bc%3Dxiangce%2C100%2C100/sign=18aa659b942bd40746c7d3f84bb2ef6c/aa18972bd40735faf743b50a9d510fb30e2408d5.jpg?referer=620367f43a12b31b9e7bf91968cc&x=.jpg)

同时， CQL 也支持自定义类型， 如下则创建了一个 fullname 类型：

    CREATE TYPE mykeyspace.fullname (
      firstname text,
      lastname text
    );

插入数据则使用如下语句：

    INSERT INTO emp (empID, deptID, first_name, last_name) VALUES (104, 15, 'jane', 'smith');

另外，与 SQL 不同，CQL 对于没有数据的字段，可以不插入一个 null 代替， 譬如 last_name 缺失，则直接不存储这个列，节省空间。同时，一些实践上的说法，可以直接用 column name 存储数据，即column name 本身就是一个数据，这样就变成一个 row key 对应一个数组的 column name 数据。 

primary key， 也称为 row key ， 如下图是 Cassandra 三种数据存储的组织方式:

![Cassandra](http://e.picphotos.baidu.com/album/s%3D550%3Bq%3D90%3Bc%3Dxiangce%2C100%2C100/sign=51e29c78b1fb43161e1f7a7f109f371e/54fbb2fb43166d2223c5e927452309f79152d2d7.jpg?referer=567cf304ea50352ae87611381ecd&x=.jpg)

根据官方的定义 “Cassandra is a partitioned row store. Rows are organized into tables with a required primary key. Partitioning means that Cassandra can distribute your data across multiple machines in an application-transparent matter. Cassandra will automatically repartition as machines are added and removed from the cluster. Row store means that like relational databases, Cassandra organizes data by rows and columns.”， Cassandra 不能被看作是列式存储数据库， 其存储结构可以用 ： 

    SortedMap<RowKey, SortedMap<ColumnKey, ColumnValue>>

这一点上与 HBase 的纯 column-oriented 实现有区别。所以 Cassandra 在实现理念上，是对关系数据库和 NoSQL 进行的揉合吧，官方的文档里甚至提到 Cassandra 是 row-oriented 的。


Cassandra 是根据 row key 来控制这一 row 数据该存储到哪一个节点上。假设我们有四个节点，四个节点的 token 分布为 0, 25, 50, 75 。节点的 token 表示其接收 row key 的值的范围，如下图：

![row key](http://www.datastax.com/docs/_images/ring_partitions.png) 


假设我们的 row key 的值分布在 0 到 100 的区间，token 为 0 的节点接收 row key 范围在 [76, 0] , 也就是 76到 100 以及 0。 

primary key 可以指定多个，但是除了第一个是用于 partition 的， 其他的，都是用于排序的。譬如：

    CREATE TABLE scores (
     name text,
     age int,
     score int,
     date timestamp,
     PRIMARY KEY (name, age, score)
    );

除了 name 是 row key， 用于 partition， age 和 score 则是用于 cluster sort 。CQL 查询的数据当且仅当 具有 cluster sort 的时候才可以进行 order by 。因为这些数据在写入的时候就进行了排序了。

CQL 所谓的轻量级事务支持，目前只有两种，如下：

    INSERT INTO users (login, email, name, login_count)
      VALUES ('jdoe', 'jdoe@abc.com', 'Jane Doe', 1)
      IF NOT EXISTS;

    UPDATE users
      SET email = ‘janedoe@abc.com’
      WHERE login = 'jdoe'
      IF email = ‘jdoe@abc.com’;

即轻量级事务支持，就只有 IF Not exists 和 where if 这两种。

Cassandra 不支持 join， 也不支持 max， min 这些聚集函数。

##### **集群搭建：**

Cassandra 的拓扑结构分为三层，Data Center --> Rack --> Node ，构建方式分两种， 单数据中心和多数据中心。节点间通信采用的是 Gossip 这一 P2P 协议，因此节点的增删对集群的影响很小，也很容易做到。

建立 Cassandra 集群，主要有几个概念需要理解：

 +  [vnode](http://www.datastax.com/dev/blog/virtual-nodes-in-cassandra-1-2)
 

> 从 1.2 之后就增加的 virtual node 特性，之前每个节点只有一个 token，这样其所存储的数据就只有固定的一个范围，添加新节点，损坏节点恢复等都会比较复杂一些。而有了vnode 之后， 每个节点上都有多个 token， 也即一个节点可以存储多个范围内的数据。

 +  Replication factor

>  设置为 1 表示数据只会存放在一个节点上，设置为 2 表示数据还会被复制到另一个节点上做备份，以此类推

+ [Partitioner](http://www.datastax.com/documentation/cassandra/1.2/cassandra/architecture/architecturePartitionerAbout_c.html)
 

> 决定数据是如何在节点间分布的（包括 replication），目前有三种 partitioner，Murmur3Partitioner， RandomPartitioner 和 ByteOrderedPartitioner

+ [Replication Strategy](http://www.datastax.com/documentation/cassandra/1.2/cassandra/architecture/architectureDataDistributeReplication_c.html)

> 这是用于数据备份复制的策略，目前只有两种， SimpleStrategy 和 NetworkTopologyStrategy 两种， 官方建议使用的是 NetWrokTopologyStrategy , 因为这个可以扩展至多数据中心

+ [Snitches](http://www.datastax.com/documentation/cassandra/1.2/cassandra/architecture/architectureSnitchesAbout_c.html)

> Snitches determines which data centers and racks are written to and read from 。
> Snitches inform Cassandra about the network topology so that requests are routed efficiently and allows Cassandra to distribute replicas by grouping machines into data centers and racks. All nodes must have exactly the same snitch configuration. Cassandra does its best not to have more than one replica on the same rack (which is not necessarily a physical location).

这些配置都在 conf/cassandra.yaml 里面进行配置。

由于 Cassandra 用到了比较多的端口， 所以用单机进行配置集群的试验会比较麻烦，可行的办法是下载官方提供的 [vmware OVA镜像](http://planetcassandra.org/install-cassandra-ova-on-vmware/) ， 或者使用 Docker搭建环境。

在集群管理方面，提供了基于命令行的工具如 nodestool, 也提供了 [opscenter](http://www.datastax.com/documentation/opscenter/5.0/opsc/about_c.html) 这样基于浏览器的更好的管理工具，如下图：

![opscenter](http://www.datastax.com/wp-content/themes/datastax-2013/images/opscenter/opsc4-performance-view-ext.jpg)

##### **可用的Cassandra驱动：**
目前官方提供支持的只有 C#, Java, C++, Python, Node.js, Ruby 其他语言的驱动由社区提供，具体列表在[http://planetcassandra.org/client-drivers-tools/](http://planetcassandra.org/client-drivers-tools/) 。 同时官方还提供了一个 [Spark connector](https://github.com/datastax/spark-cassandra-connector) 。

对我们常用的语言， Python，和 Erlang（Elixir），都是有驱动的。 R 语言也有一个驱动，但是一年多没有更新过，不知道能用否。看这个问答 [Connect cassandra through R](http://stackoverflow.com/questions/24272452/unable-to-connect-cassandra-through-r)， 可以使用 RJDBC 连接 Cassandra，还没有实际验证过。

 Python 执行 CQL 的完整代码: [Python example](http://www.datastax.com/documentation/developer/python-driver/1.0/python-driver/quick_start/qsSimpleClientAddSession_t.html)

##### **Cassandra 读写数据的过程：**
可以看这篇文章: [How Cassandra Read, Persists Data and Maintain Consistency](http://jonathanhui.com/how-cassandra-read-persists-data-and-maintain-consistency)
