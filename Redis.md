数据结构

#### 1.简单动态字符串（SDS）

```c
struct sdshdr{
    //记录buf数组中已使用字节的数量
    //等于SDS所保存的字符串长度
    int len;
    
    //记录buf数组中未使用字节的数量
    int free;
    
    //字节数组，用于保存字符串
    char buf[];
};
```

- **常数复杂度获取字符串的长度**    len

- **杜绝缓冲区的溢出**

  - char* strcat(char*dest, const char *src);
  - 先检查free 如果不能容纳，便扩充字符，再进行拼接，此时返回的 free的长度与len的长度相同

- **减少修改字符串是带来的内存重分配次数**

  - 对于一个包含了 N个字符的C字符串来说，这个 C字符的底层实现总是一个 N+1 个字符长的数组(额外的一个字符空间用于保存空字符 )
  - **空间预分配**
    - 如果对 SDS 进行修改之后，SDS 的长度(也即是 len 属性的值)将小于1MB那么程序分配和 len 属性同样大小的未使用空间（free），这时 SDS， len 属性的值将和free 属性的值相同。
    - 如果len >= 1MB, 则分配1MB的free
      - 如 len = 30MB，free = 1MB， 空字符 1byte
  - 惰性空间释放
    - 执行 sdstrim之后的 SDS 并没有释放多出来的8字节空间，而是将这8字节空间作为未使用空间保留在了 SDS 里面，如果将来要对 SDS 进行增长操作的话，这些未使用
    - 通过惰性空间释放策略，SDS 避免了缩短字符串时所需的内存重分配操作，并为将来可能有的增长操作提供了优化。
    - 与此同时，SDS 也提供了相应的 API，让我们可以在有需要时，真正地释放 SDS 的未使用空间，所以不用担心惰性空间释放策略会造成内存浪费。

- **二进制安全**

  - **C 字符串 除了字符串的末尾之外，字符串里不能包含空字符**，只能保存文本数据，而不能保存像图片、音频、视频、压缩文件这样的二进制数据。
  - 所有SDS API都会以**处理二进制的方式来处理SDS存放在buf数组里的数据**， 程序不会对其中的数据做任何限制、过滤、或者假设，数据在写人时是什么样的，它被读取时就是什么样。
  - 这也是我们将 SDS 的 buf 属性称为字节数组的原因-Redis 不是用这个数组来保存字符，而是用它来保存一系列二进制数据。

- 兼容部分C字符串函数

  - 数据末尾设置空字符串 可以是SDS重用<string.h> 中的函数
    - strcasecmp(sds->buf, "hello  world");
    - strcat(c_string, sds-buf)

  





#### 2.链表

![image-20230825172056352](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825172056352.png)

- **列表键**的底层实现之一就是链表

- 除了链表键之外， 发布与订阅、慢查询、监视器等功能也用到了链表

- Redis服务器本省还使用链表来保存多个客户端的状态信息、 构建客户端输出缓冲区

  ```c
  //adlist.h/listNode  --- 链表节点
  typedef struct listNode{
      //前置节点
      struct listNode *prev;
      struct listNode *next;
      
      //节点的值
      void *value;
  }listNode;
  
  // adlsit.h /list 来持有链表
  typedef struct list{
      //表头节点
      listNode *head;
      
      //表尾节点
      listNode *tail;
      
      //链表所包含的节点数量
      unsigned long len;
      
      //节点值复制函数
      void* (*dup)(void *ptr);
      
      //节点值释放函数
      void (*free) (void *ptr);
      
      //节点值对比函数
      int (*match)(void *ptr, void *key);
      
  }list;
  ```

  ![image-20230825172230023](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825172230023.png)

  ![image-20230825172309143](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825172309143.png)

  ![image-20230825172325244](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825172325244.png)

#### 3.字典

##### 3.1 字典

- 字典又称符号表、关联数组或映射
- 键值对
- 键是独一无二的
- Redis 中**数据库**就是使用字典来作为底层实现的
- 对数据库的增、删、查、改操作也是构建在对字典的操作之上的
- 字典还是哈希建的底层实现之一
- 实现
  - 使用哈希表作为底层实现
  - 一个哈希表中可以有富哦个哈希节点
  - 每个哈希表的节点就保存了一个键值对

```c++
//哈希表
//dict.h/dictht
typedef struct dictht{
    
    //哈希表数组
    dictEntry **table;   //table是一个数组，存储 dictEntry*的对象 
    //char * 表示字符数组， char**  表示 char* 的数组， 则是字符串数组
    
    //哈希表的大小
    unsigned long size;
    
    // 哈希表大小掩码， 用于计算索引值
    //总是等于 size-1
    unsigned long sizemask;
    
    //该哈希表已有的节点的数量
    unsigned long used;`
}dictht;

//哈希表节点
typedef struct dictEntry{
    
    //键
    void* key;
    
    //值
    union{
        void *val;
        uint64_t u64;
        int64_t s64;
    }v;
    
    //指向下一个哈希表节点，形成链表
    struct dictEntry *next
}dictEntry;



//字典
//dict.h/dict
typedef struct dict{
    //类型特定函数
    dictType *type;
    
    //私有数据
    void* privdata;
    
    //哈希表
    dictht ht[2];
    
    //rehash 索引
    //当 rehash 不在进行时， 值为-1
    int trehashidx;	    
}dict;

typedef struct dictType{
    //计算哈希值的函数
    unsigned int (*hashFunction) (const void* key);
    
    //复制键的函数
    void* (*keyDup)(void *privdata, const void* key);
    
    //复制值的函数
    void*(*valDup)(void *privdata, const void *obj);
    
    //对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    
    //销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
    
    //销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);
    
}dictType;
```

![image-20230825184826150](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825184826150.png)

##### 3.2哈希算法

```c++
//使用字典设置的哈希函数 ，计算键key的哈希值
hash = dict->type->hasFunction(key);

//使用哈希表的sizemask 属性和哈希值，计算出索引值
//根据情况不同，ht[x]可以是ht[0] 或者ht[1]
index = hash & dict->ht[x].sizemask;

hash = dict->type->hasFunction(k0); // redis 使用MurmurHash2算法计算哈希值
index = hash & dict->ht[0].sizemask;
```

当字典被用做数据库的底层实现，或者哈希键的底层实现是，Redis使用MurmurHash2算法来计算键的哈希值

##### 3.3 解决键冲突

- Redis 的哈希表使用**链地址法**来解决键的冲突
- 每个哈希表节点都有一个next指针
- 多个哈希表节点可以用next指针构成一个单向链表

![image-20230825192309096](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825192309096.png)

##### 3.4 rehash

- 为了让哈希表的负载因子(load factor)维持在一个合理的范围之内，当哈希表保存的键值对数量太多或者太少时，程序需要对哈希表的大小进行相应的扩展或者收缩。

- Redis对字典的哈希表执行rehash步骤
  - 为字典ht[1]分配空间，要取决于执行的操作，以及ht[0] 当前包含的键值对的数量（ht[0].used属性）
    - 扩展，ht[1] 的大小为 第一个 >= ht[0].used*2 的 2^n 次幂
    - 收缩， ht[1]的大小 为  第一个 >= ht[0].used 的 2^n 
  - 将ht[0]中所有的键值对rehash到ht[1]上
  - 当ht[0]全部迁移到ht[1]之后（ht[0]变为空表），释放ht[0], 将ht[1]设置为ht[0],并在ht[1]新创建一个空白哈希表，为下一rehash做准备

```c++
/*
当以下条件中的任意一个被满足时，程序会自动开始对哈希表执行扩展操作:
1)服务器目前没有在执行 BGSAVE 命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于1。
2)服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF命令，并且哈希表的负载因子大于等于 5。
*/
//当负载因子< 0.1时，自动执行收缩操作
// 负载因子 = 哈希表已保存的数量 / 哈希表的大小
load_factor = ht[0].used / ht[0].size
```

- 渐进式rehash步骤
  - 为ht[1]分配空间，让字段同时持有ht[0] 和 ht[1]
  - rehashidx 设置为0， 表示rehash正式开始
  - rehash期间， 每次对字典的增删改查，在执行的命令时，顺带架构rehashidx索引上的节点 rehash到ht[1]
  - 最终，所有键值对rehash完后，设置rehashidx = -1

- 为了避免 rehash 对服务器性能造成影响，服务器不是一次性将 ht[0] 里面的所有键值对全部rehash 到 ht[1]，而是分多次、渐进式地将 ht[0] 里面的键值对慢慢地 rehash到ht[1]。
- 在渐进式rehash 执行期间，新添加到字典的键值对一律会被保存到 ht[1]里面而 ht[0] 则不再进行任何添加操作，查找会先在ht[0]上找， 在到ht[1]上找

![image-20230825194004000](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825194004000.png)

![image-20230825194026076](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825194026076.png)

##### 3.4 字典API

![image-20230825194303897](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825194303897.png)

#### 4. 跳跃表

- 跳跃表(skiplist)是一种**有序数据结构**，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。
- 跳跃表支持平均 O(logN)、最坏 O(N） 复杂度的节点查找，还可以通过顺序性操作来批量处理节点
- Redis 使用跳跃表作为 **有序集合键**的底层实现之一，如果一个有序集合包含的元素较多，或元素成员是较长的字符串，跳跃表就会作为有序集合的底层实现

![image-20230825213142121](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825213142121.png)

```c++
//跳跃表节点 redis.h/zskiplistNode

