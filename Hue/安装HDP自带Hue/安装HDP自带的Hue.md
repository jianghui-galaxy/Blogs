# 安装HDP自带的hue

[TOC]





### 说明

HDP自带的hue只能安装在CentOS6/RHEL6/Oracle Linux 6/SLES，在CentOS7上安装会报各种错

Hue既可以安装在HDP集群内部安装，也可以在集群外部单独找一个CentOS6的服务器安装，（HDP安装在CentOS7，Hue安装在CentOS6，也是可以的）





HDP自带的hue文件路径

 ![hue](imgs/hue.png)

 ![hue-rpm](imgs/hue-rpm.png)



### 安装

首先在/etc/yum.repos.d/配置repo文件，新建一个HDP.repo文件，baseurl指向hue的rpm包所在地址 `vim  /etc/yum.repos.d/HDP.repo`

```
[HDP-2.6]
name=HDP-2.6
baseurl=http://10.110.200.101/7.2/FitData2.5/HDP/centos7/

path=/
enabled=1

```

创建完成后执行 `yum clean all && yum makecache` 重建缓存，`yum list |grep hue`来确认hue是否添加到yum中，如果可以看到这些包，执行`yum -y install hue`安装hue

```
yum clean all && yum makecache
yum list |grep hue
yum -y install hue
```



安装文件路径：`/usr/lib/hue`

配置文件路径：`/etc/hue`



### 配置

**Ambari配置**

 

HDFS: 开启WebHDFS

 ![A0001enableWebHDFS](imgs/A0001enableWebHDFS.png)

custom core-site

```
hadoop.proxyuser.oozie.groups=*
hadoop.proxyuser.oozie.hosts=fitdata132

hadoop.proxyuser.hive.groups=*
hadoop.proxyuser.hive.hosts=fitdata132

hadoop.proxyuser.hue.groups=*
hadoop.proxyuser.hue.hosts=*

```

 ![A0002](imgs/A0002.png)

custom hdfs-site

```
dfs.namenode.acls.enabled=true
```

 ![A0003](imgs/A0003.png)

Oozie: custom oozie-site

```
oozie.service.ProxyUserService.proxyuser.hue.groups=*
oozie.service.ProxyUserService.proxyuser.hue.hosts=*
```

  ![A0004](imgs/A0004.png)

Hive :  custom webhcat-site

```
webhcat.proxyuser.hue.groups=*
webhcat.proxyuser.hue.hosts=*
```

 ![A0005](imgs/A0005.png)

保存，**重启**集群有关服务



**Hue配置**

修改 `/etc/hue/conf/hue.ini`

主要修改配置项

```
  ##允许访问的IP段，0.0.0.0表示允许所有IP访问
  http_host=0.0.0.0
  ###hue页面端口，建议不用8000，可能与Storm的端口冲突
  http_port=8000
  ##时区
  time_zone=Asia/Shanghai
  ##hdfs地址
  fs_defaultfs=hdfs://fitdata132:8020
  ##web hdfs地址
  webhdfs_url=http://fitdata132:50070/webhdfs/v1/
  ## yarn.resourcemanager.webapp.address  （yarn resourcemanager web地址）
  resourcemanager_api_url=http://fitdata133:8088
  ## yarn.resourcemanager.address
  resourcemanager_rpc_url=http://fitdata133:8050
  ##
  proxy_api_url=http://fitdata133:8088
  ##
  history_server_api_url=http://fitdata132:19888
  ##
  app_timeline_server_api_url=http://fitdata132:8188
  ##
  node_manager_api_url=http://fitdata132:8042
  ##
  oozie_url=http://fitdata133:11000/oozie
  ##
  hive_server_host=fitdata133
  ##
  hive_server_port=10000
  ##
  templeton_url=http://fitdata133:50111/templeton/v1/
  
```



示例：

```

```



导入数据到Hive



