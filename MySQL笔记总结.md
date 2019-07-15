# 《从0到1实战新零售数据库设计与实现》笔记总结

## 一、课程介绍

### 1.1 学习准备

* 了解MySQL基础知识
* MySQL零基础的同学可以先学习《与MySQL的零距离接触》，<https://www.imooc.com/learn/122> 
* 掌握一种编程语言

### 1.2 最低硬件要求

* 英特尔酷睿i3，或者AMD 200GE
* 内存4GB
* 20GB空闲硬盘空间

### 1.3 搭建VMware虚拟机

* 下载安装VMware虚拟机

  https://www.vmware.com/cn.html

*  下载CentOS镜像文件

  http://mirrors.sohu.com/centos/7/isos/x86_64/CentOS-7-x86_64-Everything-1708.iso

* 配置虚拟机必须要选择桥接网络。如果采用默认的NAT网络模式，未来配置虚拟IP将无效，切记！

* 下载并安装MobaXterm，配置SSH连接

  https://mobaxterm.mobatek.net

### 1.4 Linux基础知识

| 目录 | 用途                             | 重要性 |
| ---- | -------------------------------- | ------ |
| bin  | 存放二进制文件                   | 高     |
| dev  | 存放硬件文件                     | 高     |
| etc  | 存放程序的配置文件               | 高     |
| home | 非root用户的目录文件             | 普通   |
| proc | 存放正在运行中的进程文件         | 高     |
| root | root用户目录                     | 高     |
| sbin | 存放root用户可以执行的二进制文件 | 高     |
| tmp  | 存放系统临时文件                 | 低     |
| usr  | 存放安装的程序                   | 高     |
| var  | 存放程序或者系统日志文件         | 高     |

* 创建文件命令

  ```shell
  touch demo.txt
  ```

* 编辑文件命令

  ```shell
  vi demo.txt
  ```

* 创建文件夹命令

  ```shell
  mkdir school
  ```

* 删除文件或者文件夹命令

  ```shell
  rm -rf demo.txt
  ```

  ```shell
  rm -rf shcool
  ```

* 查看运行中的进程列表

  ```shell
  ps aux
  ```

* 关闭进程

  ```shell
  kill -9 进程编号
  ```

* 关闭SELinux

  * 编辑文件

    ```shell
    vi /etc/selinux/config
    ```

  * 设置 SELINUX=disabled，重启系统



## 二、前置知识

### 2.1 在线安装MySQL数据库

* 替换yum源

  ```shell
  curl -o /etc/yum.repos.d/CentOS-Base.repo mirrors.163.com/.help/CentOS7-Base-163.repo
  ```

  ```shell
  yum clean all 
  yum makecache 
  ```

* 下载rpm文件

  ```shell
  yum localinstall https://repo.mysql.com//mysql80-community-release-el7-1.noarch.rpm
  ```

* 安装MySQL数据库

  ```shell
  yum install mysql-community-server -y
  ```

### 2.2 本地安装MySQL数据库

* 把本课程git工程里共享的MySQL本地安装文件上传到Linux主机的/root/mysql目录

* 执行解压缩

  ```shell
  tar xvf mysql-8.0.11-1.el7.x86_64.rpm-bundle.tar
  ```

* 安装依赖的程序包

  ```shell
  yum install perl -y
  yum install net-tools -y
  ```

* 卸载mariadb程序包

  ```shell
  rpm -qa|grep mariadb
  rpm -e mariadb-libs-5.5.60-1.el7_5.x86_64 --nodeps
  ```

* 安装MySQL程序包

  ```shell
  rpm -ivh mysql-community-common-8.0.11-1.el7.x86_64.rpm 
  rpm -ivh mysql-community-libs-8.0.11-1.el7.x86_64.rpm 
  rpm -ivh mysql-community-client-8.0.11-1.el7.x86_64.rpm 
  rpm -ivh mysql-community-server-8.0.11-1.el7.x86_64.rpm 
  ```

* 修改MySQL目录权限

  ```shell
  chmod -R 777 /var/lib/mysql/
  ```

* 初始化MySQL

  ```shell
  mysqld --initialize
  chmod -R 777 /var/lib/mysql/*
  ```

  

* 启动MySQL

  ```shell
  service mysqld start
  ```

* 查看初始密码

  ```shell
  grep 'temporary password' /var/log/mysqld.log
  ```

* 登陆数据库之后，修改默认密码

  ```mysql
  alter user user() identified by "abc123456"; 
  ```

* 允许远程使用root帐户

  ```mysql
  UPDATE user SET host = '%' WHERE user ='root';
  FLUSH PRIVILEGES;
  ```

