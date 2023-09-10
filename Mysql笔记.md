#### 1.数据库基础

**数据库**：保存有组织的数据的容器（一个文件）

**数据库管理系统**：数据库软件

**表**：某种特定类型数据的结构化清单

- ​	表名唯一
- **列**：表中的一个字段，表由一个列或多个列组成
- **数据类型**：每个字段都有相应的类型
- **行**：数据库记录
- **主键**：表中每一行都有可以唯一标识自己的一列（一组列）
  - 其值能够唯一区分表中每个行
  - 主键列不允许为NULL

#### 2.MySQL

- ​	MySQL 是一种DBMS，数据库软件

  - 基于共享文件系统的DBMS（Microsoft Access、FileMaker）
  - 基于客户机-服务器的DBMS（MySQL， Oracle、Microsoft SOL Server）

- `mysql -u brn -p -h myserver -P 9999`

  - ​       -u:用户  -p:密码      -h :ip             -P :端口号（大P）
  - ; 或\g 结束
  - help 或 \h
  - quit 或 exit 退出

  #### 命令

  ```sql
  show databases;										//查看数据库
  
  create database dbtest1; 							//创建新数据库
  
  show ctreate database dbtest1; 						// 查看数据库信息
  
  use dbtest1;										//使用数据库
  
  show tables;										//查看表
  
  create table employees(id int, name varchar(15)); 	//建表
  
  show create table employees; 						// 查看表的信息
  
  show columns from employees;						//查看表字段
  	--	同 describe employees;
  show status;										//显示服务器状态信息
  
  show grants , 用来显示授予用户的安全权限
  show errors, show warnings ， 显示错误和警告
  help show -- 了解更多 show
  
  insert into employees values(1001, 'Tom'); 			//插入数据
  
  select * from employees;							//查看表中数据
  
  drop dbtest1;  										//删除数据库
  
  
  show varibles like 'character_%';  					//显示字符集
  show varibles like 'collation_%';  					//显示比较规则
  ```

#### 3.检索数据（SELECT）

- sql 不区分大小写
- 许多SQL 开发人员 喜欢 SQL 关键词用大写
- 表名，列名用小写

```sql
SELECT prod_name FROM products;							//检索peoducts 的prod_name 列

SELECT prod_id, prod_name, prod_price FROM products;	//检索多个列
	
SELECT * FROM products;									//检索所有列，但是检索不需要的列会降低程序性能

SELECT vend_id FROM products;							//如果有重复的值
SELECT DISTINCT vend_id FROM products;					//去除重复的值

SELECT prod_name FROM products LIMIT 5;					//限制显示的行数为5
SELECT prod_name FROM products LIMIT 5,5;				//LIMIT 5,5 指示MySQL返回从行5开始的5行。

/*
MySQL 5的LIMIT语法 LIMIT 3，4的含义是从行4开始的3行还是从行3开始的4行?如前所述，它的意思是从行3开始的4行，这容易把人搞糊涂,

由于这个原因，MySQL 5支持LIMIT的另一种替代语法。LIMIT 4 0FFSET 3意为从行3开始取4行，就像LIMIT 3，4一样。
*/

SELECT products.prod_name FROM crashcourse.peoducts;	//使用完全限定的表名
```

#### 4.排序检索数据(ORDER BY、DESC 、ASC)

```sql
SELECT prod_name FROM products ORDER BY prod_name;  	//递增排序

SELECT prod_id, prod_price, prod_name 
FROM products
ORDER BY prod_price,prod_name;							//按多列排序
/*
换句话说，对于上述例子中的输出，仅在多个行具有相同的prod_price值时才对产品按prod name进行排序。
*/

SELECT prod_id，prod_price,prod_name
FROM products
ORDER BY prod_price DESC;								//按价格降序排列

SELECT prod_id，prod_price, prod_name
FROM products
DRDER BY prod_price DESC，prod_name;					   //多列排序
/*
prod price列指定DESC，对prod name列不指定。因此prod price列以降序排序，而prod name列(在每个价格内)仍然按标准的升序排序。

与DESC相反的关键字是ASC(ASCENDING),在升序排序时可以指定它但实际上，ASC没有多大用处，因为升序是默认的 
*/

//使用ORDER BY和LIMIT的组合，能够找出一个列中最高或最低的值下面的例子演示如何找出最昂贵物品的值:

SELECT prod_price FROM products
ORDER BY prod_prices DESC
LIMIT 1；
```

#### 5.过滤数据（WHERE）

```sql
SELECT prod_name, prod_price FROM products
WHERE prod_price = 2.50;

/*
WHERE子句的位置 在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后，否则将会产生错误
*/
							    WHERE子句操作符
				操作符                                  说明
				
				  =									   等于
				  <>							      不等于
				  !=							      不等于
				  <									   小于
				  <=								 小于等于
				  >									   大于
 				  >=								 大于等于
				BETWEEN   AND  						在指定的两个值之间
				
SELECT prod_name, prod_price
FROM products
WHERE prod_name = 'fuses';		//检查单个值 

SELECT prod_name,prod_price
FROM products
WHERE prod_price < 10;

SELECT prod_name,prod_price
FROM products
WHERE prod_price <= 10:

SELECT vend_id, prod_name
FROM products
WHERE vend_id <> 1003;			//不匹配检查

同

SELECT vend_id,prod_name
FROM products
WHERE vend_id != 1003;

SELECT prod_name,prod_price
FROM products
WHERE prod_price BETWEEN 5 AND 10;		//范围检查

SELECT prod_name
FROM products
WHERE prod_price IS NULL;		//空值检查

SELECT cust_id
FROM customers
WHERE cust_emai IS NULL:


```

#### 6.数据过滤（WHERE 附加 AND、OR、NOT、IN）

```sql
SELECT prod_id,prod_price,prod_name
FROM products
WHERE vend_id = 1003 AND prod_price <= 10;		// AND

SELECT prod_name,prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003;			// OR

//两者结合   
SELECT prod_name,prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;

// 但是结果中有两行数据 价格 小于 10 
// AND 优先级更高 
// 在于 在处理OR 之前 先处理了 AND ，结果就是 WHERE vend_id = 1002 OR (vend_id = 1003 AND prod_price >= 10)

//改变
SELECT prod_name,prod_price
FROM products
WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;

//同
SELECT prod_name,prod_price
FROM products
WHERE vend_id IN (1002, 1003) AND prod_price >= 10;			// IN

SELECT prod_name,prod_price
FROM products
WHERE vend_id IN (1002,1003) ORDER BY prod_name;

SELECT prod_name,prod_price
FROM products
WHERE vend_id NOT IN (1002,1003) ORDER BY prod_name;		// NOT IN

/*
MySQL中的NOTMySQL支持使 用NOT 对 IN、 BETWEEN和EXISTS子句取反，这与多数其他DBMS允许使用NOT对各种条件取反有很大的差别.
*/
```

#### 7.用通配符进行过滤（LIKE、% 、_）

```sql
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 'jet%';			/* 以jet起头的词。%告诉MySQL接受jet之后的任意字符，不管它有多少字符。*/


SELECT prod_id，prod_nameFROM products
WHERE prod_name LIKE  '%anvi7%';			/* '%anvi1%' 表示匹配任何位置包含文本anvil的值，而不论它之前或之后出现什么字符。*/

SELECT prod_name
FROM products
WHERE prod_name LIKE 's%e';  //匹配 s 开头 e 结尾的

/*注意NULL 虽然似乎%通配符可以匹配任何东西，但有一个例外，即NULL。即使是WHERE prod name LIKE%也不能匹配用值NULL作为产品名的行。*/

/*
另一个有用的通配符是下划线 _ 下划线的用途与%一样，但下划线只匹配 单个字符 而不是多个字符。
*/
SELECT prod_id,prod_name
FROM products
WHERE prod_name LIKE'_ ton anvi7';

/*
 通配符搜索的处理一般要比前面讨论的其他搜索所花时间更长
 不要过度使用通配符。如果其他操作符能达到相同的目的，应i使用其他操作符。
 */
```