typedef struct zskiplistNode{
    
    //层 level[]
    
    struct zskiplistLevel{
        
        //前进指针
        	struct zskiplistNode *forward;
        //跨度
        unsigned int span;
    }level[];
    
    //后退指针
    struct zskiplistNode *backward;
    
    //分值
    double score;
    
    //成员对象
    robj *obj;
    
}zskiplistNode;

//跳跃表
typedef struct zskiplist{
    //表头节点和表尾节点
    struct zskiplistNode *head, *tail;
    
    //表中节点的数量
   	unsigned long length;
    
    //表中层数最大的节点的层数
    int level;
}zskiplist;
```

![image-20230825213413989](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825213413989.png)

- 第一个层高为  1
- 第二个层高为  3
- 第三个层高为 5

![image-20230825213440685](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825213440685.png)

- 虚线为前进路线
- 虚线上的数值为 跨度
  - 两个及地点之间的跨度越大，他们相距得越远
  - 指向NULL的所有前进指正的跨度都为0
  - 将沿途访问符的所有层的跨度累计起来，得到的结果就是目标节点在跳跃表中的排位
- 后退指针
  - 用于从表尾想表头访问节点
- 分值与成员
  - 分值：double的浮点数
  - 成员对象： 指向一个字符串对象

##### API

![image-20230825213738160](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230825213738160.png)

#### 5. 整数集合

- 整数集合是**集合键**的底
- 
- 层实现之一
- 当一个集合值包含整数值元素，并且这个集合的元素数量不多是，Redis 就会使用整数集合作为集合键的底层实现
- 实现

```c++
//intset.h/intset
typedef struct intset{
    //编码方式
    uint32_t encoding;
    
    // 集合包含的元素数量
    uint32_t length;
    
    // 保存元素的数组， 从小到大排列， 并且不包含任何重复项
    int8_t contents[];
}intset;
```

![image-20230609152813902](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230609152813902.png)

![image-20230609152848143](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230609152848143.png)

##### 升级

- 根据新元素的类型，扩展整数集合底层数组的空间大小，并为新元素分配空间。
- 将底层数组现有的所有元素都转换成与新元素相同的类型，并将类型转换后的元素放置到正确的位上，而且在放置元素的过程中，需要继续维持底层数组的有序性质不变。
- 将新元素添加到底层数组里面



- 整数集合不支持降级操作，一旦对数组进行了升级，编码就会一直保持升级后的状态

##### API

![image-20230609160328684](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230609160328684.png)

#### 6 压缩列表

-  压缩列表是**列表键和哈希键**的底层实现之一
-  当一个列表键只包含少量列表项，并且每个列表项要么就是小整数值，要么就是长度比较短的字符串，那么 Redis 就会使用压缩列表来做列表键的底层实现。

![image-20230826001040734](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826001040734.png)

![image-20230826001209015](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826001209015.png)

**压缩列表节点的构成**

- 可以保存一个字节数组或者整数值

  **字节数组**

  - 长度小于等于63（2^6 -1）字节的字节数组
  - 长度小于等于16383（2^14 -1）字节的字节数组
  - 长度小于等于4294967295(2^32 -1) 字节的字节数组

  **整数**

  - 4位长， 0-12的无符号整数
  - 1字节长的有符号整数
  - 3字节长的有符号整数
  - int16_t 类型整数
  - int32_t 类型整数
  - int64_t 类型整数

![image-20230826001838544](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826001838544.png)

- previous_entry_length

  - 保存前一个节点的长度

  - 如果前一个节点长度 < 254 个字节， 则 previous_entry_length 的长度为1个字节。

  - 如果前一个节点长度 >= 254 个字节,则 previous_entry_length 的长度为5个字节。第一个字节保存OXFE

    后4个字节保存长度

![image-20230826002043423](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826002043423.png)

- encoding

  ![image-20230826002334383](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826002334383.png)

  - 记录了节点的content 属性所保存数据的类型以及长度

- content

  - 节点值可以是一个字节数组或者整数

、

##### 连锁更新

- 因为连锁更新在最坏情况下需要对压缩列表执行N次空间重分配操作，而每次空间重分配的最坏复杂度为O(N)， 所以连锁更新的最坏复杂度为O(N)。
- 首先，压缩列表里要恰好有多个连续的、长度介于250字节至253字节之间的节点，连锁更新才有可能被引发，在实际中，这种情况并不多见;
- 0其次，即使出现连锁更新，但只要被更新的节点数量不多，就不会对性能造成任何影响:比如说，对三五个节点进行连锁更新是绝对不会影响性能的;

##### API

![image-20230826184029419](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826184029419.png)

#### 7. 对象

- Redis基于这些数据结构创建了一个对象系统，这个系统包含**字符串对象**、**列表对象**、**哈希对象**、**集合对象**和**有序集合对象**这五种类型的对象，每种对象都用到了至少一种我们前面所介绍的数据结构。
- **字符串对象**是Redis 五种对象中唯一一种会被其他四种类型对象嵌套的对象
- Redis 的对象系统还实现了**基于引用计数技术**的**内存回收机制**，当程序不再使用某个对象的时候，这个对象所占用的内存就会被自动释放;
- 另外，Redis还通过**引用计数技术实现了对象共享机制**，这一机制可以在适当的条件下，通过让多个数据库键共享同个对象来节约内存。
- 最后，Redis的对象带有访问时间记录信息，该信息可以用于计算数据库键的空转时长，在服务器启用了maxmemory功能的情况下，空转时长较大的那些键可能会优先被服务器删除。
- Redis 使用**对象**来变数数据库中的**键和值**

```c
//键为字符串对象，值为字符串对象
SET msg "hello world"			//redis
TYPE msg						//redis
    
    
typedef struct redisObject{
    
    //类型
    unsigned type:4;
    
    //编码
    unsigned encoding:4;
    
    //地址底层实现数据结构指针
    void *ptr;
    // ...
}robj;

//键为字符串对象，值为列表对象
RPUSH numbers 1 3 5
TYPE numbers
    
//键为字符串对象，值为哈希对象
HMSET profile name Tom age 25 career Programmer
TYPE profile
    
//键为字符串对象，值为集合对象
SADD fruits apple banana cherry
TYPE fruits
    
//键为字符串对象，值为有序集合对象
ZADD price 8.5 apple 5.0 banana 6.0 cherry
TYPE price
```

![image-20230826191528055](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230826191528055.png)

![image-20230827180136034](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827180136034.png)

```c
SET msg "hello world "
OBJECT ENCODING msg
```

![image-20230609192203148](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230609192203148.png)

![image-20230609192214567](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230609192214567.png)

##### 7.1 字符串对象

- 字符串对象的编码可以是 **REDIS_ENCODING_INT** 、 **REDIS_ENCODING_EMBSTR**  、  **REDIS_ENCODING_RAW**

  - 一个字符串对象保存的是整数值

    - 这个整数值可以用long来表示

      `SET number 10086`

      `OBJECT ENCODING number`

      - 这里number是键， 键是一个字符串
      - value 是数字10086， 所以用int来保存

  - 如果字符串对象保存的是一个字符串值，并且这个值 > 32字节

    - 使用 raw 的编码， 用一个SDS（简单动态字符串）来保存

    `SET story "Long, long ago there lived a king ..."`

    `STRLEN stroy`

    `OBJECT ENCODING stroy`

  - embstr 编码是专门用于保存端字符串的一种优化编码方式

    - 这种编码和raw编码一样，都使用redisObject结构和sdshdr结构来表示字符串对象，
    - 但raw编码会调用**两次**内存分配函数来分别创建redisObject结构和sdshdr结构，
    - 而embstr编码则通过**调用一次**内存分配函数来分配-块连续的空间，空间中依次包含redisobject和sdshdr
    - 因为embstr编码的字符串对象的多有数据都保存在一块连续的内存里面
    - ![image-20230827180615885](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827180615885.png)

![image-20230827180601619](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827180601619.png)

![image-20230827180646259](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827180646259.png)

```
//embstr 编码
SET msg "hello"

