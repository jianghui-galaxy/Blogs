

```

from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession
sc = SparkContext('yarn').getOrCreate()
spark = SparkSession(sc)

hb_zk_quorum="hbase.zookeeper.quorum"
#hb_zk_ip='172.20.64.15,172.20.64.16,172.20.64.17'
hb_zk_ip='172.20.62.31,172.20.62.43,172.20.62.44'

inputable="hbase.mapreduce.inputtable"
tableName='T_CCV3_MOVEMENT'

znode_parent="zookeeper.znode.parent"
znode_parent_val="/hbase-unsecure"

columns = "hbase.mapreduce.scan.columns"
columnsValue = "CF:PERMIT_ID CF:MOV_TIME CF:MOV_DATE CF:MOV_HOUR CF:DIRECTION CF:PORT_UID CF:ACTION CF:DECISION_ID CF:TYPE CF:ILLEGAL_ACTION_TYPE_DESC CF:ILLEGAL_TYPE CF:ILLEGAL_TYPE_DESC CF:ITEM_TYPE CF:ILLEGAL_ITEM_TYPE_DESC"

dataDuration = 720

# now = pd.Period.now('D')
# BeginnDate = now - 360
# BeginnDateStr = BeginnDate.strftime('%y-%m-%d').encode('utf-8')

import dateutil
import datetime
FinalDay = datetime.datetime(2018, 1, 25, 0, 0, 0)
delta = datetime.timedelta(days=dataDuration)
BeginnDate = FinalDay - delta
BeginnDateStr = BeginnDate.strftime('%Y-%m-%d')

hbaseconf = {hb_zk_quorum:hb_zk_ip,inputable:tableName,columns:columnsValue,znode_parent:znode_parent_val}
keyConv = "org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter"
valueConv = "org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter"

moveRdd = sc.newAPIHadoopRDD("org.apache.hadoop.hbase.mapreduce.TableInputFormat","org.apache.hadoop.hbase.io.ImmutableBytesWritable","org.apache.hadoop.hbase.client.Result",keyConverter=keyConv, valueConverter=valueConv, conf=hbaseconf)

moveArr = moveRdd.filter(lambda (x,y): str(x)[8:18] > BeginnDateStr).values().flatMap(lambda x: x.split('\n'))
moveDF = spark.read.json(moveArr)

moveDF.createOrReplaceTempView("movement")

movement = spark.sql("SELECT row, qualifier, value FROM movement").toPandas()

df = movement.pivot(index = 'row', columns='qualifier', values = 'value').copy()

df.to_csv('/home/da/Data/movement_all.csv')


```



```
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn --num-executors 10 --executor-cores 5  --executor-memory 5G  --driver-memory 8G     --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/spark-examples_2.11-1.6.0-typesafe-001.jar  /home/da/Scripts/BasicDataGen/getMovement_ver1.py
```







```

```





LOG

