# 数据库(MySQL)使用笔记

###### 前言：本人不够专业若是文中表述有问题的地方还请读者批评指正，所有sql语句本人亲自敲过，若在使用过程中出现任何问题都可与我联系，本人qq：946115360。文中所有超链接打开方式，按住[^ctrl]+[^shift]再单击链接。

###### 阅读本文档须知：{}为必填项，[]为选填项，*代表所有字段，切记！！！

#### 1、数据库的操作(CU)

###### 1.1、创建数据库的过程以及使用(create and use)

```mysql
1.CREATE DATABASE {库名}; #创建数据库，jdbc为数据库
#成功的标志（下同）
Query OK, 1 row affected (1.76 sec) #表示创建成功
```

```mysql
1.USE {库名}; #使用创建的数据库
Database changed #表示成功使用此数据库
```

###### 1.2数据库字符指定（character）

```mysql
1.CREATE DATABASE {库名} character set utf8 collate utf8_bin;
2.CREATE DATABASE {库名} character set gb2312 collate gb2312_chinese_ci;
#注意创建数据库避免库名与关键字重复，比如（table、like、desc、······）
```

###### 1.3复制表结构

```mysql
 create table {新表名} as(select *[可复制选中字段] from {被复制的表名});
 #中括号内可以写你想复制字段的结构，*是默认选中表中所有字段，下面很多地方都是差不多的意思
```



#### 2、数据库表的操作(CI)

###### 2.1、建表及插入数据(create and insert)

```mysql
 1.  CREATE TABLE {表名}(   #为数据库添加一张名为tb_user的表
     id INT PRIMARY KEY AUTO_INCREMENT,#将id设置为主键，并自动递增
     NAME VARCHAR(40),
     sex VARCHAR(2),
     email VARCHAR(60),
     birthday DATE
     );
Query OK, 0 rows affected (3.77 sec) #表示添加成功
```

```mysql
1.INSERT INTO {表名}(id，NAME,sex,email,birthday) #为数据库添加数据
-> VALUES (1，'jack','男','jack@126.com','1980-01-04'),
->        （2，'Tom','男','tom@126.com','1981-02-14'),
->        (3，'Lucy','女','lucy@126.com','1979-12-28');
Query OK, 3 rows affected (1.14 sec) #表示添加成功
Records: 3  Duplicates: 0  Warnings: 0
```

###### 2.2、查看数据是否添加成功及表结构(select and desc)

```mysql
1.select * from tb_user;
+----+------+------+--------------+------------+
| id | NAME | sex  | email        | birthday   |
+----+------+------+--------------+------------+
|  1 | jack | 男   | jack@126.com | 1980-01-04 |
|  2 | Tom  | 男   | tom@126.com  | 1981-02-14 |
|  3 | Lucy | 女   | lucy@126.com | 1979-12-28 |
+----+------+------+--------------+------------+
3 rows in set (0.13 sec)
```

```mysql
1.desc tb_tea;  #查看表结构
   +-------+-------------+------+-----+---------+-------+
   | Field | Type        | Null | Key | Default | Extra |
   +-------+-------------+------+-----+---------+-------+
   | id    | int(11)     | YES  |     | NULL    |       |
   | name  | varchar(20) | YES  |     | NULL    |       |
   | pwd   | varchar(10) | YES  |     | NULL    |       |
   +-------+-------------+------+-----+---------+-------+
   3 rows in set (0.18 sec)
```



#### 3、数据库几大常用命令（IDUS)

###### 3.1、数据表信息的增加（insert和 replace）

```mysql
1.INSERT INTO {表名}(id，NAME,sex,email,birthday) #为数据库添加数据
-> VALUES (1，'jack','男','jack@126.com','1980-01-04');
2.replace into {表名} value(1,'财务部','');
#insert和 replace的区别详情见5.1
```

###### 3.2数据表信息的删除（delete）

```mysql
1.delete from * {表名} where id=1(写上它的id字段就成);#删除一条信息，此为条件删除
1.1.delete from {表名} where income>2500; #删除满足条件数据
2.truncate table {表名};，这个命令可以快速删除表中的所有数据，是不带条件的删除。慎用
#truncate 和 delete的区别详情见5.2
```

###### 3.2.1、删除表结构（drop）：

```mysql
1.drop table {表名称};
2.drop database {库名称};
```