#### 8. 正则表达式

```sql
//基本字符匹配
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000'
ORDER BY prod_name;						/*下面的语句检索列prod_name包含文本1000的所有行*/ 


SELECT prod_name
FROM products
WHERE prod_name REGEXP '.000'
ORDER BY prod_name;					/* 使用了 正则 ".000" . 表示匹配任意一个字符 */ 

// 进行OR匹配

SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000|2000'
ORDER BY prod_name;	

//匹配几个字符之一
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[123] Ton'
ORDER BY prod_name;	
/*正如所见，[]是另一种形式的OR语句。事实上，正则表达式[123]Ton为[1 2 3]Ton的缩写，也可以使用后者。*/

SELECT prod_name
FROM products
WHERE prod_name REGEXP '1|2|3 Ton'
ORDER BY prod_name;	

/*之所以这样是由于MySQL假定你的意思是"1'或2'或'3 ton'。*/


//匹配范围
/*
[0-9] 匹配0-9 
[a-z] 匹配任意字母字符
*/
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[1-5] Ton'
ORDER BY prod_name;	

//匹配特殊字符
// \\.  表示匹配 .  \\- 表示匹配 - \\\ 表示匹配 \

SELECT prod_name
FROM products
WHERE prod_name REGEXP '.'
ORDER BY prod_name;

SELECT prod_name
FROM products
WHERE prod_name REGEXP '\\.'
ORDER BY prod_name;

SELECT prod_name
FROM products
WHERE prod_name REGEXP '\\([0-9] sticks?\\)'	
ORDER BY prod_name;

/*正则表达式\\([o-9] sticks?\\)需要解说一下。
\\( 匹配 )

[0-9]匹配任意数字 (这个例子中为1和5)

sticks? 匹配stick和sticks 
(s后的?使s可选，因为?匹配它前面的任何字符的0次或1次出现)，=

\\) 匹配 )

没有?，匹配stick和sticks会非常困难。
*/

SELECT prod_name
FROM products
WHERE prod_name REGEXP '[[:digit:]]{4}'	
ORDER BY prod_name;
/*
[:digit:]匹配任意数字，因而它为数字的一个集合。
{4}确切地要求它前面的字符 (任意数字)出现4次，
所以 [[:digit:]]{4} 匹配连在一起的任意4位数字。
*/
// 同
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[0-9][0-9][0-9][0-9]'	
ORDER BY prod_name;


//定位符

SELECT prod_name
FROM products
WHERE prod_name REGEXP '^[0-9\\.]'	
ORDER BY prod_name;
/*
^[O-9\\.]只在 . 或 任意数字 为串中 第一个字符 时才匹配它们。
*/

/*
使REGEXP起类似LIKE的作用 
LIKE和REGE的不同在于，LIKE匹配 整个串 而REGEXP匹配 子串。
利用定符，通过用^开始每个表达式，用$结束每个表达式，可以REGEXP的作用与LIKE一样。
*/
```

![image-20230519163531615](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230519163531615.png)

![image-20230519163608492](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230519163608492.png)

![image-20230519163711283](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230519163711283.png)

![image-20230519164333751](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230519164333751.png)

#### 9.创建计算字段（Concat）

```sql
/*拼接字段*/
SELECT Concat(vend_name, ' (', vend_country,')')
FROM vendors
ORDER BY vend_name;


/* 删除数据右侧多余的空格来整理数据*/
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')')
FROM vendors
ORDER BY vend_name;
/* RTrim() 函数去电右边所有的空格*/
/*MySQL除了支持RTrim()(正如刚才所见，它去掉串右边的空格 )，还支持LTrim() (去掉串左边的空格) 以及Trim() ( 去掉串左右两边的空格 )*/


/* 使用别名*/

SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS 
vend_title
FROM vendors
ORDER BY vend_name;


/*执行算术计算*/

SELECT prod_id, quantity, item_price
FROM orderiterms
WHERE order_num = 20005;

SELECT prod_id, quantity, item_price, quantity*item_price AS expanded_price
FROM orderiterms
WHERE order_num = 20005;

/*输出中显示的expanded_price列为一个计算字段，此计算为quantity*item_price。
客户机应用现在可以使用这个新计算列，就像使用其他列一样。*/
```

![image-20230522205706780](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522205706780.png)

![image-20230522205736925](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522205736925.png)

#### 10 使用数据处理函数

##### 文本处理函数

```sql
/* 文本处理函数*/
Upper()

SELECT vend_name, Upper(vend_name) AS vend_name_upcase
FROM vendors
ORDER BY vend_name;

/*SOUNDEX是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。SOUNDEX考虑了类似的发音字符和音节，使得能对串进行发音比较而不是字母比较。虽然SOUNDEX不是SOL概念，但MySOL(就像多数DBMS一样)都提供对SOUNDEX的支持。*/
SELECT cust_name,cust_contact
FROM customers
WHERE Soundex(cust_contact) = Soundex('Y Lie');

```

![image-20230522205911306](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522205911306.png)



![image-20230522210020064](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522210020064.png)

SOUNDEX是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。SOUNDEX考虑了类似的发音字符和音节，使得能对串进行发音比较而不是字母比较。虽然SOUNDEX不是SOL概念，但MySOL(就像多数DBMS一样)都提供对SOUNDEX的支持。

##### 日期和时间处理函数

```sql
SELECT cusr_id, order_num,
FROM orders
WHERE order_date='2005-09-01';

SELECT cusr_id, order_num,
FROM orders
WHERE Date(order_date) ='2005-09-01';
/*
指示MySQL仅将给出的日期与列中的日期部分进行比较，而不是将给出的日期与整个列值进行比较。为此，必须使用Date()函数。Date(order date)指示MySOL仅提取列的日期部分，更可靠的
*/

/* 匹配 2005 年9月*/
SELECT cusr_id, order_num,
FROM orders
WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';

//同
SELECT cust_id,order_num
FROM orders
WHERE Year(order_date) = 2005 AND Month(order_date) = 9;
```

![image-20230522210425212](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522210425212.png)

##### 数值处理函数



![image-20230522211038573](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522211038573.png)

#### 11 汇总数据

##### 聚集函数

![image-20230522211216371](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522211216371.png)

```sql
/*AVG()*/
SELECT AVG(prod_price) AS avg_price
FROM products;

SELECT AVG(prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;
/* AVG() 只能用来确定特定数值列的平均值
	获得多个平均值需要使用多个AVG()
	NULL 自动忽略NULL的行
*/

/*COUNT()*/
SELECT COUNT(*) AS num_cust
FROM customers;

SELECT COUNT(cust_email) AS num_cust
FROM customers; /*只对具有电子邮件地址的客户计数*/


/*MAX()*/

SELECT MAX(prod_price) AS max_price
FROM products;

/*MIN()*/
SELECT MIN(prod_price) AS min_price
FROM products;
//返回数值列中的最小值，返回文本列中的最小值，自动忽略NULL行

/*SUM()*/

SELECT SUM(quantity) AS items_ordered
FROM orderitems
WHERE order_num = 20005; /* 订单中所有物品数量之和*/

SELECT SUM(item_price*quantity) AS total_price
FROM orderitems
WHERE order_num = 20005;
```

##### 聚集不同值（DISTINCT）

```sql
/*ALL为默认 ALL参数不需要指定，因为它是默认行为。如果不指定DISTINCT，则假定为ALL。*/
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM products
WHERE vendid=1003;
```

##### 组合聚集函数

```sql
SELECT COUNT(*) AS num_items 
	MIN(prod_price) AS price_min,
	MAX(prod_price) AS price_max,
	AVG(prod_price) AS price_avg
FROM products;
```

