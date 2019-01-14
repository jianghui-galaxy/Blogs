 

# Python3操作HBase

[TOC]



export PYTHON_EGG_CACHE=/tmp/python_eggs/
export PYTHON_EGG_DIR=/tmp/python_eggs/



source /etc/profile && mkdir /tmp/python_eggs/ && chmod -R 777 /tmp/python_eggs/



hdfs dfs -mv hdfs://172.20.62.44:8020/tmp/part-* hdfs://172.20.62.44:8020/tmp/T_CCV3/



```
os.environ['PYTHON_EGG_CACHE']='/tmp/python_eggs/'
```







环境





Sample code of pyspark on HBase

https://community.hortonworks.com/questions/96265/sample-of-pyspark-on-hbase.html



https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_spark-component-guide/content/ch_introduction-spark.html



### pyenv简介

安装不同的Python版本并不是一件容易的事情，在不同的Python版本之间来回切换更加困难，而且，多版本并存非常容易互相干扰。因此，我们需要一个名为pyenv的工具。pyenv是一个Python版本管理工具，它能够进行全局的Python版本切换，也可以为单个项目提供对应的Python版本。使用pyenv以后，可以在服务器上安装多个不同的Python版本，也可以安装不同的Python实现。

pyenv的工作原理，用一句话概括，那就是：修改系统环境变量PATH。对于系统环境变量PATH，相信大家都不陌生，里面包含了一串由冒号分隔的路径，例如`/usr/local/bin:/usr/bin:/bin`。每当在系统中执行一个命令时，例如python或pip，操作系统就会在PATH的所有路径中从左至右依次寻找对应的命令。因为是依次寻找，因此排在左边的路径具有更高的优先级。而pyenv做的，就是在PATH最前面插入一个$(pyenv root)/shims目录。这样，pyenv就可以通过控制shims目录中的Python版本号，来灵活地切换至我们所需的Python版本。

如果还想了解更多细节，可以查看pyenv的文档介绍及其源码实现。



### 安装pyenv

在一台可以连接外网的CentOS7机器上，直接从github上clone项目到本地

```
yum install git

git clone  https://github.com/pyenv/pyenv.git  ~/.pyenv
```



修改用户的.bashrc

```
cat << "EOF" >> $HOME/.bashrc

export PYENV_ROOT="${HOME}/.pyenv"
if [ -d "${PYENV_ROOT}" ]; then
  export PATH="$HOME/.pyenv/libexec:${PYENV_ROOT}/bin:${PATH}"
  eval "$(pyenv init -)"
fi
EOF

source  $HOME/.bashrc
```

如果报错：no such command init，是因为 `$HOME/.pyenv/libexec` 没有加入到PATH



pyenv用法总览：