###### 3.3、简单修改表中信息（update）

```mysql
1.update {表名} set phone=15226332115 where id=1; #指定修改列
2.update {表名} set phone=15226332115 ; #默认将phone这一列全修改为指定值
3.update {表名} set NAME='王六',password=258369 where id =3;  #一次修改多列数据
4.update {表名} set income=income+100; #让表中数据自增相应的值
```

###### 3.4简单条件查询及大集锦（select）

###### 3.4.1.简单查询，拼接SQL，之模糊查询（关于like的详细讲解请移步至3.4.3）及范围查询（select）

```mysql
 1.select name from {表名} where name like '%王%'; #指定列查询只查出名字，没有其它列信息
 2.select * from {表名} where name like '%王%'; #关于王的所有信息都将会被查询出来
 3.select [name,linux,html,mysql] from {表名};  #不带条件的显示指定列的内容
 4.select [name,linux,html,mysql] from {表名} where id=6;  #带条件的显示指定列的内容
 5.select * from {表名} where mysql>80;        #单一范围查询
 6.select * from {表名} where mysql>80 and mysql<90;   #复合范围查询
 7.select [id,studentid,name] from {表名} where mysql>80 and mysql<90; #组合使用查出有用信息
 8.select * from {表名} where username='lisi';
```



###### 3.4.2.数据统计

```mysql
#select 统计公式
#关键字
1. sum #求选中字段的和，也可指定范围
select sum({字段名}) from {表名};
select sum({字段名}) from {表名} where {主键} between {范围常量1} and {范围常量2};(下面的略同)
2.max #求选中字段的最大值，也可指定范围
select max({字段名}) from {表名};
3. min #求选中字段的最小值，也可指定范围
select min({字段名}) from {表名};
4. count #统计表中数据量
select count(*) from {表名};
5. avg #求选中字段的平均值，也可指定范围
select avg({字段名}) from {表名};
#6. 统计每一条数据的总和（例如学生各科成绩的总和，员工的基本工资，奖金，补贴的总和）
select *,(`{字段名1}`+`{字段名2}`) as [自定义字段保存求和数据] from {表名} where {主键}=1;
select *,(`{字段名1}`+`{字段名2}`) as sum from [自定义字段保存求和数据] where {主键} between {范围常量1} and {范围常量2};
#7.select 函数（year,month,day,hour,minute,second，now）
select [year](now()) #当前年份，依次类推，函数名后的括号中也可以跟时间类型数据的字段，提取相应年份
select year(now()) as 年,month(now()) as 月,day(now()) as 日; #查询当前系统的年月日
select hour(now()) as 时,minute(now()) as 分,second(now()) as 秒; #同上
#8.常用逻辑运算符（not 逻辑非，and 逻辑与，in 作为一个容器存储返回满足条件结果集，between...and... 用于限制某一字段满足条件的区间）
where {表达式} [not/and/in] #也可组合使用
where {表达式} between {条件1} and {条件1};
#9.转义字符(当查询条件中存在 _ 需用到转义字符)
where gdName like '华为P9\_PLUS';
#10.is null判断某一列是否为空值
select * from {表名} where {字段名} is [not] null;
#11.数据的排序(默认升序asc,降序为desc)
select * from {表名} order by {字段名} [asc/desc]
select * from {表名} order by {字段名} [asc/desc],{字段名} [asc/desc] #多列字段排序
#12.limit限制结果集返回的函数
select * from {表名} limit {要显示的行数}
select * from {表名} limit {要开始显示的行数，要结束显示的行数}
#13.数据的分组(用途：结合count 统计数据，消除重复项等等...)
select * from {表名} group by {字段名}
```

###### 3.4.3REGEXP和like的比较

```mysql
#1. 查找(类似于like)
select * from {表名} where {字段名} regexp '具体的值';
select * from {表名} where {字段名} like '%具体的值%';
# 2.查找字段中以“xxx”开头的记录
select * from {表名} where {字段名} regexp '^xxx'
select * from {表名} where {字段名} like 'xxx%'
# 3.查找字段中以“xxx”结尾的记录
select * from {表名} where {字段名} regexp 'xxx$'
select * from {表名} where {字段名} like '%xxx'
# 4.查找字段中包含“xxx”、“xxxx”或“xxxxx”
select * from {表名} where {字段名} REGEXP 'xxx|xxx|xxx'
# 5.查找字段不包含“xxx”字符串的记录
select * from {表名} where {字段名} not REGEXP 'xxx'
#6.mysql 如何判断 "xxx" 是否为 "数字"
select ('123a' REGEXP '[^0-9.]'); ( #mysql中常量true输出为1 false输出为0,很显然结果为1)
select ('select gdName from goods where gdID =10' REGEXP '[^0-9.]') as 判断是否为字符串1代表是0代表否;

```