#### 12 分组数据(gruop by、having)

```sql
SELECT COUNT(*) AS num_prods
FROM products
WHERE vend_id = 1003;

/* 创建分组*/
SELECT vend_id,COUNT(*) AS num_prods
FROM products
GROUP BY vend_id;
/*
上面的SELECT语句指定了两个列，vend id包含产品供应商的ID,
num_prods为计算字段(用COUNT(*)函数建立)。
GROUP BY子句指示MySQL按vend id排序并分组数据。
这导致对每个vend id而不是整个表计算num prods一次。
从输出中可以看到，供应商1001有3个产品,供应商1092有2个产品，供应商1093有7个产品，而供应商1095有2个产品。
*/

/*
GROUP BY子句可以包含任意数目的列。这使得能对分组进行嵌套为数据分组提供更细致的控制。

如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算(所以不能从个别的列取回数据)。

GROUP BY子句中列出的每个列都必须是检索列或有效的表达式(但不能是聚集函数)。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。

如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。

GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前
*/



/*过滤分组*/
/*
HAVING非常类似于WHERE。
事实上，目前为止所学过的所有类型的WHERE子句都可以用HAVING来替代。
唯一的差别是WHERE过滤行，而HAVING   过滤分组。
*/
SELECT cust_id,COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;
/*WHERE 在这里不起作用，因为这里是过滤分组聚值， 而不是行值*/


SELECT vend_id, COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >= 2;
/*
这条语句中，第一行是使用了聚集函数的基本SELECT，它与前面的例子很相像。
WHERE子句过滤所有prod price至少为10的行。
然后按vend id分组数据，HAVING子句过滤计数为2或2以上的分组。
如果没有WHERE子句，将会多检索出两行(供应商1992，销售的所有产品价格都在10以下:供应商1091，销售3个产品，但只有一个产品的价格大于等于10)
*/


/*分组和排序*/
SELECT order_num,SUM(quantity*item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity*item_price) >= 50;

SELECT order_num,SUM(quantity*item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity*item_price) >= 50
ORDER BY ordertotal;

```

![image-20230522214301607](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230522214301607.png)

#### 13 使用子查询

```sql
SELECT order_num
FROM orderitems
WHERE prod_id = 'TNT2';  // 20005 、20007

SELECT cust_id
FROM orders
WHERE order_num IN(20005, 20007); //10001、10004

//组合
SELECT cust_id
FROM orders
WHERE order_num IN (SELECT order_num
					FROM orderitems
					WHERE prod_id = 'TNT2';)


SELECT cust_name,cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
					FROM orders
					WHERE order_num IN (SELECT order_num
										FROM orderitems
										WHERE prod_id = 'TNT2';))
										


/*作为计算字段使用子查询*/

SELECT COUNT(*) AS orders
FROM orders
WHERE cust_id = 10001;

SELECT cust_name,
	   cust_state,
	   (SELECT COUNT(*)
       FROM orders 
       WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
/* 必须主义限制有歧义性的列名*/

```

#### 14 联结表（INNER JOIN、JOIN）

```sql
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id  	/*完全限定列名*/
ORDER BY  vend_name, prod_name;
/*在联结两个表时，你实际上做的是将第一个表中的每一行与第二个表中的每一行配对。WHERE子句作为过滤条件，它只包含那些匹配给定条件 (这里是联结条件) 的行。*/
/*
笛卡儿积 (cartesian product)由没有联结条件的表关系返回的结果为笛卡儿积。检索出的行的数目将是第一个表中的行数乘以第二个表中的行数。
*/

/*内部联结*/

//同上
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.end_id = products.vend_id;
/* 首选 INNER JOIN*/

/*联结多个表*/
SELECT prod_name,vend_name, prod_price, quantity
FROM orderitems,products,vendors 
WHERE products.vend_id = vendors.vend_id 
	AND orderitems.prod_id = products.prod id
	AND order_num =20005;
/*
此例子显示编号为20005的订单中的物品。
订单物品存储在orderitems表中
每个产品按其产品ID存储，它引用products表中的产品。
这些产品通过供应商ID联结到vendors表中相应的供应商，供应商ID存储在每个产品的记录中。
这里的FROM子句列出了3个表，而WHERE子句定义了这两个联结条件，
而第三个联结条件用来过滤出订单20305中的物品。

MySOL在运行时关联指定的每个表以处理联结这种处理可能是非常耗费资源的，因此应该仔细，不要联结不必要的表。
联结的表越多，性能下降越厉害
*/

SELECT cust_name,cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
					FROM orders
					WHERE order_num IN (SELECT order_num
										FROM orderitems
										WHERE prod_id = 'TNT2';))
同

SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE costomers.cust_id = orders.cust_id
	AND orders.order_num = orderitems.order_num
	AND prod_id = 'TNT2';
```

#### 15 创建高级联结

```sql
SELECT Concat(RTrim(vend_name), ' (', RTrim(vend_country), ')') AS vend_title
FROM vendors
ORDER BY vend_name;

SELECT cust_name, cust_contact
FROM costomers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
	AND oi.order_num = o.order_num
	AND prod_id = 'TNT2';
	
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id
                FROM products
                WHERE prod_id = 'DTNTR');   //子查询

SELECT prod_id, prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
	AND p2.prod_id = 'DTNTR'
/*
此查询中需要的两个表实际上是相同的表，因此products表在FROM子句中出现了两次。
虽然这是完全合法的，但对products的引用具有二义性，因为MySOL不知道你引用的是products表中的哪个实例。
为解决此问题，使用了表别名。products的第一次出现为别名p1;第二次出现为别名p2。
现在可以将这些别名用作表名。例如，SELECT语句使用p1前缀明确地给出所需列的全名。
如果不这样，MySQL将返回错误，因为分别存在两个名为prod_id、prod_name的列。
MySQL不知道想要的是哪一个列(即使它们事实上是同一个列)。WHERE(通过匹配p1中的vend_id和p2中的vend_id) 首先联结两个表，然后按第二个表中的prod_id过滤数据，返回所需的数据
*/

/*自然联结*/

SELECT c.*, o.order_num, o.order_date,oi.prod_id,oi.quantity,oi.item_price
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
	AND oi,order_num = o,order_num
	AND prod_id='FB';
	
/*外部联结  OUTER JOIN*/
/*联结包含了哪些在相关表中没有关联行的行，这种类型的连接称为外部联结*/
/*
如下例：
 对每个客户下了多少订单进行计数，包括那些至今尚未下订单的客户:
 列出所有产品以及订购数量，包括没有人订购的产品; 
 计算平均销售规模，包括那些至今尚未下订单的客户.
*/
SELECT customers.cust_id, orders.order_num
FROM customers INNER JOIN orders
ON costomers.cust_id = orders.cust_id; //内部联结

/*检索所有客户，包括那些没有订单的客户*/
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id;

/*
在使用OUTER JOIN语法时，必须使用RIGHT或LEFT关键字指定包括其所有行的表 (RIGHT指出的是OUTER JOIN右边的表，而LEFT指出的是OUTER JOIN左边的表)。
上面的例子使用LEFT OUTER JOIN从FROM子句的左边表(customers表) 中选择所有行。
为了从右边的表中选择所有行，应该使用RIGHT OUTER JOIN，
*/
 SELECT customers.cust_id, orders.order_num
 FROM customers RIGHT OUTER JOIN orders
 ON orders.cust_id = customers.cust_id;
 
 /* 使用带有聚集函数的联结*/
 
 SELECT customers.cust_name,
 		customers.cust_id,
 		COUNT(orders.order_num) AS num_ord
FROM cutomers INNER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
/*
此SELECT语句使用INNER JOIN将customers和orders表互相关联GROUP BY子句按客户分组数据，因此，函数调用COUNT(orders.order num)对每个客户的订单计数，将它作为num_ord返回。
*/

/*
对每个客户下了多少订单进行计数，包括那些至今尚未下订单的客户:
*/
SELECT customers.cust_name,
		customers.cust_id,
		COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY costomers.cust_id;
```