```
[hdfs@fitdata162 ~]$ pyenv 
pyenv 1.2.1
Usage: pyenv <command> [<args>]

Some useful pyenv commands are:
   commands    List all available pyenv commands
   local       Set or show the local application-specific Python version
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using python-build
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable

See `pyenv help <command>' for information on a specific command.
For full documentation, see: https://github.com/pyenv/pyenv#readme
[hdfs@fitdata162 ~]$ 
```





### 使用pyenv安装python3

使用pyenv安装python的原理为：如果路径 $HOME/.pyenv/cache有该版本的python包，则直接解压安装，如果没有则到https://www.python.org/ftp/python/（此地址由配置文件指定）下载指定的python到~/.pyenv/cache下，然后解压安装

- 安装python3.5.5卸载2.7.14（spark2.0支持的最高python版本为3.5.5）

  ```
  pyenv  install --list   ##查看pyenv当前支持哪些Python版本，https://www.python.org/ftp/python/

  wget http://mirrors.sohu.com/python/3.5.5/Python-3.5.5.tar.xz  -P ~/.pyenv/cache

  pyenv install -v 3.5.5  ##安装Python3.5 

  pyenv versions			##查看已经安装的版本，*表示当前生效的版本
  * system (set by /home/hdfs/.pyenv/version)
    3.5.5

  pyenv uninstall 2.7.14   ##卸载2.7.14 

  ##为所有已安装的可执行文件创建 shims，如：~/.pyenv/versions//bin/，因此，每当你增删了 Python 版本或带有可执行文件的包（如 pip）以后，都应该执行一次本命令
  pyenv rehash   

  ```

  ​


- 切换python版本

  pyenv可以从三个维度来管理Python环境，简称为：当前系统（用户）、当前目录、当前shell。这三个维度的优先级从左到右依次升高，即当前系统的优先级最低、当前shell的优先级最高。三个维度的python版本分别记录在 `$HOME/.pyenv/version`  `$HOME/.python-version`  `PYENV_VERSION environment variable` 中。

  1）切换系统全局的python环境（设定的python版本只对当前用户全局有效）

  ```
  pyenv global 3.5.5		##设置全局版本为3.5.5

  pyenv versions
    system
  * 3.5.5 (set by /home/hdfs/.pyenv/version)

  [hdfs@fitdata162 ~]$  python
  Python 3.5.5 (default, Feb 26 2018, 16:56:43) 
  [GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux
  Type "help", "copyright", "credits" or "license" for more information.
  >>> 

  pyenv global --unset

  ```

  2）切换当前目录python环境（设定的python版本只对当前目录有效）

  ```
  pyenv local  3.5.5

  pyenv versions
    system
  * 3.5.5 (set by /home/hdfs/.python-version)

  pyenv local --unset

  ```

  3）切换当前shell的python环境（设定的python版本只对当前shell有效）

  ```
  pyenv shell  3.5.5

  pyenv versions
    system
  * 3.5.5 (set by PYENV_VERSION environment variable)

  pyenv shell --unset

  ```

  ​


- 安装常用的python库（注意先切换好python版本，库被安装到当前版本的python目录下）

  pyenv在安装python的时候，已经自动将pip安装好了。首先切换pip镜像源为国内镜像

  ```
  mkdir ~/.pip

  cat << "EOF" >> ~/.pip/pip.conf
  [global]
  timeout = 6000
  index-url = https://pypi.douban.com/simple
  trusted-host = pypi.douban.com
  EOF
  ```

  ​

  ```
  pip install tensorflow
  ```



### 安装jupyter notebook



（ipython的notebook参考连接：https://ipython.org/ipython-doc/3/notebook/public_server.html）

生成jupyter配置文件

`jupyter notebook --generate-config`



修改配置文件

`vim $HOME/.jupyter/jupyter_notebook_config.py`

```
c.NotebookApp.ip='*'				    #
c.NotebookApp.open_browser = False		 #
c.NotebookApp.password=123456
#c.NotebookApp.port = 8888				#端口号，也可以在下一步启动时指定
```



启动jupyter

`jupyter notebook --port 8888 --ip=10.110.200.161`



把shell终端的token http://10.110.200.161:8888/?token=d37b965eee69975f17315d68e2732c62da6855152c05cca2 复制到浏览器





### jupyter notebook安装Spark kernel

到目前为止jupyter只有一个默认的python3的kernel,而且并没有连接任何spark.使用以下命令查看

`jupyter kernelspec list`



**1、基于pyspark的jupyter notebook**

在.bashrc中加入下列选项：

`vim $HOME/.bashrc`

```
export PYSPARK_DRIVER_PYTHON=jupyter 
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"
```

在spark bin目录下测试notebook是否安装了pyspark，成功即出现如下图：

`/usr/hdp/current/spark2-client/bin/pyspark`





**2、基于Scala spark的jupyter notebook**

此处使用Apache toree给notebook安装scala kernel

[toree官网下载页](https://github.com/apache/incubator-toree),不需要解压，直接使用pip install安装

`wget https://dist.apache.org/repos/dist/dev/incubator/toree/0.2.0/snapshots/dev1/toree-pip/toree-0.2.0.dev1.tar.gz`
`pip install toree-0.2.0.dev1.tar.gz`



接着使用以下命令安装

`jupyter toree install --spark_opts='--master=yarn' --user --kernel_name=Spark2.1 --spark_home=/usr/hdp/current/spark2-client/`

--spark_opts ：

--kernel_name：

--spark_home ：



测试是否安装成功，列出kernel列表，发现有两个kernel：python3 和spark 2.0_scala，使用以下命令查看

`jupyter kernelspec list`



重新启动pyspark，把shell终端里新的token http://10.110.200.161:8888/?token=d37b965eee69975f17315d68e2732c62da6855152c05cca2 复制到浏览器











### 通过thrift连接HBase

**1、启动thrift进程**

首先要启动hbase的thrift服务,一般在HMaster上启动（Region Server和HBase Master已经启动）

```
/usr/hdp/current/hbase-master/bin/hbase-daemon.sh start thrift
```

jps可以看到ThriftServer已成功启动,然后我们就可以使用多种语言，通过Thrift来访问HBase了

```
[root@fitdata161 ~]# jps
22626 HMaster
30999 ThriftServer
9812 HRegionServer
```



pip安装hbase-thrift

```
sudo pip install thrift
sudo pip install hbase-thrift
```





**2、用python连接HBase**

以下程序获取fitdata161集群hbase中的表

```python
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol

from hbase import Hbase
from hbase.ttypes import *

transport = TSocket.TSocket('fitdata161', 9090) 
transport = TTransport.TBufferedTransport(transport) 
protocol = TBinaryProtocol.TBinaryProtocol(transport) 
client = Hbase.Client(protocol) 
transport.open() 
print(client.getTableNames()) 

```



**3、通过pyspark连接HBase**

http://blog.csdn.net/cssdongl/article/details/77750512



T_EDI_DECLARATION

 `pyspark --port 8888 --ip=172.20.64.15`



```
from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession
sc = SparkContext('yarn').appName('JJdeCompany').getOrCreate()
spark = SparkSession(sc)

#spark = SparkSession.builder.master('yarn-client').appName('JJdeCompany').getOrCreate()

##test2是hbase中的一张测试表
hbaseconf = {"hbase.zookeeper.quorum":'fitdata161,fitdata162,fitdata163',"hbase.mapreduce.inputtable":'T_EDI_DECLARATION',"zookeeper.znode.parent":"/hbase-unsecure"}  
keyConv = "org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter"
valueConv = "org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter"

hbase_EDI_rdd = spark.sparkContext.newAPIHadoopRDD(\
"org.apache.hadoop.hbase.mapreduce.TableInputFormat",\
"org.apache.hadoop.hbase.io.ImmutableBytesWritable",\
"org.apache.hadoop.hbase.client.Result",\
keyConverter=keyConv, valueConverter=valueConv, conf=hbaseconf)

print (hbase_EDI_rdd.count())


```

`/usr/hdp/current/spark2-client/bin/spark-submit --master yarn  /home/hdfs/WorkSpace/ConnHbase.py`

```
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn  --num-executors 10 --executor-memory 10G   --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/home/da/spark-examples_2.11-1.6.0-typesafe-001.jar   /home/hdfs/WorkSpace/ConnHbase.py 
```







`/usr/hdp/current/spark2-client/bin/spark-submit --master local  /home/hdfs/WorkSpace/ConnHbase.py`



 /usr/hdp/current/spark2-client/bin/spark-submit --master yarn /usr/hdp/current/spark2-client/examples/src/main/python/pi.py

 /usr/hdp/current/spark2-client/bin/spark-submit --master local /usr/hdp/current/spark2-client/examples/src/main/python/pi.py









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
--py-files /path/file.egg /path/file.py
```









































```
transport = TSocket.TSocket('localhost', 9090)
transport = TTransport.TBufferedTransport(transport)
protocol = TBinaryProtocol.TBinaryProtocol(transport)

client = Hbase.Client(protocol)
transport.open()

contents = ColumnDescriptor(name='cf:', maxVersions=1)
# client.deleteTable('test')
client.createTable('test', [contents])

print client.getTableNames()

# insert data
transport.open()

row = 'row-key1'

mutations = [Mutation(column="cf:a", value="1")]
client.mutateRow('test', row, mutations)

# get one row
tableName = 'test'
rowKey = 'row-key1'

result = client.getRow(tableName, rowKey)
print result
for r in result:
    print 'the row is ', r.row
    print 'the values is ', r.columns.get('cf:a').value
```



### phoenix连接HBase

http://blog.csdn.net/gredn/article/details/52524938







### 离线安装Python库

系统的python路径

/usr/lib/python2.7/site-packages/resource_monitoring/psutil/



**1、通过pip方式安装**

首先查看pip是否安装，这里未找到命令

```
[root@fitdata163 psutil]# pip
-bash: pip: 未找到命令

```

安装pip依赖和python2-pip

```
yum -y install python-setuptools
wget  http://mirrors.yun-idc.com/epel/7/x86_64/p/python2-pip-8.1.2-5.el7.noarch.rpm
rpm -ivh python2-pip-8.1.2-5.el7.noarch.rpm
```

查看pip版本

```
pip -V
pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)

