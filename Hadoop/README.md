Hadoop

# Hive 0.8
1. 用sql查询HDFS上的数据<br />
2. Hive的数据都存储在HDFS的路径/user/hive/warehouse下，一个表一个子目录，使用./hive命令进入Hive shell<br />
/home<br />
/system<br />
/user<br />
	下的/hive<br />
		下的/warehouse<br />
	下的/huang<br />
3. Hive里的数据类型<br />
integer<br />
float<br />
boolean<br />
struct，两个string<br />
map，key value<br />
array，数组<br />
<img src="hive_1.jpg" width="60%" height="60%" alt="hive_1"/><br />
当然可以使用delimiter或terminator指定row、col分隔符，使用create external table ... file ...指定由HDFS文件创建外部表，指定partitioned by创建分区字段。<br />
普通表的存储路径例子：hdfs://master_server/user/hive/warehouse/mydb.db/employees<br />
分区表的存储路径例子：<br />
hdfs://master_server/user/hive/warehouse/mydb.db/employees/country=CA/state=AB<br />
hdfs://master_server/user/hive/warehouse/mydb.db/employees/country=CA/state=BC<br />
hdfs://master_server/user/hive/warehouse/mydb.db/employees/country=US/state=AL<br />
hdfs://master_server/user/hive/warehouse/mydb.db/employees/country=US/state=AK<br />
4. Hive虽然作为Hadoop里的数据仓库，但它与传统的RDBMS比如Oracle相比依旧有很大的区别。它无法做到在记录行级别上的插入、删除和更新操作。它对自己表里的数据的唯一更新办法就是整批载入（bulk load）。原因：基于HDFS。<br />
5. Hive的查询优化技巧<br />
5.1 在Hive里将aggr参数设置为true，可以显著地提升SELECT时进行group by统计count等分组计算的效率<br />
原因：aggr参数为false时，Hive先用SELECT将所有数据查出来，最后统一放到一台reduce节点机器上进行group by，这时所有的压力都压在这一台节点上；aggr为true时，Hive的SELECT会在每台节点上进行预group by，最后将group by后的数据放在reduce节点上，最终group by，缓解了最终的压力，将group by压力分摊，因此提升了效率。<br />
<img src="hive_2.jpg" width="60%" height="60%" alt="hive_2"/><br />
5.2 ni<br />

# HBase

# Map Reduce

# 