OBJECT ENCODING msg

SET pi 3.14
OBJECT ENCODING pi	

```



###### 编码转换

- int编码的字符创对象和embstr编码的字符创对象在条件满足的情况下，会被转换为raw编码的字符串对象

- Redis 没有为embstr 编码的字符串对象编写任何相应的修改程序（只有int和raw编码的字符串对象有这些程序）

  ```
  SET number 10086
  
  OBJECT ENCODING number // "int"
  
  APPEND number "is a good number!"
  
  GET number // "10086 is a good number!"
  
  OBJECT ENCODING number //"raw"
  ```

###### 字符串命令的实现

- SET
- GET
- APPEND
- INCRBYFLOAT
- INCRBY
- DECRBY
- STRLEN
- SETRANCE
- GETRANCE

![image-20230827182050243](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182050243.png)

##### 7.2 列表对象

- 可以用压缩列表编码 **ziplist** 

![image-20230827182235900](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182235900.png)



- 可以用双端链表编码 **linkedlist**
  - 字符串对象是redis五种类型对象中唯一会被其他四种类型对象嵌套的对象


![image-20230827182251783](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182251783.png)

![image-20230827182530047](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182530047.png)



```
RPUSH numbers 1 "three" 5
```

###### 编码转换

- 当列表对象同时满足一下两个条件时，列表对象使用ziplist编码

  - 保存的所有字符串元素的长度 < 64字节
  - 保存的元素数量 <=  512

- 否则会使用linkedlist编码

  ```
  RPUSH blah "hello" "world" "again"
  
  OBJECT ENCODING blah  //"ziplist"
  
  RPUSH blah "wwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwwww"
  
  OBJECT ENCODING blah //"linkedlist"
  
  //保存元素超过512
  EVAL "for i = 1, 512 do redis.call('RPUSH', KEYS[1], i)end" 1 "intergers"
  
  LLen integers
  
  OBJECT ENCODING integers // "ziplist"
  
  RPUSH integers 513
  
  OBJECT ENCODING integers // "linkedlist"
  ```

###### 列表命令的实现

- LPUSH			--压入表头
- RPUSH           --压人表尾
- LPOP             --返回表头节点，并删除
- RPOP            -- 返回表尾节点并删除
- LINDEX        -- 返回指定节点
- LLEN            -- 返回长度
- LINSERT      --将新节点插入指定位置
- LTRIM         --删除所有不在指定范围的节点
- LSET           --改变指定位置的节点值

![image-20230827182719671](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182719671.png)

##### 7.3 哈希对象

- 可以是ziplist编码
  - 保存键的节点在前， 保存值的节点在后
  - 先添加的值放在表头，后添加的值放在表尾

```
HSET profile name "Tom"

HSET profile age 25

HSET profile career "Programmer"
```

![image-20230827182806698](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182806698.png)

![image-20230611144355083](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230611144355083.png)



![image-20230827182817261](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827182817261.png)

- hashtable 编码

  - 哈希对象中的每个简直对都使用一个字典的键值对来保存

  - 字典的每个键都是一个字符串对象，对象中保存了键值对的键

  - 字典的每个值都是一个字符串对象，对象中保存了键值对的值


![image-20230827183524555](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827183524555.png)

###### 编码转换

- 满足下面两个条件使用ziplist编码
  - 所有键值对的键和值的字符串长度 < 64 字节
  - 键值对数量 <=  512
- 否则使用hashtable



```
HSET book name "Mastering c++ in 21 days"

OBJECT ENCODING book //ziplist

//键超过长度
HSET book 1ong_1ong_1ong_1ong_1ong_1ong_1ong_1ong_1ong_1ong_1ong_description "content" 

OBJECT ENCODING book // "hashtable"

EVAL "for i = 1, 512 do redis.call('RPUSH', KEYS[1], i, i)end" 1 "numbers"

HLEN numbers //512

HMSET numbers "key" "value"

HLEN numbers //513

OBJECT ENCODING numbers // "hashtable"
```

###### 哈希命令的实现

- HSET 只能设置一个字段
- HMSET  可以设置多个字段

- HGET
- HEXISTS
- HDEL
- HLEN
- HGETALL

![image-20230827184003802](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827184003802.png)



##### 7.4  集合对象

- 可以使用intset（整数集合）

  ```
  SADD numbers 1 3 5
  ```

  ![image-20230827184056476](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827184056476.png)

- 或者hashtable编码

  ```
  SADD Dfruits "apple" "banana" "cherry"
  ```

  

  ![image-20230611152128171](C:\Users\11709\AppData\Roaming\Typora\typora-user-images\image-20230611152128171.png)

###### 编码转换

- 两个条件， 用intset 编码

  - 所有元素都是整数值
  - 元素数量  <= 512 

- 否则 用hashtable 编码

  ```
  SADD numbers 1 3 5
  
  OBJECT ENCODING numbers  // "intset"
  
  SADD numbers "seven"
  
  OBJECT ENCODING numbers // "hashtable"
  
  EVAL "for i = 1, 512 do redis.call('RPUSH', KEYS[1], i)end" 1 "integers"
  
  SCARD integers   //512
  
  OBJECT ENCODING numbers  // "intset"
  
  SADD integers 10086
  
  SCARD integers   //513
  
  OBJECT ENCODING numbers // "hashtable"
  ```

###### 集合命令的实现

![image-20230827184156403](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827184156403.png)

- SADD				添加元素
- SCARD              返回元素数量
- SISMERBER       判断一个元素是否在集合中
- SMEBERS          返回集合元素
- SPOP                 随机返回集合中一个元素，并删除它
- SREM                 删除给定的元素

##### 7.5 有序集合对象

- 压缩列表 ziplist

  - 第一个节点保存元素的成员，第二个节点保存元素的分值

    ```
    ZADD price 8.5 apple 5.0 banana 6.0 cherry
    ```


![image-20230827185509794](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827185509794.png)

![image-20230827185527431](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827185527431.png)

- 跳跃表 skiplist

  - 底层实现使用zset结构作为底层实现

  - 一个zset结构同时包含一个字典和一个 跳跃表

    ```c
    typedef  struct zset{
        zskiplist *zsl;
        dict *dict;
    }zset;
    ```

  - zsl 跳跃表按分值从小到大保存了所有集合元素

  - 比如ZRANK、ZRANGE 等命令就是基于跳跃表API实现的

  - dict 为有序集合创建了一个成员到分值的映射

    - 键：元素成员
    - 值：分值

  - 程序可以用O(1)复杂度。 查找给定成员的分值

  - 有序集合每个元素成员都是一个字符串对象

  - 分值都是double

  - 虽然 zset 结构同时使用跳跃表和字典来保存有序集合元素但这两种数据结构都会通过指针来共享相同元素的成员和分值，所以同时使用跳跃表和字典，不会浪费额外的内存

- 为什么要同时使用跳跃表和字典实现

  - 单使用跳跃表：可以排序，根据成员查找分值这个特性的复杂度从O(1) ->O(logn)
  - 单使用字典：无序， 每次使用ZRANK、ZRANGE 都需要对字典保存的所有元素进行排序，

![image-20230827194241920](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827194241920.png)

###### 编码转换

- 两个条件，使用ziplist
  - 元素数量< 128
  - 元素成员长度 < 64
- 否则使用skiplist

```
EVAL "for i=1, 128 do redis.call('ZADD',KEYS[1], i, i) end" 1 numbers

ZCARD numbers  //128
OBJECT ENCODING numbers	//ziplist

ZADD numbers 3.14 pi

ZCARD numbers  //129

OBJECT ENCODING numbers	//skiplist

ZADD blah 1.0 www
OBJECT ENCODING blah  //ziplist

ZADD blah 2.0 ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo
OBJECT ENCODING blah //skiplist