#### 16 组合查询（UNION）

- 多数SOL查询都只包含从一个或多个表中返回数据的单条SELECT语句。
- MySQL也允许执行多个查询(多条SELECT语句)，并将结果作为单个查询结果集返回。这些组合查询通常称为并 (union ) 或复合查询
  - 在单个查询中从不同的表返回**类似结构的数据**
  - 对**单个表执行多个查询**，按单个查询返回数据

```sql
/*创建组合查询*/
SELECT vend_id,prod_id,prod_price
FROM products
WHERE prod_price <= 5;

SELECT vend_id,prod_id,prod_price
FROM products
WHERE vend_id IN (1001,1002);

/*组合*/
SELECT vend_id,prod_id,prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id,prod_id,prod_price
FROM products
WHERE vend_id IN (1001,1002);

/*
UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔(因此，如果组合4条SELECT语句，将要使用3个UNION关键字)。
UNION中的每个查询必须包含相同的列、表达式或聚集函数(不过各个列不需要以相同的次序列出）
列数据类型必须兼容：类型不必完全相同，但还是DBMS可以隐含的转换的类型

UNION 从查询结果集中自动去除了重复的行，要返回所有的行可以用 UNION ALL
*/

SELECT vend_id,prod_id,prod_price
FROM products
WHERE prod_price <= 5
UNION ALL
SELECT vend_id,prod_id,prod_price
FROM products
WHERE vend_id IN (1001,1002);


/*对组合查询进行排序*/
SELECT vend_id,prod_id,prod_price
FROM products
WHERE prod_price <= 5
UNION ALL
SELECT vend_id,prod_id,prod_price
FROM products
WHERE vend_id IN (1001,1002)
ORDER BY vend_id, prod_price;
```

#### 17 全文本搜索（Match 和Against）

-  MySQL 支持集中基本的数据流引擎
- 并非所有引擎都支持全文本搜索
- 最常使用的引擎为MyISAM 和 InnoDB
  - MyISAM 支持全文本搜索
  - InnoDB 不支持
- 通配符和正则存在的限制
  - 性能低
    - 通配符和正则通常要求MySQL 尝试匹配表中所有行（极少使用表索引），这些搜索非常耗时
    - 很难明确的控制匹配什么不匹配什么
    - 都不能提供一种只能话的选择结果
      - 不能提供最佳的匹配来排列
- 使用全文本搜索来解决问题

```sql
/* 一般在创建表时启用全文本搜索*/
CREATE TABLE productnotes
(
	note_id		int			NOT NULL 	AUTO_INCREMENT,
    prod_id		char(10)	NOT NULL,
    note_date	datetime	NOT NULL,
    note_text	text		NULL,
    PRIMARY KEY(note_id),
    FULLTEXT(note_text),
)ENGINE=MyISAM;

/*
这些列中有一个名为note text的列，为了进行全文本搜索，MySQL根据子句FULLTEXT(note text)的指示对它进行索引。这里的FULLTEXT索引单个列，如果需要也可以指定多个列。

不要在导入数据时使用FULLTEXT 更新索引要花时间，虽然不是很多，但毕竟要花时间。
如果正在导入数据到一个新表此时不应该启用FULLTEXT索引。应该首先导入所有数据，然后再修改表，定义FULLTEXT。
*/

SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');

/*
此SELECT语句检索单个列note text。
由于WHERE子句，一个全文本搜索被执行。
Match(note text)指示MySQL针对指定的列进行搜索，
Against('rabbit )指定词rabbit作为搜索文本。由于有两行包含词rabbit，这两个行被返回。

传递给 Match() 的值 必须 与FULLTEXT()定义中的相同。如果指定多个列，则必须列出它们 (而且次序正确 )

除非使用BINARY方式，否则全文本搜索不区分大小写
*/

//同
SELECT note_text
FROM productnotes
WHERE note_text LIKE '%rabbit%';

/*两个行都包含词rabbit，但包含词rabbit作为第3个词的行的等级比作为第20个词的行高。这很重要。全文本搜索的一个重要部分就是对结果排序。具有较高等级的行先返回*/

SELECT note_text,
	Match(note_text) Against('rabbit') AS rank
FROM productnotes;

/*
这里，在SELECT而不是WHERE子句中使用Match()和Against()。这
使所有行都被返回(因为没有WHERE子句)。
Match()和Against()用来建立一个计算列(别名为rank)，此列包含全文本搜索计算出的等级值。

文本中词靠前的行的等级值比词口后的行的等级值高


全文本搜索提供了简单LIKE 搜索不能提供的功能
而且由于书库是索引的，全文本搜索还相当快。
*/

/*  使用查询扩展 

利用查询扩展，能找出可能相关的结果，即使它们并不精确包含所查找的词。
*/ 

SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils');  //没有使用查询扩展， 只返回包含'anvils'的一行


SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils' WITH QUERY EXPANSION); //使用查询扩展
/*
这次返回了7行。第一行包含词anvils，因此等级最高。
第二行与anvils无关，但因为它包含第一行中的两个词(customer和recommend)，所以也被检索出来。
第3行也包含这两个相同的词，但它们在文本中的位置更靠后且分开得更远，因此也包含这一行，但等级为第三。
第三行确实也没有涉及anvils (按它们的产品名)。

查询扩展极大地增加了返回的行数，但这样也增加了实际中并不想要的数目
*/

/*  布尔文本搜索
	-要匹配的词
	-要排斥的词
	-排列提示
	-表达式分组
*/

//搜索包含词heavy的所有行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy' IN BOOLEAN MODE);


//包含heavy 的行 但是不包含任意以rope开始的词的行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE);

//搜索匹配包含词rabbit和bait的行
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+rabbit +bait' IN BOOLEAN MODE);

//没有指定操作符，这个搜索匹配包含rabbit和bait中的至少一个词的行。
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);

//这个搜索匹配短语rabbit bait而不是匹配两个词rabbit和bait.
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('"rabbit bait"' IN BOOLEAN MODE);

//匹配rabbit和carrot，增加前者的等级，降低后者的等级
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('>rabbit <carrot' IN BOOLEAN MODE);

//这个搜索匹配词safe和combination，降低后者的等级
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+safe +(combination)' IN BOOLEAN MODE);

/*
 在索引全文本数据时，短词被忽略且从索引中排除。短词定义为那些具有3个或3个以下字符的词(如果需要,这个数目可以更改 )。
 
 MySQL带有一个内建的非用词 (stopword) 列表，这些词在索引全文本数据时总是被忽略。
 
 如果需要，可以覆盖这个列表 (请参阅MySOL文档以了解如何完成此工作)。许多词出现的频率很高，搜索它们没有用处 (返回太多的结果).因此，MySQL规定了一条50%规则，如果一个词出现在50%以上的行中，则将它作为一个非用词忽略。50%规则不用于IN BOOLEANMODE。
 
如果表中的行数少于3行，则全文本搜索不返回结果(因为每个词或者不出现，或者至少出现在50%的行中)。

忽略词中的单引号。例如，don't索引为dont不具有词分隔符 (包括日语和汉语) 的语言不能恰当地返 回全文本搜索结果。

 如前所述，仅在MyISAM数据库引擎中支持全文本搜索
*/
```

![image-20230531112941128](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230531112941128.png)

#### 18 插入数据（Insert into）

