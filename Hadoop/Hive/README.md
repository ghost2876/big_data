Hadoop 2.3 -> Hive 2.1

# Hive 2.1
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

# Hive 2.1 案例
## 整体数据仓库的架构
处理之前的导入：通过 hadoop 命令将日志数据导入到 hdfs 文件系统，<br />
处理完成之后的导出：利用 hive 处理完成之后的数据，通过 sqoop 导出到 mysql或HBase 数据库中，以供报表层使用。<br />
## 你在项目中遇到了哪些难题，是怎么解决的？
某些任务执行时间过长，且失败率过高，检查日志后发现没有执行完就失败，原因出在hadoop 的 job 的 timeout 过短（相对于集群的能力来说），设置长一点即可<br />
其实就是-Dmapreduce.task.timeout=1200000单位是毫秒<br />
链接：http://www.tuicool.com/articles/EBFFve
## 一个网络商城 1 天大概产生多少 G 的日志？
4tb
## 你们大概有多少条日志记录（在不清洗的情况下）？
7-8 百万条。开发时使用的是部分数据，不是全量数据，有将近一亿或8、9 千万吧
## 你们日访问量大概有多少个？
百万
## 你们注册数大概多少？
不清楚，几十万吧 
## 你们的日志是不是除了 apache 的访问日志是不是还有其他的日志？
关注信息
## 你们的集群多少台,数据量多大,吞吐量是多大,每天处理多少G 的数据？
之前我们公司采用双节点的Oracle RAC，IBM小型机，由于成本和性能的压力，转为Hadoop<br />
目前我们整个的数据仓库平台(EDW)都基于Hadoop    有30台机器，数据从装载服务器批量导入到集群，周期为天。数据经过处理以后放入由Hive搭建的数据仓库，数据仓库的查询统计结果会放入由HBase搭建的集群，由HBase负责为应用提供服务接口。前端Portal会通过API连接到HBase<br />
目前集群有700 TB的容量，一天300 GB的数据增量（包括分析数据和业务数据）。使用了3年。<br />
默认情况下, Hive的元数据保存在了内嵌的 derby 数据库里, 但一般情况下生产环境使用 MySQL 来存放 Hive 元数据。<br />
## 你们集群中机器每台的配置怎么样，比如内存、CPU、存储等等，都怎么分布的（地理上，配机架感知了吗）？
考虑的时候，分Namenode这类管理节点，和Datanode这类普通节点，前者对内存要求高、需要非常可靠的硬件，所以一般还是推荐使用小型机之类内存在48GB到64GB甚至128MB的，不过对存储要求不太高；后者对存储要求较高，要求硬件可随时方便替换，一般最好也是能用至少32GB内存的机器来配置。<br />
反正一般不建议直接用普通的PC机搭建Hadoop集群，除非是实验性质或者所处理的问题真的非常非常小。一般6到8节点的小型集群也建议用稍微专业的机器组建。20台所有的集群已经可以胜任绝大多数企业的需要。<br />
当然，搭建时还需考虑你是I/O密集型的应用（比如ETL等）还是计算密集型的应用（比如数据挖掘、机器学习）。<br />
## 如何优化Hive，对Hive的一些优化的案例-1笔记
比如应付数据倾斜，使用分区等。
## Hive中实现权限控制-1笔记
## Hive HQL的优化和stored procedure-1笔记
## 整体你们项目用Hive干了些啥事儿描述一下
用Hive一方面业务部分有日常的提取数据需求，写HQL提数<br />
一方面构建real time的recommendation system（就是sales representative在customer的现场需要分钟级获取到推荐内容，做customer interaction的时候，录入customer的一些profile等等之后）。<br />