```

###### 有序集合的实现

![image-20230827194949031](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827194949031.png)

##### 7.6 类型检查与命令多态

###### 两种类型

- 一种命令对任何类型的键执行

  - DEL、EXPIRE、RENAME、TYPE、OBJECT

- 只能针对特定类型的键执行

  - SET、GET、APPEND、STRLEN 	字符键
  - HDEL、HSET、HGET、HLEN         哈希键
  - RPUSH、LPOP、LINSERT、LLEN   列表键
  - SADD、SPOP、SINTER、SCARD   集合键
  - ZADD、ZCARD、ZRANK、CSORCE   有序集合键

- 类型特定命令所进行的类型检查是通过redisObject 结构的type属性实现的

  ![image-20230827200256810](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827200256810.png)

###### 多态命令实现

Redis除了根据值对象的类型来判断键是否能够执行执行命令之外，还会根据值对象的编码方式，选择争取的命令实现代码来执行命令

比如一个列表建的编码可以是 ziplist 和linkedlist，当执行LLEN命令

- 如果是ziplist 编码，程序将使用ziplistLen函数
- 如果是linkedlist ，将使用listLength函数

因此我们认为LLEN命令是多态的

实际上DEL、EXPIRE、TYPE也可以认为是多态的，是基于类型多态，一个命令可以同时用于多种不同类型的键

LLEN 是基于编码的多态

![image-20230827201117079](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827201117079.png)

##### 7.7内存回收

Redis在自己的对象系统中构建了一个引用技术实现的内存回收机制，通过这一机制程序可以通过跟踪对象的引用j计数信息，在适当的时候自动释放对象并进行内存回收

- 当创建一个新对象是，引用技术的值会被初始化为1
- 当对象被一个新程序使用时，它的引用计数会+1
- 当对象不在被一个程序使用时， -1

- 引用计数=0，对象所占内存释放

##### 7.8对象共享

当键B 同键A的值对象一样时

- 将数据库键的值指针指向一个现有的对象

- 将共享的值对象的引用计数+1

- Redis在初始化服务器时，穿件一万个字符串对象，包含0-9999的整数值，当用到这些时，不是创建是共享

- 但是Redis支队包含整数值的字符串对象进行共享
  - 如果共享对象是保存整数值的字符串对象，那么验证操作的复杂度为 O(1);
  - 如果共享对象是保存字符串值的字符串对象，那么验证操作的复杂度为 O(N);
  - 如果共享对象是包舍了多个值(或者对象的 ) 对象，比如列表对象或者哈希对象，那么验证操作的复杂度将会是 O(N)。

##### 7.9 对象的空转时长

- lru属性

- 空转时间 = 当前时间 - 键的值对象的lru时间

- OBJECT IDLETIME  打印空转时间
- OBJECT IDLETIME 在访问键的值对象时， 不会修改对象的lru属性

#### 单机数据库的实现

##### 8 数据库

![image-20230827203041409](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827203041409.png)

###### 切换数据库 SELECT

- 每个客户端都有自己的目标数据库
- 默认情况 Redis客户端的目标数据库为0号数据库
- 用SELECT 切换目标数据库

```
SET msg "hello world"		//ok

GET msg			//hello world

SELECT 2	//切换目标数据库

GET msg 	//nil

SET msg "another world"

GET msg 	// another world
```

![image-20230827210522123](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827210522123.png)

- 在使用FLUSHDB这样危险的指令时，最后先执行一个SELECT命令，显示的切换到指定的数据库
  - 因为Redis目前没有可以返回客户端目标数据库的命令

###### 数据库的键空间

- Redis是一个键值对数据库服务器，每个数据库否有一个redisDb结构表示，其中redisDb结构中的**字典**保存了数据库的所有键值对，将这个字典称为**键空间**
- 键空间的键也及时数据库的键，每个键都是一个字符串对象
- 键空间的值也是数据库的值，每个值是字符串对象、类别对象、哈希表对象、集合对象和有序集合对象的任意一种

```
SET message "hello world"

RPUSH alphabet "a" "b" "c"

HSET book name "Redis in Action"

HSET book author "Josiah L.Carlson"

HSET book publisher "Manning"
```

![image-20230910135926577](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910135926577.png)

**添加新键**

```
SET date "2013.12.1"
```

![image-20230910140010153](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910140010153.png)

**删除键**

```
DEL book
```

![image-20230910140207127](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910140207127.png)

**更新键**

```
SET message "blah blah"
```



![image-20230910140104819](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910140104819.png)

```
HSET book page 320
```

![image-20230910140123437](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910140123437.png)

**对键取值**

```
GET message
```

![image-20230827211759244](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230827211759244.png)



- 清空数据库的FULSHDB，就是通过删除键空间中的多有键值对实现的
- RANDOMKEY   在键空间随机返回一个键
- DBSIZE 命令，就是通过返回键空间中包含的键值对的数量来实现的。
- 类似的命令还有 EXISTS、RENAME、KEYS 等，这些命都是通过对键空间进行操作来实现的。

**键空间的读写维护**

- 读一个键后，根据键是否存在来更新键空间命中次数（hit） 或不命中次数(miss)
  - INFO Status 查看 keyspace_hits、kerspace_misses属性
- 读取一个键后， 更新LRU(最后一次使用)时间
  - OBJECT idletime<key>可用于查看该键的空转时间
- 读取一个键时，发现键已过期，删除这个过期键
- WATCH监视这个键时，修改后被标记为脏
- 开启数据库通知功能，对键修改后,按配置发送相应通知

###### **设置键的生存时间或过期时间**

- EXPIRE 或 PEXPIRE   客户端以秒或者毫秒精度为数据库汇总的某个键设置生存时间（TTL），经过指定的秒数或者毫秒数之后，服务器会自动删除生存时间为0的键

```
SET key value

EXPIRE key 5

GET key		//5秒之内  value

GET key		//5秒之后 nil
```

- EXPIREAT 或 PEXPIREAT  以秒或毫秒的精度给数据库中的某个键设置过期时间（一个时间戳）

```
EXPIREAT key 1377257300
```

- TTL 或 PTTL 接收一个带有生存时间或者过期时间的键，返回这个键的剩余生存时间
- EXPIRE、PEXPIRE、EXPIREAT、PEXPIREAT 都是使用 **PEXPIREAT**  实现的
  - EXPIREAT  将 PEXPIREAT  **的毫秒数/ 1000**
  - PEXPIRE   将 PEXPIREAT **的时间戳 - 当前时间** 
  - EXPIRE  将 PEXPIREAT 的 **(时间戳 - 当前时间) /1000**

**保存过期时间**

redisDb 结构中的expires字典保存了数据库中所有键的过期时间，称为过期字典

- 过期字典的键是一个指针，指向键空间的某个键对象
- 值是一个 long long 类型的整数， 保存这个键对象的过期时间（毫秒精度的时间戳）

![image-20230910140604106](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910140604106.png)

- 上图是展示，但是键空间的键和过期字典的键都指向同一个键对象，不会出现任何重复对象

```
PEXPIREAT message 1391234400000 
```

![image-20230910140834205](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910140834205.png)

**移除过期时间**

```
PEXPIREAT message 1391234400000

TTL message		//13893281