```sql
/*
	-插入完整的行
	-插入行的一部分
	-插入多行
	-插入某些查询的结果
*/

//插入完整的行
INSERT INTO customers
VALUES(NULL,
'Pep E. LaPew',
'100 Main Street',
'Los Angeles',
'CA',
'190046',
'USA',
NULL,
NULL);
  
//同上  
//编写INSERT语句的更安全 (不过更烦琐) 的方法如下
INSERT INTO customers(cust_name,
                     cust_address,
                     cust_city,
                     cust_zip,
                     cust_country,
                     cust_contact,
                     cust_email)
            VALUES('Pep E. LaPew',
                    '100 Main Street',
                    'Los Angeles',
                    'CA',
                    '190046',
                    'USA',
                    NULL,
                    NULL);
                    
// 下面的INSERT语句填充所有列 (与前面的一样)，但以一种不同的次序填充。因为给出了列名，所以插入结果仍然正确:

INSERT INTO customers(cust_name,
					cust_contact,
					cust_emai1,
                    cust_address,
                    cust_city,
                    cust_state,
                    cust_zip,
                    cust_country)
            VALUES('Pep E.LaPew',
                    NULL,
                    NULL,
                    '100 Main Street',
                    'Los Angeles',
                    'CA',
                    90046,
                    'USA');
                    
//插入多个行
//只要每条INSERT语句中的列名 (和次序) 相同，
INSERT INTO customers(cust_name,
                cust_address,
                cust_city,
                cust_state,
                cust_zip,
                cust_country)
            VALUES('Pep E.LaPew',
                '100 Main Street',
                'Los Angeles',
                'CA',
                90046,
                'USA');
INSERT INTO customers(cust_name,
                    cust_address,
                    cust_city,
                    cust_state,
                    cust_zip,
                    cust_country)
            VALUES('M.Martian',
                   '42 Galaxy way',
                   'New York',
                   'NY',
                   11213,
                   'USA');
                   
INSERT INTO customers(cust_name,
                cust_address,
                cust_city,
                cust_state,
                cust_zip,
                cust_country)
            VALUES(
                    'Pep E.LaPew',
                    '100 Main Street',
                    'Los Angeles',
                    'CA',
                    90046,
                    'USA'
            	),
				(
                    'M.Martian',
                    '42 Galaxy way',
                    'New York',
                    'NY',
                    11213,
                    'USA'
                );
                
//插入检索出的数据
INSERT INTO customers(cust_id.
                     cust_contact,
                     cust_email,
                     cust_name,
                     cust_address,
                     cust_city,
                     cust_state,
                     cust_zip,
                     cust_country)
           		SELECT cust_id,
           			 cust_contact,
                     cust_email,
                     cust_name,
                     cust_address,
                     cust_city,
                     cust_state,
                     cust_zip,
                     cust_country
               FROM custnew;
```

#### 19 更新和删除数据（update、delete）

```sql
/*
	-更新表中特定行
	-更新表中所有行
*/
UPDATE customers
SET cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;		//如果不用where 指定，便会更新表中所有行的email

UPDATE customers
SET cust_name = 'The Fudds',
cust_email = 'elmer@fudd.com'	
WHERE cust_id = 10005;	

UPDATE customers
SET cust_email = NULL
WHERE cust_id = 10005; //NULL 用来删除cust_email列中的值

/*删除一行*/
DELETE FROM customers
WHERE cust_id = 10006;

/*DELETE不需要列名或通配符。DELETE删除整行而不是删除列。为了删除指定的列，请使用UPDATE语句。

删除表的内容而不是表 DELETE语句从表中删除行，甚至是删除表中所有行。但是，DELETE不删除表本身。

更快的删除 truncate table 
		实际上是删除原来的表并重新创建一个表，而不是逐行删除表中的数据
		
更新删除请谨慎，先备份在操作
*/
```

#### 20 创建和操作表(create 、alter、drop)

```sql
CREATE TABLE customers
(
	cust_id			int			NOT NULL  AUTO_INCREMENT,
    cust_name		char(50)	NOT NULL,
    cust_address	char(50)	NULL,
    cust_city		char(50)	NULL,
    cust_state		char(5)		NULL,
    cust_zip		char(50)	NULL,
    cust_country	char(50)	NULL,
    cust_contact	char(50)	NULL,
    cust_email		char(255)	NULL,
    PRIMARY	KEY	(cust_id)
)ENGINE=InnoDB;
/*
NULL值就是没有值或缺值。允许NULL值的列也允许在插入行时不给出该列的值。不允许NULL值的列不接受该列没有值的行，换句话说，在插入或更新行时，该列必须有值。
*/
CREATE TABLE orders
(

    order_num 		int 		NOT NULL AUTO INCREMENT,
    order_date 		datetime    NOT NULL,
    cust_id			int			NOT NULL,
    PRIMARY KEY (order_num)
)ENGINE=InnoDB;

CREATE TABLE vendors
(
    vend_id				int			NOT NULL AUTO INCREMENT,
    vend_name			char(50)	NOT NULL,
    vend_address		char(50)	NULL,
    vend_city			char(50)	NULL,
    vend_state			char(5)		NULL,
    vend_zip			char(10)	NULL,
    cust_country		char(50)	NULL,
    PRIMARY KEY (vend_id)
)ENGINE=InnoDB;

//主键值必须唯一

CREATE TABLE orderitems
(
    order_num		int 		NOT NULL,
    order_item    	int 		NOT NULL,
    prod_id			char(10)	NOT NULL,
    quantity 		int			NOT NULL,
    item_price 		decimal(8,2) NOT NULL,
    PRIMARY KEY (order_num，order_item)
)ENGINE=InnoDB;

/*
每个表只允许一个AUTO INCREMENT列，而且它必须被索引(如，通过使它成为主键)。
*/

//指定默认值
CREATE TABLE orderitems
(
    order_num		int 		NOT NULL,
    order_item    	int 		NOT NULL,
    prod_id			char(10)	NOT NULL,
    quantity 		int			NOT NULL	DEFAULT 1,
    item_price 		decimal(8,2) NOT NULL,
    PRIMARY KEY (order_num，order_item)
)ENGINE=InnoDB;
//在此例子中，给该列的描述添加文本DEFAULT 1指示MySQL，在未给出数量的情况下使用数量1

/*引擎类型

	-MySOL与其他DBMS不一样，它具有多种引擎。它打包多个引擎，这些引擎都隐藏在MySOL服务器内，全都能执行CREATE TABLE和SELECT等命令


	-InnoDB是一个可靠的事务处理引擎，它不支持全文本搜索;
	-MEMORY在功能等同于MyISAM，但由于数据存储在内存(不是磁盘)中，速度很快(特别适合于临时表)
	-MyISAM是一个性能极高的引擎,它支持全文本搜索,但不支持事务处理。
	
	引擎类型可以混用。
	除productnotes表使用MyISAM外，本书中的样例表都使用InnoDB。
	原因是作者希望支持事务处理(因此，使用InnoDB),但也需要在productnotes中支持全文本搜索(因此，使用MyISAM)。
*/

// 添加一个列
ALTER TABLE vendors
ADD vend_phone CHAR(20);

//删除列
ALTER TABLE vendors
DROP COLUMN	vend_phone;

/* alter table 最常见的用途是定义外键*/

ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);

ALTER TABLE orderitems
ADD CONSTRAINT fk_orderitems_products
FOREIGN KEY (pord_id) REFERENCES products (prod_id);

ALTER TABLE orders
ADD CONSTRAINT fk_orders_customers
FOREIGN KEY (cust_id) REFERENCES customers (cust_id);

ALTER TABLE products
ADD CONSTRAINT fk_products_vendors
FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);

//删除表
DROP TABLE customers2;

//重命名表
RENAME TABLE customers2 T0 customers;

RENAME TABLE backup_customers TO customers,
            backup_vendors TO vendors,
            backup_products TO products;
```

#### 21 使用视图

