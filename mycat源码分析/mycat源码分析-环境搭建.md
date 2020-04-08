# MyCAT源码分析环境搭建

## 1.mycat简介

### 1.1什么是Mycat？

简单的说，Mycat就是（官网:[http://www.mycat.org.cn](http://www.mycat.org.cn/)）：

- 一个彻底开源的，面向企业应用开发的“大数据库集群”
- 支持事务、ACID、可以替代MySQL的加强版数据库
- 一个可以视为“MySQL”集群的企业级数据库，用来替代昂贵的Oracle集群
- 一个融合内存缓存技术、Nosql技术、HDFS大数据的新型SQL Server
- 结合传统数据库和新型分布式数据仓库的新一代企业级数据库产品
- 一个新颖的数据库中间件产品

### 1.2Mycat的关键特性：

- 支持 SQL 92标准
- 支持MySQL集群，可以作为Proxy使用
- 支持JDBC连接ORACLE、DB2、SQL Server，将其模拟为MySQL Server使用
- 支持galera for MySQL集群，percona-cluster或者mariadb cluster，提供高可用性数据分片集群
- 自动故障切换，高可用性
- 支持读写分离，支持MySQL双主多从，以及一主多从的模式
- 支持全局表，数据自动分片到多个节点，用于高效表关联查询
- 支持独有的基于E-R 关系的分片策略，实现了高效的表关联查询
- 多平台支持，部署和实施简单

## 2.环境搭建

### 2.1依赖工具

> Maven 、Git、JDK 、MySQL 、IDEA

### 2.2源码地址

官网：https://github.com/MyCATApache/Mycat-Server 

### 2.3 数据库配置

- 创建数据库 db01

- 创建数据表：test

  ```sql
  CREATE TABLE `test` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT,
    `name` varchar(255) CHARACTER SET latin1 DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_bin
  ```

### 2.4mycat配置

为了避免对实现源码产生影响，我们选择对 `test` 目录做变更。

- 在 `resources` 目录下新建文件夹 `backups` ，将原 `resources` 下的所有文件移到 `backups` 下，这样我们的环境就干干净了。

- 在 `resources` 目录下新建 `schema.xml` 文件，配置 `MyCAT` 的逻辑库、表、数据节点、数据源。

  ```xml
  <?xml version="1.0"?>
  <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
  <mycat:schema xmlns:mycat="http://io.mycat/">
  
      <schema name="db01" checkSQLschema="true" sqlMaxLimit="100">
          <table name="test" dataNode="dn1" autoIncrement="true" primaryKey="id" />
      </schema>
  
  	<dataNode name="dn1" dataHost="localhost1" database="db1" />
  
  	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
  			  writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
  		<heartbeat>select user()</heartbeat>
  		<writeHost host="hostM1" url="127.0.0.1:3306" user="root" password="123456"> <!-- ‼️‼️‼️ url、user、password 设置成你的数据库 -->
  		</writeHost>
  	</dataHost>
  
  </mycat:schema>
  ```

- 在 `resources` 目录下新建 `server.xml` 文件，配置 `MyCAT` 系统配置

  ```sql
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mycat:server SYSTEM "server.dtd">
  <mycat:server xmlns:mycat="http://io.mycat/">
  	<system>
          <property name="nonePasswordLogin">0</property> <!-- 0为需要密码登陆、1为不需要密码登陆 ,默认为0，设置为1则需要指定默认账户-->
          <property name="useHandshakeV10">1</property>
          <property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
          <property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->
  		<property name="sequnceHandlerType">2</property>
  		<property name="processorBufferPoolType">0</property>
  		<property name="handleDistributedTransactions">0</property>
  		<property name="useOffHeapForMerge">1</property>
          <property name="memoryPageSize">64k</property>
  		<property name="spillsFileBufferSize">1k</property>
  		<property name="useStreamOutput">0</property>
  		<property name="systemReserveMemorySize">384m</property>
  		<property name="useZKSwitch">false</property>
  	</system>
  
  	<user name="root" defaultAccount="true">
  		<property name="password">123456</property>
  		<property name="schemas">db01</property>
  	</user>
  
  </mycat:server>
  ```

# 3. MyCAT 启动

- 在 `java` 目录下新建 `debugger` 包，和原先已存在的包做区分
- 在 `debbuger` 包下新建 `MycatStartupTest.java` ：

```java
package demo.debugger;

import io.mycat.MycatStartup;

/**
 * @Author smallmartial
 * @Date 2020/4/7
 * @Email smallmarital@qq.com
 */
public class MycatStartupTest {

    public static void main(String[] args) {
        MycatStartup.main(args);
    }

}
```

- 运行 `MycatStartupTest.java` ，当看到输出日志 `MyCAT Server startup successfully. see logs in logs/mycat.log` 即为启动成功。

  ![image-20200407152623914](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/image-20200407152623914.png)

# 4. MyCAT 测试

使用 `MySQL` 客户端连接 `MyCAT` ：

- HOST ：127.0.0.1
- PORT ：8066
- USERNAME ：root
- PASSWORD ：123456

![image-20200407152758685](mycat%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/image-20200407152758685.png)