```

到这里有两种方案来安装库：

1）下载需要的包whl文件，地址：https://pypi.python.org/pypi    

```
wget https://pypi.python.org/packages/62/5a/6ba7ea4f097343021efd721200126969c603295b1b76c5469795e2f9ea38/numpy-1.14.1-cp27-cp27mu-manylinux1_x86_64.whl#md5=0c2c6637c5c8ca639e1b7b3fa4ac64cc

pip install   /root/numpy-1.14.1-cp27-cp27mu-manylinux1_x86_64.whl


wget https://pypi.python.org/packages/source/b/bandersnatch/bandersnatch-1.5.tar.gz
tar -zxvf bandersnatch-1.5.tar.gz
```



2)搭建pip的内网源：可以把国内的https://pypi.douban.com/simple同步到本地，上传到内网

配置pip源为本地源

豆瓣：https://pypi.douban.com/simple

中国科学技术大学：http://pypi.mirrors.ustc.edu.cn/simple

```
mkdir ~/.pip

cat << "EOF" >> ~/.pip/pip.conf
[global]
timeout = 6000
index-url = https://pypi.douban.com/simple
trusted-host = pypi.douban.com

EOF
```

安装

```
pip install -i http://localhost:3134/simple/   some-package
```

```
pip install turtle --trusted-host mirrors.aliyun.com
```



**2、通过zip包解压方式安装**

```
unzip  numpy-1.14.1.zip
cd numpy-1.14.1
python setup.py install