```sql
/*
	视图不包含表中应该有的任何列和数据，它包含的是一个SQL查询
	
	为什么使用视图
		-重用SQL语句
		-简化复杂的SQL 操作
		-使用表的组成部分而不是整个表
		-保护数据
		-更改数据格式和表示
		
	下面是关于视图创建和使用的一些最常见的规则和限制。
 		-与表一样，视图必须唯一命名 (不能给视图取与别的视图或表相同的名字)。
 		-对于可以创建的视图数目没有限制
		-为了创建视图，必须具有足够的访问权限。这些限制通常由数据库管理人员授予。
		-视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造个视图。
		-ORDER BY可以用在视图中，但如果从该视图检索数据SELECT中也含有ORDER BY，那么该视图中的ORDER BY将被覆盖。
		- 视图不能索引，也不能有关联的触发器或默认值。
		- 视图可以和表一起使用。例如，编写一条联结表和视图的SELECT语句。
		
	使用视图
		-创建  CREATE VIEW
		- SHOW CREATE VIEW viewname
		- DORP删除视图
		- 更新视图 
				-可以先DROP 再 CREATE
				-CREATE or REPLACE VIEW
*/

SELECT cust_name, cust_contact
FROM customers,orders,orderitems
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = orders.order_num
	AND prod_id='TNT2';
	
//转化为视图
CREATE VIEW productcustomers AS
			SELECT cust_name, cust_contact,prod_id
			FROM customers,orders, orderitems
			WHERE customers.cust_id = orders.cust_id
			AND orderitems.order_num = orders.order_num;			
//使用视图查询
SELECT cust_name,cust_contact
FROM productcustomers
WHERE prod_id='TNT2';

/*视图另一种常见用途是重新格式化检索出的数据*/
SELECT Concat(RTrim(vend_name),'(',Trim(vend_country),')')
		AS vend_title
FROM vendors
ORDER BY vend_name;

CREATE VIEW vendorlocations AS
		SELECT Concat(RTrim(vend_name), '(', RTrim(vend_country), ')')
        AS vend_title
        FROM vendors
        ORDER BY vend_name;
SELECT * FROM vendorlocations;

/*用视图过滤掉不想要的数据*/
CREATE VIEW customeremaillist AS
SELECT cust_id，cust_name， cust_emai
FROM customers
WHERE cust_email IS NOT NULL;

SELECT * FROM costomeremaillist;

/*使用视图与计算字段*/

SELECT prod_id,
		quantity,
		item_price,
		quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;

//转换为视图
CREATE VIEW orderitemsexpanded AS
		SELECT order_num,
				prod_id,
                quantity,
                item_price,
                quantity*item_price AS expanded_price
		FROM orderitems;
SELECT * FROM orderitemsexpanded WHERE order_num = 20005;

/*更新视图
    通常，视图是可更新的(即，可以对它们使用INSERT、UPDATE和DELETE)。更新一个视图将更新其基表(可以回忆一下，视图本身没有数据)。如果你对视图增加或删除行，实际上是对其基表增加或删除行。
    但是，并非所有视图都是可更新的。基本上可以说，如果MySOL不能正确地确定被更新的基数据，则不允许更新(包括插入和删除)。这实际上意味着，如果视图定义中有以下操作，则不能进行视图的更新
        - 分组 (使用GROUP BY和HAVING):
        - 联结:
        - 子查询:
        - 并:
        - 聚集函数 (Min()、Count()、Sum()等):
        - DISTINCT
        - 导出(计算)列

视图为虚拟的表。 他们包含的不是数据而是根据需要检索数据的查询结果。
*/
```

#### 22 使用存储过程

```sql
/*
	为了在这些场景下使用：
	
	为了处理订单，需要核对以保证库存中有相应的物品。
	如果库存有物品，这些物品需要预定以便不将它们再卖给别的人，并且要减少可用的物品数量以反映正确的库存量。
	库存中没有的物品需要订购，这需要与供应商进行某种交互关于哪些物品入库(并且可以立即发货) 和哪些物品退订，需要通知相应的客户
	
	存储过程，就是为以后的使用而保存的一条或多条MySQL 语句的集合
	
	为什么要使用存储过程
		- 通过把处理封装在容易使用的单元中，简化复杂操作
		- 由于不要求反复建立一系列处理步骤，这保证数据的完整性
		- 简化对变动的管理
		- 提高性能：使用存储过程比使用单独的SQL要快
		
		简单、安全、高性能
		
	缺点
		-存储过程的编写比基本SQL语句要复杂
*/

/*使用存储过程*/
//执行名为productpricing的存储过程，它计算并返回产品的最低、最高和平均价格。
CALL productpricing(@pricelow,
                   @pricehigh,
                   @priceaverage);
                   
//创建存储过程
CREATE PROCEDURE productpricing()
BEGIN
	SELECT AVg(prod_price) AS priceaverage
	FROM products;
END;

/*
mysq1命令行客户机的分隔符实用程序，应该仔细阅读此说明。
	默认的MySQL语句分隔符为 ;
	mysql命令行实用程序也使用;作为语句分隔符。
	如果命令行实用程序要解释存储过程自身内的;字符，则它们最终不会成为存储过程的成分，这会使存储过程中的SQL出现句法错误。
	解决办法是临时更改命令行实用程序的语句分隔符，如下所示:
*/
DELIMITER //

CREATE PROCEDURE productpricing()
BEGIN
	SELECT AVg(prod_price) AS priceaverage
	FROM products;
END//

DELIMITER;

/*使用存储过程*/
CALL productpricing(); //有()

/*删除存储过程*/
DROP PROCEDURE productpricing; //没有()

/*使用参数*/
CREATE PROCEDUREproductpricing(
	OUT Pl DECIMAL(8,2),
	OUT ph DECIMAL(8,2),
	OUT pa DECIMAL(8,2)
)
BEGIN
	SELECT Min(prod_price)
	INTO pl
	FROM products;
	SELECT Max(prod_price)
	INTO ph
	FROM products;
	SELECT Avg(prod_price)
	INTO pa
	FROM products;
END;
/* pl 存储产品的最低价格
   ph 存储产品的最高价格
   pa 存储产品的平均价格
*/

/*使用*/
CALL productpricing(@pricelow,
                   @pricehigh,
                   @priceaverage);
                   
SELECT @priceaverage;
SELECT @pricelow, @pricehigh, @priceaverage;


CREATE PROCEDURE ordertotal(
    IN onumber INT,		//输入
    OUT ototal DECIMAL(8,2) 	//输出
    )
BEGIN
    SELECT Sum(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO ototal;
END;

CALL ordertota1(20005,@total);
SELECT @total;

CALL ordertota1(20009,@total);
SELECT @total;

/* 建立智能存储过程
	
	考虑这个场景。你需要获得与以前一样的订单合计，但需要对合计增加营业税，不过只针对某些顾客。那么，你需要做下面几件事情:
 	-获得合计 (与以前一样):
 	-把营业税有条件地添加到合计;
 	-返回合计 (带或不带税)。
*/

-- Name： ordertotal   
-- Parameters: onumber = order number
--				taxable = 0 if not taxable, 1 if taxable
--				ototal	= order total variable

CREATE PROCEDURE ordertotal(
	IN onumber INT,
    IN taxable BOOLEAN,
    OUT ototal DECIMAL(8,2)  // 十进制
) COMMENT 'Obtain order total, optionally adding tax'

BEGIN
	-- 声明变量
	DECLARE total DECIMAL(8,2);
	-- 声明税率
	DECLARE taxrate INT DEFAULT 6;
	
	-- 获得合计
	SELECT Sum(item_price*quantity)
    FROM orderitems
    WHERE order_num = onumber
    INTO total;
    
    -- 如果加税
    IF taxable THEN
    	-- 加
    	SELECT total + (total/100*taxrate) INTO total;
    END IF;
    
    -- 最后保存变量
    SELECT total INTO ototal;
    
END;
	
CALL ordertotal(20005,0,@total);S
ELECT @tota1;

CALL ordertotal(20005,1, @total);
SELECT @total;  /* MySQL 中的变量前都加 @ */

/* 检查存储过程*/
SHOW CREATE PROCEDURE ordertotal;

SHOW PROCEDURE STATUS;
```