* 允许远程访问MySQL数据库（/etc/my.cnf）

  ```ini
  character_set_server = utf8
  bind-address = 0.0.0.0
  ```

* 开启防火墙3360端口

  ```shell
  firewall-cmd --zone=public --add-port=3306/tcp --permanent
  firewall-cmd --reload
  ```



## 三、企业级解决方案

### 3.1 主键用数字还是UUID？

UUID 是通用唯一识别码的缩写，其目的是让分布式系统中的所有元素，都能有唯一的辨识信息，而不需要通过中央控制端来做辨识信息的指定。在数据库集群中，为了避免每个MySQL各自生成的主键产生重复，所以有人考虑采用UUID方式。

MySQL中有UUID()函数，可以生成UUID()。

#### 使用UUID的好处

* 使用UUID，分布式生成主键，降低了全局节点的压力，使得主键生成速度更快
* 使用UUID生成的主键值全局唯一
* 跨服务器合并数据很方便

#### UUID主键的缺点

* UUID占用16个字节，比4字节的INT类型和8字节的BIGINT类型更加占用存储空间
* UUID是字符串类型，查询速度很慢
* UUID不是顺序增长，作为主键，数据写入IO随机性很大

#### 主键自动增长的优点

* INT和BIGINT类型占用存储空间较小
* MySQL检索数字类型速度远快过字符串
* 主键值是自动增长的，所以IO写入连续性较好

**无论什么场合，都不推荐使用UUID作为数据表的主键，而是要利用数据库中间件来生成全局主键**

### 3.2 订单号和流水号的关系

* 订单号既是订单的唯一编号，而且经常被用来检索，所以应当是数字类型的主键
* 流水号是打印在购物单据上的字符串，便于阅读，但是不用做查询


流水号的规则由自己定义

### 3.3 在线修改表结构

在业务系统运行的过程中随意删改字段，会造成重大事故。常规的做法是业务停机，维护表结构，但是不影响正常业务的表结构是允许在线修改的。

【ALTER TABLE 修改表结构的弊病】

* 由于修改表结构是表级锁，因此在修改表结构时，影响表写入操作
* 如果修改表结构失败，必须还原表结构，所以耗时更长
* 大数据表记录多，修改表结构锁表时间很久

【使用Percona-Toolkit工具】

* 安装第三方依赖包

  ```shell
  yum install  -y perl-DBI
  yum install  -y perl-DBD-mysql
  yum install  -y perl-IO-Socket-SSL
  yum install  -y perl-Digest-MD5
  yum install  -y perl-TermReadKey
  ```

* 安装Percona-Toolkit

  ```shell
  #进入到Percona-Tookit离线文件所在的目录
  rpm -ivh *.rpm
  ```

* 把客户收货地址表中的name字段改成VARCHAR(20)

  ```shell
  pt-online-schema-change --host=192.168.99.202 --port=3306 --user=root --password=abc123456 --alter "MODIFY name VARCHAR(20) NOT NULL COMMENT '收货人'" D=neti, t=t_customer_address --print --execute
  ```

### 3.4 架设本地图床服务器

* 设置Nginx安装源

  ```shell
  rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
  ```

  ```shell
  yum install -y nginx
  ```

* 防火墙开放80端口

  ```shell
  firewall-cmd --zone=public --add-port=80/tcp --permanent
  firewall-cmd --reload
  ```

* 在/usr/share/nginx/html目录下创建文件夹，上传图片即可



### 3.5 读多写少和写多读少(对数据库负载的理解)

普遍来说，绝大多数系统都是读多写少的

#### 读多写少的解决方案

可以把MySQL组建集群，并且设置上读写分离

#### 写多读少的解决方案(冷热数据分离)

如果是低价值的数据，可以采用NoSQL数据库来存储这些数据

如果是高价值的数据，可以用TokuDB来保存

#### 写多读多的解决方案
采用NoSQL

### 3.6 逻辑删除还是物理删除？

物理删除就是用DELETE、TRUNCATE、DROP语句删除数据。物理删除是把数据从硬盘中删除，可以释放存储空间，缩小数据表的体积，对性能提升有帮助
物理删除会造成主键的不连续，导致分页查询变慢
核心业务表的数据不建议物理删除，只做状态变更。

我们如何实现既不删除数据，又能缩小数据表体积呢，可以把记录转移到历史表
逻辑删除就是在数据表添加一个字段valid，用字段值标记该数据已经逻辑删除，查询的时候跳过这些数据。同时将逻辑删除的记录转移到历史表
```mysql
//克隆表
CREATE TABLE t_user_history LIKE t_user;
```

```mysql
SELECT ……  FROM ……  LIMIT 1000, 20;
```