RERSIST message			//移除过期时间
TTL message		//-1
```

**剩余生存时间**

- TTL 以秒为单位返回键的剩余生存时间
- PTTL 则以毫秒为单位返回键的甚于生存时间

都是通过计算键的过期时间和当前时间之间的差来实现的

```
EXPIREAT alphabet 1385877600000
TTL alphabet	//8549007
PTTL alphabet	//8549001011
```

**过期键的判定**

- 检查给定键是否存在于过期字典，存在取得过期时间
- 检查UNIX时间错是否大于键的过期时间， 是，则过期，否则未过期

**过期键的删除策略**

- **定时删除**：

  - 设置键的过期时间的同时，创建一个定时器，过期时间来临， 立即删除键
  - 对内存友好，及时释放福哦其键所占用的内存
  - 对CPU时间不友好，花大量时间在删除键上，无法及时处理请求
  - 此外，定时器的实现方式是无需链表，查找以恶个时间的时间复杂度为O(N)
  - 让服务器创建大量的定时器，实现定时删除，不现实

- **惰性删除**：

  - 程序只有在取出键是才对键的过期时间进行检查，过期则删除

  - 因此这样对CPU时间来说最友好
  - 对内存最不友好，可能会造成内存中有大量的过期键，占用内存
  - 实现
    - db.c/ expireIfNeeded 函数实现
      - 键过期则删除
      - 否则不做动作

  ![image-20230910143713781](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910143713781.png)

- **定期删除：**

  - 上述两种方式的折中
    - 定时删除占用太多CPU时间，影响服务器的相应时间和吞吐量
    - 惰性删除浪费太多内存，有内存泄露的风险
  - 每隔一段时间执行一次删除过期键的操作，并限制执行时长和频率
  - 有效减少过期键带来的内存浪费

​	**难点**：如何设置执行时长和频率

- 删除执行时间长 或 太频繁 则退化成  定时删除

- 删除执行时间短 或 不频繁 则退化成  惰性删除

实现

- redis.c /activeExpireCycle 函数实现
  - 当redis.c/serverCron 函数执行时， activeExpireCycle 函数被调用
  - 它在规定的时间内，多次遍历服务器中的各个数据库，从expires字典中检查异步键的过期时间，删除其中的过期键		

- 简单概括
  - 函数每次运行时，都从一定数量的数据库中取出一定数量的随机键进行检查，并删除其中过期的键
  - current_db 会记录当前检查的进度，并在下一次调用函数的时候接着上次进度处理
  - 随着这个函数的不断运行，服务器中的所有数据库都会被检查一遍

```python
//伪代码
#默认每次检查的数据库数量
DEFAULT DB NUMBERS = 16;
#默认每个数据库检查的键数量
DEFAULT KEY NUMBERS = 20;
#全局变量，记录检查进度
current db = 0
def activeExpireCycle():
#初始化要检查的数据库数量
#如果服务器的数据库数量比 DEFAULTDBNUMBERS要小
#那么以服务器的数据库数量为准
if server.dbnum < DEFAULT DB NUMBERS:
	db numbers = server.dbnum
else:
	db numbers  DEFAULT DB NUMBERS
#遍历各个数据库
for i in range(db numbers):
    #如果current db 的值等于服务器的数据库数量
    #这表示检查程序已经遍历了服务器的所有数据库一次
    #将current db 重置为 0，开始新的一轮遍历
    
    if current db == server.dbnum:
        current db = 0
        
    # 获取当前要处理的数据库
    redisDb = server.db[current db]
    # 将数据库索引增 1，指向下一个要处理的数据库current db + 1
    #检查数据库键
for j in range(DEFAULT KEY NUMBERS):
    
	#如果教据库中没有一个键带有过期时间，那么跳过这个教据库
	# 如果数据库中没有一个键带有过期时间，那么跳过这个数据库
    if redisDb.expires.size() == 0: break
        
	#随机获取一个带有过期时间的键
    key_with_ttl = redisDb.expires.get random key()
	#检查键是否过期，如果过期就删除它
    if is_expired(key_with_ttl):delete key(key_with_ttl)
	# 已达到时间上限，停止处理
	if reach time limit(): return
```

###### AOF、RDB对过期键的处理

**RDB**

- 在执行SAVE或者BGSAVE命令创建一个新的RDB文件时，程序会对数据库中的键进行检查，已过期的键不会被处保存到性穿件的RDB文件中
  - 主服务器模式运行，那么在载入RDB文件时，未过期的键会被载入到数据库中，过期键忽略
  - 从服务器模式运行时， 不管RDB文件中保存的键是否过期都会被载入到数据库中。等主从进行数据同步的时候，才会删除

**AOF**

- 当以AOF持久化模式运行时， 如果数据库中某个键过期，但是还没被删除，那么AOF文件不会对这个键做任何操作
- 当被惰性删除或则定期删除之后，程序会向AOF文件追加一天DEL语句

- 在执行AOF重写时， 对数据库键进行检查，已过期的键不会保存在重写后的AOF文件中

###### 在复制模式下对过期键的处理

- 主服务器在删除一个过期键后，会显示的向所有从服务器发送一条DEL命令
- 从服务器在处理客户端的读请求时，遇到过期键也不会将过期键删除，像处理未过期键一样处理
- 从服务器只有接收到DEL命令，才会删除过期键

这样做可以保证主从服务器数据的一致性

###### **数据库通知**

- redis 2.8版本后才有的功能
- 让客户端通过订阅给定的频道或模式，知晓数据库键的变化、以及数据库中命令的执行情况
  - 某个键执行了什么命令属于键空间通知

![image-20230910161113172](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910161113172.png)

- 某个命令被什么键执行了  是 键时间通知

  ![image-20230910170004860](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910170004860.png)

  AKE     ----         服务器发送所有类型的键空间通知和键事件通知

  AK       ----        服务器发送所有类型的键空间通知

  AE       ----        服务器发送所有类型的键事件通知

  K$ 	 ----        只发送字符串键有关的键空间通知

  El       ----        只发送和列表键有关的键事件通知

  等

##### 9.RDB持久化

- 因为redis是内存数据库，它将自己的数据库状态存储在内存里面，如果不持久化到磁盘中， 那么进程结束，数据就消失了
- RDB文件是一个经过压缩的二进制文件，可以通过该文件还原保存RDB文件时，数据库的状态
- RDB文件存在磁盘，只要RDB在，数据库就可以恢复

![image-20230910171534835](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910171534835.png)

###### RDB文件的创建和载入

**SAVE / BGSAVE**

**SAVE**

- 会阻塞redis服务器进程，指导RDB文件创建完成
- 服务器会拒绝所有请求

**BGSAVE**

- 会派生一个子进程负责创建RDB文件
- 服务器仍然可以处理请求
  - 但是在BGSAVE执行期间， 服务器处理SAVE、BGSAVE、BGREWRITEAOF 会和平时不同
  - 客户端发送的SAVE命令会被拒绝
    - 避免父进程和子进程同时调用rdbSave()，产生竞态
  - 客户端发送的BGSAVE命令也会被拒绝
    - 也避免产生竞态
  - BGSAVE执行时，BGREWRITEAOF 命令会被延迟BGSAVE命令执行完毕之后执行
  - BGREWRITEAOF执行时， BGSAVE被拒绝
    - 这两都是子进程在执行，两子进程同时执大量的I/O操作，性能不高

```python
#伪代码
	def SAVE():
        rdbSave()
 	def BGSAVE():
       #创建子进程
        pid = fork()
        if pid == 0:
            rdbSave()
            #完成之后向父进程发送信号
            signal_parent()
        else if pid > 0:
            handle_request_amd_wait_signal()
        else
        	#处理出错的情况
            handle_fork_error()
        
```

**RDB载入工作**	

- 是服务器启动时自动执行的
- 只要启动时检测到RDB文件的存在，就会自动载入RDB文件
- 载入RDB是，会一直处于阻塞状态，直到载入完成

**RDB与AOF对比**

- AOF文件的更新频率比RDB文件的更新频率高
  - 如果服务器开启AOF持久化，那么服务器有限使用AOF文件来还原数据库
  - 只有AOF持久化关闭是，服务器才会使用RDB文件来还原数据库

![image-20230910172853320](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910172853320.png)

**自动间隔保存**

- 设置服务器的save选项，让服务器每隔一段时间自动执行一个BGSAVE命令

  ![image-20230910173838386](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910173838386.png)

```c
struct redisServer{
    //记录保存条件的数组
struct saveparam *saveparams;
    
//修改计数器
    long long dirty;
    
// 上一次执行保存时间
    time_t lastsave;
};

struct saveparam{
    //秒数
    time_t seconds;
    
    //修改数
    int changes;
};

```

```python
#检查条件保存是否满足
def serverCron():
    
    #遍历所有保存条件
    for saveparam in server.saveparams:
        #计算距离上次保存 有多少秒
        save_interval = unictime_now() - server.lastsave
        
        if server.dirty >= saveparam.changes and save_interval > saveparam.seconds:
            BGSAVE()