##启动python验证

[root@fitdata132 ~]# python
Python 2.7.5 (default, Nov 20 2015, 02:00:19) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import numpy
>>> import numpy as np
>>> np.random.randn(100)
array([-0.80512041,  0.09890768,  2.48858955, -0.8917867 , -1.3706733 ,
        0.13193466, -0.75196573, -1.67466228,  0.14479676, -1.19825861,
       -0.85769927, -0.61535727,  0.99948917,  1.27245741, -0.71637388,
        1.84995659, -1.03130426, -1.48238643, -0.80583333, -0.31408132,
       -0.08688419, -1.33317426,  0.76114269,  0.16480492,  0.52756175,
        0.8728405 ,  0.3204389 ,  1.14390394,  0.61475328,  0.31275065,
       -0.79240742,  0.35233747, -0.78229307, -0.8104049 ,  0.12479561,
       -0.8476245 ,  0.07387393, -0.42521415,  0.60987544, -1.74864215,
       -1.61386963, -0.8185159 ,  0.69162984,  0.80826118,  0.15449445,
        0.02695697, -0.21675079,  0.3293489 , -0.69314852, -0.96703392,
       -0.89103496,  1.18279172, -0.3967728 , -1.00529183, -0.51351358,
       -0.20847335,  0.77124209, -0.65511737, -0.33277414,  2.20705325,
        0.73271568,  0.14306268,  0.56474339,  1.07501893,  0.98987395,
        0.49642343, -0.52131644, -0.7849968 ,  1.19514754, -1.1323867 ,
        0.63274917,  0.94949847,  0.19398685, -0.76015634, -0.50451761,
        0.90161186, -1.16071514,  0.48564091, -0.12041452, -1.99426676,
       -0.52635483,  0.98089942,  0.76508737,  1.40188727, -1.89425401,
        0.83562326, -0.32684437, -0.84248088,  1.09772661,  0.29900357,
       -0.36554102, -1.07856259, -1.50029502, -0.70790169, -0.2940151 ,
        0.91983832, -0.0941203 ,  1.21276865,  0.64794013,  1.31674398])
>>>

```





Cython-0.27.3-cp27-cp27m-manylinux1_x86_64.whl

Cython-0.27.3-cp27-cp27mu-manylinux1_x86_64.whl

```
>>> import sys
>>> print sys.maxunicode
1114111
>>> 

```



需要的库

```
##版本问题2.1.2
bleach (1.5.0)

##版本问题6.2.1
##ipython (5.5.0)

##2.2.0rc
##matplotlib (2.1.2)

absl-py (0.1.10)
bokeh (0.12.14)
Cython (0.27.3)
h5py (2.7.1)
hbase-thrift
jupyter (1.0.0)
Keras (2.1.4)
numpy (1.14.1)
pandas (0.22.0)
requests (2.18.4)
scikit-image (0.13.1)
scikit-learn (0.19.1)
scipy (1.0.0)
seaborn (0.8.1)
sklearn (0.0)
tensorflow (1.5.0)
thrift (0.11.0)