```mysql
SELECT ……  FROM ……  WHERE  id>=1000  AND  id<= 1020;
```

核心业务表的数据不建议做物理删除，只做状态变更。比如订单作废、账号禁用、优惠券作废等等。既不删除数据，又能缩小数据表体积，可以把记录转移到历史表。

逻辑删除就是在数据表添加一个字段（is_deleted），用字段值标记该数据已经逻辑删除，查询的时候跳过这些数据

### 3.7 海量记录快速分页
```mysql
SELECT * FROM t_test WHERE id>=5000000 LIMIT 100,10;//耗时0.09秒
SELECT * FROM t_test WHERE id>=5000000 LIMIT 10000,10;//耗时0.387秒
SELECT * FROM t_test WHERE id>=5000000 LIMIT 50000000,10;//耗时1.839秒
```

利用主键索引来加速分页查询

```mysql
SELECT * FROM t_test WHERE id>=5000000 LIMIT 100;
SELECT * FROM t_test WHERE id>=5000000 AND id<=5000000+100;
```


使用逻辑删除，不会造成主键不连续
如果用物理删除，主键不连续，就不能用主键索引来加速分页，所以只能使用折中的方案

```mysql
SELECT t.id, t.name FROM t_test t JOIN ( SELECT id FROM t_test LIMIT 5000000, 100) tmp ON t.id = tmp.id;
```

或者通过业务上的设置，避免用户查询过于早期的数据，减轻系统负担


### 3.8 乐观锁

#### 删除数据时如何避免锁表
InnoDB采用行级锁，删改数据的时候，MySQL会锁住记录。
行级锁分为共享锁(S锁)和排它锁(X锁)
共享锁和排它锁，都不允许其他事务执行写操作，但是可以读数据
##### 共享锁
排它锁不允许对数据再另行加锁
- 只有serializable事务隔离级别，才会给数据读取添加共享锁`SELECT * FROM t_user LOCK IN SHARE MODE`

##### 排它锁
MySQL默认会给添加、修改、和删除记录，设置排它锁

``` mysql
SELECT ... FROM ... FOR UPDATE//手动加锁
```
如何减少并发操作的锁冲突？
将复杂的SQL语句，拆分成多条简单的SQL语句，减少事务时间。



在并发环境下，如果多个客户端访问同一条数据，此时就会产生数据不一致的问题，如何解决，通过加锁的机制，常见的有两种锁，乐观锁和悲观锁，可以在一定程度上解决并发访问。

乐观锁，顾名思义，对加锁持有一种乐观的态度，即先进行业务操作，不到最后一步不进行加锁，"乐观"的认为加锁一定会成功的，在最后一步更新数据的时候在进行加锁，乐观锁的实现方式一般为每一条数据加一个版本号，更新数据的时候，比较版本号，系统就知道有没有出现数据的并发更新。如果小于等于当前版本号的更新，都会被放弃。

### 3.9 为什么要放弃存储过程、触发器和自定义函数？

因为在数据库集群的场景里，由于存储过程、触发器和自定义函数都是在本地数据库节点上运行，它们与数据库集群业务产生了冲突，所以为了顾全大局，放弃使用数据库本地编程，甚至连数据库本地生成主键的机制也都放弃了。

### 3.10 如何避免偷换交易中的商品信息？

B2B电商平台，通常采用保存历次商品修改信息、降低搜索排名

B2C电商平台，只需要保存历次商品修改信息即可

### 3.11 如何抵御XSS攻击？

XSS攻击，是通过在网页上嵌入恶意的JavaScript代码，然后当浏览器渲染DOM组件的时候，这段恶意的脚本就执行了，然后盗取个人账号信息。
![enter description here](./images/1562677548018.png)
![enter description here](./images/1562677788378.png)
(**当图片加载失败，就会触发事件**)
第一种，大家都已经知道了，那就是在网页里面植入恶意代码即可。植入过的过程就是你到网站上发帖与回帖。别人看到这个网页，他的账号就被你窃取到了。为什么说XSS攻击跟数据库有关系呢？毕竟发帖回帖的内容，是保存到数据库上的。如果我们对保存到MySQL的数据，先做一下转义，那么将来输出到页面上，那么浏览器就不会当它是标签了，因此也就不会执行恶意代码。

第二种XSS攻击的常用方式，就是发送HTML格式的邮件。因为邮件本身就是一个HTML页面，所以往里面挂恶意代码非常容易。当用户在浏览器上登陆网易邮箱，然后点开这个邮件，那么恶意脚本就开始执行，马上就窃取到你的账户信息。抵御这种XSS攻击，就只能靠电子信箱的运营商，加强邮件内容的过滤，筛选出恶意的脚本。