```
##说明
##1、Comments不能有中文，否则hue页面会报错
##2、fields terminated by '\t'只支持单个字符，“::”是不能导入进去的，原始数据需要使用awk之类的工具处理以下分隔符
create database if not exists movieLength;
use movieLength;
create table if not exists movieLength.users(
UserID int  comment 'user ID',
Gender string comment 'gender',
Age int comment 'age',
Occupation  string comment 'Occupation0-20',
ZipCode string comment 'zip code'
) comment 'movieLength users table'
row format delimited 
fields terminated by '\t'
lines terminated by '\n'
stored as textfile; 
desc users;

load data local inpath "/root/users.dat" into table movieLength.users ;
```



### 启动

```
/etc/init.d/hue start
```



 ![Screenshot from 2018-12-11 09_03_15](imgs/Screenshot from 2018-12-11 09_03_15.png)



关闭iptables，否则从外部访问看不到页面。如果hue.ini中写的都是域名不是ip，还要配置/etc/hosts



 ![Screenshot from 2018-12-11 09:13:39](imgs/Screenshot from 2018-12-11 09:13:39.png)



 ![01Login](imgs/01Login.png)

 

  ![02AboutHue-Configuration](imgs/02AboutHue-Configuration.png)



 ![03AboutHue-CheckForMisConfiguration](imgs/03AboutHue-CheckForMisConfiguration.png)



 ![04AboutHue-ServerDetails](imgs/04AboutHue-ServerDetails.png)



 ![05AboutHue-ServerLogs](imgs/05AboutHue-ServerLogs.png)



 ![06Hive-QueryEditor](imgs/06Hive-QueryEditor.png)



 ![07Hive-QueryEditor-Results](imgs/07Hive-QueryEditor-Results.png)



 ![08Hive-QueryEditor-SaveAs](imgs/08Hive-QueryEditor-SaveAs.png)



 ![09Hive-MyQueries](imgs/09Hive-MyQueries.png)



 ![10Hive-MyQueries-RecentRunQueries](imgs/10Hive-MyQueries-RecentRunQueries.png)



 ![11Hive-SavedQueries](imgs/11Hive-SavedQueries.png)



 ![12Hive-History](imgs/12Hive-History.png)



 ![13Hive-Databases](imgs/13Hive-Databases.png)



 ![14Hive-Tables](imgs/14Hive-Tables.png)



 ![15Hive-Tables-Columns](imgs/15Hive-Tables-Columns.png)



 ![16Hive-Tables-Sample](imgs/16Hive-Tables-Sample.png)



  ![17Hive-Settings](imgs/18Pig-MyScrips.png)



 ![18Pig-MyScrips](imgs/18Pig-MyScrips.png)



 ![19Pig-QueryHistory](imgs/19Pig-QueryHistory.png)



 ![20HCAT-Databases](imgs/20HCAT-Databases.png)



 ![21HCAT-Databases-CreateNewDB](imgs/21HCAT-Databases-CreateNewDB.png)



 ![22HCAT-Databases-CreateNewDB-Location](imgs/22HCAT-Databases-CreateNewDB-Location.png)



 ![23HCAT-Tables](imgs/23HCAT-Tables.png)



 ![24FileBrowser](imgs/24FileBrowser.png)



 ![25FileBrowser-HDFS-FileOperations](imgs/25FileBrowser-HDFS-FileOperations.png)



 ![26JobBrowser](imgs/27JobDesigner-NewAction.png)



 ![27JobDesigner-NewAction](imgs/27JobDesigner-NewAction.png)



 ![28Oozie-Editor-DashBoard](imgs/28Oozie-Editor-DashBoard.png)



 ![29Oozie-Editor-WorkFlows](imgs/29Oozie-Editor-WorkFlows.png)



 ![30Oozie-Editor-Coordinators](imgs/30Oozie-Editor-Coordinators.png)  



 ![31Oozie-Editor-Bundles](imgs/31Oozie-Editor-Bundles.png)



 ![32UserAdmin](imgs/32UserAdmin.png)



 ![33Help](imgs/33Help.png)



 





参考文章

<https://community.hortonworks.com/questions/87595/hue-failed-to-start-on-machine-which-has-python-27.html>

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.6/bk_installing_manually_book/content/configure_hdp_support_hue.html