```



![image-20230910174411529](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910174411529.png)

###### RDB文件结构

![image-20230910204001992](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910204001992.png)

- REDIS 这5个字符，可以帮助程序快速检查载入的文件是不是为RDB文件
- db_version 占4个字节， 记录RDB的版本号
- database 可以包含另个或任意多个数据库， 空 为0字节
- EOF长度为1个字节，标志着RDB文件正文内容的结束
- check_num 是一个8个字节常的无符号整数，保存着一个校验和，检查RDB文件是否出错或者损坏

**database**

0号数据库和3号数据库非空

![image-20230910204457229](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910204457229.png)

![image-20230910204508859](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910204508859.png)

- SELECTDB 长度为1个字节，当读入程序遇到这个值的时候，他知道接下来要读入的将是一个数据库号码
- db_number 是 1个字节、2个字节、5个字节
- key_value_paires  保存了数据库中的所有键值对数据
  - 不带过期时间的键值对
  - key总是一个字符串对象
  - ![image-20230910205525044](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910205525044.png)

![image-20230910205555549](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910205555549.png)

- 带有过期时间的键值对

  ![image-20230910205724453](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910205724453.png)

  - EXPIRETIME_MS , 	1个字节
  - ms 8个字节的到符号整数， 以毫秒为单位的UNIX时间戳

**VALUE**

- TYPE: REDIS_RDB_TYPE_STRING ----      **字符串对象**

  - 编码为REDIS_ENCODING_INT 或 REDIS_ENCODING_RAW

    - REDIS_RDB_ENC_INT8、REDIS_RDB_ENC_INT16、REDIS_RDB_ENC_INT32
    - 分别存储8位、16位、32位整型数

  - REDIS_ENCODING_RAW

    - 有压缩和不压缩两种方法
      - 长度 <= 20字节， 不压缩
      - 长度 > 20 字节 压缩
      - REDIS_RDB_ENC_LZF 标志着字符串已经被LZF算法压缩

    ![image-20230910210643982](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910210643982.png)

    ![image-20230910210655627](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910210655627.png)

![image-20230910210704951](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910210704951.png)

- TYPE : REDIS_RDB_TYPE_LIST   ---------   **列表对象**

- ![image-20230910210904628](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910210904628.png)

- ![image-20230910210915973](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910210915973.png)

  - 第一个列表项的长度为5， 字符串 “hello”

  - 第二个  也为5， “world”

  - 第三个  为1， “!”

- TYPE: REDIS_RDB_TYPE_SET  ---------    **集合对象**

  ![image-20230910213743237](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910213743237.png)

  ![image-20230910213753501](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910213753501.png)

- TYPE ： REDIS_RDB_TYPE_HASH   -----  哈希对象

  ![image-20230910213912605](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910213912605.png)

![image-20230910213921854](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910213921854.png)

![image-20230910213932060](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910213932060.png)

- TYPE: REDIS_RDB_TYPE_ZSET   -----   有序集合对象

  ![image-20230910214037278](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910214037278.png)

  ![image-20230910214146281](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910214146281.png)

  ![image-20230910214155657](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910214155657.png)

- ZIPLIST编码的列表、哈希表或者有序集合

  - 将压缩列表转换成一个字符串
  - 将转换所得的字符串对象保存到RDB文件
  - 通过type类型转换
    - 如REDIS_RDB_TYPE_HASH_ZIPLIST 那么就把压缩列表随想的类型转换为哈希表

###### 分析RDB文件

![image-20230910214916073](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910214916073.png)

![image-20230910215031106](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910215031106.png)

![image-20230910215133086](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910215133086.png)

##### 10.AOF 持久化

- RDB 持久化通过保存数据库中的键值对来记录数据库的状态
- AOF是通过保存Redis 服务器所执行的写命令来记录数据库的状态

![image-20230910215548405](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910215548405.png)

![image-20230910215556060](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910215556060.png)

**实现**

- 命令追加
  - 会议协议格式将被执行的写命令追缴到服务器状态的aof_buf 缓冲区末尾

```c
struct rediserver{
    
    
    //AOF 缓冲区
    sds aof_buf;
};

 SET KEY VALUE
 *3\r\n$3\r\nSET\r\n$3\r\nKEY\r\n$5\r\nVALUE\r\n
```



- AOF 文件的写入与同步
  - redis 服务器进程就是一个事件循环（loop）,负责接收客户端的命令请求，以及想客户端发送命令回复称为文件事件
  - 时间事件 是serverCron函数这样定时运行的函数（定期删除过期键的  定期事件）
  - redis 在处理文件事件时，不断地点aof_buf缓冲区追加内容。在结束事件循环之前，会调用flushAppendonlyFile 函数，将aof_buf中的内容写到AOF文件中

```python
def eventloop:
    while True:
        #处理文件事件
        # 将新内容追加到aof_buf
        processFileEvents()
        #处理时间事件
        processTimeEvents()
        #考虑是否将aof_buf 中的内容写入和保存到AOF文件中
        flushAppendoOnlyFile()
```

![image-20230910221242568](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230910221242568.png)

- 现代操作系统中，单用户调用write函数，将数据写入文件的时候，操作系统会暂时将数据保存到内存缓冲区，直到缓冲区满，才将数据写入磁盘，但是计算机突然停机了怎么办？
- 一次系统提供了fsync 和fdatasync 两个同步函数， 它们可以强制让操作系统立即将缓冲区数据写入磁盘

- always同步，最安全，但是效率不高，出现故障，只会丢失一个事件循环中发生的命令数据
- everysec ， 也足够快，出现故障丢失1秒的命令数据
-  no ，速度最快， 发生故障，会丢失上次同步AOF文件之后的所有写命令数据

###### AOF文件的载入和还原

- AOF文件里面包含了重建数据库状态所需的所有写命令

- 只需要读入并重新执行一遍AOF文件里面保存的写命令就行

- 如何执行

  - 创建一个不带网络连接的伪客户端
    - redis命令只能在客户端上下文中执行
  - 从AOF文件中分析并读取出一条写命令
  - 使用尾客户端执行被读出的写命令

  ![image-20230911210228673](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230911210228673.png)

###### AOF重写

- 随着时间的流逝，AOF文件的内容会越来越多，不加以控制，会对服务器造成影响

- redis可以创建一个新的AOF文件来替代现有的AOF文件

  ![image-20230911210506584](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230911210506584.png)

- 直接执行 `RPUSH list "C", "D", "E",F","G"` 来代替这六条指令
- 重写原理：首先从数据库中读取键现在的值，然后用一天命令去记录键值对代替之间记录这个键值对的多条命令，所以新生成的AOF文件不会浪费任何硬盘空间

```python
def aof_rewrite(new_aof_file_name):
    #创建新的AOF 文件
    f = create_file(new_aof_file_name)
    #遍历数据库
    for db in redisServer.db:
        #忽略空的数据库
        if db.is_empty():continue
        
        #根据键的类型对键进行重写
        if key.type == String:
            rewrite_string(key)
        elif key.type == List:
            rewrite_lsit(key)
        elif key.type == Hash:
            rewrite_hash(key)
        elif key.type == Set:
            rewrite_set(key)
        elif key.type == SortedSet:
            rewrite_sorted_set(key)
            
        #如果键带有过期时间，那么过期时间也要被重写
        if key.have_expire_time():
            rewrite_expire_time(key)
  #写入完毕，关闭文件
	f.close()
    
    
def rewrite string(key):
	#使用 GET命令获取字符串键的值
	value = GET(key)
	#使用 SET命今重写字符串键
	f.write command(SET, key, value)
    
def rewrite list(key):
	#使用 LRANGE 命获取列表键包的所有元素
	iteml，item2，..·, itemN = LRANGE(key， 0，-1)
	#使用RPUSH命今重写列表键
	f.write command(RPUSH, key, iteml, item2， ...itemN)
    
def rewrite hash(key):
	# 使用 HGETALL 命今获取哈希键包含的所有键值对
	fieldl，valuel，field2，value2，...， fieldN，valueN = HGETALL(key)
	#使用HMSET命令重写哈希键
    f.write_command(HMSET， key， fieldl， valuel， field2， value2，... fieldNvalueN)
    
def rewrite set(key);
	#使用SMEMBERS 命令获取集合键包含的所有元素
	elem1，elem2， ... elemN = SMEMBERS(key)
	#使用 SADD命令重写集合键
    f.write command(SADD， key, elem1，elem2，...  elemN)
    
def rewrite sorted set (key):
	# 使用 ZRANGE 命令获取有序集合键包含的所有元素
	memberl， scorel，member2，score2，... memberN， scoreN = ZRANGE(key，0，-1"WITHSCORES")
	# 使用 ZADD命令重写有序集合键
	f.write command(ZADD， key， scorel， memberl， score2， member2，.. scoreN,memberN)
def rewrite expire time(key):
	# 获取毫秒精度的键过期时间戳
    timestamp = get expire time in unixstamp(key)
	#使用PEXPIREAT 命令重写键的过期时间
    f.write command(PEXPIREAT， key,timestamp)
```

![image-20230911212042595](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230911212042595.png)

![image-20230911212052170](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230911212052170.png)