### 3.12 数据库缓存（查询缓存）、程序缓存应该选择哪个？

MySQL的查询缓存结构的是KV的，数据库会把执行过的SELECT结果集缓存到内存里面，KEY是SQL语句，VALUE是结果集。
![enter description here](./images/1562680852579.png)
数据库缓存注意事项：
- 所有对数据加锁的事务中(包含UPDATE,DELETE等...)，不会使用查询缓存
- 查询语句必须一模一样，才有机会命中缓存

**查询缓存的缺点在于缓存的颗粒度不够细**
查询缓存最大的缺点就是，只要用户对数据表修改一条记录，就会让这个数据表的缓存大面积的失效，这就造成的IO压力突然增大，所以最好的办法就是不使用查询缓存，而改用数据缓存，数据缓存是把InnoDB数据表和索引中的一部分记录缓存到内存中。用户更新数据的时候，更新了数据表多少条记录，响应个缓存就更改多少条，并不会出现缓存大面积失效的情况。


数据库的查询缓存因为不可以细颗粒度的设置哪张数据表结果集被缓存，但是程序查询缓存可以详细设置哪一条的SQL的结果集被缓存。所以我们可以避开内容经常变化的数据表，对哪些数据不经常变化的数据表设置查询缓存。

SpringCache技术是Java领域里面比较成熟的缓存方案，使用注解就能管理缓存。结合Redis，可以充分发挥查询缓存的优势。

### 3.13 新零售系统的智能拆分订单



### 3.14 中文分词技术

MySQL的全文检索功能，既支持英文也支持中文。
MySQL全文检索对英文支持很好，但是对中文支持很不好。不能按照语义切词，只能按照字符切词。

#### MySQL全文索引
通过数值比较，范围过滤就可以完成绝大多数我们需要的查询，但是，如果希望通过关键字的匹配来进行查询过滤，那么就需要基于相似度的查询，而不是原来的精确数值比较。全文索引就是为这种场景设计的
原本`like + %`就可以实现模糊匹配了，为什么还要全文索引呢？ike + % 在文本比较少时是合适的，但是对于大量的文本数据检索，是不可想象的。全文索引在大量的数据面前，能比 like + % 快 N 倍，速度不是一个数量级，但是全文索引可能存在精度问题。

##### 版本支持
- MySQL 5.6 以前的版本，只有 MyISAM 存储引擎支持全文索引；
- MySQL 5.6 及以后的版本，MyISAM 和 InnoDB 存储引擎均支持全文索引;
- 只有字段的数据类型为 char、varchar、text 及其系列才可以建全文索引

##### 使用全文索引
和常用的模糊匹配使用 like + % 不同，全文索引有自己的语法格式，使用 match 和 against 关键字，比如
``` mysql
SELECT id,title,images,price
FROM t_sku
WHERE MATCH(title） AGAINST("小米9")
```
ps：match() 函数中指定的列必须和全文索引中指定的列完全相同，否则就会报错，无法使用全文索引，这是因为全文索引不会记录关键字来自哪一列。如果想要对某一列使用全文索引，请单独为该列创建全文索引

MySQL中的全文索引，有两个变量，最小搜索长度和最大搜索长度。对于长度小于最小搜索长度和大于最大搜索长度的词语，都不会被索引。通俗点就是说，想对一个词语使用全文索引搜索，那么这个词语的长度必须在以上两个变量的区间内。
这两个的默认值可以使用以下命令查看

``` mysql
show variables like '%ft%';
```

> // MyISAM
ft_min_word_len = 4;
ft_max_word_len = 84;
// InnoDB
innodb_ft_min_token_size = 3;
innodb_ft_max_token_size = 84;


##### 配置最小搜索长度
全文索引的相关参数都无法进行动态修改，必须通过修改 MySQL 的配置文件来完成。修改最小搜索长度的值为 1，首先打开 MySQL 的配置文件 /etc/my.cnf，在 [mysqld] 的下面追加以下内容

>[mysqld]
innodb_ft_min_token_size = 1
ft_min_word_len = 1

然后重启 MySQL 服务器，并修复全文索引。注意，修改完参数以后，一定要修复下索引，不然参数不会生效
两种修复方式，可以使用下面的命令修复

``` mysql
repair table test quick;
```

##### 查询权重
全文索引除了需要注意搜索长度之外，还有另外一个问题值得注意。明明很多条数据都包含这个词，但返回的结果却是空的，原来MySQL还会计算一个词的权重，以决定是否出现在结果集中，具体如下：MySQL在集和查询中对每个合适的词都会先计算它们的权重，一个出现在多个文档中的词将有较低的权重(甚至可能有零权重)，因为在这个特定的集中，它有较低的语义值。反之，如果词是较少的，它将得到一个较高的权重。(MySQL默认的阈值是50%，只有低于50%的才会出现在结果集中)

