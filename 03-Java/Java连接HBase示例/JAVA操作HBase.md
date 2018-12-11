# JAVA对HBase进行建表、增删改查



[HBase写入的各种方式总结汇总（代码）](http://www.cnblogs.com/teagnes/p/6446526.html)

1. 单条put
2. 批量put
3. 使用Mapreduce
4. bluckload




[HBase导出CSV格式数据的方法](http://www.dataguru.cn/thread-472859-1-1.html)

[HappyBase: 在 Python 中快速访问 HBase](http://classfoo.com/ccby/article/rfJ3bVG)

http://happybase.readthedocs.io/en/latest/







参考文章：

http://blog.csdn.net/liuyuan185442111/article/details/45250027  （API已经过时）

https://www.cnblogs.com/teagnes/p/6446526.html

https://www.cnblogs.com/invban/p/5667701.html



以下使用CentOS7 下的Ambari2.5 + HDP2.6  Java(TM) SE Runtime Environment (build 1.8.0_25-b17) ，HBase 1.1.2

```
javac -cp $(hbase classpath) HbaseTest2.java   ##编译.java
	
java -cp .:$(hbase classpath) HbaseTest2	  ##运行class文件
```



```java
import java.io.File;
import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.util.Bytes;

public class HbaseTest2 {
    public static Configuration configuration;
    public static Connection connection;
    public static Admin admin;

    public static void main(String[] args) throws IOException {
        listTables();
        System.out.println(">>>>>>>>>>creating table t2<<<<<<<<<");
	    createTable("t2", new String[] { "cf1", "cf2" });
        listTables();
        System.out.println(">>>>>>>>>>insert data to t2<<<<<<<<<");
        insterRow("t2", "rw1", "cf1", "q1", "val1"); 
        System.out.println(">>>>>>>>>>get data from t2<<<<<<<<<");
		getData("t2", "rw1", "cf1", "q1"); 
        System.out.println(">>>>>>>>>>scan data of t2<<<<<<<<<");
		scanData("t2", "rw1", "rw2");
        System.out.println(">>>>>>>>>>delete row of t2<<<<<<<<<");
        deleRow("t2","rw1","cf1","q1"); 
        System.out.println(">>>>>>>>>>delete  t2<<<<<<<<<");
		deleteTable("t2");
        listTables();
        
    }

    // 初始化链接
    public static void init() {
        configuration = HBaseConfiguration.create();

        configuration.set("hbase.zookeeper.property.clientPort", "2181");
        configuration.set("hbase.zookeeper.quorum", "10.110.200.161,10.110.200.162,10.110.200.163");
        configuration.set("hbase.master", "10.110.200.161:16010");
        //configuration.set("zookeeper.znode.parent","/hbase");
        
        File workaround = new File(".");
        System.getProperties().put("hadoop.home.dir",workaround.getAbsolutePath());
        
	   //
        try {
            connection = ConnectionFactory.createConnection(configuration);
            admin = connection.getAdmin();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 关闭连接
    public static void close() {
        try {
            if (null != admin)
                admin.close();
            if (null != connection)
                connection.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    // 建表
    public static void createTable(String tableNmae, String[] cols) throws IOException {

        init();
        TableName tableName = TableName.valueOf(tableNmae);

        if (admin.tableExists(tableName)) {
            System.out.println("talbe is exists!");
        } else {
            HTableDescriptor hTableDescriptor = new HTableDescriptor(tableName);
            for (String col : cols) {
                HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(col);
                hTableDescriptor.addFamily(hColumnDescriptor);
            }
            admin.createTable(hTableDescriptor);
        }
        close();
    }

    // 删表
    public static void deleteTable(String tableName) throws IOException {
        init();
        TableName tn = TableName.valueOf(tableName);
        if (admin.tableExists(tn)) {
            admin.disableTable(tn);
            admin.deleteTable(tn);
        }
        close();
    }

    // 查看已有表
    public static void listTables() throws IOException {
        init();
        HTableDescriptor hTableDescriptors[] = admin.listTables();
        for (HTableDescriptor hTableDescriptor : hTableDescriptors) {
            System.out.println(hTableDescriptor.getNameAsString());
        }
        close();
    }

    // 插入数据
    public static void insterRow(String tableName, String rowkey, String colFamily, String col, String val)
            throws IOException {
        init();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put put = new Put(Bytes.toBytes(rowkey));
        put.addColumn(Bytes.toBytes(colFamily), Bytes.toBytes(col), Bytes.toBytes(val));
        table.put(put);

        // 批量插入
        /*
         * List<Put> putList = new ArrayList<Put>(); puts.add(put);
         * table.put(putList);
         */
        table.close();
        close();
    }

    // 删除数据
    public static void deleRow(String tableName, String rowkey, String colFamily, String col) throws IOException {
        init();
        Table table = connection.getTable(TableName.valueOf(tableName)); 
        Delete delete = new Delete(Bytes.toBytes(rowkey)); 
        // 删除指定列族
        // delete.addFamily(Bytes.toBytes(colFamily)); 
        // 删除指定列
        // delete.addColumn(Bytes.toBytes(colFamily),Bytes.toBytes(col));
        table.delete(delete);
        // 批量删除
        /*
         * List<Delete> deleteList = new ArrayList<Delete>();
         * deleteList.add(delete); table.delete(deleteList);
         */
        table.close();
        close();
    }

    // 根据rowkey查找数据
    public static void getData(String tableName, String rowkey, String colFamily, String col) throws IOException {
        init();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(Bytes.toBytes(rowkey));
        // 获取指定列族数据
        // get.addFamily(Bytes.toBytes(colFamily));
        // 获取指定列数据
        // get.addColumn(Bytes.toBytes(colFamily),Bytes.toBytes(col));
        Result result = table.get(get);

        showCell(result);
        table.close();
        close();
    }

    // 格式化输出
    public static void showCell(Result result) {
        Cell[] cells = result.rawCells();
        for (Cell cell : cells) {
            System.out.println("RowName:" + new String(CellUtil.cloneRow(cell)) + " ");
            System.out.println("Timetamp:" + cell.getTimestamp() + " ");
            System.out.println("column Family:" + new String(CellUtil.cloneFamily(cell)) + " ");
            System.out.println("row Name:" + new String(CellUtil.cloneQualifier(cell)) + " ");
            System.out.println("value:" + new String(CellUtil.cloneValue(cell)) + " ");
        }
    }

    // 批量查找数据
    public static void scanData(String tableName, String startRow, String stopRow) throws IOException {
        init();
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        // scan.setStartRow(Bytes.toBytes(startRow));
        // scan.setStopRow(Bytes.toBytes(stopRow));
        ResultScanner resultScanner = table.getScanner(scan);
        for (Result result : resultScanner) {
            showCell(result);
        }
        table.close();
        close();
    }

}

```





运行结果

```
[root@fitdata161 jianghui]# java -cp .:$(hbase classpath) HbaseTest2
2018-02-06 17:09:32,114 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2018-02-06 17:09:32,294 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x4961f6af connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:32,301 INFO  [main] zookeeper.ZooKeeper: Client environment:zookeeper.version=3.4.6-8--1, built on 04/01/2017 21:27 GMT
2018-02-06 17:09:32,301 INFO  [main] zookeeper.ZooKeeper: Client environment:host.name=fitdata161
2018-02-06 17:09:32,301 INFO  [main] zookeeper.ZooKeeper: Client environment:java.version=1.8.0_25
2018-02-06 17:09:32,301 INFO  [main] zookeeper.ZooKeeper: Client environment:java.vendor=Oracle Corporation
2018-02-06 17:09:32,301 INFO  [main] zookeeper.ZooKeeper: Client environment:java.home=/usr/lib/java/jdk1.8.0_25/jre
2018-02-06 17:09:32,302 INFO  [main] zookeeper.ZooKeeper: Client environment:java.class.path=.:/usr/hdp/2.6.0.3-8/hbase/conf:/usr/lib/java/jdk1.8.0_25//lib/tools.jar:/usr/hdp/2.6.0.3-8/hbase:/usr/hdp/2.6.0.3-8/hbase/lib/activation-1.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/aopalliance-1.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/apacheds-i18n-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hbase/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hbase/lib/api-asn1-api-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hbase/lib/api-util-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hbase/lib/asm-3.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/avro-1.7.4.jar:/usr/hdp/2.6.0.3-8/hbase/lib/azure-keyvault-core-0.8.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/azure-storage-4.2.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-beanutils-1.7.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-beanutils-core-1.8.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-codec-1.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-collections-3.2.2.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-compress-1.4.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-configuration-1.6.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-daemon-1.0.13.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-digester-1.8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-el-1.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-lang3-3.4.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-logging-1.2.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-math-2.2.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-math3-3.1.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/commons-net-3.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/curator-client-2.7.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/curator-framework-2.7.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/curator-recipes-2.7.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/disruptor-3.3.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/findbugs-annotations-1.3.9-1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/gson-2.2.4.jar:/usr/hdp/2.6.0.3-8/hbase/lib/guava-12.0.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/guice-3.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/guice-servlet-3.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-annotations.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-client-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-client.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-common-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-common.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-examples-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-examples.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-hadoop2-compat.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-hadoop-compat.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-it-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-it.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-prefix-tree.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-procedure.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-protocol.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-resource-bundle.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-rest-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-rest.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-rsgroup.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-server-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-server.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-shell-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-shell.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/hbase-thrift.jar:/usr/hdp/2.6.0.3-8/hbase/lib/htrace-core-3.1.0-incubating.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-annotations-2.2.3.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-core-2.6.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-databind-2.2.3.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-jaxrs-1.9.13.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jackson-xc-1.9.13.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jamon-runtime-2.4.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jasper-compiler-5.5.23.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jasper-runtime-5.5.23.jar:/usr/hdp/2.6.0.3-8/hbase/lib/javax.inject-1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/java-xmlbuilder-0.4.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jaxb-api-2.2.2.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jaxb-impl-2.2.3-1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jcip-annotations-1.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jcodings-1.0.8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jersey-client-1.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jersey-guice-1.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jersey-json-1.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jets3t-0.9.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jettison-1.3.3.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jetty-sslengine-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hbase/lib/joni-2.1.2.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jruby-complete-1.6.8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jsch-0.1.54.jar:/usr/hdp/2.6.0.3-8/hbase/lib/json-smart-1.1.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jsp-2.1-6.1.14.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jsp-api-2.1-6.1.14.jar:/usr/hdp/2.6.0.3-8/hbase/lib/jsr305-1.3.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/junit-4.12.jar:/usr/hdp/2.6.0.3-8/hbase/lib/leveldbjni-all-1.8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/libthrift-0.9.3.jar:/usr/hdp/2.6.0.3-8/hbase/lib/log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hbase/lib/metrics-core-2.2.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/netty-3.2.4.Final.jar:/usr/hdp/2.6.0.3-8/hbase/lib/netty-all-4.0.23.Final.jar:/usr/hdp/2.6.0.3-8/hbase/lib/nimbus-jose-jwt-3.9.jar:/usr/hdp/2.6.0.3-8/hbase/lib/ojdbc6.jar:/usr/hdp/2.6.0.3-8/hbase/lib/okhttp-2.4.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/okio-1.4.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/paranamer-2.3.jar:/usr/hdp/2.6.0.3-8/hbase/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/ranger-hbase-plugin-shim-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/ranger-plugin-classloader-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hbase/lib/servlet-api-2.5-6.1.14.jar:/usr/hdp/2.6.0.3-8/hbase/lib/servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/hbase/lib/slf4j-api-1.7.7.jar:/usr/hdp/2.6.0.3-8/hbase/lib/snappy-java-1.0.4.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/spymemcached-2.11.6.jar:/usr/hdp/2.6.0.3-8/hbase/lib/xercesImpl-2.9.1.jar:/usr/hdp/2.6.0.3-8/hbase/lib/xml-apis-1.3.04.jar:/usr/hdp/2.6.0.3-8/hbase/lib/xmlenc-0.52.jar:/usr/hdp/2.6.0.3-8/hbase/lib/xz-1.0.jar:/usr/hdp/2.6.0.3-8/hbase/lib/zookeeper.jar:/usr/hdp/2.6.0.3-8/hadoop/conf:/usr/hdp/2.6.0.3-8/hadoop/lib/ojdbc6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-core-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ranger-hdfs-plugin-shim-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-annotations-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ranger-plugin-classloader-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ranger-yarn-plugin-shim-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/stax-api-1.0-2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/activation-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jaxb-impl-2.2.3-1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/apacheds-i18n-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-databind-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jcip-annotations-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/api-asn1-api-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/api-util-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/asm-3.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/xmlenc-0.52.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/avro-1.7.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-jaxrs-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/aws-java-sdk-core-1.10.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jersey-json-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/aws-java-sdk-kms-1.10.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jsp-api-2.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/aws-java-sdk-s3-1.10.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/azure-keyvault-core-0.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/json-smart-1.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/azure-storage-4.2.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jsr305-3.0.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-beanutils-1.7.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-beanutils-core-1.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/xz-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/junit-4.11.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-codec-1.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-xc-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-collections-3.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-compress-1.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/java-xmlbuilder-0.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-configuration-1.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/mockito-all-1.8.5.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-digester-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/nimbus-jose-jwt-3.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-lang3-3.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/netty-3.6.2.Final.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-logging-1.1.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/paranamer-2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-math3-3.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-net-3.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/curator-client-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/curator-framework-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/slf4j-api-1.7.10.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/curator-recipes-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/gson-2.2.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/guava-11.0.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/slf4j-log4j12-1.7.10.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/hamcrest-core-1.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jaxb-api-2.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/htrace-core-3.1.0-incubating.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/snappy-java-1.0.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/httpclient-4.5.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/httpcore-4.4.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jets3t-0.9.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jettison-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/zookeeper-3.4.6.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/joda-time-2.9.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jetty-sslengine-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jsch-0.1.54.jar:/usr/hdp/2.6.0.3-8/hadoop/.//azure-data-lake-store-sdk-2.1.4.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-annotations-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-annotations.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-auth-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-auth.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-aws-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-aws.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-azure-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-azure-datalake-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-azure-datalake.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-azure.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-common-2.7.3.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-common-tests.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-common.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-nfs-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/.//hadoop-nfs.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/./:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/asm-3.2.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/commons-codec-1.4.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/commons-daemon-1.0.13.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/commons-logging-1.1.3.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/guava-11.0.2.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/htrace-core-3.1.0-incubating.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jackson-annotations-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jackson-core-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jackson-databind-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/jsr305-3.0.0.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/leveldbjni-all-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/netty-3.6.2.Final.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/netty-all-4.0.23.Final.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/okhttp-2.4.0.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/okio-1.4.0.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/xercesImpl-2.9.1.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/xml-apis-1.3.04.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/lib/xmlenc-0.52.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/.//hadoop-hdfs-2.7.3.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/.//hadoop-hdfs-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/.//hadoop-hdfs-nfs-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/.//hadoop-hdfs-nfs.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/.//hadoop-hdfs-tests.jar:/usr/hdp/2.6.0.3-8/hadoop-hdfs/.//hadoop-hdfs.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/activation-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/aopalliance-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jsch-0.1.54.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/apacheds-i18n-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/api-asn1-api-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jetty-sslengine-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/api-util-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/asm-3.2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/avro-1.7.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/java-xmlbuilder-0.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/azure-keyvault-core-0.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/azure-storage-4.2.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/json-smart-1.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-beanutils-1.7.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/javassist-3.18.1-GA.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-beanutils-core-1.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jsp-api-2.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-codec-1.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/javax.inject-1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-collections-3.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jsr305-3.0.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-compress-1.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jersey-guice-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-configuration-1.6.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/leveldbjni-all-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-digester-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/metrics-core-3.0.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-lang3-3.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/netty-3.6.2.Final.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-logging-1.1.3.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/nimbus-jose-jwt-3.9.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-math3-3.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/commons-net-3.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/objenesis-2.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/curator-client-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/paranamer-2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/curator-framework-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/curator-recipes-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/fst-2.24.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/gson-2.2.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/guava-11.0.2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/guice-3.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/guice-servlet-3.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jersey-json-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/htrace-core-3.1.0-incubating.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/snappy-java-1.0.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/httpclient-4.5.2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/httpcore-4.4.4.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-annotations-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/stax-api-1.0-2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-core-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/xmlenc-0.52.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/xz-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-databind-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/zookeeper-3.4.6.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-jaxrs-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jets3t-0.9.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/zookeeper-3.4.6.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jackson-xc-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jaxb-api-2.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jaxb-impl-2.2.3-1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jcip-annotations-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jersey-client-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/lib/jettison-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-api-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-api.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-applications-distributedshell-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-applications-distributedshell.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-applications-unmanaged-am-launcher-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-applications-unmanaged-am-launcher.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-client-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-client.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-common.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-registry-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-registry.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-applicationhistoryservice-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-applicationhistoryservice.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-common.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-nodemanager-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-nodemanager.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-resourcemanager-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-resourcemanager.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-sharedcachemanager-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-sharedcachemanager.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-tests-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-tests.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-timeline-pluginstorage-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-timeline-pluginstorage.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-web-proxy-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-yarn/.//hadoop-yarn-server-web-proxy.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/aopalliance-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/asm-3.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/avro-1.7.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/commons-compress-1.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/guice-3.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/guice-servlet-3.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/hamcrest-core-1.3.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/javax.inject-1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/jersey-guice-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/junit-4.11.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/leveldbjni-all-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/netty-3.6.2.Final.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/paranamer-2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/snappy-java-1.0.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/lib/xz-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jaxb-impl-2.2.3-1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//activation-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-rumen.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//apacheds-i18n-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-core-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//apacheds-kerberos-codec-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-sls-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//api-asn1-api-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-sls.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//api-util-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jsr305-3.0.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//asm-3.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jcip-annotations-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//avro-1.7.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-gridmix.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//azure-keyvault-core-0.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hamcrest-core-1.3.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-beanutils-1.7.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-core.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-beanutils-core-1.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-streaming.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-codec-1.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-hs-plugins.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-collections-3.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//httpcore-4.4.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-compress-1.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-jobclient.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-configuration-1.6.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//htrace-core-3.1.0-incubating.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-digester-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//httpclient-4.5.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-httpclient-3.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jersey-json-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jackson-jaxrs-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-lang3-3.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//java-xmlbuilder-0.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-logging-1.1.3.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-math3-3.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//commons-net-3.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jackson-xc-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//curator-client-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jaxb-api-2.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//curator-framework-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jetty-sslengine-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//curator-recipes-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jets3t-0.9.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//gson-2.2.4.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jettison-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//guava-11.0.2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-hs.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-ant-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-ant.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-jobclient-tests.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-archives-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-archives.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-examples.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-auth-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-auth.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-shuffle.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-datajoin-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jsch-0.1.54.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-datajoin.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-streaming-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-distcp-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//json-smart-1.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-distcp.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-openstack-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-extras-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//jsp-api-2.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-extras.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-openstack.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//xz-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-gridmix-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-hs-plugins-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-app-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-rumen-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-app.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-common.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-hs-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-jobclient-2.7.3.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-jobclient-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-client-shuffle-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//hadoop-mapreduce-examples-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//junit-4.11.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//metrics-core-3.0.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//mockito-all-1.8.5.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//netty-3.6.2.Final.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//nimbus-jose-jwt-3.9.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//okhttp-2.4.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//okio-1.4.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//paranamer-2.3.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//snappy-java-1.0.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//stax-api-1.0-2.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//xmlenc-0.52.jar:/usr/hdp/2.6.0.3-8/hadoop-mapreduce/.//zookeeper-3.4.6.2.6.0.3-8.jar::jdbc-mysql.jar:mysql-connector-java-5.1.30-bin.jar:mysql-connector-java.jar:/usr/hdp/2.6.0.3-8/tez/tez-api-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-common-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-dag-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-examples-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-history-parser-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-job-analyzer-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-mapreduce-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-runtime-internals-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-runtime-library-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-tests-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-yarn-timeline-cache-plugin-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-yarn-timeline-history-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-yarn-timeline-history-with-acls-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/tez-yarn-timeline-history-with-fs-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/azure-data-lake-store-sdk-2.1.4.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-codec-1.4.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-collections-3.2.2.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-collections4-4.1.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/tez/lib/commons-math3-3.1.1.jar:/usr/hdp/2.6.0.3-8/tez/lib/guava-11.0.2.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-annotations-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-aws-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-azure-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-azure-datalake-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-mapreduce-client-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-mapreduce-client-core-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-yarn-server-timeline-pluginstorage-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/hadoop-yarn-server-web-proxy-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/tez/lib/jersey-client-1.9.jar:/usr/hdp/2.6.0.3-8/tez/lib/jersey-json-1.9.jar:/usr/hdp/2.6.0.3-8/tez/lib/jettison-1.3.4.jar:/usr/hdp/2.6.0.3-8/tez/lib/jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/tez/lib/jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/tez/lib/jsr305-2.0.3.jar:/usr/hdp/2.6.0.3-8/tez/lib/metrics-core-3.1.0.jar:/usr/hdp/2.6.0.3-8/tez/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/tez/lib/servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/tez/lib/slf4j-api-1.7.5.jar:/usr/hdp/2.6.0.3-8/tez/conf:/usr/hdp/2.6.0.3-8/hadoop/conf:/usr/hdp/2.6.0.3-8/hadoop/azure-data-lake-store-sdk-2.1.4.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-annotations-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-annotations.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-auth-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-auth.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-aws-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-aws.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-azure-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-azure-datalake-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-azure-datalake.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-azure.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-common-2.7.3.2.6.0.3-8-tests.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-common-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-common-tests.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-common.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-nfs-2.7.3.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/hadoop-nfs.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ojdbc6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-core-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ranger-hdfs-plugin-shim-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-annotations-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ranger-plugin-classloader-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-core-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/ranger-yarn-plugin-shim-0.7.0.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/stax-api-1.0-2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/activation-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jaxb-impl-2.2.3-1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/apacheds-i18n-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-databind-2.2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jcip-annotations-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/api-asn1-api-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jersey-core-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/api-util-1.0.0-M20.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/asm-3.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/xmlenc-0.52.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/avro-1.7.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-jaxrs-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/aws-java-sdk-core-1.10.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jersey-json-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/aws-java-sdk-kms-1.10.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jsp-api-2.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/aws-java-sdk-s3-1.10.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-mapper-asl-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/azure-keyvault-core-0.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/json-smart-1.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/azure-storage-4.2.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jsr305-3.0.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-beanutils-1.7.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jetty-util-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-beanutils-core-1.8.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/xz-1.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-cli-1.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/junit-4.11.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-codec-1.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jackson-xc-1.9.13.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-collections-3.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/log4j-1.2.17.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-compress-1.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/java-xmlbuilder-0.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-configuration-1.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/mockito-all-1.8.5.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-digester-1.8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-io-2.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/nimbus-jose-jwt-3.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-lang-2.6.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-lang3-3.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/netty-3.6.2.Final.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-logging-1.1.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/paranamer-2.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-math3-3.1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/commons-net-3.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/protobuf-java-2.5.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/curator-client-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/servlet-api-2.5.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/curator-framework-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/slf4j-api-1.7.10.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/curator-recipes-2.7.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/gson-2.2.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/guava-11.0.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/slf4j-log4j12-1.7.10.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/hamcrest-core-1.3.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jaxb-api-2.2.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/htrace-core-3.1.0-incubating.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/snappy-java-1.0.4.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/httpclient-4.5.2.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/httpcore-4.4.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jersey-server-1.9.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jets3t-0.9.0.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jettison-1.1.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/zookeeper-3.4.6.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jetty-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/joda-time-2.9.4.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jetty-sslengine-6.1.26.hwx.jar:/usr/hdp/2.6.0.3-8/hadoop/lib/jsch-0.1.54.jar:/usr/hdp/2.6.0.3-8/zookeeper/zookeeper-3.4.6.2.6.0.3-8.jar:/usr/hdp/2.6.0.3-8/zookeeper/zookeeper.jar:
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:java.io.tmpdir=/tmp
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:java.compiler=<NA>
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:os.name=Linux
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:os.arch=amd64
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:os.version=3.10.0-514.el7.x86_64
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:user.name=root
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:user.home=/root
2018-02-06 17:09:32,303 INFO  [main] zookeeper.ZooKeeper: Client environment:user.dir=/root/jianghui
2018-02-06 17:09:32,304 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@21b2e768
2018-02-06 17:09:32,326 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.162/10.110.200.162:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:32,373 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.162/10.110.200.162:2181, initiating session
2018-02-06 17:09:32,384 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.162/10.110.200.162:2181, sessionid = 0x261411ca391007c, negotiated timeout = 60000
2018-02-06 17:09:33,286 WARN  [main] shortcircuit.DomainSocketFactory: The short-circuit local reads feature cannot be used because libhadoop cannot be loaded.
T_CCV3_ILLEGAL_ACTION
T_CCV3_MOVEMENT
T_CMS_IASCFICHA
T_CMS_IASFICHA
T_CMS_IASITEM
T_CMS_IASITEMLAW
T_CMS_ILLEGAL_ACTS
T_CMS_PCSFICHA_HIS
T_CMS_PCSITEMLAW_HIS
T_CMS_PCSITEM_HIS
T_CMS_PROCESSO
T_TEST_MOVEMENT
crawler_result
pentaho_mappings
test
test2
2018-02-06 17:09:33,789 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
2018-02-06 17:09:33,789 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x261411ca391007c
2018-02-06 17:09:33,792 INFO  [main] zookeeper.ZooKeeper: Session: 0x261411ca391007c closed
2018-02-06 17:09:33,792 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
>>>>>>>>>>creating table t2<<<<<<<<<
2018-02-06 17:09:33,830 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x2c22a348 connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:33,830 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@7bd69e82
2018-02-06 17:09:33,831 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.162/10.110.200.162:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:33,831 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.162/10.110.200.162:2181, initiating session
2018-02-06 17:09:33,833 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.162/10.110.200.162:2181, sessionid = 0x261411ca391007d, negotiated timeout = 60000
2018-02-06 17:09:35,184 INFO  [main] client.HBaseAdmin: Created t2
2018-02-06 17:09:35,184 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
2018-02-06 17:09:35,185 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x261411ca391007d
2018-02-06 17:09:35,188 INFO  [main] zookeeper.ZooKeeper: Session: 0x261411ca391007d closed
2018-02-06 17:09:35,188 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
2018-02-06 17:09:35,227 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x1162410a connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:35,228 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@b09fac1
2018-02-06 17:09:35,229 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.161/10.110.200.161:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:35,229 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.161/10.110.200.161:2181, initiating session
2018-02-06 17:09:35,235 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.161/10.110.200.161:2181, sessionid = 0x161411ca3990089, negotiated timeout = 60000
T_CCV3_ILLEGAL_ACTION
T_CCV3_MOVEMENT
T_CMS_IASCFICHA
T_CMS_IASFICHA
T_CMS_IASITEM
T_CMS_IASITEMLAW
T_CMS_ILLEGAL_ACTS
T_CMS_PCSFICHA_HIS
T_CMS_PCSITEMLAW_HIS
T_CMS_PCSITEM_HIS
T_CMS_PROCESSO
T_TEST_MOVEMENT
crawler_result
pentaho_mappings
t2
test
test2
2018-02-06 17:09:35,250 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
2018-02-06 17:09:35,250 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x161411ca3990089
2018-02-06 17:09:35,254 INFO  [main] zookeeper.ZooKeeper: Session: 0x161411ca3990089 closed
2018-02-06 17:09:35,254 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
>>>>>>>>>>insert data to t2<<<<<<<<<
2018-02-06 17:09:35,290 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x545f80bf connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:35,290 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@66f66866
2018-02-06 17:09:35,291 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.162/10.110.200.162:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:35,291 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.162/10.110.200.162:2181, initiating session
2018-02-06 17:09:35,293 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.162/10.110.200.162:2181, sessionid = 0x261411ca391007e, negotiated timeout = 60000
2018-02-06 17:09:35,379 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x261411ca391007e
2018-02-06 17:09:35,381 INFO  [main] zookeeper.ZooKeeper: Session: 0x261411ca391007e closed
2018-02-06 17:09:35,381 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
>>>>>>>>>>get data from t2<<<<<<<<<
2018-02-06 17:09:35,417 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x37ff4054 connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:35,417 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@894858
2018-02-06 17:09:35,419 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.161/10.110.200.161:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:35,419 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.161/10.110.200.161:2181, initiating session
2018-02-06 17:09:35,423 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.161/10.110.200.161:2181, sessionid = 0x161411ca399008a, negotiated timeout = 60000
RowName:rw1 
Timetamp:1517908175358 
column Family:cf1 
row Name:q1 
value:val1 
2018-02-06 17:09:35,446 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x161411ca399008a
2018-02-06 17:09:35,448 INFO  [main] zookeeper.ZooKeeper: Session: 0x161411ca399008a closed
2018-02-06 17:09:35,448 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
>>>>>>>>>>scan data of t2<<<<<<<<<
2018-02-06 17:09:35,485 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x70d8de connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:35,485 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@42561fba
2018-02-06 17:09:35,486 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.162/10.110.200.162:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:35,487 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.162/10.110.200.162:2181, initiating session
2018-02-06 17:09:35,498 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.162/10.110.200.162:2181, sessionid = 0x261411ca391007f, negotiated timeout = 60000
RowName:rw1 
Timetamp:1517908175358 
column Family:cf1 
row Name:q1 
value:val1 
2018-02-06 17:09:35,518 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x261411ca391007f
2018-02-06 17:09:35,521 INFO  [main] zookeeper.ZooKeeper: Session: 0x261411ca391007f closed
2018-02-06 17:09:35,521 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
>>>>>>>>>>delete row of t2<<<<<<<<<
2018-02-06 17:09:35,559 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x253c1256 connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:35,560 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@8dfe921
2018-02-06 17:09:35,561 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.162/10.110.200.162:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:35,561 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.162/10.110.200.162:2181, initiating session
2018-02-06 17:09:35,564 INFO  [main-SendThread(10.110.200.162:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.162/10.110.200.162:2181, sessionid = 0x261411ca3910080, negotiated timeout = 60000
2018-02-06 17:09:35,585 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x261411ca3910080
2018-02-06 17:09:35,587 INFO  [main] zookeeper.ZooKeeper: Session: 0x261411ca3910080 closed
2018-02-06 17:09:35,587 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
>>>>>>>>>>delete  t2<<<<<<<<<
2018-02-06 17:09:35,625 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x3961a41a connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:35,625 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@5a4ed68f
2018-02-06 17:09:35,627 INFO  [main-SendThread(10.110.200.163:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.163/10.110.200.163:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:35,628 INFO  [main-SendThread(10.110.200.163:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.163/10.110.200.163:2181, initiating session
2018-02-06 17:09:35,630 INFO  [main-SendThread(10.110.200.163:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.163/10.110.200.163:2181, sessionid = 0x361411ca39c006c, negotiated timeout = 60000
2018-02-06 17:09:35,645 INFO  [main] client.HBaseAdmin: Started disable of t2
2018-02-06 17:09:37,868 INFO  [main] client.HBaseAdmin: Disabled t2
2018-02-06 17:09:39,088 INFO  [main] client.HBaseAdmin: Deleted t2
2018-02-06 17:09:39,088 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
2018-02-06 17:09:39,088 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x361411ca39c006c
2018-02-06 17:09:39,091 INFO  [main] zookeeper.ZooKeeper: Session: 0x361411ca39c006c closed
2018-02-06 17:09:39,091 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down
2018-02-06 17:09:39,126 INFO  [main] zookeeper.RecoverableZooKeeper: Process identifier=hconnection-0x60c16548 connecting to ZooKeeper ensemble=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181
2018-02-06 17:09:39,127 INFO  [main] zookeeper.ZooKeeper: Initiating client connection, connectString=10.110.200.161:2181,10.110.200.162:2181,10.110.200.163:2181 sessionTimeout=90000 watcher=org.apache.hadoop.hbase.zookeeper.PendingWatcher@68105edc
2018-02-06 17:09:39,128 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Opening socket connection to server 10.110.200.161/10.110.200.161:2181. Will not attempt to authenticate using SASL (unknown error)
2018-02-06 17:09:39,129 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Socket connection established to 10.110.200.161/10.110.200.161:2181, initiating session
2018-02-06 17:09:39,131 INFO  [main-SendThread(10.110.200.161:2181)] zookeeper.ClientCnxn: Session establishment complete on server 10.110.200.161/10.110.200.161:2181, sessionid = 0x161411ca399008b, negotiated timeout = 60000
T_CCV3_ILLEGAL_ACTION
T_CCV3_MOVEMENT
T_CMS_IASCFICHA
T_CMS_IASFICHA
T_CMS_IASITEM
T_CMS_IASITEMLAW
T_CMS_ILLEGAL_ACTS
T_CMS_PCSFICHA_HIS
T_CMS_PCSITEMLAW_HIS
T_CMS_PCSITEM_HIS
T_CMS_PROCESSO
T_TEST_MOVEMENT
crawler_result
pentaho_mappings
test
test2
2018-02-06 17:09:39,138 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
2018-02-06 17:09:39,138 INFO  [main] client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x161411ca399008b
2018-02-06 17:09:39,140 INFO  [main] zookeeper.ZooKeeper: Session: 0x161411ca399008b closed
2018-02-06 17:09:39,140 INFO  [main-EventThread] zookeeper.ClientCnxn: EventThread shut down

```





