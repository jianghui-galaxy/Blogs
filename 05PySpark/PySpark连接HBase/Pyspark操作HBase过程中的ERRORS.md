# ERROR

[TOC]





### pyenv  no such command init

解决办法:

把libexec文件夹添加到PATH   （亲测可以）

```
export PATH="$HOME/.pyenv/bin:$HOME/.pyenv/libexec:$PATH"
```





### NameError:name 'xrange' is not defined

原因是我的python版本为python 3.4，而xrange( )函数时在python 2.x中的一个函数，在Python 3中，range()的实现方式与xrange()函数相同，所以就不存在专用的xrange( )，因此，当遇到这种问题时，有两种方法可以解决这个问题。

解决办法:

> - 第一种：若你想在python 3中运行程序，将xrange( )函数全部换为range( )即可
> - 第二种：将出现此问题的程序放在python 2.x版本的环境中运行即可

`vim` 

`%s/xrange/range/gc`



### HBase Region Server Down

```
2018-03-09 01:54:53,825 INFO  [regionserver/bigdata-data10.novalocal/172.20.62.40:16020] regionserver.HRegionServer: regionserver/bigdata-data10.novalocal/172.20.62.40:16020 exiting
2018-03-09 01:54:53,826 ERROR [main] regionserver.HRegionServerCommandLine: Region server exiting
java.lang.RuntimeException: HRegionServer Aborted
        at org.apache.hadoop.hbase.regionserver.HRegionServerCommandLine.start(HRegionServerCommandLine.java:68)
        at org.apache.hadoop.hbase.regionserver.HRegionServerCommandLine.run(HRegionServerCommandLine.java:87)
        at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:76)
        at org.apache.hadoop.hbase.util.ServerCommandLine.doMain(ServerCommandLine.java:126)
        at org.apache.hadoop.hbase.regionserver.HRegionServer.main(HRegionServer.java:2801)
2018-03-09 01:54:53,830 INFO  [pool-4-thread-1] regionserver.ShutdownHook: Shutdown hook starting; hbase.shutdown.hook=true; fsShutdownHook=org.apache.hadoop.fs.FileSystem$Cache$ClientFinalizer@2555fff0
2018-03-09 01:54:53,830 INFO  [pool-4-thread-1] regionserver.ShutdownHook: Starting fs shutdown hook thread.
2018-03-09 01:54:53,835 INFO  [pool-4-thread-1] regionserver.ShutdownHook: Shutdown hook finished.


```



http://wenda.chinahadoop.cn/question/2571

Check Time  起了ntp，就好了



### ClassNotFoundException:org.apache.hadoop.io.ImmutableBytesWritable

### ClassNotFoundException:org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter

(http://blog.csdn.net/otie99/article/details/79343984)

- java.lang.ClassNotFoundException:org.apache.hadoop.io.ImmutableBytesWritable

  解决办法:启动jupyter时把hbase的lib/下面的hbase-*.jar添加到启动路径，使用--jars选项，多个jar使用逗号隔开。

  ​

- java.lang.ClassNotFoundException:org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter

  解决办法:启动jupyter时把[spark-examples_2.11-1.6.0-typesafe-001.jar](https://mvnrepository.com/artifact/org.apache.spark/spark-examples_2.11/1.6.0-typesafe-001)加入到 --jars路径中

  ​

```
su - da
/usr/hdp/current/spark2-client/bin/pyspark  --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/tmp/spark-examples_2.11-1.6.0-typesafe-001.jar

```





### Spark submmit ERROR  YARN

```
su - da
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn  --executor-memory 50G  --total-executor-cores 100 --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar   /home/da/Scripts/getMovement.py

```



```
 java.lang.IllegalArgumentException: Required executor memory (51200+5120 MB) is above the max threshold (36864 MB) of this cluster! Please check the values of 'yarn.scheduler.maximum-allocation-mb' and/or 'yarn.nodemanager.resource.memory-mb'.

```

解决办法:

`--executor-memory 20G`



在集群上提交pyspark脚本，如果是多个py脚本之间有相互依赖关系，需要将所有脚本打包为.egg文件，然后用一个.py主脚本来调用这些文件。提交的时候同时提交.egg文件和.py文件。

将多个文件.py文件打包为一个.egg文件的代码如下，需要注意：

1. 在文件夹下面必须有一个__init__.py文件，文件内部不需要有任何内容
2. py文件引用的时候要使用相对路径，避免更换了包的路径后导致程序出错。./代表当前路径，../代表上一层目录
3. 只有指定目录中的.py文件会被打包为.egg文件，其余格式的文件不会被处理
4. .比如将haha文件夹打包进hehe.egg里再放入my_path路径下，直接设定mypath为当前工作路径，import haha即可，不需要写成from hehe import haha

```
${SPARK_HOME}/bin/spark-submit \
--master yarn \
--deploy-mode cluster \
--driver-memory 8g \
--num-executors 4 \
--executor-memory 8g \
--executor-cores 4 \
--files ${SPARK_HOME}/conf/hive-site.xml \
--py-files /path/file.egg \
/path/file.py
```





### Permission denied: user=da, access=WRITE

```
 Permission denied: user=da, access=WRITE, inode="/user/da/.sparkStaging/application_1510736816673_0010":hdfs:hdfs:drwxr-xr-x

```

解决办法:

`su hdfs` 

`hdfs dfs -mkdir /user/da`

`hdfs dfs -chown da:da  /user/da`





### [Errno 13] Permission denied: '/home/.cache'

###  The Python egg cache directory is currently set to: /home/.cache/Python-Eggs.Perhaps your account does not have write access to this directory?You can change the cache directory by setting the PYTHON_EGG_CACHE environment variable to point to an accessible directory

pi.py估算pi的值，spark-submit --master yarn 报错，spark-submit --master local正常

```
 /usr/hdp/current/spark2-client/bin/spark-submit --master local /usr/hdp/current/spark2-client/examples/src/main/python/pi.py
```

```
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn /usr/hdp/current/spark2-client/examples/src/main/python/pi.py
```

```
18/03/08 17:10:54 WARN TaskSetManager: Lost task 0.0 in stage 0.0 (TID 0, bigdata-test2.novalocal, executor 2): org.apache.spark.SparkException:
Error from python worker:
  Traceback (most recent call last):
    File "/usr/lib64/python2.7/runpy.py", line 151, in _run_module_as_main
      mod_name, loader, code, fname = _get_module_details(mod_name)
    File "/usr/lib64/python2.7/runpy.py", line 101, in _get_module_details
      loader = get_loader(mod_name)
    File "/usr/lib64/python2.7/pkgutil.py", line 464, in get_loader
      return find_loader(fullname)
    File "/usr/lib64/python2.7/pkgutil.py", line 474, in find_loader
      for importer in iter_importers(fullname):
    File "/usr/lib64/python2.7/pkgutil.py", line 430, in iter_importers
      __import__(pkg)
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/__init__.py", line 44, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/context.py", line 40, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/rdd.py", line 51, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/shuffle.py", line 33, in <module>
    File "build/bdist.linux-x86_64/egg/psutil/__init__.py", line 89, in <module>
    File "build/bdist.linux-x86_64/egg/psutil/_pslinux.py", line 24, in <module>
    File "build/bdist.linux-x86_64/egg/_psutil_linux.py", line 7, in <module>
    File "build/bdist.linux-x86_64/egg/_psutil_linux.py", line 4, in __bootstrap__
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1226, in resource_filename
      self, resource_name
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1729, in get_resource_filename
      self._extract_resource(manager, self._eager_to_zip(name))
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1759, in _extract_resource
      self.egg_name, self._parts(zip_path)
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1293, in get_cache_path
      self.extraction_error()
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1273, in extraction_error
      raise err
  pkg_resources.ExtractionError: Can't extract file(s) to egg cache

  The following error occurred while trying to extract file(s)
  to the Python egg cache:

    [Errno 13] Permission denied: '/home/.cache'

  The Python egg cache directory is currently set to:

    /home/.cache/Python-Eggs

  Perhaps your account does not have write access to this directory?
  You can change the cache directory by setting the PYTHON_EGG_CACHE
  environment variable to point to an accessible directory.

PYTHONPATH was:
  /data01/hadoop/yarn/local/filecache/12/spark2-hdp-yarn-archive.tar.gz/spark-core_2.11-2.1.0.2.6.0.3-8.jar:/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip:/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/py4j-0.10.4-src.zip
java.io.EOFException
        at java.io.DataInputStream.readInt(DataInputStream.java:392)
        at org.apache.spark.api.python.PythonWorkerFactory.startDaemon(PythonWorkerFactory.scala:181)
        at org.apache.spark.api.python.PythonWorkerFactory.createThroughDaemon(PythonWorkerFactory.scala:103)
        at org.apache.spark.api.python.PythonWorkerFactory.create(PythonWorkerFactory.scala:79)
        at org.apache.spark.SparkEnv.createPythonWorker(SparkEnv.scala:117)
        at org.apache.spark.api.python.PythonRunner.compute(PythonRDD.scala:128)
        at org.apache.spark.api.python.PythonRDD.compute(PythonRDD.scala:63)
        at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:323)
        at org.apache.spark.rdd.RDD.iterator(RDD.scala:287)
        at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:87)
        at org.apache.spark.scheduler.Task.run(Task.scala:99)
        at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:322)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

18/03/08 17:10:54 INFO TaskSetManager: Starting task 0.1 in stage 0.0 (TID 2, bigdata-test2.novalocal, executor 2, partition 0, PROCESS_LOCAL, 378440 bytes)
18/03/08 17:10:54 INFO TaskSetManager: Lost task 0.1 in stage 0.0 (TID 2) on bigdata-test2.novalocal, executor 2: org.apache.spark.SparkException (
Error from python worker:
  Traceback (most recent call last):
    File "/usr/lib64/python2.7/runpy.py", line 151, in _run_module_as_main
      mod_name, loader, code, fname = _get_module_details(mod_name)
    File "/usr/lib64/python2.7/runpy.py", line 101, in _get_module_details
      loader = get_loader(mod_name)
    File "/usr/lib64/python2.7/pkgutil.py", line 464, in get_loader
      return find_loader(fullname)
    File "/usr/lib64/python2.7/pkgutil.py", line 474, in find_loader
      for importer in iter_importers(fullname):
    File "/usr/lib64/python2.7/pkgutil.py", line 430, in iter_importers
      __import__(pkg)
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/__init__.py", line 44, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/context.py", line 40, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/rdd.py", line 51, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/shuffle.py", line 33, in <module>
    File "build/bdist.linux-x86_64/egg/psutil/__init__.py", line 89, in <module>
    File "build/bdist.linux-x86_64/egg/psutil/_pslinux.py", line 24, in <module>
    File "build/bdist.linux-x86_64/egg/_psutil_linux.py", line 7, in <module>
    File "build/bdist.linux-x86_64/egg/_psutil_linux.py", line 4, in __bootstrap__
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1226, in resource_filename
      self, resource_name
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1729, in get_resource_filename
      self._extract_resource(manager, self._eager_to_zip(name))
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1759, in _extract_resource
      self.egg_name, self._parts(zip_path)
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1293, in get_cache_path
      self.extraction_error()
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1273, in extraction_error
      raise err
  pkg_resources.ExtractionError: Can't extract file(s) to egg cache

  The following error occurred while trying to extract file(s)
  to the Python egg cache:

    [Errno 13] Permission denied: '/home/.cache'

  The Python egg cache directory is currently set to:

    /home/.cache/Python-Eggs

  Perhaps your account does not have write access to this directory?
  You can change the cache directory by setting the PYTHON_EGG_CACHE
  environment variable to point to an accessible directory.

PYTHONPATH was:
  /data01/hadoop/yarn/local/filecache/12/spark2-hdp-yarn-archive.tar.gz/spark-core_2.11-2.1.0.2.6.0.3-8.jar:/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip:/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/py4j-0.10.4-src.zip
java.io.EOFException) [duplicate 1]
18/03/08 17:10:54 INFO TaskSetManager: Starting task 0.2 in stage 0.0 (TID 3, bigdata-test2.novalocal, executor 2, partition 0, PROCESS_LOCAL, 378440 bytes)
18/03/08 17:10:55 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-test4.novalocal:37362 (size: 2.9 KB, free: 366.3 MB)
18/03/08 17:10:55 INFO TaskSetManager: Lost task 0.2 in stage 0.0 (TID 3) on bigdata-test2.novalocal, executor 2: org.apache.spark.SparkException (
Error from python worker:
  Traceback (most recent call last):
    File "/usr/lib64/python2.7/runpy.py", line 151, in _run_module_as_main
      mod_name, loader, code, fname = _get_module_details(mod_name)
    File "/usr/lib64/python2.7/runpy.py", line 101, in _get_module_details
      loader = get_loader(mod_name)
    File "/usr/lib64/python2.7/pkgutil.py", line 464, in get_loader
      return find_loader(fullname)
    File "/usr/lib64/python2.7/pkgutil.py", line 474, in find_loader
      for importer in iter_importers(fullname):
    File "/usr/lib64/python2.7/pkgutil.py", line 430, in iter_importers
      __import__(pkg)
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/__init__.py", line 44, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/context.py", line 40, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/rdd.py", line 51, in <module>
    File "/data01/hadoop/yarn/local/usercache/hdfs/appcache/application_1510736816673_0034/container_e02_1510736816673_0034_01_000003/pyspark.zip/pyspark/shuffle.py", line 33, in <module>
    File "build/bdist.linux-x86_64/egg/psutil/__init__.py", line 89, in <module>
    File "build/bdist.linux-x86_64/egg/psutil/_pslinux.py", line 24, in <module>
    File "build/bdist.linux-x86_64/egg/_psutil_linux.py", line 7, in <module>
    File "build/bdist.linux-x86_64/egg/_psutil_linux.py", line 4, in __bootstrap__
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1226, in resource_filename
      self, resource_name
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1729, in get_resource_filename
      self._extract_resource(manager, self._eager_to_zip(name))
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1759, in _extract_resource
      self.egg_name, self._parts(zip_path)
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1293, in get_cache_path
      self.extraction_error()
    File "/usr/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1273, in extraction_error
      raise err
  pkg_resources.ExtractionError: Can't extract file(s) to egg cache

  The following error occurred while trying to extract file(s)
  to the Python egg cache:

    [Errno 13] Permission denied: '/home/.cache'

  The Python egg cache directory is currently set to:

    /home/.cache/Python-Eggs

  Perhaps your account does not have write access to this directory?
  You can change the cache directory by setting the PYTHON_EGG_CACHE
  environment variable to point to an accessible directory.

PYTHONPATH was:
  /data01/hadoop/yarn/local/filecache/12/spark2-hdp-yarn-archive.tar.gz/spark-core_2.11-2.
```

解决办法:

http://blog.csdn.net/imzkz/article/details/5568585

http://www.cnblogs.com/fbiswt/p/5417523.html



`vim /etc/profile`

```
export PYTHON_EGG_CACHE=/tmp/python_eggs/
export PYTHON_EGG_DIR=/tmp/python_eggs/
```

`source /etc/profile && mkdir /tmp/python_eggs/ && chmod -R 777 /tmp/python_eggs/`

Restart Yarn NodeManager

Submmit on yarn 



方案二：

解压python_eggs/下的zip文件

方案三：（http://www.georgevreilly.com/blog/2015/01/28/PythonEggCache.html）

`easy_in­stall --always-unzip pycrypto`







### Required executor memory (10240+1024 MB) is above the max threshold (8192 MB) of this cluster! Please check the values of 'yarn.scheduler.maximum-allocation-mb' and/or 'yarn.nodemanager.resource.memory-mb'



```
18/03/08 15:59:22 INFO Client: Requesting a new application from cluster with 4 NodeManagers
18/03/08 15:59:22 INFO Client: Verifying our application has not requested more than the maximum memory capability of the cluster (8192 MB per container)
18/03/08 15:59:22 ERROR SparkContext: Error initializing SparkContext.
java.lang.IllegalArgumentException: Required executor memory (10240+1024 MB) is above the max threshold (8192 MB) of this cluster! Please check the values of 'yarn.scheduler.maximum-allocation-mb' and/or 'yarn.nodemanager.resource.memory-mb'.
        at org.apache.spark.deploy.yarn.Client.verifyClusterResources(Client.scala:334)
        at org.apache.spark.deploy.yarn.Client.submitApplication(Client.scala:168)
        at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:56)
        at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:156)
        at org.apache.spark.SparkContext.<init>(SparkContext.scala:509)
        at org.apache.spark.api.java.JavaSparkContext.<init>(JavaSparkContext.scala:58)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:408)
        at py4j.reflection.MethodInvoker.invoke(MethodInvoker.java:247)
        at py4j.reflection.ReflectionEngine.invoke(ReflectionEngine.java:357)
        at py4j.Gateway.invoke(Gateway.java:236)
        at py4j.commands.ConstructorCommand.invokeConstructor(ConstructorCommand.java:80)
        at py4j.commands.ConstructorCommand.execute(ConstructorCommand.java:69)
        at py4j.GatewayConnection.run(GatewayConnection.java:214)
        at java.lang.Thread.run(Thread.java:745)
18/03/08 15:59:22 INFO ServerConnector: Stopped Spark@256c05e2{HTTP/1.1}{0.0.0.0:4041}
```



https://stackoverflow.com/questions/36872618/on-spark-1-6-0-get-org-apache-spark-sparkexception-related-with-spark-driver-ma

http://bourneli.github.io/scala/spark/2016/09/21/spark-driver-maxResultSize-puzzle.html

https://medium.com/@wx.london.cun/spark-out-of-memory-for-drivers-result-size-e7ed7532c13a

spark2-defaults.xml:    spark.driver.maxResultSize=4096M

Spark常见的两类OOM问题：Driver OOM和Executor OOM。如果发生在executor，可以通过增加分区数量，减少每个executor负载。但是此时，会增加driver的负载。所以，可能同时需要增加driver内存。定位问题时，一定要先判断是哪里出现了OOM，对症下药，才能事半功倍。



解决办法:

 Increase both `maxResultSize` and `memoryOverhead`.

`--conf "spark.driver.maxResultSize=8g spark.yarn.driver.memoryOverhead=4g"` 



Spark On YARN内存分配

http://blog.javachen.com/2015/06/09/memory-in-spark-on-yarn.html







### GC overhead limit exceeded

```
18/03/11 13:07:43 INFO BlockManagerInfo: Added broadcast_0_piece0 in memory on bigdata-data9.novalocal:42540 (size: 31.8 KB, free: 4.1 GB)
Traceback (most recent call last):
  File "/home/da/Scripts/BasicDataGen/getEDIItems.py", line 38, in <module>
    edi = spark.sql("SELECT row, qualifier, value FROM ediItem").toPandas()
  File "/usr/hdp/current/spark2-client/python/lib/pyspark.zip/pyspark/sql/dataframe.py", line 1585, in toPandas
  File "/usr/hdp/current/spark2-client/python/lib/pyspark.zip/pyspark/sql/dataframe.py", line 391, in collect
  File "/usr/hdp/current/spark2-client/python/lib/py4j-0.10.4-src.zip/py4j/java_gateway.py", line 1133, in __call__
  File "/usr/hdp/current/spark2-client/python/lib/pyspark.zip/pyspark/sql/utils.py", line 63, in deco
  File "/usr/hdp/current/spark2-client/python/lib/py4j-0.10.4-src.zip/py4j/protocol.py", line 319, in get_return_value
py4j.protocol.Py4JJavaError: An error occurred while calling o53.collectToPython.
: java.lang.OutOfMemoryError: GC overhead limit exceeded
        at org.apache.spark.sql.execution.SparkPlan$$anon$1.next(SparkPlan.scala:258)
        at org.apache.spark.sql.execution.SparkPlan$$anon$1.next(SparkPlan.scala:254)
        at scala.collection.Iterator$class.foreach(Iterator.scala:893)
        at org.apache.spark.sql.execution.SparkPlan$$anon$1.foreach(SparkPlan.scala:254)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeCollect$1.apply(SparkPlan.scala:276)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeCollect$1.apply(SparkPlan.scala:275)
        at scala.collection.IndexedSeqOptimized$class.foreach(IndexedSeqOptimized.scala:33)
        at scala.collection.mutable.ArrayOps$ofRef.foreach(ArrayOps.scala:186)
        at org.apache.spark.sql.execution.SparkPlan.executeCollect(SparkPlan.scala:275)
        at org.apache.spark.sql.Dataset$$anonfun$collectToPython$1.apply$mcI$sp(Dataset.scala:2760)
        at org.apache.spark.sql.Dataset$$anonfun$collectToPython$1.apply(Dataset.scala:2757)
        at org.apache.spark.sql.Dataset$$anonfun$collectToPython$1.apply(Dataset.scala:2757)
        at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:57)
        at org.apache.spark.sql.Dataset.withNewExecutionId(Dataset.scala:2780)
        at org.apache.spark.sql.Dataset.collectToPython(Dataset.scala:2757)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:483)
        at py4j.reflection.MethodInvoker.invoke(MethodInvoker.java:244)
        at py4j.reflection.ReflectionEngine.invoke(ReflectionEngine.java:357)
        at py4j.Gateway.invoke(Gateway.java:280)
        at py4j.commands.AbstractCommand.invokeMethod(AbstractCommand.java:132)
        at py4j.commands.CallCommand.execute(CallCommand.java:79)
        at py4j.GatewayConnection.run(GatewayConnection.java:214)
        at java.lang.Thread.run(Thread.java:745)

18/03/11 13:07:43 INFO SparkContext: Invoking stop() from shutdown hook
18/03/11 13:07:43 INFO ServerConnector: Stopped Spark@58e1efe0{HTTP/1.1}{0.0.0.0:4040}
18/03/11 13:07:43 INFO SparkUI: Stopped Spark web UI at http://172.20.62.40:4040
18/03/11 13:07:44 INFO YarnClientSchedulerBackend: Interrupting monitor thread
18/03/11 13:07:44 INFO YarnClientSchedulerBackend: Shutting down all executors
18/03/11 13:07:44 INFO YarnSchedulerBackend$YarnDriverEndpoint: Asking each executor to shut down
18/03/11 13:07:44 INFO SchedulerExtensionServices: Stopping SchedulerExtensionServices
(serviceOption=None,
 services=List(),
 started=false)
18/03/11 13:07:44 INFO YarnClientSchedulerBackend: Stopped
18/03/11 13:07:44 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
18/03/11 13:07:44 INFO MemoryStore: MemoryStore cleared
18/03/11 13:07:44 INFO BlockManager: BlockManager stopped
18/03/11 13:07:44 INFO BlockManagerMaster: BlockManagerMaster stopped
18/03/11 13:07:44 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
18/03/11 13:07:44 INFO SparkContext: Successfully stopped SparkContext
18/03/11 13:07:44 INFO ShutdownHookManager: Shutdown hook called
18/03/11 13:07:44 INFO ShutdownHookManager: Deleting directory /tmp/spark-e723d267-fc5d-499a-833d-a1f5ede9072e
18/03/11 13:07:44 INFO ShutdownHookManager: Deleting directory /tmp/spark-e723d267-fc5d-499a-833d-a1f5ede9072e/pyspark-c2cda5c3-37b5-4672-864e-1963b083a26f

```











































































### pyspark启动jupyter

Installing Jupyter with the PySpark and R kernels for Spark development：http://cleverowl.uk/2016/10/15/installing-jupyter-with-the-pyspark-and-r-kernels-for-spark-development/

http://blog.csdn.net/xmo_jiao/article/details/72674687?utm_source=itdadao&utm_medium=referral

pip install toree-0.2.0.dev1.tar.gz安装spark的kernel



- java.lang.ClassNotFoundException:org.apache.hadoop.io.ImmutableBytesWritable

  解决办法，启动jupyter时把hbase的lib/下面的hbase-*.jar添加到启动路径，使用--jars选项，多个jar使用逗号隔开。

  ​

- java.lang.ClassNotFoundException:org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter

  解决办法，启动jupyter时把/tmp/spark-examples_2.11-1.6.0-typesafe-001.jar加入到 --jars路径中

  ​

```
/usr/hdp/current/spark2-client/bin/pyspark --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/tmp/spark-examples_2.11-1.6.0-typesafe-001.jar

```



搭建pip本地源

http://blog.csdn.net/u014000150/article/details/49025173





/data1/httpd-storage/7.2/pip-source1/pypi.doubanio.com/simple/





```
awk -v ORS="," '{print $0}' test
```



SED

http://wanggen.myweb.hinet.net/ach3/ach3.html









http://ju.outofmemory.cn/entry/171843



虽然spark的资料不少，但其内核与原理介绍不多，而pyspark原理说明的就更少了。所以项目里遇到的问题，只能自己分析解决了。最近比较关注的一个问题是在broadcast大词典时，内存消耗非常大，而driver端java进程的内存用量更是达到python的2倍+。为什么呢？为了回答这个问题，还是得先搞清楚pyspark与spark的集成方式，因为一个看起来简单的spark map、reduce、broadcast，其中包含了非常多的进程、网络、线程交互，以及序列化、反序列化、压缩等等。

首先，看一下整合了pyspark后的spark runtime architecure，但在此之前，得先回顾一下没有python时简单的情况：

[![spark-arch-1](http://flykobe.com/wp-content/uploads/2015/04/spark-arch-1.jpg)](http://flykobe.com/wp-content/uploads/2015/04/spark-arch-1.jpg)

一个stage对应的tasks，从driver端传送至master，尤其选择合适的worker node，将tasks下发给Executor执行，结果直接返回给Driver Program。（根据不同的master方式，实现细节可能有区别）

那么当添加了python以后呢？可能是为了保持spark核心代码的精简以及用统一的模式未来适配到更多语言，pyspark的实现者选择了尽量不改变交互协议下的外围封装，这也造成非常多的python与java进程间的交互。看一下修改后的架构图：

[![spark-arch-2](http://flykobe.com/wp-content/uploads/2015/04/spark-arch-21.jpg)](http://flykobe.com/wp-content/uploads/2015/04/spark-arch-21.jpg)

driver与worker端都有所修改，其中白色是新增的python进程，其通过py4j开源包、socket、local file system等多种方式，与java进程交互。之所以有多种交互方式，分别是为了应对不同的使用场景。例如py4j是为了python调用scala方法；直接的socket用于java主动发起与python的交互；file system用于交互大量数据，例如broadcast的值。

有了以上的基本知识后，再来看交互序列图：

[![spark-sequence](http://flykobe.com/wp-content/uploads/2015/04/spark-sequence-1024x509.jpg)](http://flykobe.com/wp-content/uploads/2015/04/spark-sequence.jpg)

- spark-submit *your-app *.py 启动spark job，由于是python脚本，故由PythonRunner处理（注意，它是scala代码），它首先启动py4j的socket server，然后fork 子进程执行python your-app.py
- python解析your-app.py，计算执行的rdd族谱关系等，并通过py4j socket调用scala，生成PythonRDD
- driver上的scala进程，处理PythonRDD，并将任务提交给master
- master根据不同的策略，选择worker node，发起launchExecutor命令给后者
- worker node上的java slave程序，处理launchExecutor命令，最主要的是复用或fork出新的python进程（先忽略其中与daemon.py的交互，下面再说），并与python进程间建立socket连接
- worker node上的python worker进程，从socket接收job信息，开始执行，并将结果通过socket返回给java worker进程
- java worker进程通过网络将结果返回给driver上的java进程
- driver的java 进程，再返回给driver的python进程

以上忽略了很多容错、监控、数据统计的细节，但可以看到已经比较复杂了。

再来看一下daemon.py的作用，由于父子进程会COW复制内存，如果在一个已经分配了大内存的java进程上fork子python进程的话，会很容易造成OOM。所以spark的做法是 **尽量 **在java进程还没有消耗很多内存的开始阶段，就fork出一个pyspark/daemon.py子进程，后续由它再按需fork python worker进程。当然这是对unix-like（包括linux）系统有效，windows无法享受该红利。

以上涉及的细节很多，得看代码才可以真正了解，列举关键性代码路径如下，由于master没有看，所以欠奉：

- Driver端

- - core/src/main/scala/org/apache/spark/deploy/PythonRunner.scala
  - python/pyspark/rdd.py
  - core/src/main/scala/org/apache/spark/api/python/PythonRDD.scala
  - core/src/main/scala/org/apache/spark/rdd/RDD.scala
  - core/src/main/scala/org/apache/spark/SparkContext.scala
  - core/src/main/scala/org/apache/spark/scheduler/DAGScheduler.scala

- Worker端

- - core/src/main/scala/org/apache/spark/deploy/worker/Worker.scala
  - core/src/main/scala/org/apache/spark/api/python/PythonRDD.scala->compute()
  - core/src/main/scala/org/apache/spark/SparkEnv.scala->createPythonWorker()
  - python/pyspark/daemon.py
  - python/pyspark/worker.py

关于broadcast时为什么消耗内存过多的问题，您有答案了嘛？后续将单独成文予以介绍 haha！

