##### 全文索引的弊病
- 中文字段创建全文索引，切词结果太多，占用大量存储空间
- 更新字段内容，全文索引不会更新，必须定期手动维护
- 在数据库集群中维护全文索引难度很大

#### 使用专业的全文检索引擎
Lucene是Apache基金会的开源全文检索引擎，支持中文分词

#### Lucene的使用
- 引入Lucene依赖
- Lucene自带的中文分词插件功能较弱，需要引入第三方中文分词插件，对中文内容准确分词(http://hanlp.com/)--引入依赖


![enter description here](./images/1562725648346.png)
![enter description here](./images/1562725660024.png)
![enter description here](./images/1562725674152.png)

``` java
	public static void main(String[] args) {
		try {
			Directory directory=FSDirectory.open(new File("D:/index").toPath());
			Analyzer analyzer=new HanLPAnalyzer();
			IndexWriterConfig config=new IndexWriterConfig(analyzer);
			IndexWriter writer=new IndexWriter(directory, config);
			DriverManager.registerDriver(new Driver());
			String url="jdbc:mysql://localhost:3306/neti?serverTimezone=GMT%2B8";
			String username="root";
			String password="abc123456";
			Connection con=DriverManager.getConnection(url,username,password);
			String sql="SELECT id,title FROM t_sku";
			PreparedStatement pst=con.prepareStatement(sql);
			ResultSet set=pst.executeQuery();
			while(set.next()) {
				String id=set.getString("id");
				String title=set.getString("title");
				Document document=new Document();
				document.add(new TextField("id", id, Field.Store.YES));
				document.add(new TextField("title", title, Field.Store.YES));
				writer.addDocument(document);
			}
			con.close();
			writer.close();
		}
		catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}


``` java
public class Demo2 {
	public static void main(String[] args) {
		try {
			Directory directory=FSDirectory.open(Paths.get("D:/index"));
			IndexReader reader=DirectoryReader.open(directory);
			IndexSearcher searcher=new IndexSearcher(reader);
			String text="拍手机会";
			String field="title";
			Analyzer analyzer=new HanLPAnalyzer();
			QueryParser parser=new QueryParser(field, analyzer);
			Query query=parser.parse(text);
			TopDocs docs=searcher.search(query, 100);
			searcher.
			System.out.println("命中的记录数："+docs.totalHits);
			ScoreDoc[] array=docs.scoreDocs;
			for (ScoreDoc one : array) {
				Document document=searcher.doc(one.doc);
				String id=document.get("id");
				String title=document.get("title");
				System.out.println("id - >"+id);
				System.out.println("title - >"+title);
			}
			reader.close();
		}
		catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```



#### Lucene注意事项
- 不是所有数据表的记录，都要保存到Lucene上面。只对需要全文检索的字段使用Lucene即可
#### Lucene与MySQL的结合
![enter description here](./images/1562727632707.png)
### 3.15 商品秒杀
商品秒杀过程中出现的超售现象：卖出了超过预期数量的商品
#### 预防数据库超售现象？

#### 设置数据库的隔离己别Serializable-不可行




#### 在数据表上设置乐观锁字段(每次更新都比较下乐观锁字段的版本号)
程序实现乐观锁
![enter description here](./images/1562667065545.png)

什么表需要设置乐观锁？
- 出现同时修改同一条记录的业务，相应的数据表要设置乐观锁

#### 利用Redis防止超售
将秒杀商品放入内存，相比硬盘，内存的读写速度快得多，应对更多地并发请求，同时防止超售.
Redis是单线程，可以将并发请求串行化；同时Redis的pop操作是原子性的，也就是不会被中断
具体实现思路
- 定义Redis队列名为sku:awards。里面的元素的值都是比如1，只是用来代表一个商品，元素的个数则是供秒杀的商品总数
- 所有请求打到Redis上，都是从sku:awards队列上pop出一个元素
  - 如果有:说明还有商品，那么需要把用户ID加入到Redis的Set名为candidate:userids里：
    -  如果加入成功，说明用户是第一次抢购
    -  否则说明用户已经成功抢购，不能重复抢购，需要往队列sku:awards弥补一个商品标识元素
- 没有取到，说明都被秒杀完了

``` java
public class Application {
	public static ThreadPoolExecutor pool=new ThreadPoolExecutor(
			10,100,10,TimeUnit.SECONDS,new LinkedBlockingDeque<Runnable>());
	public static void main(String[] args) {
		Jedis jedis=new Jedis("192.168.99.209",6379);
		jedis.auth("abc123456");
		jedis.select(0);
		jedis.set("kill_num","50");
		jedis.del("kill_user");
		jedis.close();
		for(int i=0;i<1000;i++) {
			pool.execute(new KillTask());
		}
	}
}
```

``` java
public class KillTask implements Runnable {

	@Override
	public void run() {
		Jedis jedis=new Jedis("192.168.99.209",6379);
		jedis.auth("abc123456");
		jedis.select(0);
		int num=Integer.parseInt(jedis.get("kill_num"));
		if(num>0) {
			jedis.watch("kill_num","kill_user");
			Transaction transaction=jedis.multi();
			transaction.decr("kill_num");
			transaction.rpush("kill_user", "9527");
			transaction.exec();
		}
		else {
			Application.pool.shutdown();
		}
		jedis.close();
	}

}
```
Redis的单线程是非阻塞执行的，所以并发修改数据容易产生超售结果
为了避免Redis出现超售现象，引入Redis事务机制，一次性把多条命令传递给Redis执行，这就避免了其他客户端中间插队，出现超售现象。

### 3.16 如何避免偷换交易中的商品信息？
- B2B电商平台，通常采用保存历次商品修改信息、降低搜索排名
- B2C电商平台，只需要保存历次商品修改信息即可

### 3.17 CRUD需要注意的细节


#### 引擎
TokuDB引擎适用于写多读少的业务场景，InnoDB引擎当数据量大于2000w时性能就会产生明显的下降，这时我们需要将一部分旧数据归档入TokuDB引擎下的数据库。
TokuDB引擎写速度远快于InnoDB引擎，同时数据压缩比更大。

#### 数据插入
1. 批量插入数据，因为一条记录的问题，全部数据都写入失败，回滚事务。
添加IGNORE则可以忽略失误：`INSERT IGNORE INTO t_dept(deptno,dname,loc) VALUES(...)`

2. 如何实现不存在就插入，存在就更新？添加 `ON DUPLICATE KEY UPDATE`即可`INSERT INTO t_emp_ip(ip,empno,ip) VALUES(...) ON DUPLICATE KEY UPDATE ip = VALUES(ip)`

#### 要不要使用子查询？
MySQL数据库默认关闭了缓存，所以每个子查询都是相关子查询。
**相关子查询就是要循环执行多次的子查询，每次筛选数据时都要执行子查询**
![enter description here](./images/1562679647279.png)
因为MyBatis等持久层框架开启了缓存功能，其中一级缓存就会保存子查询的结果，所以可以放心使用子查询
结论：SQL控制台不要使用子查询，在持久层框架中则可以使用

#### 如何替代子查询？
**使用FROM子查询，替代WHERE子查询**
![enter description here](./images/1562679858097.png)
FROM子句用来确定数据来源，在SQL语句中最先被执行且只被执行一次


#### 外连接的JOIN条件
- 内连接里，查询条件写在ON子句或者WHERE子句，效果相同
- 外连接里，查询条件写在ON子句或者WHERE子句，效果不同



#### 表连接修改
UPDATE语句中的WHERE子查询如何改成表连接？
![enter description here](./images/1562680416478.png)

#### 表连接删除
![enter description here](./images/1562680573161.png)
(删除的表是DELETE与逗号之间的表)

### 什么是存储过程？
存储过程是一个编译后的SQL脚本，可以单独调用，但是不能用在SQL语句
存储过程优点：
- 存储过程是编译过的SQL脚本，执行速度非常快
- 实现了SQL编程，可以降低锁表的时间和锁表的范围(因为可以通过变量存储，不需要保留整张表和整个事务)
- 对外封装表结构，提高保密性

### 什么是函数？


### 什么是触发器？


### 为什么放弃存储过程、触发器和自定义函数？
在数据库集群场景里，因为存储过程、触发器和自定义函数，都是在本地数据库执行，无法兼容集群场景。
## 四、新零售系统数据库性能优化进阶




 
### 4.1 MySQL压力测试

压力测试是针对系统的一种性能测试，但是测试数据与业务逻辑无关，更加简单直接的测试读写性能。

压力测试有4个重要测试指标：TPS、QPS、响应时间和并发量

* QPS是每秒钟处理完的请求数量
* TPS是每秒钟处理完的事务数量
* 响应时间是一次请求的平均处理时间
* 并发量是系统能同时处理的请求数量

安装sysbench工具

```shell
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash 
```

```shell
yum -y install sysbench
```

准备测试数据

```shell
sysbench /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --mysql-host=192.168.99.202 --mysql-port=3306 --mysql-user=root --mysql-password=abc123456 --oltp-tables-count=10 --oltp-table-size=100000 prepare
```

执行测试

```shell
sysbench /usr/share/sysbench/tests/include/oltp_legacy/oltp.lua --mysql-host=192.168.99.202 --mysql-port=3306 --mysql-user=root --mysql-password=abc123456 --oltp-test-mode=complex --threads=10 --time=300 --report-interval=10 run >> /home/mysysbench.log
```

### 4.2 SQL语句优化

* 不要把SELECT子句写成 SELECT *

  ```mysql
  SELECT * FROM t_emp;
  ```

* 谨慎使用模糊查询

  ```mysql
  SELECT ename FROM t_emp WHERE ename LIKE '%S%'; #不使用索引
  SELECT ename FROM t_emp WHERE ename LIKE 'S%';
  ```

* 对ORDER BY排序的字段设置索引

* 少用IS NULL
索引是二叉树，NULL值无法进行排序，所以不会记录在索引里面。
  ```mysql
  SELECT ename FROM t_emp WHERE comm IS NULL; #不使用索引
  SELECT ename FROM t_emp WHERE comm =-1;
  ```

* 尽量少用 != 运算符

索引是二叉树，大于、小于、等于可以使用到二叉树机制，但是!=没办法应用二叉树机制

  ```mysql
  SELECT ename FROM t_emp WHERE deptno!=20; #不使用索引
  SELECT ename FROM t_emp WHERE deptno<20 AND deptno>20;
  ```

* 尽量少用 OR 运算符
OR前面的字段会使用索引，OR后面的字段不会使用索引
  ```mysql
  SELECT ename FROM t_emp WHERE deptno=20 OR deptno=30; #不使用索引
  SELECT ename FROM t_emp WHERE deptno=20 
  UNION ALL
  SELECT ename FROM t_emp WHERE deptno=30;
  ```

* 尽量少用 IN 和 NOT IN 运算符

  ```mysql
  SELECT ename FROM t_emp WHERE deptno IN (20,30); #不使用索引
  SELECT ename FROM t_emp WHERE deptno=20 
  UNION ALL
  SELECT ename FROM t_emp WHERE deptno=30;
  ```

* 避免条件语句中的数据类型转换

  ```mysql
  SELECT ename FROM t_emp WHERE deptno='20';
  ```

* 在表达式左侧使用运算符和函数都会让索引失效

  ```mysql
  SELECT ename FROM t_emp WHERE salary*12>=100000; #不使用索引
  SELECT ename FROM t_emp WHERE salary>=100000/12;
  SELECT ename FROM t_emp WHERE year(hiredate)>=2000; #不使用索引
  SELECT ename FROM t_emp 
  WHERE hiredate>='2000-01-01 00:00:00';
  ```

### 4.3 MySQL参数优化

**修改/etc/my.cnf**

* 最大连接数
  * max_connections是MySQL最大并发连接数，默认值151
  * MySQL允许的最大连接数上限是16384
  * 实际连接数是最大连接数的85%较为合适

``` mysql
show variables like 'max_connections'; # 最大并发连接上限
show status like 'max_used_connections'; # 当前被使用的连接数
```
MySQL会为每个连接创建缓冲区，所以不应该盲目上调最大连接数

(类似于任务队列)
* 请求堆栈的大小
  * back_log是存放执行请求的堆栈大小，默认值是50
  * 一般堆栈大小设置成最大连接数的1/3
* 修改并发线程数
  * innodb_thread_concurrency代表并发线程数，默认是0
  * 并发线程数应该设置为CPU核心数的两倍
* 修改连接超时时间
  * wait-timeout是超时时间，单位是秒
  * 连接默认超时为8小时，连接长期不用又不销毁，浪费资源
* 数据缓存(不是查询缓存)

InnoDB缓存中保存数据表数据和索引数据
  * innodb_buffer_pool_size是InnoDB的缓存容量，默认是128M
  * InnoDB缓存的大小可以设置为主机内存的70%~80%

### 4.4 慢查询日志

慢查询日志会把查询耗时超过规定时间的SQL语句记录下来，利用慢查询日志，定位分析性能的瓶颈。

slow_query_log 可以设置慢查询日志的开闭状态

long_query_time 可以规定查询超时的时间，单位是秒

```ini
slow_query_log = ON
long_query_time = 1
```



## 五、数据库集群

### 5.1 **数据库集群能解决什么问题？**

如果在低并发的情况下，单节点MySQL的读写速度更快。因为在数据库集群中，多个MySQL节点的数据要通过网络同步，所以读写速度不如单节点MySQL

但是在高并发的情况下，大量的读写请求会让单节点MySQL的硬盘无法承受，所以速度会变得很慢。比如说12306最开始上线的时候，大量用户涌入网站购票，结果系统就崩溃了。

高并发恰恰是数据库集群的主场。大量的读写请求会被分散发往多个节点执行。众人拾柴火焰高，多个MySQL执行读写请求，肯定比单节点MySQL快，所以在高负载的情况下，单节点MySQL接近崩溃，反而数据库集群的读写速度更快。

### 5.2 如何使用Docker虚拟机

#### 5.2.1 Docker镜像

Docker的镜像文件，相当于是一个只读层，不能往里面写入数据。我们可以通过编写dockerfile文件，定义需要安装的程序，比如说MySQL、Redis、JDK、Node.js等等，然后执行这个dockerfile文件，创建出镜像文件。

手写dockerfile还是比较麻烦的，所以我们可以在Docker的镜像仓库里面寻找别人已经创建好的镜像，比如说你想部署Java程序，那么就在线下载Java镜像即可，非常简单。

毕竟Docker镜像是只读层，如果我们想要往里面部署层序应该怎么办呢？这个很简单，我们可以给镜像创建一个容器，容器是可读可写的，我们把程序部署在容器里就行了。

#### 5.2.2 Docker容器

我们说的在Docker虚拟机中创建实例，指的就是容器。因为镜像的内容是只读的，想要部署程序，我们需要创建出容器实例，容器的内容是可以读写的，可以用来部署程序。

而且容器之间是完全隔离的，我们不用担心一个容器中部署程序，会影响到另一个容器。就比如说我们在CentOS上直接安装MySQL 8.0，它跟Percona Toolkit有冲突，跟Sysbench也有冲突，所以我们做在线修改表结构，以及做压力测试的时候，都是挑选新的虚拟机实例来安装这些程序，访问MySQL的。如果用上了Docker，我可以在A容器里安装MySQL，在B容器跑压力测试，根本不会有冲突。

再有，必须先有镜像，才能创建出容器，镜像和容器之间是关联的关系。而且一个镜像可以创建出多个容器，像是SaaS云计算，运营商可以把进销存系统打成镜像。有企业购买进销存系统，那么运营商就给客户创建一个容器，客户的进销存数据保存在容器A里面。再有客户购买进销存系统，运营商就创建容器B，以此类推。云计算服务商就是这么卖软件的。

#### 5.2.3 安装Docker虚拟机

```shell
yum install -y docker
```

```shell
service docker start
```

```shell
service docker stop
```

#### 5.2.4 操作Docker虚拟机

![](笔记图片\1.png)

##### 设置镜像加速器

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

编辑/etc/docker/daemon.json文件，把结尾的逗号去掉

#### 管理镜像

```shell
#搜索镜像
docker search 关键字
#下载镜像
docker pull 镜像名字
#查看镜像
docker images
#重命名镜像
docker tag 旧镜像 新镜像
#删除镜像
docker rmi 镜像名字
#导出镜像
docker save -o 压缩文件路径 镜像名字
#导入镜像
docker load < 压缩文件路径
```

#### 创建容器

```shell
#创建普通容器
docker run -it --name 别名 镜像名字 程序名字
#创建含有端口映射的容器
docker run -it --name 别名 -p 宿主机端口:容器端口 镜像名字 程序名字
#创建含有挂载目录的容器
docker run -it --name 别名 -v 宿主机目录:容器目录 --privileged 镜像名字 程序名字
```

#### 操作容器状态

```shell
#暂停容器
docker pasue 容器
#恢复容器
docker unpause 容器
#停止容器
docker stop 容器
#启动容器
docker start -i 容器
```

#### 创建Swarm集群

因为我们要利用Docker环境搭建数据库集群，如果把所有的MySQL节点都部署在同一个Docker虚拟机之内，要是宿主机宕机，那么Docker里面所有的容器都不能使用了，数据库集群就彻底瘫痪了。所以我们应该采用分布式部署的方案，把MySQL节点部署在不同的Docker虚拟机之上。不光是数据库集群要采用分布式部署，像什么Java程序、PHP程序也都要采用分布式部署。

![](笔记图片\2.png)

```shell
#创建Swarm集群（该节点自动变成管理节点）
docker swarm init
#查看Swarm集群中的Docker节点（管理节点上执行）
docker node ls
#删除Swarm集群的Docker节点（管理节点上执行）
docker noode rm 节点ID -f
#退出Swarm集群（Workd节点上执行）
docker swarm leave
#退出Swarm集群（管理节点上执行）
docker swarm leave -f
```

```shell
#查看虚拟网络
docker network ls
#创建虚拟网络
docker network create -d overlay --attachable 虚拟网络名称
#删除虚拟网络（先删除该网络上部署的容器）
docker network rm 虚拟网络名称
```