注意: 如果元素的数量超过了redis.h/REDIS_AOF_REWRITE_ITEMS_PER_CMD 常量的值，为64， 那么重写程序将使用多条命令来记录键的值，而不单单使用一条命令。

- AOF重写程序aof_rewrite 函数，包含大量的写入操作
  - 所以redis决定将APF重写程序放到子进程中执行
  - 子进程进行AOF重写期间，服务器进程（父进程）可以继续处理命令请求
  - 子进程带有服务器进程的数据副本，使用子进程而不是线程，可以避免使用锁的情况下， 保证数据的安全性
- 在子进程重写期间，如果父进程又有新的写命令，那么久写入AOF重写缓冲区
- redis服务器执行完一个写命令之后，它会同时将这个写命令发送给AOF缓冲区和AOF重写缓冲区

**在子进程执行AOF重写期间，服务器进程的三个工作**

- 执行客户端发送来的命令
- 将执行后的写命令追加到AOF缓冲区
- 将执行后的写命令追加到AOF重写缓冲区

子进程写完后向父进程发送信号，调用一个信号处理函数

- 将AOF重写缓冲区中的内容写入新的AOF文件
- 对新的AOF文件进行改名， 原子地覆盖现有的AOF文件

##### 11. 事件

redis是一个事件驱动程序

​	文件事件：redis服务器通过套接字与客户端通信产生相应的文件事件

​	时间事件：一些定时执行的函数（serverCron 实时删除）

redis基于Reactor模式开发了自己的网络事件处理器，称为文件处理器

- 当一个套接字准备好执行连接应答、写入、读取、官兵等操作是，就会产生一个文件事件
- 所有产生事件的套接字都放到一个队列里面。然后有序、同步的向文件事件丰排期发送套接字，根据事件类型调用相应的事件处理器

![image-20230912010316624](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912010316624.png)

###### I/O多路复用的实现

- 封装select、epoll、evport，每个I/O多路复用函数库都实现相同的API

**事件类型**

- 可读事件 AE_READABLE（client -> socketfd write / close/ 有新的可应答（acceptable）套接字出现）
- 可写事件 AE_WRITABLE（client -> socketfd  read）

**API**

```c
//ae.c
aeCreateFileEvent(fd, eventType, handler); // 加入监听，并绑定事件和事件处理器

aeDeleteFileEvent(fd, eventType); //监听的事件类型, 取消对该事件的监听

aeGetFileEvents(fd); //返回被监听的事件类型 
	//没有事件		返回AE_NONE
	//读事件		AE_READABLE
	//写事件		AE_WRITAABLE
aeWait(fd, eventType, unixtime);  //在指定时间内，阻塞并等待fd给定事件的产生

aeApiPoll(timeval); //在给定事件内阻塞并等待aeCreateFileEvent监听的fd产生文件事件，当产生文件事件，或超时则返回
```

**事件处理器**

- 连接应答处理器

  - networking.c/acceptTcpHander
    - 包装 sys/socket.h/accept函数
  - redis服务器初始化时，将 这个处理器与 socketfd的可读时间关联，当客户端connect时，产生可读事件，引发应答处理器执行

- 命令请求处理器

  - networking.c/readQueryFromClient
    - 封装 unistd.h/read
  - 当客户端连接服务器后，服务器会将 客户端fd 与 可读事件关联起来
  - 当client ->server 发送命令请求是，就调用命令请求服务器

- 命令回复处理器

  - networking.c /sendRelyToClient
    - 封装 unistd/write
  - 当客户端准备好接收服务器传回的命令是，执行
  - 发送完毕接触 client fd 与可读事件的关联

- 复制处理器

  ![image-20230912015633190](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912015633190.png)

###### 时间事件

- 定时事件：在一段时间后执行一次
- 周期性事件：每隔一段时间执行一次
- 时间事件有三个属性组成
  - ID： 全局唯一性ID，从小到大递增排序、新事件ID > 旧事件ID
  - when： 毫秒精度UNIX时间戳， 记录到达时间
  - timeProc： 时间事件处理器，一个回调函数
  - ae.h/AE_NOMORE    -----    定时事件
  - 非AE_NOMORE         -----    周期事件

**实现**

- 所有时间事件存储在一个**无序链表**中，每当时间事件执行器运行时，遍历整个链表，查找到期时间事件，调用相应事件处理器
- 新的时间事件总是插入链头，按ID逆序排序（无序链表指不按when排序）
- 无序链表并不影响性能
  - 因为正常模式下只有serverCron一个时间事件
  - 在benMark模式下，只使用两个时间事件

**API**

ae.c/aeCreateTimerEvent ( 毫秒数， timeProc)；

- 在时间达到后，调用timeProc

ae.c/aeDeletaFileEvent (ID);

- 删除 指定ID 的时间事件

ae.c/aeSearchNearestTimer()

- 返回到达时间距离当前时间最接近的那个时间事件

**实例** serverCron函数（周期性事件）

- 更新服务器各类统计信息，时间、内存占用、数据库占用
- 清理数据库过期的键值对
- 关闭和清理连接失败的客户端
- 尝试进行AOF 或RDB持久化
- 服务器为主服务器，对从服务器定期同步
- 处于集群模式，对集群进行定期同步和连接测试

- 2。6版本  每秒运行10次

###### 事件调度和执行

- 何时处理文件事件、何时处理时间事件

- ae.c/aeProcessEvents

  ```python
  def aeProcessEvents():
      
      #获取到达时间离当前时间最接近的时间事件
      time_event = aeSearchNearestTimer()
      
      #计算最接近的时间事件距离到达还有多少毫秒
      remaind_ms = time_event.when-unix_ts_now()
      
     #如果时间已到达， 那么remained_ms可能为负数，将它设置为0
  	if remained_ms < 0:
          remained_ms = 0
      #根据remained_ms 的值，创建timeval结构
      timeval = create timeval with ms(remaind ms)
  	# 阻寒并等待文件事件产生，最大阻塞时间由传入的 timeval 结构决定开如果remaind_ms 的值为 0，那么		#aeApiPol1 调用之后马上返回，不阻寨
      aeApiPoll(timeval)
  	#处理所有已产生的文件事件
      processFileEvents ()  //虚构的
  	#处理所有已到达的时间事件
  	processTimeEvents ()
  ```

  ![image-20230912023626379](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912023626379.png)

  ![image-20230912023651213](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912023651213.png)

##### 12 客户端

- redis 是典型的一对多的服务器程序
- redis.h/redisClient
  - 客户端套接字描述符
  - 客户端名字
  - 标志值（flag）
  - 正在使用的数据库指针
  - 执行的命令， 命令的参数。命令参数的个数，执行命令实现函数的指针
  - 输入缓冲区和输出缓冲区
  - 赋值状态信息
  - BRPOP， BLPOP 等列表阻塞命令时使用改的数据结构
  - 事务状态执行WATCH命令用到的数据结构
  - 执行发布与订阅时用到的数据结构
  - 身份验证
  - 创建时间，与服务器最后一次通信时间，以及客户端的输出缓冲区超出软性限制的时间
- clients 属性时一个链表，保存了所有与服务器连接的客户端的状态结构

![image-20230912025449040](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912025449040.png)

###### 客户端属性

- 通用属性

  - fd
    - 伪客户端的fd属性为-1
    - 普通客户端的fd属性的值 > -1 的整数
  - name
    - 默认情况下没有名字
    - CLIENT setname 可以为客户端设置一个名字
    - name 指向一个字符串对象
  - flags
    - 记录客户端的角色
    - 在主从复制时， 主是从的客户端， 从是主的客户端
      - REDIS_MASTER  主服务器
      - REDIS_SLAVE 从服务器
      - REDIS_LUA_CLIENT 伪客端
    - 记录客户端的状态
      - REDIS_MONITOR 标志表示客户端正在执行 MONITOR 命令。
      - REDIS_UNIX_ OCKET 标志表示服务器使用UNIX 套接字来连接客户端。
      -  REDIS_BLOCKED 标志表示客户端正在被 BRPOP、BLPOP 等命令阻塞。
      - REDIS_UNBLOCKED标志表示客户端已经从REDIS_BLOCKED标志所表示的阻塞状态脱离
      - REDIS_MULTI 表示客户端正在执行事务
      - REDIS_DIRTY_CAS 事务使用WATCH监视的数据库键已经被修改
      - 。。。。p166页
    - PUBSUB  和 SCRIPT LOAD 命令 修改客端状态（及时没有修改数据） 但也会 使用REDIS_FORCE_AOF
  - 输入缓冲区
    - 用于保存客户端发送的命令请求
    - 会根据输入内容动态地缩小或扩大，不能超过1GB， 否则服务器将关闭这个客户端