```





spark将数据写入hbase以及从hbase读取数据

http://blog.csdn.net/u013468917/article/details/52822074









 sh par_exec.sh -f hosts.all "pip install Cython"
 sh par_exec.sh -f hosts.all "pip install pandas"

 sh par_exec.sh -f hosts.all "pip install tensorflow"

 sh par_exec.sh -f hosts.all "pip install Keras"

 sh par_exec.sh -f hosts.all "pip install scikit-image"

 sh par_exec.sh -f hosts.all "pip install scikit-learn"

 sh par_exec.sh -f hosts.all "pip install scipy"

 sh par_exec.sh -f hosts.all "pip install seaborn"

 sh par_exec.sh -f hosts.all "pip install sklearn"

 sh par_exec.sh -f hosts.all "pip install thrift"

 sh par_exec.sh -f hosts.all "pip install requests"

 sh par_exec.sh -f hosts.all "pip install numpy"

 sh par_exec.sh -f hosts.all "pip install absl-py"

sh par_exec.sh -f hosts.all "pip install matplotlib"

sh par_exec.sh -f hosts.all "pip install bleach"

sh par_exec.sh -f hosts.all "pip install ipython"

 sh par_exec.sh -f hosts.all "pip install bokeh"

 sh par_exec.sh -f hosts.all "pip install h5py"

 sh par_exec.sh -f hosts.all "pip install hbase-thrift"

 sh par_exec.sh -f hosts.all "pip install jupyter"













Installed



 sh par_exec.sh -f hosts.all "pip install /root/pip/absl-py-0.1.10.tar.gz"

 sh par_exec.sh -f hosts.all "pip install /root/pip/Cython-0.27.3-cp27-cp27mu-manylinux1_x86_64.whl"

 sh par_exec.sh -f hosts.all "pip install /root/pip/h5py-2.7.1-cp27-cp27mu-manylinux1_x86_64.whl"

 sh par_exec.sh -f hosts.all "pip install /root/pip/numpy-1.14.1-cp27-cp27mu-manylinux1_x86_64.whl"

sh par_exec.sh -f hosts.all "pip install /root/pip/scipy-1.0.0-cp27-cp27mu-manylinux1_x86_64.whl"

sh par_exec.sh -f hosts.all "pip install /root/pip/scikit_learn-0.19.1-cp27-cp27mu-manylinux1_x86_64.whl"

 sh par_exec.sh -f hosts.all "pip install /root/pip/sklearn-0.0.tar.gz"

 sh par_exec.sh -f hosts.all "pip install /root/pip/thrift-0.11.0.tar.gz"





Dependence

 sh par_exec.sh -f hosts.all "pip install /root/pip/bleach-2.1.2.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/bokeh-0.12.14.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/hbase-thrift-0.20.4.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/ipython-6.2.1.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/jupyter-1.0.0.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/Keras-2.1.4.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/matplotlib-2.2.0rc1-cp27-cp27mu-manylinux1_x86_64.whl"
 sh par_exec.sh -f hosts.all "pip install /root/pip/pandas-0.22.0-cp27-cp27mu-manylinux1_x86_64.whl"
 sh par_exec.sh -f hosts.all "pip install /root/pip/requests-2.18.4.tar.gz"
 sh par_exec.sh -f hosts.all "pip install /root/pip/scikit_image-0.13.1-cp27-cp27mu-manylinux1_x86_64.whl"
 sh par_exec.sh -f hosts.all "pip install /root/pip/seaborn-0.8.1.tar.gz"

 sh par_exec.sh -f hosts.all "pip install /root/pip/tensorflow-1.6.0-cp27-cp27mu-manylinux1_x86_64.whl"









 sh par_exec.sh -f hosts.all "pip install  http://172.20.62.44/7.2/Pyspark/argparse-1.4.0-py2.py3-none-any.whl"

 sh par_exec.sh -f hosts.all "pip install  http://172.20.62.44/7.2/Pyspark/python-dateutil-2.6.1.tar.gz"

 







### Pandas



  Downloading pandas-0.22.0-cp27-cp27mu-manylinux1_x86_64.whl (24.3MB)

    100% |████████████████████████████████| 24.3MB 28kB/s 
Collecting python-dateutil (from pandas)
  Using cached python_dateutil-2.6.1-py2.py3-none-any.whl
Collecting numpy>=1.9.0 (from pandas)
  Downloading numpy-1.14.1-cp27-cp27mu-manylinux1_x86_64.whl (12.1MB)
    100% |████████████████████████████████| 12.1MB 42kB/s 
Collecting pytz>=2011k (from pandas)
  Using cached pytz-2018.3-py2.py3-none-any.whl
Collecting six>=1.5 (from python-dateutil->pandas)
  Using cached six-1.11.0-py2.py3-none-any.whl







### TensorFlow



  Downloading tensorboard-1.6.0-py2-none-any.whl (3.0MB)

    100% |████████████████████████████████| 3.1MB 68kB/s 
Collecting grpcio>=1.8.6 (from tensorflow==1.6.0)
  Downloading grpcio-1.10.0-cp27-cp27mu-manylinux1_x86_64.whl (7.4MB)
    100% |████████████████████████████████| 7.4MB 65kB/s 
Requirement already satisfied: six>=1.10.0 in ./.pyenv/versions/2.7.5/lib/python2.7/site-packages (from tensorflow==1.6.0)
Collecting absl-py>=0.1.6 (from tensorflow==1.6.0)
  Using cached absl-py-0.1.10.tar.gz
Collecting termcolor>=1.1.0 (from tensorflow==1.6.0)
  Downloading termcolor-1.1.0.tar.gz
Collecting gast>=0.2.0 (from tensorflow==1.6.0)
  Downloading gast-0.2.0.tar.gz
Collecting enum34>=1.1.6 (from tensorflow==1.6.0)
  Downloading enum34-1.1.6-py2-none-any.whl
Collecting mock>=2.0.0 (from tensorflow==1.6.0)
  Downloading mock-2.0.0-py2.py3-none-any.whl (56kB)
    100% |████████████████████████████████| 61kB 113kB/s 
Requirement already satisfied: wheel in ./.pyenv/versions/2.7.5/lib/python2.7/site-packages (from tensorflow==1.6.0)
Collecting backports.weakref>=1.0rc1 (from tensorflow==1.6.0)
  Downloading backports.weakref-1.0.post1-py2.py3-none-any.whl
Collecting astor>=0.6.0 (from tensorflow==1.6.0)
  Downloading astor-0.6.2-py2.py3-none-any.whl
Collecting protobuf>=3.4.0 (from tensorflow==1.6.0)
   Downloading protobuf-3.5.1-cp27-cp27mu-manylinux1_x86_64.whl (6.4MB)

    100% |████████████████████████████████| 6.4MB 29kB/s 
Collecting futures>=3.1.1; python_version < "3" (from tensorboard<1.7.0,>=1.6.0->tensorflow==1.6.0)
  Downloading futures-3.2.0-py2-none-any.whl
Collecting werkzeug>=0.11.10 (from tensorboard<1.7.0,>=1.6.0->tensorflow==1.6.0)
  Using cached Werkzeug-0.14.1-py2.py3-none-any.whl
Collecting html5lib==0.9999999 (from tensorboard<1.7.0,>=1.6.0->tensorflow==1.6.0)
  Using cached html5lib-0.9999999.tar.gz
Collecting bleach==1.5.0 (from tensorboard<1.7.0,>=1.6.0->tensorflow==1.6.0)
  Using cached bleach-1.5.0-py2.py3-none-any.whl
Collecting markdown>=2.6.8 (from tensorboard<1.7.0,>=1.6.0->tensorflow==1.6.0)
  Using cached Markdown-2.6.11-py2.py3-none-any.whl
Collecting funcsigs>=1; python_version < "3.3" (from mock>=2.0.0->tensorflow==1.6.0)
  Downloading funcsigs-1.0.2-py2.py3-none-any.whl
Collecting pbr>=0.11 (from mock>=2.0.0->tensorflow==1.6.0)
  Downloading pbr-3.1.1-py2.py3-none-any.whl (99kB)
    100% |████████████████████████████████| 102kB 38kB/s 
Requirement already satisfied: setuptools in ./.pyenv/versions/2.7.5/lib/python2.7/site-packages (from protobuf>=3.4.0->tensorflow==1.6.0)





```
[hdfs@fitdata161 ~]$ pip install Keras
Collecting Keras
/home/hdfs/.pyenv/versions/2.7.5/lib/python2.7/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:318: SNIMissingWarning: An HTTPS request has been made, but the SNI (Subject Name Indication) extension to TLS is not available on this platform. This may cause the server to present an incorrect TLS certificate, which can cause validation failures. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#snimissingwarning.
  SNIMissingWarning
