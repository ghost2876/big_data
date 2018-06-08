Hadoop 2.3, CDH 5.0.0

# Hive 2.1
参见对应子文件夹

# HBase


# Map Reduce
## 案例项目：User Analysis of China Mobile - Digital China
大部分forgot了，统计drop rate because sometimes the telecom connection will be terminated automatically and occasionaly, we have to estimate the drop rate of that accident. 参见这里链接：https://blog.csdn.net/yangshaojun1992/article/details/77917823   有相关表结构。<br />
详细参见BMA-1笔记之“2.8.6	第2课：电信运营商数据分析”和“第3课：LBS签到与区域用户流动分析”两节。这里不摘什么了，具体回去快速扫吧。<br />
8台普通PC构建的cluster，500万 5 million customers，上网数据、位置数据，记录（user id，时间段duration-一天分割为不同的时间段，地点location经纬度lati和long）。基于Java开发Map Reduce分析程序。最后遵从大部分电信运营商的喜好，用Sqoop将分析数据导出至RDBMS比如Oracle进一步分析和展现。<br />
因为其实有很多条记录，比如用户A于9:00在X基站出现，9:15于X基站上网，9:30于Y基站上网，9:45出现在Y基站。于是，我们认定该用户在X基站停留了30分钟。所以我们以user id和时间段duration为key，location和所有时间为value，于是所有相同user id和duration的派发到一台reducer，然后就能统计出在每个location的停留时长。于是就有数据 user id + 时间段 + location + 停留时长。这个过程就是map reduce。<br />
Self-reported positioning<br />