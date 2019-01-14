

```
[hdfs@node111 ~]$ hdfs fsck /
Connecting to namenode via http://node111:50070/fsck?ugi=hdfs&path=%2F
FSCK started by hdfs (auth:SIMPLE) from /192.168.0.111 for path / at Mon Jan 14 20:01:52 CST 2019

/apps/hbase/data/hbase.version:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741828_1004. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/atsv2/hbase/data/data/hbase/meta/1588230740/.regioninfo:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741835_1011. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/hdp/apps/3.0.1.0-187/mapreduce/mapreduce.tar.gz:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741826_1002. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/hdp/apps/3.0.1.0-187/mapreduce/mapreduce.tar.gz:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741831_1007. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/hdp/apps/3.0.1.0-187/spark2/spark2-hdp-yarn-archive.tar.gz:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741894_1073. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/ambari-qa/.staging/job_1547369026297_0004/job.jar:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742562_1772. Target Replicas is 10 but found 3 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/ambari-qa/.staging/job_1547369026297_0004/job.split:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742563_1773. Target Replicas is 10 but found 3 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/ambari-qa/.staging/job_1547373562013_0004/job.jar:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742653_1879. Target Replicas is 10 but found 3 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/ambari-qa/.staging/job_1547373562013_0004/job.split:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742654_1880. Target Replicas is 10 but found 3 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/hdfs/.staging/job_1547373562013_0005/job.jar:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742676_1902. Target Replicas is 10 but found 3 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/hdfs/.staging/job_1547373562013_0005/job.split:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742677_1903. Target Replicas is 10 but found 3 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/oozie/share/lib/lib_20181209140216/pig/RoaringBitmap-0.4.9.jar:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742024_1210. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/oozie/share/lib/lib_20181209140216/pig/gson-2.7.jar:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742018_1204. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/oozie/share/lib/lib_20181209140216/pig/jpam-1.1.jar:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073742012_1198. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/zeppelin/notebook/2CAX5JCTA/note.json:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741861_1040. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

/user/zeppelin/notebook/2CCBNZ5YY/note.json:  Under replicated BP-1930959271-192.168.0.111-1544331236324:blk_1073741863_1042. Target Replicas is 3 but found 2 live replica(s), 0 decommissioned replica(s), 0 decommissioning replica(s).

Status: HEALTHY
 Number of data-nodes:	3
 Number of racks:		1
 Total dirs:			850
 Total symlinks:		0

Replicated Blocks:
 Total size:	1558185398 B (Total open files size: 397 B)
 Total files:	553 (Files currently being written: 9)
 Total blocks (validated):	423 (avg. block size 3683653 B) (Total open file blocks (not validated): 8)
 Minimally replicated blocks:	423 (100.0 %)
 Over-replicated blocks:	0 (0.0 %)
 Under-replicated blocks:	16 (3.782506 %)
 Mis-replicated blocks:		0 (0.0 %)
 Default replication factor:	3
 Average block replication:	2.9763594
 Missing blocks:		0
 Corrupt blocks:		0
 Missing replicas:		52 (3.9664378 %)

Erasure Coded Block Groups:
 Total size:	0 B
 Total files:	0
 Total block groups (validated):	0
 Minimally erasure-coded block groups:	0
 Over-erasure-coded block groups:	0
 Under-erasure-coded block groups:	0
 Unsatisfactory placement block groups:	0
 Average block group size:	0.0
 Missing block groups:		0
 Corrupt block groups:		0
 Missing internal blocks:	0
FSCK ended at Mon Jan 14 20:01:52 CST 2019 in 70 milliseconds


The filesystem under path '/' is HEALTHY
[hdfs@node111 ~]$ 
```