/home/hdfs/.pyenv/versions/2.7.5/lib/python2.7/site-packages/pip/_vendor/requests/packages/urllib3/util/ssl_.py:122: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
  Using cached Keras-2.1.4-py2.py3-none-any.whl
Requirement already satisfied: numpy>=1.9.1 in ./.pyenv/versions/2.7.5/lib/python2.7/site-packages (from Keras)
Collecting scipy>=0.14 (from Keras)
  Downloading scipy-1.0.0-cp27-cp27mu-manylinux1_x86_64.whl (46.7MB)
    100% |████████████████████████████████| 46.7MB 18kB/s 
Requirement already satisfied: six>=1.9.0 in ./.pyenv/versions/2.7.5/lib/python2.7/site-packages (from Keras)
Collecting pyyaml (from Keras)
  Using cached PyYAML-3.12.tar.gz
Building wheels for collected packages: pyyaml
  Running setup.py bdist_wheel for pyyaml ... done
  Stored in directory: /home/hdfs/.cache/pip/wheels/2c/f7/79/13f3a12cd723892437c0cfbde1230ab4d82947ff7b3839a4fc
Successfully built pyyaml
Installing collected packages: scipy, pyyaml, Keras
Successfully installed Keras-2.1.4 pyyaml-3.12 scipy-1.0.0

```



tensorboard-1.6.0-py2-none-any.whl (3.0MB)







### jupyter Pyspark

http://jupyter-notebook.readthedocs.io/en/latest/public_server.html





`su -  da`

```
jupyter notebook --generate-config
```

修改配置文件

`vim $HOME/.jupyter/jupyter_notebook_config.py`

```


c.NotebookApp.ip=172.20.62.40

#端口号，也可以启动jupyter时指定
c.NotebookApp.port = 8888
c.NotebookApp.open_browser = False
c.NotebookApp.password_required = True
c.NotebookApp.password=123456
c.NotebookApp.allow_password_change = True

```



`vim $HOME/.bashrc`

```
export PYSPARK_PYTHON=python2
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
export SPARK_HOME=/usr/hdp/current/spark2-client/
export PATH==$SPARK_HOME/bin:$PATH