###### 3.4.4MySQL 正则表达式常用功能对照表

| 模式       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| ^          | 匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，^ 也匹配 '\n' 或 '\r' 之后的位置。 |
| $          | 匹配输入字符串的结束位置。如果设置了RegExp 对象的 Multiline 属性，$ 也匹配 '\n' 或 '\r' 之前的位置。 |
| .          | 匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用象 '[.\n]' 的模式。 |
| [...]      | 字符集合。匹配所包含的任意一个字符。例如， '[abc]' 可以匹配 "plain" 中的 'a'。 |
| [^...]     | 负值字符集合。匹配未包含的任意字符。例如， '[^abc]' 可以匹配 "plain" 中的'p'。 |
| p1\|p2\|p3 | 匹配 p1 或 p2 或 p3。例如，'z\|food' 能匹配 "z" 或 "food"。'(z\|f)ood' 则匹配 "zood" 或 "food"。 |
| *          | 匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。 |
| +          | 匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。 |
| {n}        | n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。 |
| {n,m}      | m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。 |

(备注：此表来源于网络包括REGEXP和like的比较，出处：）

https://www.jb51.net/article/180533.htm

###### 3.4.5连接查询多表数据

```mysql
#1.内链接使用比较运算符比较两个表中共有的字段列，只返回满足条件的记录行
select * from {表一} a join {表二} b on a.{字段名}=b.{字段名} [where] {表达式};
select * from {表一} a join {表二} b join {表三} c on a.{字段名}=b.{字段名} and b.{字段名}=c.{字段名} [where] {表达式}; #以此类推
#2.外连接，显示满足条件和不满足的主表（对于左外连接，left join 左边的表为主表，反之右外连接同理）记录行。
select * from {表一} a [left/right] join {表二} b on a.{字段名}=b.{字段名} [where] {表达式} [order by/group by];   #(注意有where条件时分组和排序需写在where之后)
#3.联合查询多表数据
select 语句1 union [all] select 语句2 [union [all] <select 语句3>] [...]
#4.子查询多表数据
select * from {表一} where {字段名1} {=/in} (select {字段名1} from {表一} where {字段名2} = 'xxx')
#(注意有时select子句查询出来的不是一个确定的值，而是符合条件的一列值，这时再用 = 就显得不太合适了，用in就行。)
#5.内链接，子查询，排序，分组，聚合，结合的复杂查询语句
(1)select {待查询的字段名} from {表一} u join {表二} o on u.uID=o.uID where year(o.{xxx字段名})=2017 and o.{字段名1} in (select {字段名1} from {表二} group by {字段名1} having sum({xxx字段名})>=1000) group by o.{字段名1};
(2)select {待查询的字段名1} as 别名1 , sum({待查询的字段名2}) as 别名2 from  {表一} g2 join {表二} g on g.tID = g2.tID where {字段名1} in (select {字段名1} from {表三} where {字段名2} in (select {字段名2} from {表四} where year({xxx字段名})=2017)) group by {待查询的字段名1} order by 别名2 desc;
```

###### 3.4.6子查询之封装数据

```mysql
#对于一些复杂的业务逻辑，普通子查询或删除已经不能满足我们的需求，于是我引入封装数据进行查询或删除
[DELETE ,SELECT]
FROM
	{表明} 
WHERE
	商品 ID = ( SELECT {封装字段1} FROM (SELECT MAX({字段名}) AS {封装字段1} FROM {表明} ) AS {封装字段2} );
#将查询的结果值作为表明 {eg:(SELECT MAX({字段名}) AS {封装字段1} FROM {表明} )},前提是查询的结果值的命名必须是外层查询的字段名，这样就达到封装数据的目的。
```



#### 4、补充知识(AS)

###### 4.1、要想让乱掉的id重新排序并自增，执行下面两行代码(alter)

```mysql
1. alter table {表名} drop column id; #id为主键名，stu_sro为表名
2. alter table {表名} add id mediumint(8) not null primary key auto_increment first;
```

###### 4.3、运行SQL脚本(source)及导出命令（mysql dump）

> ```mysql
> 1.source +sql文件路径#导入SQL脚本命令
> #2.包含数据的导出(解决导出乱问题，需注意：字符类型要与数据库的全局编码一致，导出若不指定位置，默认在mysql的bin目录下。)
> mysqldump {表名} --default-character-set={字符类型} >[导出路径]\[sql 文件名].sql -uroot -p[密码]
> mysqldump -uroot -p[密码] {库名} [表名] --default-character-set={字符类型} > [导出路径]\[sql 文件名].sql
> #3.不包含数据的导出，只包含结构
> mysqldump -d {库名} --default-character-set={字符类型} >[导出路径]\[sql 文件名].sql -uroot -p[密码]
> mysqldump -uroot -p[密码] -d {库名} [表名] --default-character-set={字符类型} > [导出路径]\[sql 文件名].sql(注意导出语句，须在DOS窗口，数据库环境下不能导出)
> ```

###### 4.2.2中文乱码的产生以及导入数据库产生的问题

```mysql
/*中文乱码是由不同的编码格式存储汉字的字节数不同造成的*/
set names gbk/utf8 /*可以在gbk和utf8之间转换*/
```



###### 4.2、中文乱码解决办法之一(alter)

```mysql
1. alter table {表名} convert to character set utf8; #修改不能兼容中文字符的表，其中tb_user为表名
2. alter DATABASE {库名} character set utf8 collate utf8_bin;#修改不能兼容中文字符的数据库，其中jdbc为库名
```

###### 4.2.1中文不能插入解决方案

![](F:\Users\notbook\mysqlnotbook\QQ截图20210412210345.png)

![](F:\Users\notbook\mysqlnotbook\QQ截图20210412205902.png)

![QQ截图20210412210009](F:\Users\notbook\mysqlnotbook\QQ截图20210412210009.png)

![QQ截图20210412210046](F:\Users\notbook\mysqlnotbook\QQ截图20210412210046.png)

![QQ截图20210412210259](F:\Users\notbook\mysqlnotbook\QQ截图20210412210259.png)

这样就会解决中文不能插入问题。记住表和字段都要设置为utf8。

#### 5.同种功能不同命令的区别（delete和truncate，insert 和replace）

###### 5.1insert 和replace的区别

当一个表中存在主键或唯一索引时，使用replace into 语句插入数据时，会先把冲突的旧数据删除，然后插入新数据。而insert into则会报错。用图片更直观的告诉你。



![QQ截图20210412230853](F:\Users\notbook\mysqlnotbook\QQ截图20210412230853.png)

![QQ截图20210412231048](F:\Users\notbook\mysqlnotbook\QQ截图20210412231048.png)

###### 5.2delete和truncate的区别

1、 Truncate比Delete所用的事务日志空间更少

2、 Truncate比Delete使用锁通常较少

3、 TRUNCATE对表中的所有页都清空

#### 6alter的常用方法（作用修改表结构，设置数据类型）

###### 6.1  alter公式大集锦

```mysql
#alter table {表名} +关键词+操作对象  #alert基本公式
#下面介绍关键词的作用及用法
1.rename#换表名用法如下
alter table {旧表名} rename {新表名};
2.change #改字段名及字符类型
alter table {表名} change {旧字段名} {新字段名} {数据类型};
3.modify #修改字符类型，调整字段位置,定义唯一性约束
alter table {表名} modify {字段名} {数据类型};
alter table {表名} modify {字段名1} {数据类型} [first,after] {字段名2};
alter table {表名} modify {字段名} {数据类型} not null unique;
4.drop #删除指令（字段的删除，库的删除，表的删除详情看3.2.1）
alter table {表名} drop {字段名};
5.add #添加字段指令
alter table {表名} add {字段名} {数据类型} after {定位字段名};
6.default #指定默认值
alter table {表名} modify {字段名} enum('[字段限制值1]','[字段限制值2]') default '[enum中的一个]';
#指定默认值为当前系统时间
alter table {表名} modify {字段名} timestamp default current_timestamp;
7.设置主表与从表的联系
  cascade     #主表的更新与删除操作，如果修改和删除的值被其他表引用，从表也要删除更新相关数据
  set null   #在更新和删除表记录时，从表相应值设为空
  no action  #不做任何操作
  restrict   #拒绝主表更新或修改外键的关联列
alter table {从表名} add constraint {外键名}
foreign key({外键字段名}) references {主表名}({主键字段名})
    -> on update [cascade,set null,restrict,no action]
    -> on delete [cascade,set null,restrict,no action];
8.删除主从表间关系
foreign key#外键关键字
alter table {表名} drop foreign key {外键名};
```

#### 7.初识java链接数据库(JDBC)

1.java连接数据库是需要驱动的，请移步到官网下载相对应的驱动 https://downloads.mysql.com/archives/c-j/

所谓的驱动就是一个jar包，jar包的导入上一次已经说过了（新用户请私信我）。

```java
package FirstJDBC;

import java.sql.*;

public abstract class tb_tea {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Connection conn = null;
		Statement stmt = null;
		ResultSet rs = null;
		try {
			Class.forName("com.mysql.jdbc.Driver");
			String url = "jdbc:mysql://localhost:3306/jdbc"; //jdbc为库名
			String username = "root";    //数据库用户名
			String password = "945942"; //数据库密码
			conn = DriverManager.getConnection(url, username, password);
			stmt = conn.createStatement();  
			String sql = "select * from tb_tea"; //sql语句
			rs = stmt.executeQuery(sql); //调用executeQuery方法执行sql语句，将查询的结果存储在rs中
			System.out.println("id  |  name  |  pwd  \t|   ");
			while (rs.next()) {  //通过循环取出rs中的结果
				int id = rs.getInt("id"); 
				String name = rs.getString("name");
				String pwd = rs.getString("pwd");

				System.out.println(id + "\t|" + name + "\t|" + pwd + "|");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
            //查询完成后需要关闭资源。切记！！！
			if (rs != null) {
				try {
					rs.close();
				} catch (SQLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if (stmt != null) {
				try {
					stmt.close();
				} catch (SQLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			if (conn != null) {
				try {
					conn.close();
				} catch (SQLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			System.out.println("数据库连接成功！");
		}
	}

}

```

> 事实上java链接mysql远不止这么简单，在实际环境中我们往往需要防止sql注入，和mysql的资源分配，以及响应速度，资源重用。其实原生的jdbc已经被mybatis和mybatis-plus框架替代了，这也并不是说jdbc没有用武之地，恰恰相反，jdbc是我们学习数据库连接框架的基础。

#### 8.SQL流程控制（SQL编程）

###### 8.1初始sql编程

> 8.1.1变量的定义与声明
>
> sql语言中变量分为4中

- 用户变量：用@做为前导标识，在mysql会话末端结束其定义

- 系统变量和服务器变量：用@@做为前导标识申明

- 结构化变量：是系统变量的一种特例。mysql目前只在定义更多的MyISAM索引缓存才会使用

- 局部变量：没有前导标识，定义时注意与数据库表中的字段加以区别

  > (1)用户变量例子

```mysql
set @id = 3;                    #定义整型
select @name := 'backpackerxl'; #定义字符串
select * from {表明} where id = @id and name = @name;
```

> (2)局部变量例子

```mysql
CREATE PROCEDURE proc_add ( IN a INT, IN b INT ) BEGIN
	DECLARE
		c INT DEFAULT 0;
	
	SET c = a + b;
	SELECT
		c AS 'result';
	
END;
```

> （3）系统变量

```mysql
set @@wait_timeout = 10000; --会话变量
set @@global.wait_timeout = 1000; --全局变量

```

> （4）常量

- 字符串常量：eg 'backpackerxl',"backpackerxl"都是字符串常量，还有转义字符，与java类似不做过多解释
- 数值常量
- 日期时间常量
- 布尔值常量
- null值常量

###### 8.1.2运算符与表达式

- 算术运算符：+，-，*，/，%
- 赋值运算符： = ，:=
- 逻辑运算符：!(NOT)  ,&& (AND)  ,|| (OR)  ,XOR
- 位运算符：& ,^ , <<  ,>>  , ~
- 比较运算符：= ， <> ,(!=) ,<=> ,< , > ,<= , >= , is null

> 例子

```mysql
set @x=5,@y=6;
set @x=@x+@y;
select @x;
```