```bash
[da@bigdata-data10 BasicDataGen]$ /usr/hdp/current/spark2-client/bin/spark-submit --master yarn --num-executors 10 --executor-cores 5  --executor-memory 5G  --driver-memory 8G     --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/spark-examples_2.11-1.6.0-typesafe-001.jar  /home/da/Scripts/BasicDataGen/getMovement_ver1.py
/usr/lib/python2.7/site-packages/pkg_resources/__init__.py:1324: UserWarning: /tmp/python_eggs/ is writable by group/others and vulnerable to attack when used with get_resource_filename. Consider a more secure location (set with .set_extraction_path or the PYTHON_EGG_CACHE environment variable).
  warnings.warn(msg, UserWarning)
18/03/10 13:51:01 INFO SparkContext: Running Spark version 2.1.0.2.6.0.3-8
18/03/10 13:51:02 INFO SecurityManager: Changing view acls to: da
18/03/10 13:51:02 INFO SecurityManager: Changing modify acls to: da
18/03/10 13:51:02 INFO SecurityManager: Changing view acls groups to:
18/03/10 13:51:02 INFO SecurityManager: Changing modify acls groups to:
18/03/10 13:51:02 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(da); groups with view permissions: Set(); users  with modify permissions: Set(da); groups with modify permissions: Set()
18/03/10 13:51:02 INFO Utils: Successfully started service 'sparkDriver' on port 59094.
18/03/10 13:51:02 INFO SparkEnv: Registering MapOutputTracker
18/03/10 13:51:02 INFO SparkEnv: Registering BlockManagerMaster
18/03/10 13:51:02 INFO BlockManagerMasterEndpoint: Using org.apache.spark.storage.DefaultTopologyMapper for getting topology information
18/03/10 13:51:02 INFO BlockManagerMasterEndpoint: BlockManagerMasterEndpoint up
18/03/10 13:51:02 INFO DiskBlockManager: Created local directory at /tmp/blockmgr-18602ade-3ea7-4c6f-8315-1eb17cbe72d9
18/03/10 13:51:02 INFO MemoryStore: MemoryStore started with capacity 4.1 GB
18/03/10 13:51:02 INFO SparkEnv: Registering OutputCommitCoordinator
18/03/10 13:51:02 INFO log: Logging initialized @2241ms
18/03/10 13:51:03 INFO Server: jetty-9.2.z-SNAPSHOT
18/03/10 13:51:03 INFO Server: Started @2315ms
18/03/10 13:51:03 INFO ServerConnector: Started ServerConnector@5a0c2a7e{HTTP/1.1}{0.0.0.0:4040}
18/03/10 13:51:03 INFO Utils: Successfully started service 'SparkUI' on port 4040.
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@153aec8c{/jobs,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@2290eac{/jobs/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@417ab336{/jobs/job,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@466f7573{/jobs/job/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@3c2866de{/stages,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@146b9153{/stages/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@7de83a99{/stages/stage,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@7a5215c2{/stages/stage/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@7dffa61c{/stages/pool,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@47f047f0{/stages/pool/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@65b0997e{/storage,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@3898cbb5{/storage/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@4484955d{/storage/rdd,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@51a71104{/storage/rdd/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@6039122c{/environment,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@4338a25a{/environment/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@51fd410f{/executors,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@8b98c43{/executors/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@26bce846{/executors/threadDump,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@42bb1636{/executors/threadDump/json,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@f44ba39{/static,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@691857f7{/,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@38f57bfa{/api,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@42e5276e{/jobs/job/kill,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@1d1182e1{/stages/stage/kill,null,AVAILABLE,@Spark}
18/03/10 13:51:03 INFO SparkUI: Bound SparkUI to 0.0.0.0, and started at http://172.20.62.40:4040
18/03/10 13:51:03 INFO RequestHedgingRMFailoverProxyProvider: Looking for the active RM in [rm1, rm2]...
18/03/10 13:51:04 INFO RequestHedgingRMFailoverProxyProvider: Found active RM [rm2]
18/03/10 13:51:04 INFO Client: Requesting a new application from cluster with 10 NodeManagers
18/03/10 13:51:04 INFO Client: Verifying our application has not requested more than the maximum memory capability of the cluster (36864 MB per container)
18/03/10 13:51:04 INFO Client: Will allocate AM container, with 896 MB memory including 384 MB overhead
18/03/10 13:51:04 INFO Client: Setting up container launch context for our AM
18/03/10 13:51:04 INFO Client: Setting up the launch environment for our AM container
18/03/10 13:51:04 INFO Client: Preparing resources for our AM container
18/03/10 13:51:05 INFO Client: Use hdfs cache file as spark.yarn.archive for HDP, hdfsCacheFile:hdfs://MCluster/hdp/apps/2.6.0.3-8/spark2/spark2-hdp-yarn-archive.tar.gz
18/03/10 13:51:05 INFO Client: Source and destination file systems are the same. Not copying hdfs://MCluster/hdp/apps/2.6.0.3-8/spark2/spark2-hdp-yarn-archive.tar.gz
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-annotations-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-client-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-common-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-common-1.1.2.2.6.0.3-8-tests.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-examples-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-it-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:05 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-it-1.1.2.2.6.0.3-8-tests.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-procedure-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-protocol-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-rest-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-rsgroup-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-server-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-server-1.1.2.2.6.0.3-8-tests.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-shell-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/hbase-thrift-1.1.2.2.6.0.3-8.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/hbase-client/lib/spark-examples_2.11-1.6.0-typesafe-001.jar -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/spark-examples_2.11-1.6.0-typesafe-001.jar
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/spark2-client/python/lib/pyspark.zip -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/pyspark.zip
18/03/10 13:51:06 INFO Client: Uploading resource file:/usr/hdp/current/spark2-client/python/lib/py4j-0.10.4-src.zip -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/py4j-0.10.4-src.zip
18/03/10 13:51:06 INFO Client: Uploading resource file:/tmp/spark-a0913fac-d59f-4d75-95c3-91e8ecb40415/__spark_conf__4967229208790888814.zip -> hdfs://MCluster/user/da/.sparkStaging/application_1520606297884_0005/__spark_conf__.zip
18/03/10 13:51:06 INFO SecurityManager: Changing view acls to: da
18/03/10 13:51:06 INFO SecurityManager: Changing modify acls to: da
18/03/10 13:51:06 INFO SecurityManager: Changing view acls groups to:
18/03/10 13:51:06 INFO SecurityManager: Changing modify acls groups to:
18/03/10 13:51:06 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(da); groups with view permissions: Set(); users  with modify permissions: Set(da); groups with modify permissions: Set()
18/03/10 13:51:06 INFO Client: Submitting application application_1520606297884_0005 to ResourceManager
18/03/10 13:51:07 INFO YarnClientImpl: Submitted application application_1520606297884_0005
18/03/10 13:51:07 INFO SchedulerExtensionServices: Starting Yarn extension services with app application_1520606297884_0005 and attemptId None
18/03/10 13:51:08 INFO Client: Application report for application_1520606297884_0005 (state: ACCEPTED)
18/03/10 13:51:08 INFO Client:
         client token: N/A
         diagnostics: AM container is launched, waiting for AM container to Register with RM
         ApplicationMaster host: N/A
         ApplicationMaster RPC port: -1
         queue: default
         start time: 1520661066933
         final status: UNDEFINED
         tracking URL: http://bigdata-master2.novalocal:8088/proxy/application_1520606297884_0005/
         user: da
18/03/10 13:51:09 INFO Client: Application report for application_1520606297884_0005 (state: ACCEPTED)
18/03/10 13:51:10 INFO Client: Application report for application_1520606297884_0005 (state: ACCEPTED)
18/03/10 13:51:10 INFO YarnSchedulerBackend$YarnSchedulerEndpoint: ApplicationMaster registered as NettyRpcEndpointRef(null)
18/03/10 13:51:10 INFO YarnClientSchedulerBackend: Add WebUI Filter. org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter, Map(PROXY_HOSTS -> bigdata-master1.novalocal,bigdata-master2.novalocal, PROXY_URI_BASES -> http://bigdata-master1.novalocal:8088/proxy/application_1520606297884_0005,http://bigdata-master2.novalocal:8088/proxy/application_1520606297884_0005), /proxy/application_1520606297884_0005
18/03/10 13:51:10 INFO JettyUtils: Adding filter: org.apache.hadoop.yarn.server.webproxy.amfilter.AmIpFilter
18/03/10 13:51:11 INFO Client: Application report for application_1520606297884_0005 (state: RUNNING)
18/03/10 13:51:11 INFO Client:
         client token: N/A
         diagnostics: N/A
         ApplicationMaster host: 172.20.62.40
         ApplicationMaster RPC port: 0
         queue: default
         start time: 1520661066933
         final status: UNDEFINED
         tracking URL: http://bigdata-master2.novalocal:8088/proxy/application_1520606297884_0005/
         user: da
18/03/10 13:51:11 INFO YarnClientSchedulerBackend: Application application_1520606297884_0005 has started running.
18/03/10 13:51:11 INFO Utils: Successfully started service 'org.apache.spark.network.netty.NettyBlockTransferService' on port 38573.
18/03/10 13:51:11 INFO NettyBlockTransferService: Server created on 172.20.62.40:38573
18/03/10 13:51:11 INFO BlockManager: Using org.apache.spark.storage.RandomBlockReplicationPolicy for block replication policy
18/03/10 13:51:11 INFO BlockManagerMaster: Registering BlockManager BlockManagerId(driver, 172.20.62.40, 38573, None)
18/03/10 13:51:11 INFO BlockManagerMasterEndpoint: Registering block manager 172.20.62.40:38573 with 4.1 GB RAM, BlockManagerId(driver, 172.20.62.40, 38573, None)
18/03/10 13:51:11 INFO BlockManagerMaster: Registered BlockManager BlockManagerId(driver, 172.20.62.40, 38573, None)
18/03/10 13:51:11 INFO BlockManager: Initialized BlockManager: BlockManagerId(driver, 172.20.62.40, 38573, None)
18/03/10 13:51:11 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@508882e3{/metrics/json,null,AVAILABLE,@Spark}
18/03/10 13:51:11 INFO EventLoggingListener: Logging events to hdfs:///spark2-history/application_1520606297884_0005
18/03/10 13:51:13 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.40:42763) with ID 2
18/03/10 13:51:13 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data10.novalocal:60579 with 2.5 GB RAM, BlockManagerId(2, bigdata-data10.novalocal, 60579, None)
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.38:53146) with ID 8
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data8.novalocal:48332 with 2.5 GB RAM, BlockManagerId(8, bigdata-data8.novalocal, 48332, None)
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.37:59023) with ID 4
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.36:54992) with ID 7
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data7.novalocal:52755 with 2.5 GB RAM, BlockManagerId(4, bigdata-data7.novalocal, 52755, None)
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.31:55503) with ID 5
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data6.novalocal:36422 with 2.5 GB RAM, BlockManagerId(7, bigdata-data6.novalocal, 36422, None)
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.32:57729) with ID 3
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.35:44597) with ID 1
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data1.novalocal:44851 with 2.5 GB RAM, BlockManagerId(5, bigdata-data1.novalocal, 44851, None)
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data2.novalocal:48413 with 2.5 GB RAM, BlockManagerId(3, bigdata-data2.novalocal, 48413, None)
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.33:59778) with ID 6
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data5.novalocal:34005 with 2.5 GB RAM, BlockManagerId(1, bigdata-data5.novalocal, 34005, None)
18/03/10 13:51:14 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data3.novalocal:45293 with 2.5 GB RAM, BlockManagerId(6, bigdata-data3.novalocal, 45293, None)
18/03/10 13:51:14 INFO YarnClientSchedulerBackend: SchedulerBackend is ready for scheduling beginning after reached minRegisteredResourcesRatio: 0.8
18/03/10 13:51:14 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.39:55642) with ID 9
18/03/10 13:51:15 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data9.novalocal:54909 with 2.5 GB RAM, BlockManagerId(9, bigdata-data9.novalocal, 54909, None)
18/03/10 13:51:15 INFO YarnSchedulerBackend$YarnDriverEndpoint: Registered executor NettyRpcEndpointRef(null) (172.20.62.34:33506) with ID 10
18/03/10 13:51:15 INFO MemoryStore: Block broadcast_0 stored as values in memory (estimated size 367.0 KB, free 4.1 GB)
18/03/10 13:51:15 INFO BlockManagerMasterEndpoint: Registering block manager bigdata-data4.novalocal:53033 with 2.5 GB RAM, BlockManagerId(10, bigdata-data4.novalocal, 53033, None)
18/03/10 13:51:15 INFO MemoryStore: Block broadcast_0_piece0 stored as bytes in memory (estimated size 31.8 KB, free 4.1 GB)
18/03/10 13:51:15 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on 172.20.62.40:38573 (size: 31.8 KB, free: 4.1 GB)
18/03/10 13:51:15 INFO SparkContext: Created broadcast 0 from newAPIHadoopRDD at PythonRDD.scala:598
18/03/10 13:51:15 INFO MemoryStore: Block broadcast_1 stored as values in memory (estimated size 364.9 KB, free 4.1 GB)
18/03/10 13:51:15 INFO MemoryStore: Block broadcast_1_piece0 stored as bytes in memory (estimated size 31.8 KB, free 4.1 GB)
18/03/10 13:51:15 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on 172.20.62.40:38573 (size: 31.8 KB, free: 4.1 GB)
18/03/10 13:51:15 INFO SparkContext: Created broadcast 1 from broadcast at PythonRDD.scala:579
18/03/10 13:51:15 INFO Converter: Loaded converter: org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter
18/03/10 13:51:15 INFO Converter: Loaded converter: org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter
18/03/10 13:51:15 INFO RecoverableZooKeeper: Process identifier=hconnection-0x433c18a connecting to ZooKeeper ensemble=172.20.62.31:2181,172.20.62.43:2181,172.20.62.44:2181
18/03/10 13:51:15 INFO ZooKeeper: Client environment:zookeeper.version=3.4.6-8--1, built on 04/01/2017 21:09 GMT
18/03/10 13:51:15 INFO ZooKeeper: Client environment:host.name=bigdata-data10.novalocal
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.version=1.8.0_25
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.vendor=Oracle Corporation
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.home=/usr/lib/java/jdk1.8.0_25/jre
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.class.path=/usr/hdp/current/spark2-client/conf/:/usr/hdp/current/spark2-client/jars/jsr305-1.3.9.jar:/usr/hdp/current/spark2-client/jars/antlr-2.7.7.jar:/usr/hdp/current/spark2-client/jars/api-asn1-api-1.0.0-M20.jar:/usr/hdp/current/spark2-client/jars/hadoop-yarn-client-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/eigenbase-properties-1.1.5.jar:/usr/hdp/current/spark2-client/jars/pyrolite-4.13.jar:/usr/hdp/current/spark2-client/jars/aopalliance-repackaged-2.4.0-b34.jar:/usr/hdp/current/spark2-client/jars/jpam-1.1.jar:/usr/hdp/current/spark2-client/jars/jta-1.1.jar:/usr/hdp/current/spark2-client/jars/hk2-utils-2.4.0-b34.jar:/usr/hdp/current/spark2-client/jars/super-csv-2.2.0.jar:/usr/hdp/current/spark2-client/jars/jersey-client-2.22.2.jar:/usr/hdp/current/spark2-client/jars/jsp-api-2.1.jar:/usr/hdp/current/spark2-client/jars/nimbus-jose-jwt-3.9.jar:/usr/hdp/current/spark2-client/jars/hadoop-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/hive-metastore-1.2.1.spark2.hdp.jar:/usr/hdp/current/spark2-client/jars/snappy-java-1.1.2.6.jar:/usr/hdp/current/spark2-client/jars/parquet-encoding-1.8.1.jar:/usr/hdp/current/spark2-client/jars/snappy-0.2.jar:/usr/hdp/current/spark2-client/jars/pmml-model-1.2.15.jar:/usr/hdp/current/spark2-client/jars/httpcore-4.4.4.jar:/usr/hdp/current/spark2-client/jars/commons-httpclient-3.1.jar:/usr/hdp/current/spark2-client/jars/hadoop-yarn-server-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/jackson-core-asl-1.9.13.jar:/usr/hdp/current/spark2-client/jars/scala-parser-combinators_2.11-1.0.4.jar:/usr/hdp/current/spark2-client/jars/jackson-annotations-2.6.5.jar:/usr/hdp/current/spark2-client/jars/mx4j-3.0.2.jar:/usr/hdp/current/spark2-client/jars/breeze_2.11-0.12.jar:/usr/hdp/current/spark2-client/jars/jcip-annotations-1.0.jar:/usr/hdp/current/spark2-client/jars/okhttp-2.4.0.jar:/usr/hdp/current/spark2-client/jars/jackson-core-2.6.5.jar:/usr/hdp/current/spark2-client/jars/hadoop-mapreduce-client-shuffle-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/breeze-macros_2.11-0.12.jar:/usr/hdp/current/spark2-client/jars/lz4-1.3.0.jar:/usr/hdp/current/spark2-client/jars/spark-hive_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/metrics-json-3.1.2.jar:/usr/hdp/current/spark2-client/jars/hadoop-yarn-api-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/metrics-jvm-3.1.2.jar:/usr/hdp/current/spark2-client/jars/jersey-media-jaxb-2.22.2.jar:/usr/hdp/current/spark2-client/jars/libthrift-0.9.3.jar:/usr/hdp/current/spark2-client/jars/javax.ws.rs-api-2.0.1.jar:/usr/hdp/current/spark2-client/jars/spark-streaming_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/osgi-resource-locator-1.0.1.jar:/usr/hdp/current/spark2-client/jars/hadoop-annotations-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/spark-repl_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/commons-cli-1.2.jar:/usr/hdp/current/spark2-client/jars/JavaEWAH-0.3.2.jar:/usr/hdp/current/spark2-client/jars/calcite-core-1.2.0-incubating.jar:/usr/hdp/current/spark2-client/jars/gson-2.2.4.jar:/usr/hdp/current/spark2-client/jars/javolution-5.5.1.jar:/usr/hdp/current/spark2-client/jars/stax-api-1.0.1.jar:/usr/hdp/current/spark2-client/jars/pmml-schema-1.2.15.jar:/usr/hdp/current/spark2-client/jars/scala-xml_2.11-1.0.2.jar:/usr/hdp/current/spark2-client/jars/bonecp-0.8.0.RELEASE.jar:/usr/hdp/current/spark2-client/jars/commons-logging-1.1.3.jar:/usr/hdp/current/spark2-client/jars/commons-dbcp-1.4.jar:/usr/hdp/current/spark2-client/jars/datanucleus-rdbms-3.2.9.jar:/usr/hdp/current/spark2-client/jars/jaxb-api-2.2.2.jar:/usr/hdp/current/spark2-client/jars/commons-beanutils-core-1.8.0.jar:/usr/hdp/current/spark2-client/jars/log4j-1.2.17.jar:/usr/hdp/current/spark2-client/jars/antlr4-runtime-4.5.3.jar:/usr/hdp/current/spark2-client/jars/apacheds-i18n-2.0.0-M15.jar:/usr/hdp/current/spark2-client/jars/spark-sql_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/curator-framework-2.6.0.jar:/usr/hdp/current/spark2-client/jars/okio-1.4.0.jar:/usr/hdp/current/spark2-client/jars/hadoop-mapreduce-client-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/jodd-core-3.5.2.jar:/usr/hdp/current/spark2-client/jars/azure-keyvault-core-0.8.0.jar:/usr/hdp/current/spark2-client/jars/aopalliance-1.0.jar:/usr/hdp/current/spark2-client/jars/xmlenc-0.52.jar:/usr/hdp/current/spark2-client/jars/xz-1.0.jar:/usr/hdp/current/spark2-client/jars/avro-1.7.7.jar:/usr/hdp/current/spark2-client/jars/netty-3.8.0.Final.jar:/usr/hdp/current/spark2-client/jars/shapeless_2.11-2.0.0.jar:/usr/hdp/current/spark2-client/jars/hadoop-aws-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/commons-math3-3.4.1.jar:/usr/hdp/current/spark2-client/jars/leveldbjni-all-1.8.jar:/usr/hdp/current/spark2-client/jars/spark-yarn_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/kryo-shaded-3.0.3.jar:/usr/hdp/current/spark2-client/jars/spire-macros_2.11-0.7.4.jar:/usr/hdp/current/spark2-client/jars/commons-codec-1.10.jar:/usr/hdp/current/spark2-client/jars/py4j-0.10.4.jar:/usr/hdp/current/spark2-client/jars/datanucleus-api-jdo-3.2.6.jar:/usr/hdp/current/spark2-client/jars/hadoop-yarn-server-web-proxy-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/parquet-column-1.8.1.jar:/usr/hdp/current/spark2-client/jars/hive-beeline-1.2.1.spark2.hdp.jar:/usr/hdp/current/spark2-client/jars/xercesImpl-2.9.1.jar:/usr/hdp/current/spark2-client/jars/guava-14.0.1.jar:/usr/hdp/current/spark2-client/jars/stream-2.7.0.jar:/usr/hdp/current/spark2-client/jars/scala-compiler-2.11.8.jar:/usr/hdp/current/spark2-client/jars/javax.inject-2.4.0-b34.jar:/usr/hdp/current/spark2-client/jars/hadoop-hdfs-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/json-smart-1.1.1.jar:/usr/hdp/current/spark2-client/jars/hadoop-mapreduce-client-jobclient-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/compress-lzf-1.0.3.jar:/usr/hdp/current/spark2-client/jars/jersey-container-servlet-2.22.2.jar:/usr/hdp/current/spark2-client/jars/spark-mllib-local_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/aws-java-sdk-kms-1.10.6.jar:/usr/hdp/current/spark2-client/jars/spark-mllib_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/spark-sketch_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/parquet-format-2.3.0-incubating.jar:/usr/hdp/current/spark2-client/jars/jetty-6.1.26.hwx.jar:/usr/hdp/current/spark2-client/jars/scalap-2.11.8.jar:/usr/hdp/current/spark2-client/jars/jersey-server-2.22.2.jar:/usr/hdp/current/spark2-client/jars/spark-catalyst_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/hadoop-openstack-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/aws-java-sdk-s3-1.10.6.jar:/usr/hdp/current/spark2-client/jars/mail-1.4.7.jar:/usr/hdp/current/spark2-client/jars/netty-all-4.0.42.Final.jar:/usr/hdp/current/spark2-client/jars/commons-digester-1.8.jar:/usr/hdp/current/spark2-client/jars/hadoop-azure-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/json4s-ast_2.11-3.2.11.jar:/usr/hdp/current/spark2-client/jars/jersey-guava-2.22.2.jar:/usr/hdp/current/spark2-client/jars/jackson-module-paranamer-2.6.5.jar:/usr/hdp/current/spark2-client/jars/hadoop-mapreduce-client-core-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/azure-storage-4.2.0.jar:/usr/hdp/current/spark2-client/jars/aws-java-sdk-core-1.10.6.jar:/usr/hdp/current/spark2-client/jars/scala-library-2.11.8.jar:/usr/hdp/current/spark2-client/jars/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/hdp/current/spark2-client/jars/spark-launcher_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/curator-client-2.6.0.jar:/usr/hdp/current/spark2-client/jars/jersey-container-servlet-core-2.22.2.jar:/usr/hdp/current/spark2-client/jars/javax.servlet-api-3.1.0.jar:/usr/hdp/current/spark2-client/jars/javax.annotation-api-1.2.jar:/usr/hdp/current/spark2-client/jars/jackson-xc-1.9.13.jar:/usr/hdp/current/spark2-client/jars/commons-compress-1.4.1.jar:/usr/hdp/current/spark2-client/jars/oro-2.0.8.jar:/usr/hdp/current/spark2-client/jars/parquet-common-1.8.1.jar:/usr/hdp/current/spark2-client/jars/spark-network-common_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/metrics-graphite-3.1.2.jar:/usr/hdp/current/spark2-client/jars/commons-lang-2.6.jar:/usr/hdp/current/spark2-client/jars/hive-exec-1.2.1.spark2.hdp.jar:/usr/hdp/current/spark2-client/jars/bcprov-jdk15on-1.51.jar:/usr/hdp/current/spark2-client/jars/slf4j-api-1.7.16.jar:/usr/hdp/current/spark2-client/jars/httpclient-4.5.2.jar:/usr/hdp/current/spark2-client/jars/datanucleus-core-3.2.10.jar:/usr/hdp/current/spark2-client/jars/json4s-core_2.11-3.2.11.jar:/usr/hdp/current/spark2-client/jars/jetty-sslengine-6.1.26.hwx.jar:/usr/hdp/current/spark2-client/jars/jackson-dataformat-cbor-2.6.5.jar:/usr/hdp/current/spark2-client/jars/spark-cloud_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/spire_2.11-0.7.4.jar:/usr/hdp/current/spark2-client/jars/antlr-runtime-3.4.jar:/usr/hdp/current/spark2-client/jars/jetty-util-6.1.26.hwx.jar:/usr/hdp/current/spark2-client/jars/commons-compiler-3.0.0.jar:/usr/hdp/current/spark2-client/jars/calcite-avatica-1.2.0-incubating.jar:/usr/hdp/current/spark2-client/jars/hive-jdbc-1.2.1.spark2.hdp.jar:/usr/hdp/current/spark2-client/jars/commons-net-2.2.jar:/usr/hdp/current/spark2-client/jars/htrace-core-3.1.0-incubating.jar:/usr/hdp/current/spark2-client/jars/opencsv-2.3.jar:/usr/hdp/current/spark2-client/jars/activation-1.1.1.jar:/usr/hdp/current/spark2-client/jars/parquet-hadoop-1.8.1.jar:/usr/hdp/current/spark2-client/jars/validation-api-1.1.0.Final.jar:/usr/hdp/current/spark2-client/jars/jcl-over-slf4j-1.7.16.jar:/usr/hdp/current/spark2-client/jars/parquet-hadoop-bundle-1.6.0.jar:/usr/hdp/current/spark2-client/jars/metrics-core-3.1.2.jar:/usr/hdp/current/spark2-client/jars/arpack_combined_all-0.1.jar:/usr/hdp/current/spark2-client/jars/jackson-databind-2.6.5.jar:/usr/hdp/current/spark2-client/jars/univocity-parsers-2.2.1.jar:/usr/hdp/current/spark2-client/jars/jersey-common-2.22.2.jar:/usr/hdp/current/spark2-client/jars/spark-hive-thriftserver_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/libfb303-0.9.3.jar:/usr/hdp/current/spark2-client/jars/jackson-module-scala_2.11-2.6.5.jar:/usr/hdp/current/spark2-client/jars/avro-ipc-1.7.7.jar:/usr/hdp/current/spark2-client/jars/commons-lang3-3.5.jar:/usr/hdp/current/spark2-client/jars/avro-mapred-1.7.7-hadoop2.jar:/usr/hdp/current/spark2-client/jars/guice-3.0.jar:/usr/hdp/current/spark2-client/jars/hadoop-client-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/slf4j-log4j12-1.7.16.jar:/usr/hdp/current/spark2-client/jars/objenesis-2.1.jar:/usr/hdp/current/spark2-client/jars/hadoop-yarn-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/core-1.1.2.jar:/usr/hdp/current/spark2-client/jars/ivy-2.4.0.jar:/usr/hdp/current/spark2-client/jars/paranamer-2.3.jar:/usr/hdp/current/spark2-client/jars/apache-log4j-extras-1.2.17.jar:/usr/hdp/current/spark2-client/jars/hive-cli-1.2.1.spark2.hdp.jar:/usr/hdp/current/spark2-client/jars/commons-collections-3.2.2.jar:/usr/hdp/current/spark2-client/jars/stringtemplate-3.2.1.jar:/usr/hdp/current/spark2-client/jars/jdo-api-3.0.1.jar:/usr/hdp/current/spark2-client/jars/spark-graphx_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/parquet-jackson-1.8.1.jar:/usr/hdp/current/spark2-client/jars/hadoop-yarn-registry-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/chill-java-0.8.0.jar:/usr/hdp/current/spark2-client/jars/stax-api-1.0-2.jar:/usr/hdp/current/spark2-client/jars/commons-io-2.4.jar:/usr/hdp/current/spark2-client/jars/guice-servlet-3.0.jar:/usr/hdp/current/spark2-client/jars/azure-data-lake-store-sdk-2.1.4.jar:/usr/hdp/current/spark2-client/jars/spark-unsafe_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/zookeeper-3.4.6.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/minlog-1.3.0.jar:/usr/hdp/current/spark2-client/jars/json4s-jackson_2.11-3.2.11.jar:/usr/hdp/current/spark2-client/jars/scala-reflect-2.11.8.jar:/usr/hdp/current/spark2-client/jars/calcite-linq4j-1.2.0-incubating.jar:/usr/hdp/current/spark2-client/jars/hk2-api-2.4.0-b34.jar:/usr/hdp/current/spark2-client/jars/jtransforms-2.4.0.jar:/usr/hdp/current/spark2-client/jars/jline-2.12.1.jar:/usr/hdp/current/spark2-client/jars/spark-core_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/spark-network-shuffle_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/hk2-locator-2.4.0-b34.jar:/usr/hdp/current/spark2-client/jars/javassist-3.18.1-GA.jar:/usr/hdp/current/spark2-client/jars/ST4-4.0.4.jar:/usr/hdp/current/spark2-client/jars/protobuf-java-2.5.0.jar:/usr/hdp/current/spark2-client/jars/derby-10.12.1.1.jar:/usr/hdp/current/spark2-client/jars/hadoop-mapreduce-client-app-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/janino-3.0.0.jar:/usr/hdp/current/spark2-client/jars/spark-tags_2.11-2.1.0.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/commons-configuration-1.6.jar:/usr/hdp/current/spark2-client/jars/joda-time-2.9.3.jar:/usr/hdp/current/spark2-client/jars/jackson-mapper-asl-1.9.13.jar:/usr/hdp/current/spark2-client/jars/RoaringBitmap-0.5.11.jar:/usr/hdp/current/spark2-client/jars/java-xmlbuilder-1.0.jar:/usr/hdp/current/spark2-client/jars/commons-crypto-1.0.0.jar:/usr/hdp/current/spark2-client/jars/jackson-jaxrs-1.9.13.jar:/usr/hdp/current/spark2-client/jars/javax.inject-1.jar:/usr/hdp/current/spark2-client/jars/base64-2.3.8.jar:/usr/hdp/current/spark2-client/jars/curator-recipes-2.6.0.jar:/usr/hdp/current/spark2-client/jars/hadoop-auth-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/api-util-1.0.0-M20.jar:/usr/hdp/current/spark2-client/jars/jets3t-0.9.3.jar:/usr/hdp/current/spark2-client/jars/hadoop-azure-datalake-2.7.3.2.6.0.3-8.jar:/usr/hdp/current/spark2-client/jars/jul-to-slf4j-1.7.16.jar:/usr/hdp/current/spark2-client/jars/chill_2.11-0.8.0.jar:/usr/hdp/current/spark2-client/jars/commons-pool-1.5.4.jar:/usr/hdp/current/spark2-client/jars/xbean-asm5-shaded-4.4.jar:/usr/hdp/current/spark2-client/jars/commons-beanutils-1.7.0.jar:/usr/hdp/current/hadoop-client/conf/
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.library.path=/usr/hdp/current/hadoop-client/lib/native:/usr/hdp/current/hadoop-client/lib/native/Linux-amd64-64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.io.tmpdir=/tmp
18/03/10 13:51:15 INFO ZooKeeper: Client environment:java.compiler=<NA>
18/03/10 13:51:15 INFO ZooKeeper: Client environment:os.name=Linux
18/03/10 13:51:15 INFO ZooKeeper: Client environment:os.arch=amd64
18/03/10 13:51:15 INFO ZooKeeper: Client environment:os.version=3.10.0-327.el7.x86_64
18/03/10 13:51:15 INFO ZooKeeper: Client environment:user.name=da
18/03/10 13:51:15 INFO ZooKeeper: Client environment:user.home=/home/da
18/03/10 13:51:15 INFO ZooKeeper: Client environment:user.dir=/home/da/Scripts/BasicDataGen
18/03/10 13:51:15 INFO ZooKeeper: Initiating client connection, connectString=172.20.62.31:2181,172.20.62.43:2181,172.20.62.44:2181 sessionTimeout=180000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@304a2a2b
18/03/10 13:51:15 INFO ClientCnxn: Opening socket connection to server 172.20.62.43/172.20.62.43:2181. Will not attempt to authenticate using SASL (unknown error)
18/03/10 13:51:15 INFO ClientCnxn: Socket connection established to 172.20.62.43/172.20.62.43:2181, initiating session
18/03/10 13:51:15 INFO ClientCnxn: Session establishment complete on server 172.20.62.43/172.20.62.43:2181, sessionid = 0x26206c49ad00125, negotiated timeout = 60000
18/03/10 13:51:15 INFO RegionSizeCalculator: Calculating region sizes for table "T_CCV3_MOVEMENT".
18/03/10 13:51:16 INFO ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
18/03/10 13:51:16 INFO ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x26206c49ad00125
18/03/10 13:51:16 INFO ZooKeeper: Session: 0x26206c49ad00125 closed
18/03/10 13:51:16 INFO ClientCnxn: EventThread shut down
18/03/10 13:51:16 INFO SparkContext: Starting job: take at SerDeUtil.scala:203
18/03/10 13:51:16 INFO DAGScheduler: Got job 0 (take at SerDeUtil.scala:203) with 1 output partitions
18/03/10 13:51:16 INFO DAGScheduler: Final stage: ResultStage 0 (take at SerDeUtil.scala:203)
18/03/10 13:51:16 INFO DAGScheduler: Parents of final stage: List()
18/03/10 13:51:16 INFO DAGScheduler: Missing parents: List()
18/03/10 13:51:16 INFO DAGScheduler: Submitting ResultStage 0 (MapPartitionsRDD[1] at map at PythonHadoopUtil.scala:181), which has no missing parents
18/03/10 13:51:16 INFO MemoryStore: Block broadcast_2 stored as values in memory (estimated size 2.9 KB, free 4.1 GB)
18/03/10 13:51:16 INFO MemoryStore: Block broadcast_2_piece0 stored as bytes in memory (estimated size 1812.0 B, free 4.1 GB)
18/03/10 13:51:16 INFO BlockManagerInfo: Added broadcast_2_piece0 in memory on 172.20.62.40:38573 (size: 1812.0 B, free: 4.1 GB)
18/03/10 13:51:16 INFO SparkContext: Created broadcast 2 from broadcast at DAGScheduler.scala:996
18/03/10 13:51:16 INFO DAGScheduler: Submitting 1 missing tasks from ResultStage 0 (MapPartitionsRDD[1] at map at PythonHadoopUtil.scala:181)
18/03/10 13:51:16 INFO YarnScheduler: Adding task set 0.0 with 1 tasks
18/03/10 13:51:16 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, bigdata-data5.novalocal, executor 1, partition 0, RACK_LOCAL, 6035 bytes)
18/03/10 13:51:16 INFO BlockManagerInfo: Added broadcast_2_piece0 in memory on bigdata-data5.novalocal:34005 (size: 1812.0 B, free: 2.5 GB)
18/03/10 13:51:16 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data5.novalocal:34005 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:18 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 2230 ms on bigdata-data5.novalocal (executor 1) (1/1)
18/03/10 13:51:18 INFO YarnScheduler: Removed TaskSet 0.0, whose tasks have all completed, from pool
18/03/10 13:51:18 INFO DAGScheduler: ResultStage 0 (take at SerDeUtil.scala:203) finished in 2.248 s
18/03/10 13:51:18 INFO DAGScheduler: Job 0 finished: take at SerDeUtil.scala:203, took 2.315755 s
18/03/10 13:51:18 INFO SharedState: Warehouse path is 'file:/home/da/Scripts/BasicDataGen/spark-warehouse'.
18/03/10 13:51:18 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@782e4621{/SQL,null,AVAILABLE,@Spark}
18/03/10 13:51:18 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@5872cbca{/SQL/json,null,AVAILABLE,@Spark}
18/03/10 13:51:18 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@7530166e{/SQL/execution,null,AVAILABLE,@Spark}
18/03/10 13:51:18 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@6ed087e6{/SQL/execution/json,null,AVAILABLE,@Spark}
18/03/10 13:51:18 INFO ContextHandler: Started o.s.j.s.ServletContextHandler@de3a2a4{/static/sql,null,AVAILABLE,@Spark}
18/03/10 13:51:18 INFO SparkContext: Starting job: json at NativeMethodAccessorImpl.java:0
18/03/10 13:51:18 INFO DAGScheduler: Got job 1 (json at NativeMethodAccessorImpl.java:0) with 23 output partitions
18/03/10 13:51:18 INFO DAGScheduler: Final stage: ResultStage 1 (json at NativeMethodAccessorImpl.java:0)
18/03/10 13:51:18 INFO DAGScheduler: Parents of final stage: List()
18/03/10 13:51:18 INFO DAGScheduler: Missing parents: List()
18/03/10 13:51:18 INFO DAGScheduler: Submitting ResultStage 1 (MapPartitionsRDD[5] at json at NativeMethodAccessorImpl.java:0), which has no missing parents
18/03/10 13:51:18 INFO MemoryStore: Block broadcast_3 stored as values in memory (estimated size 10.4 KB, free 4.1 GB)
18/03/10 13:51:18 INFO MemoryStore: Block broadcast_3_piece0 stored as bytes in memory (estimated size 6.4 KB, free 4.1 GB)
18/03/10 13:51:18 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on 172.20.62.40:38573 (size: 6.4 KB, free: 4.1 GB)
18/03/10 13:51:18 INFO SparkContext: Created broadcast 3 from broadcast at DAGScheduler.scala:996
18/03/10 13:51:18 INFO DAGScheduler: Submitting 23 missing tasks from ResultStage 1 (MapPartitionsRDD[5] at json at NativeMethodAccessorImpl.java:0)
18/03/10 13:51:18 INFO YarnScheduler: Adding task set 1.0 with 23 tasks
18/03/10 13:51:18 INFO TaskSetManager: Starting task 0.0 in stage 1.0 (TID 1, bigdata-data5.novalocal, executor 1, partition 0, RACK_LOCAL, 6035 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 1.0 in stage 1.0 (TID 2, bigdata-data4.novalocal, executor 10, partition 1, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 2.0 in stage 1.0 (TID 3, bigdata-data2.novalocal, executor 3, partition 2, RACK_LOCAL, 6063 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 3.0 in stage 1.0 (TID 4, bigdata-data1.novalocal, executor 5, partition 3, RACK_LOCAL, 6063 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 4.0 in stage 1.0 (TID 5, bigdata-data7.novalocal, executor 4, partition 4, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 5.0 in stage 1.0 (TID 6, bigdata-data3.novalocal, executor 6, partition 5, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 6.0 in stage 1.0 (TID 7, bigdata-data6.novalocal, executor 7, partition 6, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 7.0 in stage 1.0 (TID 8, bigdata-data8.novalocal, executor 8, partition 7, RACK_LOCAL, 6063 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 8.0 in stage 1.0 (TID 9, bigdata-data9.novalocal, executor 9, partition 8, RACK_LOCAL, 6063 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 9.0 in stage 1.0 (TID 10, bigdata-data10.novalocal, executor 2, partition 9, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 10.0 in stage 1.0 (TID 11, bigdata-data5.novalocal, executor 1, partition 10, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 11.0 in stage 1.0 (TID 12, bigdata-data4.novalocal, executor 10, partition 11, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 12.0 in stage 1.0 (TID 13, bigdata-data2.novalocal, executor 3, partition 12, RACK_LOCAL, 6038 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 13.0 in stage 1.0 (TID 14, bigdata-data1.novalocal, executor 5, partition 13, RACK_LOCAL, 6038 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 14.0 in stage 1.0 (TID 15, bigdata-data7.novalocal, executor 4, partition 14, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 15.0 in stage 1.0 (TID 16, bigdata-data3.novalocal, executor 6, partition 15, RACK_LOCAL, 6038 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 16.0 in stage 1.0 (TID 17, bigdata-data6.novalocal, executor 7, partition 16, RACK_LOCAL, 6063 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 17.0 in stage 1.0 (TID 18, bigdata-data8.novalocal, executor 8, partition 17, RACK_LOCAL, 6063 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 18.0 in stage 1.0 (TID 19, bigdata-data9.novalocal, executor 9, partition 18, RACK_LOCAL, 6033 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 19.0 in stage 1.0 (TID 20, bigdata-data10.novalocal, executor 2, partition 19, RACK_LOCAL, 6033 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 20.0 in stage 1.0 (TID 21, bigdata-data5.novalocal, executor 1, partition 20, RACK_LOCAL, 6033 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 21.0 in stage 1.0 (TID 22, bigdata-data4.novalocal, executor 10, partition 21, RACK_LOCAL, 6037 bytes)
18/03/10 13:51:18 INFO TaskSetManager: Starting task 22.0 in stage 1.0 (TID 23, bigdata-data2.novalocal, executor 3, partition 22, RACK_LOCAL, 6031 bytes)
18/03/10 13:51:18 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data5.novalocal:34005 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data7.novalocal:52755 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data10.novalocal:60579 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data8.novalocal:48332 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data3.novalocal:45293 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data2.novalocal:48413 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data1.novalocal:44851 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data4.novalocal:53033 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data6.novalocal:36422 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_3_piece0 in memory on bigdata-data9.novalocal:54909 (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data7.novalocal:52755 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data3.novalocal:45293 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data1.novalocal:44851 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data8.novalocal:48332 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data6.novalocal:36422 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data10.novalocal:60579 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data2.novalocal:48413 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data4.novalocal:53033 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data9.novalocal:54909 (size: 31.8 KB, free: 2.5 GB)
18/03/10 13:51:19 INFO TaskSetManager: Finished task 20.0 in stage 1.0 (TID 21) in 744 ms on bigdata-data5.novalocal (executor 1) (1/23)
18/03/10 13:51:21 INFO TaskSetManager: Finished task 22.0 in stage 1.0 (TID 23) in 2531 ms on bigdata-data2.novalocal (executor 3) (2/23)
18/03/10 13:51:21 INFO TaskSetManager: Finished task 19.0 in stage 1.0 (TID 20) in 2590 ms on bigdata-data10.novalocal (executor 2) (3/23)
18/03/10 13:51:21 INFO TaskSetManager: Finished task 18.0 in stage 1.0 (TID 19) in 2648 ms on bigdata-data9.novalocal (executor 9) (4/23)
18/03/10 13:51:24 INFO TaskSetManager: Finished task 0.0 in stage 1.0 (TID 1) in 5239 ms on bigdata-data5.novalocal (executor 1) (5/23)
18/03/10 13:51:32 INFO TaskSetManager: Finished task 16.0 in stage 1.0 (TID 17) in 14105 ms on bigdata-data6.novalocal (executor 7) (6/23)
18/03/10 13:51:33 INFO TaskSetManager: Finished task 17.0 in stage 1.0 (TID 18) in 14424 ms on bigdata-data8.novalocal (executor 8) (7/23)
18/03/10 13:51:38 INFO TaskSetManager: Finished task 2.0 in stage 1.0 (TID 3) in 19449 ms on bigdata-data2.novalocal (executor 3) (8/23)
18/03/10 13:51:39 INFO TaskSetManager: Finished task 3.0 in stage 1.0 (TID 4) in 20667 ms on bigdata-data1.novalocal (executor 5) (9/23)
18/03/10 13:51:42 INFO TaskSetManager: Finished task 21.0 in stage 1.0 (TID 22) in 23279 ms on bigdata-data4.novalocal (executor 10) (10/23)
18/03/10 13:51:42 INFO TaskSetManager: Finished task 14.0 in stage 1.0 (TID 15) in 24154 ms on bigdata-data7.novalocal (executor 4) (11/23)
18/03/10 13:51:43 INFO TaskSetManager: Finished task 4.0 in stage 1.0 (TID 5) in 25182 ms on bigdata-data7.novalocal (executor 4) (12/23)
18/03/10 13:51:46 INFO TaskSetManager: Finished task 5.0 in stage 1.0 (TID 6) in 27308 ms on bigdata-data3.novalocal (executor 6) (13/23)
18/03/10 13:51:55 INFO TaskSetManager: Finished task 7.0 in stage 1.0 (TID 8) in 36594 ms on bigdata-data8.novalocal (executor 8) (14/23)
18/03/10 13:51:56 INFO TaskSetManager: Finished task 8.0 in stage 1.0 (TID 9) in 37489 ms on bigdata-data9.novalocal (executor 9) (15/23)
18/03/10 13:51:57 INFO TaskSetManager: Finished task 1.0 in stage 1.0 (TID 2) in 38190 ms on bigdata-data4.novalocal (executor 10) (16/23)
18/03/10 13:52:12 INFO TaskSetManager: Finished task 6.0 in stage 1.0 (TID 7) in 53601 ms on bigdata-data6.novalocal (executor 7) (17/23)
18/03/10 13:52:46 INFO TaskSetManager: Finished task 9.0 in stage 1.0 (TID 10) in 88034 ms on bigdata-data10.novalocal (executor 2) (18/23)
18/03/10 13:52:52 INFO TaskSetManager: Finished task 11.0 in stage 1.0 (TID 12) in 93965 ms on bigdata-data4.novalocal (executor 10) (19/23)
18/03/10 13:52:54 INFO TaskSetManager: Finished task 10.0 in stage 1.0 (TID 11) in 95321 ms on bigdata-data5.novalocal (executor 1) (20/23)
18/03/10 13:53:24 INFO TaskSetManager: Finished task 12.0 in stage 1.0 (TID 13) in 125888 ms on bigdata-data2.novalocal (executor 3) (21/23)
18/03/10 13:53:37 INFO TaskSetManager: Finished task 13.0 in stage 1.0 (TID 14) in 139156 ms on bigdata-data1.novalocal (executor 5) (22/23)
18/03/10 13:53:53 INFO TaskSetManager: Finished task 15.0 in stage 1.0 (TID 16) in 155132 ms on bigdata-data3.novalocal (executor 6) (23/23)
18/03/10 13:53:53 INFO YarnScheduler: Removed TaskSet 1.0, whose tasks have all completed, from pool
18/03/10 13:53:53 INFO DAGScheduler: ResultStage 1 (json at NativeMethodAccessorImpl.java:0) finished in 155.159 s
18/03/10 13:53:53 INFO DAGScheduler: Job 1 finished: json at NativeMethodAccessorImpl.java:0, took 155.183601 s
18/03/10 13:53:54 INFO BlockManagerInfo: Removed broadcast_1_piece0 on 172.20.62.40:38573 in memory (size: 31.8 KB, free: 4.1 GB)
18/03/10 13:53:54 INFO BlockManagerInfo: Removed broadcast_2_piece0 on 172.20.62.40:38573 in memory (size: 1812.0 B, free: 4.1 GB)
18/03/10 13:53:54 INFO BlockManagerInfo: Removed broadcast_2_piece0 on bigdata-data5.novalocal:34005 in memory (size: 1812.0 B, free: 2.5 GB)
18/03/10 13:53:54 INFO SparkSqlParser: Parsing command: movement
18/03/10 13:53:54 INFO SparkSqlParser: Parsing command: SELECT row, qualifier, value FROM movement
18/03/10 13:53:55 INFO CodeGenerator: Code generated in 239.302495 ms
18/03/10 13:53:55 INFO SparkContext: Starting job: toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43
18/03/10 13:53:55 INFO DAGScheduler: Got job 2 (toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43) with 23 output partitions
18/03/10 13:53:55 INFO DAGScheduler: Final stage: ResultStage 2 (toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43)
18/03/10 13:53:55 INFO DAGScheduler: Parents of final stage: List()
18/03/10 13:53:55 INFO DAGScheduler: Missing parents: List()
18/03/10 13:53:55 INFO DAGScheduler: Submitting ResultStage 2 (MapPartitionsRDD[10] at toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43), which has no missing parents
18/03/10 13:53:55 INFO MemoryStore: Block broadcast_4 stored as values in memory (estimated size 16.1 KB, free 4.1 GB)
18/03/10 13:53:55 INFO MemoryStore: Block broadcast_4_piece0 stored as bytes in memory (estimated size 8.9 KB, free 4.1 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on 172.20.62.40:38573 (size: 8.9 KB, free: 4.1 GB)
18/03/10 13:53:55 INFO SparkContext: Created broadcast 4 from broadcast at DAGScheduler.scala:996
18/03/10 13:53:55 INFO DAGScheduler: Submitting 23 missing tasks from ResultStage 2 (MapPartitionsRDD[10] at toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43)
18/03/10 13:53:55 INFO YarnScheduler: Adding task set 2.0 with 23 tasks
18/03/10 13:53:55 INFO TaskSetManager: Starting task 0.0 in stage 2.0 (TID 24, bigdata-data9.novalocal, executor 9, partition 0, RACK_LOCAL, 6152 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 1.0 in stage 2.0 (TID 25, bigdata-data8.novalocal, executor 8, partition 1, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 2.0 in stage 2.0 (TID 26, bigdata-data2.novalocal, executor 3, partition 2, RACK_LOCAL, 6180 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 3.0 in stage 2.0 (TID 27, bigdata-data3.novalocal, executor 6, partition 3, RACK_LOCAL, 6180 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 4.0 in stage 2.0 (TID 28, bigdata-data7.novalocal, executor 4, partition 4, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 5.0 in stage 2.0 (TID 29, bigdata-data4.novalocal, executor 10, partition 5, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 6.0 in stage 2.0 (TID 30, bigdata-data1.novalocal, executor 5, partition 6, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 7.0 in stage 2.0 (TID 31, bigdata-data6.novalocal, executor 7, partition 7, RACK_LOCAL, 6180 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 8.0 in stage 2.0 (TID 32, bigdata-data5.novalocal, executor 1, partition 8, RACK_LOCAL, 6180 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 9.0 in stage 2.0 (TID 33, bigdata-data10.novalocal, executor 2, partition 9, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 10.0 in stage 2.0 (TID 34, bigdata-data9.novalocal, executor 9, partition 10, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 11.0 in stage 2.0 (TID 35, bigdata-data8.novalocal, executor 8, partition 11, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 12.0 in stage 2.0 (TID 36, bigdata-data2.novalocal, executor 3, partition 12, RACK_LOCAL, 6155 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 13.0 in stage 2.0 (TID 37, bigdata-data3.novalocal, executor 6, partition 13, RACK_LOCAL, 6155 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 14.0 in stage 2.0 (TID 38, bigdata-data7.novalocal, executor 4, partition 14, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 15.0 in stage 2.0 (TID 39, bigdata-data4.novalocal, executor 10, partition 15, RACK_LOCAL, 6155 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 16.0 in stage 2.0 (TID 40, bigdata-data1.novalocal, executor 5, partition 16, RACK_LOCAL, 6180 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 17.0 in stage 2.0 (TID 41, bigdata-data6.novalocal, executor 7, partition 17, RACK_LOCAL, 6180 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 18.0 in stage 2.0 (TID 42, bigdata-data5.novalocal, executor 1, partition 18, RACK_LOCAL, 6150 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 19.0 in stage 2.0 (TID 43, bigdata-data10.novalocal, executor 2, partition 19, RACK_LOCAL, 6150 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 20.0 in stage 2.0 (TID 44, bigdata-data9.novalocal, executor 9, partition 20, RACK_LOCAL, 6150 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 21.0 in stage 2.0 (TID 45, bigdata-data8.novalocal, executor 8, partition 21, RACK_LOCAL, 6154 bytes)
18/03/10 13:53:55 INFO TaskSetManager: Starting task 22.0 in stage 2.0 (TID 46, bigdata-data2.novalocal, executor 3, partition 22, RACK_LOCAL, 6148 bytes)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data7.novalocal:52755 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data8.novalocal:48332 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data10.novalocal:60579 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data3.novalocal:45293 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data5.novalocal:34005 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data1.novalocal:44851 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data2.novalocal:48413 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data9.novalocal:54909 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data4.novalocal:53033 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO BlockManagerInfo: Added broadcast_4_piece0 in memory on bigdata-data6.novalocal:36422 (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:53:55 INFO TaskSetManager: Finished task 19.0 in stage 2.0 (TID 43) in 490 ms on bigdata-data10.novalocal (executor 2) (1/23)
18/03/10 13:53:55 INFO TaskSetManager: Finished task 22.0 in stage 2.0 (TID 46) in 521 ms on bigdata-data2.novalocal (executor 3) (2/23)
18/03/10 13:53:55 INFO TaskSetManager: Finished task 18.0 in stage 2.0 (TID 42) in 523 ms on bigdata-data5.novalocal (executor 1) (3/23)
18/03/10 13:53:55 INFO TaskSetManager: Finished task 20.0 in stage 2.0 (TID 44) in 545 ms on bigdata-data9.novalocal (executor 9) (4/23)
18/03/10 13:53:59 INFO BlockManagerInfo: Added taskresult_24 in memory on bigdata-data9.novalocal:54909 (size: 1572.5 KB, free: 2.5 GB)
18/03/10 13:53:59 INFO TransportClientFactory: Successfully created connection to bigdata-data9.novalocal/172.20.62.39:54909 after 5 ms (0 ms spent in bootstraps)
18/03/10 13:54:00 INFO TaskSetManager: Finished task 0.0 in stage 2.0 (TID 24) in 4629 ms on bigdata-data9.novalocal (executor 9) (5/23)
18/03/10 13:54:00 INFO BlockManagerInfo: Removed taskresult_24 on bigdata-data9.novalocal:54909 in memory (size: 1572.5 KB, free: 2.5 GB)
18/03/10 13:54:07 INFO BlockManagerInfo: Added taskresult_41 in memory on bigdata-data6.novalocal:36422 (size: 3.8 MB, free: 2.5 GB)
18/03/10 13:54:07 INFO TransportClientFactory: Successfully created connection to bigdata-data6.novalocal/172.20.62.36:36422 after 2 ms (0 ms spent in bootstraps)
18/03/10 13:54:07 INFO BlockManagerInfo: Added taskresult_40 in memory on bigdata-data1.novalocal:44851 (size: 4.3 MB, free: 2.5 GB)
18/03/10 13:54:07 INFO TransportClientFactory: Successfully created connection to bigdata-data1.novalocal/172.20.62.31:44851 after 3 ms (0 ms spent in bootstraps)
18/03/10 13:54:07 INFO TaskSetManager: Finished task 17.0 in stage 2.0 (TID 41) in 11807 ms on bigdata-data6.novalocal (executor 7) (6/23)
18/03/10 13:54:07 INFO BlockManagerInfo: Removed taskresult_41 on bigdata-data6.novalocal:36422 in memory (size: 3.8 MB, free: 2.5 GB)
18/03/10 13:54:07 INFO TaskSetManager: Finished task 16.0 in stage 2.0 (TID 40) in 11863 ms on bigdata-data1.novalocal (executor 5) (7/23)
18/03/10 13:54:07 INFO BlockManagerInfo: Removed taskresult_40 on bigdata-data1.novalocal:44851 in memory (size: 4.3 MB, free: 2.5 GB)
18/03/10 13:54:12 INFO BlockManagerInfo: Added taskresult_26 in memory on bigdata-data2.novalocal:48413 (size: 5.7 MB, free: 2.5 GB)
18/03/10 13:54:12 INFO TransportClientFactory: Successfully created connection to bigdata-data2.novalocal/172.20.62.32:48413 after 2 ms (0 ms spent in bootstraps)
18/03/10 13:54:12 INFO TaskSetManager: Finished task 2.0 in stage 2.0 (TID 26) in 17418 ms on bigdata-data2.novalocal (executor 3) (8/23)
18/03/10 13:54:12 INFO BlockManagerInfo: Removed taskresult_26 on bigdata-data2.novalocal:48413 in memory (size: 5.7 MB, free: 2.5 GB)
18/03/10 13:54:13 INFO BlockManagerInfo: Added taskresult_27 in memory on bigdata-data3.novalocal:45293 (size: 6.4 MB, free: 2.5 GB)
18/03/10 13:54:13 INFO TransportClientFactory: Successfully created connection to bigdata-data3.novalocal/172.20.62.33:45293 after 2 ms (0 ms spent in bootstraps)
18/03/10 13:54:13 INFO TaskSetManager: Finished task 3.0 in stage 2.0 (TID 27) in 18216 ms on bigdata-data3.novalocal (executor 6) (9/23)
18/03/10 13:54:13 INFO BlockManagerInfo: Removed taskresult_27 on bigdata-data3.novalocal:45293 in memory (size: 6.4 MB, free: 2.5 GB)
18/03/10 13:54:15 INFO BlockManagerInfo: Added taskresult_45 in memory on bigdata-data8.novalocal:48332 (size: 8.0 MB, free: 2.5 GB)
18/03/10 13:54:15 INFO TransportClientFactory: Successfully created connection to bigdata-data8.novalocal/172.20.62.38:48332 after 2 ms (0 ms spent in bootstraps)
18/03/10 13:54:15 INFO TaskSetManager: Finished task 21.0 in stage 2.0 (TID 45) in 19999 ms on bigdata-data8.novalocal (executor 8) (10/23)
18/03/10 13:54:15 INFO BlockManagerInfo: Removed taskresult_45 on bigdata-data8.novalocal:48332 in memory (size: 8.0 MB, free: 2.5 GB)
18/03/10 13:54:17 INFO BlockManagerInfo: Added taskresult_38 in memory on bigdata-data7.novalocal:52755 (size: 7.9 MB, free: 2.5 GB)
18/03/10 13:54:17 INFO TransportClientFactory: Successfully created connection to bigdata-data7.novalocal/172.20.62.37:52755 after 2 ms (0 ms spent in bootstraps)
18/03/10 13:54:17 INFO TaskSetManager: Finished task 14.0 in stage 2.0 (TID 38) in 22159 ms on bigdata-data7.novalocal (executor 4) (11/23)
18/03/10 13:54:17 INFO BlockManagerInfo: Removed taskresult_38 on bigdata-data7.novalocal:52755 in memory (size: 7.9 MB, free: 2.5 GB)
18/03/10 13:54:17 INFO BlockManagerInfo: Added taskresult_28 in memory on bigdata-data7.novalocal:52755 (size: 8.1 MB, free: 2.5 GB)
18/03/10 13:54:17 INFO TaskSetManager: Finished task 4.0 in stage 2.0 (TID 28) in 22492 ms on bigdata-data7.novalocal (executor 4) (12/23)
18/03/10 13:54:17 INFO BlockManagerInfo: Removed taskresult_28 on bigdata-data7.novalocal:52755 in memory (size: 8.1 MB, free: 2.5 GB)
18/03/10 13:54:19 INFO BlockManagerInfo: Added taskresult_29 in memory on bigdata-data4.novalocal:53033 (size: 8.6 MB, free: 2.5 GB)
18/03/10 13:54:19 INFO TransportClientFactory: Successfully created connection to bigdata-data4.novalocal/172.20.62.34:53033 after 2 ms (0 ms spent in bootstraps)
18/03/10 13:54:19 INFO TaskSetManager: Finished task 5.0 in stage 2.0 (TID 29) in 24550 ms on bigdata-data4.novalocal (executor 10) (13/23)
18/03/10 13:54:19 INFO BlockManagerInfo: Removed taskresult_29 on bigdata-data4.novalocal:53033 in memory (size: 8.6 MB, free: 2.5 GB)
18/03/10 13:54:28 INFO BlockManagerInfo: Added taskresult_25 in memory on bigdata-data8.novalocal:48332 (size: 11.9 MB, free: 2.5 GB)
18/03/10 13:54:29 INFO TaskSetManager: Finished task 1.0 in stage 2.0 (TID 25) in 33613 ms on bigdata-data8.novalocal (executor 8) (14/23)
18/03/10 13:54:29 INFO BlockManagerInfo: Removed taskresult_25 on bigdata-data8.novalocal:48332 in memory (size: 11.9 MB, free: 2.5 GB)
18/03/10 13:54:30 INFO BlockManagerInfo: Added taskresult_32 in memory on bigdata-data5.novalocal:34005 (size: 11.7 MB, free: 2.5 GB)
18/03/10 13:54:30 INFO TransportClientFactory: Successfully created connection to bigdata-data5.novalocal/172.20.62.35:34005 after 1 ms (0 ms spent in bootstraps)
18/03/10 13:54:30 INFO TaskSetManager: Finished task 8.0 in stage 2.0 (TID 32) in 34776 ms on bigdata-data5.novalocal (executor 1) (15/23)
18/03/10 13:54:30 INFO BlockManagerInfo: Removed taskresult_32 on bigdata-data5.novalocal:34005 in memory (size: 11.7 MB, free: 2.5 GB)
18/03/10 13:54:30 INFO BlockManagerInfo: Added taskresult_31 in memory on bigdata-data6.novalocal:36422 (size: 12.5 MB, free: 2.5 GB)
18/03/10 13:54:30 INFO TaskSetManager: Finished task 7.0 in stage 2.0 (TID 31) in 35294 ms on bigdata-data6.novalocal (executor 7) (16/23)
18/03/10 13:54:30 INFO BlockManagerInfo: Removed taskresult_31 on bigdata-data6.novalocal:36422 in memory (size: 12.5 MB, free: 2.5 GB)
18/03/10 13:54:48 INFO BlockManagerInfo: Added taskresult_30 in memory on bigdata-data1.novalocal:44851 (size: 18.6 MB, free: 2.5 GB)
18/03/10 13:54:48 INFO TaskSetManager: Finished task 6.0 in stage 2.0 (TID 30) in 52733 ms on bigdata-data1.novalocal (executor 5) (17/23)
18/03/10 13:54:48 INFO BlockManagerInfo: Removed taskresult_30 on bigdata-data1.novalocal:44851 in memory (size: 18.6 MB, free: 2.5 GB)
18/03/10 13:55:20 INFO BlockManagerInfo: Added taskresult_33 in memory on bigdata-data10.novalocal:60579 (size: 29.8 MB, free: 2.5 GB)
18/03/10 13:55:20 INFO TransportClientFactory: Successfully created connection to bigdata-data10.novalocal/172.20.62.40:60579 after 1 ms (0 ms spent in bootstraps)
18/03/10 13:55:20 INFO TaskSetManager: Finished task 9.0 in stage 2.0 (TID 33) in 85126 ms on bigdata-data10.novalocal (executor 2) (18/23)
18/03/10 13:55:20 INFO BlockManagerInfo: Removed taskresult_33 on bigdata-data10.novalocal:60579 in memory (size: 29.8 MB, free: 2.5 GB)
18/03/10 13:55:22 INFO BlockManagerInfo: Added taskresult_35 in memory on bigdata-data8.novalocal:48332 (size: 31.6 MB, free: 2.5 GB)
18/03/10 13:55:22 INFO TaskSetManager: Finished task 11.0 in stage 2.0 (TID 35) in 86887 ms on bigdata-data8.novalocal (executor 8) (19/23)
18/03/10 13:55:22 INFO BlockManagerInfo: Removed taskresult_35 on bigdata-data8.novalocal:48332 in memory (size: 31.6 MB, free: 2.5 GB)
18/03/10 13:55:29 INFO BlockManagerInfo: Added taskresult_34 in memory on bigdata-data9.novalocal:54909 (size: 33.9 MB, free: 2.5 GB)
18/03/10 13:55:29 INFO TaskSetManager: Finished task 10.0 in stage 2.0 (TID 34) in 94152 ms on bigdata-data9.novalocal (executor 9) (20/23)
18/03/10 13:55:29 INFO BlockManagerInfo: Removed taskresult_34 on bigdata-data9.novalocal:54909 in memory (size: 33.9 MB, free: 2.5 GB)
18/03/10 13:56:02 INFO BlockManagerInfo: Added taskresult_36 in memory on bigdata-data2.novalocal:48413 (size: 39.8 MB, free: 2.5 GB)
18/03/10 13:56:02 INFO TaskSetManager: Finished task 12.0 in stage 2.0 (TID 36) in 127333 ms on bigdata-data2.novalocal (executor 3) (21/23)
18/03/10 13:56:02 INFO BlockManagerInfo: Removed taskresult_36 on bigdata-data2.novalocal:48413 in memory (size: 39.8 MB, free: 2.5 GB)
18/03/10 13:56:10 INFO BlockManagerInfo: Added taskresult_37 in memory on bigdata-data3.novalocal:45293 (size: 40.1 MB, free: 2.5 GB)
18/03/10 13:56:10 INFO TaskSetManager: Finished task 13.0 in stage 2.0 (TID 37) in 135289 ms on bigdata-data3.novalocal (executor 6) (22/23)
18/03/10 13:56:10 INFO BlockManagerInfo: Removed taskresult_37 on bigdata-data3.novalocal:45293 in memory (size: 40.1 MB, free: 2.5 GB)
18/03/10 13:56:27 INFO BlockManagerInfo: Added taskresult_39 in memory on bigdata-data4.novalocal:53033 (size: 52.2 MB, free: 2.4 GB)
18/03/10 13:56:27 INFO TaskSetManager: Finished task 15.0 in stage 2.0 (TID 39) in 152392 ms on bigdata-data4.novalocal (executor 10) (23/23)
18/03/10 13:56:27 INFO YarnScheduler: Removed TaskSet 2.0, whose tasks have all completed, from pool
18/03/10 13:56:27 INFO DAGScheduler: ResultStage 2 (toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43) finished in 152.405 s
18/03/10 13:56:27 INFO DAGScheduler: Job 2 finished: toPandas at /home/da/Scripts/BasicDataGen/getMovement_ver1.py:43, took 152.427407 s
18/03/10 13:56:27 INFO BlockManagerInfo: Removed taskresult_39 on bigdata-data4.novalocal:53033 in memory (size: 52.2 MB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on 172.20.62.40:38573 in memory (size: 8.9 KB, free: 4.1 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data5.novalocal:34005 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data10.novalocal:60579 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data8.novalocal:48332 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data7.novalocal:52755 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data3.novalocal:45293 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data9.novalocal:54909 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data6.novalocal:36422 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data2.novalocal:48413 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data4.novalocal:53033 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_4_piece0 on bigdata-data1.novalocal:44851 in memory (size: 8.9 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on 172.20.62.40:38573 in memory (size: 6.4 KB, free: 4.1 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data5.novalocal:34005 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data4.novalocal:53033 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data10.novalocal:60579 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data7.novalocal:52755 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data9.novalocal:54909 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data1.novalocal:44851 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data6.novalocal:36422 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data3.novalocal:45293 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data8.novalocal:48332 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 13:56:30 INFO BlockManagerInfo: Removed broadcast_3_piece0 on bigdata-data2.novalocal:48413 in memory (size: 6.4 KB, free: 2.5 GB)
18/03/10 14:07:44 INFO SparkContext: Invoking stop() from shutdown hook
18/03/10 14:07:44 INFO ServerConnector: Stopped Spark@5a0c2a7e{HTTP/1.1}{0.0.0.0:4040}
18/03/10 14:07:44 INFO SparkUI: Stopped Spark web UI at http://172.20.62.40:4040
18/03/10 14:07:44 INFO YarnClientSchedulerBackend: Interrupting monitor thread
18/03/10 14:07:44 INFO YarnClientSchedulerBackend: Shutting down all executors
18/03/10 14:07:44 INFO YarnSchedulerBackend$YarnDriverEndpoint: Asking each executor to shut down
18/03/10 14:07:44 INFO SchedulerExtensionServices: Stopping SchedulerExtensionServices
(serviceOption=None,
 services=List(),
 started=false)
18/03/10 14:07:44 INFO YarnClientSchedulerBackend: Stopped
18/03/10 14:07:44 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
18/03/10 14:07:44 INFO MemoryStore: MemoryStore cleared
18/03/10 14:07:44 INFO BlockManager: BlockManager stopped
18/03/10 14:07:44 INFO BlockManagerMaster: BlockManagerMaster stopped
18/03/10 14:07:44 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
18/03/10 14:07:44 INFO SparkContext: Successfully stopped SparkContext
18/03/10 14:07:44 INFO ShutdownHookManager: Shutdown hook called
18/03/10 14:07:44 INFO ShutdownHookManager: Deleting directory /tmp/spark-a0913fac-d59f-4d75-95c3-91e8ecb40415
18/03/10 14:07:44 INFO ShutdownHookManager: Deleting directory /tmp/spark-a0913fac-d59f-4d75-95c3-91e8ecb40415/pyspark-ad84a4a2-87b0-4179-854b-0db4ed24fa73
[da@bigdata-data10 BasicDataGen]$ ls /home/da/Data/ -l

```





