```

`source  $HOME/.bashrc`



`jupyter notebook password`

123456

123456







`pyspark`

```
[da@bigdata-data10 ~]$ pyspark
Traceback (most recent call last):
  File "/usr/bin/jupyter-notebook", line 11, in <module>
    sys.exit(main())
  File "/usr/lib/python2.7/site-packages/jupyter_core/application.py", line 266, in launch_instance
    return super(JupyterApp, cls).launch_instance(argv=argv, **kwargs)
  File "/usr/lib/python2.7/site-packages/traitlets/config/application.py", line 657, in launch_instance
    app.initialize(argv)
  File "<decorator-gen-7>", line 2, in initialize
  File "/usr/lib/python2.7/site-packages/traitlets/config/application.py", line 87, in catch_config_error
    return method(app, *args, **kwargs)
  File "/usr/lib/python2.7/site-packages/notebook/notebookapp.py", line 1505, in initialize
    self.init_configurables()
  File "/usr/lib/python2.7/site-packages/notebook/notebookapp.py", line 1209, in init_configurables
    connection_dir=self.runtime_dir,
  File "/usr/lib/python2.7/site-packages/traitlets/traitlets.py", line 556, in __get__
    return self.get(obj, cls)
  File "/usr/lib/python2.7/site-packages/traitlets/traitlets.py", line 535, in get
    value = self._validate(obj, dynamic_default())
  File "/usr/lib/python2.7/site-packages/jupyter_core/application.py", line 99, in _runtime_dir_default
    ensure_dir_exists(rd, mode=0o700)
  File "/usr/lib/python2.7/site-packages/jupyter_core/utils/__init__.py", line 13, in ensure_dir_exists
    os.makedirs(path, mode=mode)
  File "/usr/lib64/python2.7/os.py", line 157, in makedirs
    mkdir(name, mode)
OSError: [Errno 13] Permission denied: '/run/user/0/jupyter'
[da@bigdata-data10 ~]$ ll /r

```

chown -R da:da  /run/user/0/

chown -R da:da /run/user/



启动jupyter

`su - da`

```
nohup  pyspark  --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/home/da/spark-examples_2.11-1.6.0-typesafe-001.jar  &
```







ConnHBase

T_EDI_DECLARATION



```

hbaseconf = {"hbase.zookeeper.quorum":'172.20.64.15,172.20.64.16,172.20.64.17',"hbase.mapreduce.inputtable":'T_EDI_DECLARATION',"zookeeper.znode.parent":"/hbase-unsecure"}  
keyConv = "org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter"
valueConv = "org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter"

hbase_T_EDI_DECLARATION_rdd = spark.sparkContext.newAPIHadoopRDD("org.apache.hadoop.hbase.mapreduce.TableInputFormat","org.apache.hadoop.hbase.io.ImmutableBytesWritable",\
"org.apache.hadoop.hbase.client.Result",keyConverter=keyConv, valueConverter=valueConv, conf=hbaseconf)

print (hbase_T_EDI_DECLARATION_rdd.count())
```



RDD to DataFrame

```
hbase_T_EDI_DECLARATION_df = hbase_T_EDI_DECLARATION_rdd.toDF()
```





















rdd api :http://homepage.cs.latrobe.edu.au/zhe/ZhenHeSparkRDDAPIExamples.html



http://blog.einext.com/apache-spark/pyspark-working-with-hbase



https://strongyoung.gitbooks.io/hbase-reference-guide/content/hbase_spark/86sparksqldataframes.html







``` 
hbase_T_CCV3_ILLEGAL_ACTION_rdd.keys().count()
rdd   .keys()  .values() 
```



```python
from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession
sc = SparkContext('yarn').appName('Company').getOrCreate()
spark = SparkSession(sc)

hb_zk_quorum="hbase.zookeeper.quorum"
hb_zk_ip='172.20.62.31,172.20.62.43,172.20.62.44'

inputable="hbase.mapreduce.inputtable"
hb_TableName='T_TEST_DEC'

#Scan=scan
#columns='o:fname o:email'

znode_parent="zookeeper.znode.parent"
znode_parent_val="/hbase-unsecure"

MyHbaseConf = {hb_zk_quorum:hb_zk_ip, inputable:hb_TableName, znode_parent:znode_parent_val} 
#configuration = {}
#configuration['hbase.mapreduce.inputtable'] = hb_TableName
#configuration['hbase.zookeeper.quorum'] = hb_zk_ip


MyInputFormatClass = 'org.apache.hadoop.hbase.mapreduce.TableInputFormat'

MyKeyClass = 'org.apache.hadoop.hbase.io.ImmutableBytesWritable'
MyValueClass = 'org.apache.hadoop.hbase.client.Result'

MyKeyConverter = 'org.apache.spark.examples.pythonconverters.ImmutableBytesWritableToStringConverter'
MyValueConverter  = "org.apache.spark.examples.pythonconverters.HBaseResultToStringConverter"


RDD = sc.newAPIHadoopRDD(inputFormatClass = MyInputFormatClass, KeyClass = MyKeyClass, ValueClass = MyValueClass,\
						 keyConverter = MyKeyConverter, valueConverter = MyValueConverter, conf = MyHbaseConf)

##RDD=SparkContext.newAPIHadoopRDD(MyInputFormatClass, MyKeyClass, MyValueClass, MyKeyConverter,  MyValueConverter, MyHbaseConf)
##