- ![image-20230912031046962](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912031046962.png)

  - 命令与命令参数

    - robj **argv；字符串对象
    - int argc；记录argv的数组长度
    - ![image-20230912031242246](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912031242246.png)

  - 命令实现函数

    - 根据argv[0] 在命令表中查找命令所对于的命令实现函数

    - 命令表是一个字典， 

      - 键是 SDS结构，保存命令名字

      - 值是redisCommand结构

        ![image-20230912031615845](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912031615845.png)

  - 输出缓冲区

    - 命令的回复会保存在客户端的输出缓冲区

    - 两个输出缓冲区

      - 一个大小固定

        - 保存长度较小的回复
        - char buf[REDIS_REPLY_CHUNK_BYTES];   默认值为16 * 1024
        - bufpos;  //已使用的字节数量

      - 一个大小可变

        - 太大

        - 链表来连接多个字符串对象

          ![image-20230912032100849](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912032100849.png)

  - 身份验证（仅在开启身份验证功能时才有用）

    - int authenticated；
    - 为 0 未通过身份验证， 除了 AUTH命令外, 其他命令会被拒绝
    - 1  已通过身份验证

  -  时间

    - time_t  ctime；  创建客户端的时间
    - time_t lastinteraction; 与服务器最后一次互动时间,可以计算空转时间
    - time-t obuf_soft_limit_reached_time； 输入缓冲区第一次到达软性现状的时间

**客户端的创建和关闭**

- 创建普通客户端， 在客户端使用connect函数连接到服务器时，为客户端自动创建客户端状态，最新一个是c3

![image-20230912032908005](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230912032908005.png)

- 关闭客户端
  - 客户端进程退出或被 kill
  - 向服务器发送带有不符合协议格式的命令请求
  - 成为CIIENT kill 的目标
  - 用户设置了 timeout 配置选项
  - 客户端的命令请求超过服务器输入缓冲区大小（1GB）
  - 服务器发送给这个客户端的命令回复超过了输出缓冲区的限制大小
    - 硬性限制：一旦超过，立即关闭
    - 软性限制：在一段时间 > 设定时长 一直超过软限制

**LUA脚本伪客户端**

- 服务器初始化时创建负责执行Lua脚本的redis伪客户端

**AOF文件伪客户端**

- 用于执行AOF文件的伪客户端

##### 13 服务器

- redis服务器负责与多个客户端建立网络连接，处理客户端发送来的请求没在数据库中保存客户端执行命令缠身改的数据
- 并通过资源管理器维护服务器自身的运转

###### 命令请求执行过程

`SET KEY VALUE`

- 客户端向服务器发送给命令请求 `SET KEY VALUE`
  - SET KEY VALUE  将命令转换成协议   *3\r\ns3\r\nSET\r\ns3r\nKEYr\ns5\r\nVALUE\r\n
- 服务器接收并处理客户端发送来的命令请求， `SET KEY VALUE` ，在数据库中进行操作，并产生命令回复ok
  - 读取命令请求
    - 连接套接字，因为客户端的写入变得可读
    - 读取协议格式的命令请求，保存在客户端状态的输入缓冲区里
    - 对命令请求进行分析，将命令参数及个数保存到客户端状态的argv和argc属性中
    - ![image-20230914104536712](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230914104536712.png)
    - ![image-20230914104548881](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230914104548881.png)
  - 命令执行器
    - 根据 argv[0] 参数，在命令表中查找指定的命令，并保存到客户端状态的cmd中
- 服务器将命令恢复 ok返回给客户端
  - 将命令回复保存到客户端的输出缓冲区，并为套接字关联命令回复处理器，使socketfd变为可写的状态，服务器执行命令回复处理器
  - 命令回复发送完毕后，回复处理器会清空客户端状态的输出缓冲区，为下一个命令请求做准备
- 客户端接受服务器返回的命令回复，并打印给用户看

###### serverCron 函数

- 默认每个100ms执行一次，并更新服务器时间

**更新服务器时间缓存**

- 这两个属性记录的时间进度并不高

```c
struct redisServer{
    //,,,
    //保存秒级精度的系统当前的unix时间戳
    time_t unixtime;
    
    //保存了毫秒级进度的系统当前unix时间戳
    long long mstime；
};
```

- 只用在 打印日志、更新服务器LRU适中、决定是否执行出久华任务，计算服务器上线时间这类许时间精度要求不高的功能上
- 为将设置过期时间、添加慢查日志这种需要高精度的时间，服务器还是会再次执行系统调用

**更新LRU时钟**

- INFO server 查看lru_clock 域

```c
struct redisServer{
    ...
    //默认每10秒更新一次时钟缓存
    //用于计算键的空转（idle） 时长
    unsigned lruclock : 22;
};

//每个redis对象都有一个lru属性，这个属性保存了对象最后一次被命令访问的时间
typedef struct redisObject{
    unsigned lru:22;
}robj;
```

- 键对应的值对象的空转时间 idle =  lruclock - lru  

**更新服务器每秒执行命令次数**

- serverCron 函数中的trackOperationsPerSecond 函数会每100毫秒一次的频率执行

- INFO status 查看instantaneous_ops_per_sec 城
  - trackOperationsPerSecond 函数每次运行，会计算出两次trackOperationsPerSecond调用之间 ，服务器平均每一豪秒钟内处理多少个命令请求的 ， 然后*1000 ，就得到一秒钟服务器能处理多少个命令请求的估计值，这个估值值被作为一个 新的数组项被放进ops_sec_samples中


```c

struct redisServer {
	...
	// 上一次进行抽样的时间
	long long ops_sec_last_sample_time;

	//上一次抽样时，服务器已执行命令的数量
    long long ops_sec_last_sample_ops;
    
	//REDIS_OPS_SEC_SAMPLES 大小(默认值为 16)的环形数组
    // 数组中的每个项都记录了一次抽样结果。
	long long ops_sec_samples[REDIS_OPS_SEC_SAMPLES];
    
	//ops_sec_samples 数胡的案引值
	// 每次抽样后将值自增一，
	//在值等于 16 时重置为 0.
	// 让ops sec samples 数组构成一个环形数组。
	int ops_sec_idx;
    ...
};
```

**更新服务器内存峰值记录**

- 服务器状态中stat_peak_memory

- 每次serverCron函数执行时，都会常看服务器器当前使用的内存数量

-  大于 stat_peak_memory，则stat_peak_memory =  当前的内存数量 

- INFO memory 

  - used_memory_peak;
  - used_memory_peak_human;

  ```c
  struct redisServer{
      
      //已使用内存峰值
      size_t stat_peak_memory;
  };
  ```

**处理SIGTERM信号**

- 在启动服务器时，会为服务器进程的SIGTERM信号关联处理器 sigtermHandler
- 当SIGTERM信号出现是，打开服务器状态的 shutdown_asap标识
- 每次serverCron函数执行，会检查shutdown_asap 标识， 1则关闭服务器， 0则不处理
- 服务器在关闭前会进行RDB持久化操作

**管理客户资源**

- serverCron 函数每次执行会调用clientsCron 函数，做以下检查
  - 客户端与服务器之间连接超时，则释放客户端
  - 输入缓冲区大小朝富哦一定长度，释放客户端当前的输入缓冲区，重建一个默认大小的

**管理数据库资源**

- serverCron 函数每次执行会调用cdatabasesCron函数
  - 删除其中的过期键
  - 在需要是对字典进行收缩操作

**执行被延迟的BGREWRITEAOF**

```c
struct redisServer{
    //如果值为1，那么表示有BGREWRITEAOF被延迟
    int aof_rewrite_scheduled;
};
```

- 先检查BGSAVE 或 BGREWRITEAOF 是否在执行
- 否 ，这执行 BGREWRITEAOF 

##### 

#### 多机数据库的实现

##### 14 复制

- SLAVEOF 

  ```c
  127.0.0.1:12345> SLAVEOF 127.0.0.1 6379
      
  // 那么服务器 127.0.0.1:12345 将成为 127.0.0.1:6379 的从服务器，而服务器127.0.0.1:6379 则会成为 127.0.0.1:12345 的主服务器。
  ```

- 

![image-20230919145142222](C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20230919145142222.png)









 





