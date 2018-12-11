PySpark学习笔记





spark read selected colum  and write to file

http://dmtolpeko.com/2015/02/10/how-to-select-specified-columns-projection-in-spark/

https://www.nodalpoint.com/spark-data-frames-from-csv-files-handling-headers-column-types/

http://www.alpha-epsilon.de/cca175/2017/09/07/reading-and-writing-data-with-spark-and-python/

https://docs.databricks.com/spark/latest/dataframes-datasets/introduction-to-dataframes-scala.html

http://colobu.com/2014/12/09/spark-submitting-applications/



参数配置

https://datahub.packtpub.com/tutorials/sizing-configuring-hadoop-cluster/

http://blog.javachen.com/2015/06/07/spark-configuration.html

http://blog.javachen.com/2015/06/09/memory-in-spark-on-yarn.html



spark访问hbase

http://www.voidcn.com/article/p-hdwkwiyu-sk.html



PySpark操作HBase时设置scan参数

https://www.cnblogs.com/errdev/p/4500089.html



GC

https://www.cnblogs.com/hucn/p/3572384.html



主要的Library

-   PySparkSQL
-   MLlib
-   GraphFrames



jrunscrip获取JAVA_HOME

```
jrunscript -e 'java.lang.System.out.println(java.lang.System.getProperty("java.home"));'
/usr/lib/jvm/jdk1.8.0_25/jre
```



Python

-   typecast
-   list
-   set
-   tuple
-   dictionary
-   Define Call functions
-   lamada functions
-   condition && loops 



list1=[1,2,3,4,5]

print(list1[0])

len(list1)  

list1.extend([6,7,8])

list1.append(9)    list1.append([10,11])

print(list1.sort(reverse=True))



pythonTuple = (2.0,9,"a",True,"a")

pythonTuple[2]

pythonTuple.index('a')

pythonTuple.count("a")



pythonSet = {'Book','Pen','NoteBook','Pencil','Book'}

set(['Pencil', 'Pen', 'Book', 'NoteBook'])    ##重复元素被忽略

pythonSet.add("Eraser")

set(['Pencil', 'Pen', 'Book', 'Eraser', 'NoteBook'])

pythonSet1 = {'NoteBook','Pencil','Diary','Marker'}

pythonSet.union(pythonSet1)
set(['Pencil', 'NoteBook', 'Pen', 'Book', 'Diary', 'Eraser', 'Marker'])

pythonSet1.union(pythonSet)
set(['Pencil', 'Book', 'Pen', 'NoteBook', 'Diary', 'Eraser', 'Marker'])

pythonSet.intersection(pythonSet1)
set(['Pencil', 'NoteBook'])



pythonDict = {'item1':'Pencil','item2':'Pen', 'item3':'NoteBook'}

pythonDict['item3']       ##不存在会报错

pythonDict.get('item4')   ##不存在无输出

pythonDict.keys()



RDD的2种类型操作: transformation and action

Transformation on an RDD returns another RDD （RDDs are immutable; ）

the transformation is lazy because whenever a transformation is applied to an RDD,that operation is not applied to the data at the same time.but all the transformations are applied when the first action is called













You want to calculate the following:
•	 Average grades per semester, each year, for each student
•	 Top three students who have the highest average grades in the
second year
•	 Bottom three students who have the lowest average grades in the
second year
•	 All students who have earned more than an 80% average in the
second semester of the second year

[学号，学年，第1学期分数，第2学期分数]

```
studentMarksData = [["si1","year1",62.08,62.4],
...  ["si1","year2",75.94,76.75],
...  ["si2","year1",68.26,72.95],
...  ["si2","year2",85.49,75.8],
...  ["si3","year1",75.08,79.84],
...  ["si3","year2",54.98,87.72],
...  ["si4","year1",50.03,66.85],
...  ["si4","year2",71.26,69.77],
...  ["si5","year1",52.74,76.27],
...  ["si5","year2",50.39,68.58],
...  ["si6","year1",74.86,60.8],
...  ["si6","year2",58.29,62.38],
...  ["si7","year1",63.95,74.51],
...  ["si7","year2",66.69,56.92]]
studentMarksDataRDD = sc.parallelize(studentMarksData,4)
```

```
studentMarksMean = studentMarksDataRDD.map(lambda x :[x[0],x[1],(x[2]+x[3])/2])
studentMarksMean.take(2)
```

```
secondYearMarks = studentMarksMean.filter(lambda x : "year2" in x)
secondYearMarks.take(2)
```

```
sortedMarksData = secondYearMarks.sortBy(keyfunc = lambda x : -x[2])
sortedMarksData.take(3)

topThreeStudents = secondYearMarks.takeOrdered(num=3, key = lambda x :-x[2])
topThreeStudents.collect()
```

```
bottomThreeStudents = secondYearMarks.takeOrdered(num=3, key = lambda x:x[2]])
bottomThreeStudents.collect()
```

```
moreThan80Marks = secondYearMarks.filter(lambda x : x[2] > 80)
moreThan80Marks.collect()
```

































































































