ARR = RDD.values().flatMap(lambda x: x.split('\n'))
DF = spark.read.json(ARR)

DF.select("qualifier","row","value").show(100,truncate = False)

```



```
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn  --num-executors 10 --executor-memory 6G   --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/home/hdfs/spark-examples_2.11-1.6.0-typesafe-001.jar    /home/hdfs/readEDIGoodsRDD.py
```



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















HBase Region Server Down

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





spark.driver.maxResultSize

```
18/03/09 22:09:16 INFO YarnScheduler: Removed TaskSet 2.0, whose tasks have all completed, from pool
Traceback (most recent call last):
  File "/home/da/Scripts/BasicDataGen/getMovement.py", line 34, in <module>
    movement = spark.sql("SELECT row, qualifier, value FROM movement").toPandas()
  File "/usr/hdp/current/spark2-client/python/lib/pyspark.zip/pyspark/sql/dataframe.py", line 1585, in toPandas
  File "/usr/hdp/current/spark2-client/python/lib/pyspark.zip/pyspark/sql/dataframe.py", line 391, in collect
  File "/usr/hdp/current/spark2-client/python/lib/py4j-0.10.4-src.zip/py4j/java_gateway.py", line 1133, in __call__
  File "/usr/hdp/current/spark2-client/python/lib/pyspark.zip/pyspark/sql/utils.py", line 63, in deco
  File "/usr/hdp/current/spark2-client/python/lib/py4j-0.10.4-src.zip/py4j/protocol.py", line 319, in get_return_value
py4j.protocol.Py4JJavaError: An error occurred while calling o51.collectToPython.
: org.apache.spark.SparkException: Job aborted due to stage failure: Total size of serialized results of 14 tasks (1141.8 MB) is bigger than spark.driver.maxResultSize (1024.0 MB)
        at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1435)

```

http://bourneli.github.io/scala/spark/2016/09/21/spark-driver-maxResultSize-puzzle.html

https://medium.com/@wx.london.cun/spark-out-of-memory-for-drivers-result-size-e7ed7532c13a

spark2-defaults.xml:    spark.driver.maxResultSize=4096M

Spark常见的两类OOM问题：Driver OOM和Executor OOM。如果发生在executor，可以通过增加分区数量，减少每个executor负载。但是此时，会增加driver的负载。所以，可能同时需要增加driver内存。定位问题时，一定要先判断是哪里出现了OOM，对症下药，才能事半功倍。







Spark submmit ERROR

```
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn  --executor-memory 20G  --total-executor-cores 100 --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar   /home/da/Scripts/getMovement.py
```



```
 java.lang.IllegalArgumentException: Required executor memory (51200+5120 MB) is above the max threshold (36864 MB) of this cluster! Please check the values of 'yarn.scheduler.maximum-allocation-mb' and/or 'yarn.nodemanager.resource.memory-mb'.
```







real	0m16.485s
user	0m14.736s
sys	0m0.905s



real	0m21.890s
user	0m15.048s
sys	0m0.877s











# 在CentOS下用PySpark连接HBase

http://blog.csdn.net/otie99/article/details/79343984

http://colobu.com/2014/12/09/spark-submitting-applications/

```
/usr/hdp/current/spark2-client/bin/spark-submit --master yarn --num-executors 10 --executor-cores 4 --executor-memory 8g --driver-memory 8g   --conf "spark.driver.maxResultSize=8g" --conf "spark.yarn.driver.memoryOverhead=4g"   --jars /usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-annotations-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-client-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-examples-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-hadoop-compat-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-it-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-prefix-tree-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-procedure-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-resource-bundle-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rest-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-rsgroup-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-server-1.1.2.2.6.0.3-8-tests.jar,/usr/hdp/current/hbase-client/lib/hbase-shell-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/hbase-thrift-1.1.2.2.6.0.3-8.jar,/usr/hdp/current/hbase-client/lib/spark-examples_2.11-1.6.0-typesafe-001.jar     /home/da/Scripts/BasicDataGen/getMovement.py
```



--master yarn 

--num-executors 10 

--executor-cores 5 

--executor-memory 4g 

--driver-memory 15g

--total-executor-cores 100

--conf spark.yarn.executor.memoryOverhead=409











OOM









































































 useradd -G hadoop da













ACTION   PERMIT_ID   MOV_TIME DIRECTION  PORT_UID  ACTION











# Sizing and Configuring your Hadoop Cluster

https://datahub.packtpub.com/tutorials/sizing-configuring-hadoop-cluster/







tmpfs   /run/user/1001

```
~ $ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           1.2G   20K  1.2G   1% /run/user/1000
```

https://unix.stackexchange.com/questions/162900/what-is-this-folder-run-user-1000



















































hbase_master_heapsize   896   2048



metrics_collector_heapsize  512  3G
