#### 23 使用游标

```sql
/*
游标是一个存储在MySQL 服务器上的数据库查询， 在存储了游标之后，应用车讯可以根据需要滚动或浏览其中的数据

使用游标几个步骤:
	- 使用前必须声明（定义）它
	- 一旦声明，必须打开游标以供使用
	- 对于填有数据的游标，更具需要检索各行
	- 在结束游标使用是，必须关闭游标
*/


/* 创建游标 */
CREATE PROCEDURE processorders()
BEGIN
	DECLARE ordernumbers CURSOR
	FOR 
	SELECT order_num FROM orders;
END; //DECLARE语句用来定义和命名游标，这里为ordernumbers。

/*打开游标*/
OPEN ordernumbers;

/*关闭游标*/
CLOSE ordernumbers;

//修改版本
//这个存储过程声明、打开和关闭一个游标。但对检索出的数据什么也没做。
CREATE PROCEDURE processorders()
BEGIN
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	-- Open the corsor
	OPEN ordernumbers;
	-- close the corsor
	CLOSE ordernumbers;
END;

//使用游标FETCH
/*
其中FETCH用来检索当前行的order_num列(将自动从第一行开始)到一个名为o的局部声明的变量中。
*/
CREATE PROCEDURE processorders()
BEGIN
	-- Declare local variables
	DECLARE o INT;
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	-- Open the corsor
	OPEN ordernumbers;
	-- Get order number
    FETCH ordernumbers INTO o;
	-- close the corsor
	CLOSE ordernumbers;
END;


/*
与前一个例子一样，这个例子使用FETCH检索当前order_num到声明的名为o的变量中。
但与前一个例子不一样的是，这个例子中的FETCH是在REPEAT内，因此它反复执行直到done为真 (由UNTIL done END REPEAT;规定)。
为使它起作用，用一个DEFAULT (假，不结束)定义变量done。那么，done怎样才能在结束时被设置为真呢?
呢? 答案是用以下语句:
DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
这条语句定义了一个CONTINUE HANDLER，它是在条件出现时被执行的代码。这里,它指出当SOLSTATE 02000 出现时,SET done=1。
SQLSTATE '02000'是一个未找到条件，当REPEAT由于没有更多的行供循环而不能继续时，出现这个条件。

DECLARE语句的次序 
-定义的局部变量必须在定义任意游标或句柄之前定义，
-而句柄必须在游标之后定义。
-不遵守此顺序将产生错误消息。
*/
CREATE PROCEDURE processorders()
BEGIN
	-- Declare local variables
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num FROM orders;
	
	-- Declare continue handler
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
	
	-- Open the corsor
	OPEN ordernumbers;
	
	-- Loop through all rows
	REPEAT
	
        -- Get order number
        FETCH ordernumbers INTO o;
        
    -- End of loop
    UNTIL done END REPEAT;
    
	-- close the corsor
	CLOSE ordernumbers;
END;


/*
	在这个例子中增加了一个名为t的变量（存储每个订单的合计）
	此存储过程还在运行中创建了一个新表 名为 ordertotals
*/
CREATE PROCEDURE processorders()
BEGIN
	-- Declare local variables
	DECLARE done BOOLEAN DEFAULT 0;
	DECLARE o INT;
	DECLARE t DECIMAX(8,2);
	
	-- Declare the cursor
	DECLARE ordernumbers CURSOR
	FOR
	SELECT order_num from orders;
	
	-- Declare continue handler
	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
	
	-- Create a table to store the results
	CREATE TABLE IF NOT EXISTS ordertotals
	(order_num INT, total DECIMAX(8,2));
	
	-- open the cursor
	OPEN ordernumbers;
	
	-- Loop through all rows
	REPEAT
		-- Get order number
		FETCH ordernumbers INTO o;
	
		-- Get the total for this order
		CALL ordertotal(o, l, t);
		
		-- Inert order and total into ordertotals;
		INSERT INTO ordertotals(order_num, total)
		VALUES(o, t);
	-- End of loop
	UNTIL done END REPEAT
	
	-- Close the cursor
	CLOSE ordernumbers;
END;
```

#### 24 使用触发器

```sql
/*某些语句在事件发生时自动执行 例如
		- 每增加一个顾客到数据库表时， 都检查其电话号码格式是否正确
		- 每当订购一个产品是，都从库存数量中减去订购的数量
		- 无论何时删除一行，艘在某个存档表中保留一个副本
		
	触发器是MySQL想要以下任意语句而自动执行的一条MySQL语句
		-DELETE
		-INSERT
		-UPDATE
	其他MySQL不支持触发器
	
	创建触发器时
		- 唯一的触发器名
		- 触发器关联的表
		- 触发器应该响应的活动
		- 触发器何时执行（处理之前或之后）
*/

CREATE TRIGGER newproduct AFTER INSERT ON products
FOR EACH ROW SELECT 'product added';
/*
CREATE TRIGGER用来创建名为newproduct的新触发器。
触发器可在一个操作发生之前或之后执行，这里给出了AFTER INSERT所以此触发器将在INSERT语句成功执行后执行。
这个触发器还指定FOR EACH ROW，因此代码对每个插入行执行。
在这个例子中，文本Product added将对每个插入的行显示一次。


只有表才支持触发器，视图不支持（临时表也不支持）

每个表每个事件每次只允许一个触发器。因此，每个表最多支持6个触发器(每条INSERT、UPDATE和DELETE的之前和之后)。
单一触发器不能与多个事件或多个表关联，所以，如果你需要一个对INSERT和UPDATE操作执行的触发器，则应该定义两个触发器

如果BEFORE触发器失败，则MySQL将不执行请求的操作。此外，如果BEFORE触发器或语句本身失败，MySQL将不执行AFTER触发器 (如果有的话 )。
*/

/*删除触发器*/
DROP TRIGGER newproduct;
/* 触发器不能更新或覆盖吗，为了修改一个触发器，必须先删除然后在创建*/

/*使用触发器
	INSERT 触发器
		- 可引用一个名为NEW 的虚拟表，访问被插入的行
		- BEFORE INSERT 触发器中， NEW中的值也可以被更新
		- 对于 AUTO_INCREMENT 列， NEW在INSERT执行前包含0，执行后包含新的自动生成的值
		通常，将BEFORE用于数据验证和净化(目的是保证插入表中的数据确实是需要的数据）。本提示也适用于UPDATE触发器
		
	DELETE 触发器
		- 可以引用一个名为OLD的虚拟表，访问被删除的行
		- OLD的值全都是只读的，不能更新
		
	UPDATE 触发器
		- 可以引用一个名为OLD的虚拟表访问之前的值，引用NEW虚拟表访问更新的值
		- BEFORE INSERT 触发器中， NEW中的值可能也被更新
		- OLD中的值全都是只读的，不能更新
*/


//INSERT 触发器
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW SELECT NEW.order_num;

INSERT INTO orders(order_date, cust_id)
VALUES(Now(),10001);

/*
此代码创建一个名为neworder的触发器，它按照AFTER INSERTON orders执行。
在插入一个新订单到orders表时，MySQL生成一个新订单号并保存到order_num中。
触发器从NEW.order_num取得这个值并返回它。
此触发器必须按照AFTER INSERT执行，因为在BEFORE INSERT语句执行之前，新order num还没有生成。
对于orders的每次插入使用这个触发器将总是返回新的订单号。
*/

//DELETE 触发器
CREATE TRIGGER deleteorder BEFORE DELETE ON orders
FOR EACH ROW
BEGIN
	INSERT INTO archive_orders(order_num, order_date, cust_id)
	VALUES(OLD.order_num, OLD.order_date, OLD.cust_id);
END; 
/*
在任意订单被删除前将执行此触发器。
它使用一条INSERT语句将OLD中的值(要被删除的订单) 保存到一个名为archive_orders的存档表中(为实际使用这个例子，你需要用与orders相同的列创建一个名为archive orders的表)

多语句触发器 正如所见，触发器deleteorder使用BEGIN和END语句标记触发器体。
这在此例子中并不是必需的，不过也没有害处。
使用BEGIN END块的好处是触发器能容纳多条SOI语句 (在BEGIN END块中一条挨着一条 )。
*/


// UPDATE 触发器
CREATE TRIGGER updatevendor BEFORE UPDATE ON vendors
FOR EACH ROW SET NEW.vend_state = Upper(NEW.vend_sate);

/*
	任何数据精华都需要在UPDATE语句之前进行
	这个例子，在每次更新一个行时， NEW.vend_state中的值(将用来更新表行的值)都用upper(NEw.vend_state)替换
*/

```

#### 25 管理事务处理（COMMIT 和 ROLLBACK）

```sql
/*
	事务处理可以用来维护数据库的完整性，他保证呈批的MySQL 操作要么完全执行，要么不执行
	
	事务： 指一组SQL语句
	回退： 指撤销指定SQL语句的过程
	提交： 指将未存储的SQL语句结果写入数据库表
	保留点：指事务处理中设置的临时占位符
	
	管理事务处理的关键在于将SQL语句组分解为逻辑快，并明确规定数据合适应该回退，合适不应该回退
*/


/* 事务的开始*/
START TRANSACTION;

/*回退：使用 ROLLBACK*/
SELECT * FROM ordertotals;
START TRANSACTION;
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK;
SELECT * FROM ordertotals;
/*
首先执行一条SELECT以显示该表不为空。
然后开始-个事务处理，用一条DELETE语句删除ordertotals中的所有行。
另一条SELECT语句验证ordertotals确实为空。
这时用一条ROLLBACK语句回退START TRANSACTION之后的所有语句，
最后一条SELECT语句显示该表不为空。
显然，ROLLBACK只能在一个事务处理内使用(在执行一条START TRANSACTION命令之后)。

哪些语句可以回退?
	- 事务处理用来管理INSERT、UPDATE和DELETE语句。
	- 你不能回退SELECT语句。(这样做也没有什么意义。) 
	- 你不能回退CREATE或DROP操作。事务处理块中可以使用这两条语句，但如果你执行回退，它们不会被撤销。
*/


/*提交：使用COMMIT 

	一般MySQL语句都是直接针对数据库表执行和编写的，这就是所谓的隐含提交
	在事务处理中，提交不会隐含的进行，为进行明确的提交，使用 COMMIT
*/

START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20010;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;
/*
在这个例子中，从系统中完全删除订单20010。
因为涉及更新两个数据库表orders和orderItems，所以使用事务处理块来保证订单不被部分删除。
最后的COMMIT语句仅在不出错时写出更改。
如果第一条DELETE起作用，但第二条失败，则DELETE不会提交(实际上!它是被自动撤销的)。


隐含事务关闭： 当COMMIT 或 ROLLBACK 语句执行后，事务会自动关闭
*/

/* 保留点：使用保留点 
	-简单的事务使用COMMIT 和 ROLLBACK 来写入和撤销整个事务处理
	-复杂的事务处理可能需要部分提交或回退
	
	为了支持部分事务处理，必须能在事务处理块中合适的位置放置占位符
	这样的占位符成为保留点
*/
/* 创建占位符 
	-每个保留点都取标识它的唯一名字
	-保留点越多越好
	- 保留点在事务处理完成后自动释放， 也可以用 RELEASE SAVEPOINT 明确释放
*/
SAVEPOINT delete1;

ROLLBACK To delete1; //回退到保留点

/*更改默认的提交行为
	默认的MySQL 的行为是自动提交所有更改，换句话说，就是所做的更改立即生效
	
	为了指示MySQL 不自动提交更改，使用以下语句
	
	autocommit标志决定是否自动提交更改，不管有没有COMMIT语句。
	设置autocommit为(假) 指示MySQL不自动提交更改(直到autocommit被设置为真为止)。
*/
SET autocommit=0;

```

#### 26 全球化和本地化

```sql
/*
	-数据库表被用来存储和检索数据
	-字符集： 为字母和符号的集合
	-编码： 为某个字符集成员的内部表示
	-校对：为福鼎字符如何比较的指令
*/
//查看所支持的字符集完整列表
SHOW CHARACTER SET;

//查看所支持校对的完整列表
SHOW COLLATION；

//查看所用的字符集和校对
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';

//为了给表指定字符集和校对
CREATE TABLE mytable
(
	columnn1	INT,
    columnn2	VARCHAR(10)
)DEFAULT CHARACTER SET hebrew
COLLATE hebrew_general_ci;

//为列单独指定字符集
CREATE TABLE mytable
coTumnn1INT,coTumnn2VARCHAR(10)coTumn3VARCHAR(10) CHARACTER SET Tatin1 
COLLATE latinl_general_ci
)DEFAULT CHARACTER SET hebrewCOLLATE hebrew_general_ci;

// 用特定的校对排序
SELECT*FROM customers
ORDER BY lastname,firstname COLLATE Tatinl_general_cs;
```

#### 27 安全管理

```sql
/*
MySQL用户账号和信息存储在名为mysg1的MySOL数据库中。
*/
USE mysql;
SELECT user FROM user;

// mysql表中有 root表和user表

//创建用户账户
CREATE USER ben IDENTIFIED BY 'p@$$wOrd';

/*
指定散列口令IDENTIFIED BY指定的口今为纯文本，MySOL将在保存到user表之前对其进行加密。为了作为散列值指定口令，使用IDENTIFIED BY PASSWORD

使用GRANT或INSERT 
	GRANT语句 (稍后介绍) 也可以创建用户账号，但一般来说CREATE USER是最清楚和最简单的句子。
	此外，也可以通过直接插入行到user表来增加用户，不过为安全起见，一般不建议这样做。MySOL用来存储用户账号信息的表 (以及表模式等) 极为重要，对它们的任何毁坏都可能严重地伤害到MySQL服务器
*/

RENAME USER ben To bforta;
DROP USER bforta;

SHOW GRANTS FOR bforta;

GRANT SELECT ON crashcourse.* TO bforta;
/*
此GRANT允许用户在crashcourse.* (crashcourse数据库的所有表)上使用SELECT。通过只授予SELECT访问权限,用户bforta对crashcourse数据库中的所有数据具有只读访问权限。

*/
//取消刚才赋予的权限
REVOKE SELECT ON crashcourse.* FROM bforta;

/*
GRANT和REVOKE可在几个层次上控制访问权限:
	-整个服务器，使用GRANT ALL和REVOKE ALL;
	-整个数据库，使用ON database.*;
	-特定的表，使用ON database.table;
	-特定的列;
 	-特定的存储过程。

*/

//更改口令
SET PASSWORD FOR bforta = Password('n3w p@$$wOrd'); // 更新用户的口令， '' 中的内容已加密
SET PASSWORD= Password('n3w p@$$wOrd'); //更新自己的口令
```

#### 29 数据库维护

- 下面列出这个问题的可能解决方案
  -  使用命令行实用程序mysqldump转储所有数据库内容到某个外部文件。在进行常规备份前这个实用程序应该正常运行，以便能正确地备份转储文件。
  -  可用命令行实用程序mysqlhotcopy从一个数据库复制所有数据(并非所有数据库引擎都支持这个实用程序)。
  - 可以使用MySQL的BACKUP TABLE或SELECT INTO OUTFILE转储所有数据到某个外部文件。这两条语句都接受将要创建的系统文件名，此系统文件必须不存在，否则会出错。数据可以用RESTORETABLE来复原。

