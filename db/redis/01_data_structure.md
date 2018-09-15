### Redis相关数据结构底层实现

#### 1. SDS(simple dynamic string)
- 应用

- 数据结构
    ```
    struct sdshdr {

        // 记录buf数组中已使用字节的数量
        // 等于SDS所保存字符串的长度
        int len;

        // 记录buf数组中未使用字节的数量
        int free;

        // 字节数组, 用于保存字符串
        char[] buf;

    } sdshdr;
    ```

- 区别

    | C字符串 | SDS |
    |---|---|
    | 获取字符串长度的复杂度为O(N) | 获取字符串长度的复杂度为O(1) |
    | API不安全, 有可能缓冲区溢出 | API安全, 不会缓冲区溢出 |
    | 修改字符串长度N次必然需要执行N次内存重分配 | 修改字符串长度N次最多需要执行N次内存重分配 |
    | 只能保存文本数据 | 可以保存文本或者二进制数据 |
    | 可以使用所有<string.h>库中的函数 | 可以使用一部分<string.h>库中的函数 |

#### 2. 链表
- 应用

- 数据结构

    1. ListNode
        ```
        typedef struct listNode {

            // 前置节点
            struct listNode* prev;

            // 后置节点
            struct listNode* next;

            // 节点的值
            void* value;

        } listNode;
        ```
    2. List
        ```
        typedef struct list {

            // 表头节点
            listNode* head;

            // 表尾节点
            listNode* tail;

            // 链表所包含的节点数量
            unsigned long len;

            // 节点值复制函数
            void* (*dup)(void* ptr);

            // 节点值释放函数
            void (*free)(void* ptr);

            // 节点值对比函数
            int (*match)(void* ptr, void* key);

        } list;
        ```

#### 3. 字典
- 应用

- 数据结构

    1. dictht
        ```
        typedef struct dictht{

            // 哈希表数组
            dictEntry **table;

            // 哈希表大小
            unsigned long size;

            // 哈希表大小掩码, 用于计算索引值
            // 总是等于size-1
            unsigned long sizemask;

            // 该哈希表已有节点的数量
            unsigned long used;

        } dictht;
        ```

    2. dictEntry
        ```
        typedef struct dictEntry {

            // 键
            void *key;

            // 值
            union {
                void *val;
                uint64_tu64;
                int64_ts64;
            } v;

            // 指向下个哈希表节点, 形成链表
            struct dictEntry *next;

        } dictEntry;
        ```

    3. dict
        ```
        typedef struct dict {

            // 类型特定函数
            dictType *type;

            // 私有数据
            void *privdata;

            // 哈希表
            dictht ht[2];

            // rehash索引
            // 当rehash不在进行时, 值为-1
            int rehashidx; /* rehashing not in progress if rehashidx == -1 */

        } dict;
        ```

- 算法

    - 哈希值计算 - [MurmurHash2](https://blog.csdn.net/thinkmo/article/details/26833565)
    - 键冲突处理 - 拉链法
    - rehash

        - 取第一个大于等于ht\[0\].used*2或ht\[0\].used的2^n值
        - 渐进式(保证ht\[0\]的kv只减不增).
            1. rehash期间[INSERT]只在ht\[1\]执行
            2. rehash期间[DEL],[FIND],[UPDATE]在两个哈希表进行

- 负载因子

    - 计算方法: load_factor = ht\[0\].used / ht\[0\].size

- 哈希表扩展条件

    - 服务器目前没有执行BGSAVE或者BGREWRITEAOF命令, 且哈希表的负载因子大于等于1
    - 服务器目前正在执行BGSAVE或者BGREWRITEAOF命令, 且哈希表的负载因子大于等于5

#### 4. 跳跃表
- 应用

- 数据结构

    1. zskiplistNode
        ```
        typedef struct zskiplistNode {

            // 层
            struct zskiplistLevel {

                // 前进指针
                struct zskiplistNode *forward;

                // 跨度
                unsigned int span;

            } level[];

            // 后退指针
            struct zskiplistNode *backward;

            // 分值
            double score;

            // 成员对象
            robj *robj;

        } zskiplistNode
        ```

    2. zskiplist
        ```
        typedef struct zskiplist {

            // 表头节点和表尾节点
            struct zskiplistNode *header, *tail;

            // 表中节点的数量
            unsigned long length;

            // 表中层数最大的节点的层数
            int level;

        } zskiplist
        ```

#### 5. 整数集合
- 应用

- 数据结构
    ```
    typedef struct intset {

        // 编码方式
        uint32_t encoding;

        // 集合包含的元素数量
        uint32_t length;

        // 保存元素的数组
        int8_t contents[];

    } intset
    ```
- 问题
    - 是否可以包含多个不同级别编码的元素数组, 每个元素只使用刚好足够的编码, 这样子可以省去不少内存以及升级时间?
        例如:
        ```
        typedef struct betterintsetNode {

            // 编码方式
            uint32_t encoding;

            // 集合包含的元素数量
            uint32_t length;

            // 保存元素的数组
            void* contents[];

        } betterintsetNode

        typedef struct betterintset {

            // 数组下标对于一种编码方式, 按照占用内存大小从小到大排序
            // 若不存在对于编码的节点则留空
            // 如:
            // uint8_t对应下标1
            // uint16_t对应下标2
            struct betterintsetNode nodes[];

        } betterintset
        ```

#### 6. 压缩列表(待完善)
- 应用

- 压缩列表

    由一系列特殊编码的连续内存块组成的顺序型数据结构

    | 属性 | 类型 | 长度 | 用途 |
    |---|---|---|---|
    | zlbytes | uint32_t | 4字节 |  |
    | zltail | uint32_t | 4字节 |  |
    | zllen | uint16_t | 2字节 |  |
    | entryX | 列表节点 | 不定 |  |
    | zlend | uint8_t | 1字节 |  |

- 压缩列表节点(待完善)

    每个压缩列表节点可以保存一个字节数组或者一个整数值

    | 属性 | 类型 | 长度 | 用途 |
    |---|---|---|---|
    | previous_entry_length || 1或5字节 ||
    | encoding ||||
    | content ||||

- 连锁更新

#### 7. 对象
- 对象类型与编码

    - 数据结构
        ```
        typedef struct redisObject {

            // 类型
            unsigned type:4;

            // 编码
            unsigned encoding:4;

            // 指向底层实现数据结构的指针
            void *ptr;

            // ...
        } robj;
        ```
    - 对象类型

        | 类型常量 | 对象名称 |
        |---|---|
        | REDIS_STRING | 字符串对象 |
        | REDIS_LIST | 列表对象 |
        | REDIS_HASH | 哈希对象 |
        | REDIS_SET | 集合对象 |
        | REDIS_ZSET | 有序集合对象 |

    - 对象编码(待完善)

    - 内存回收(引用计数法)

    - 对象共享(只共享整数值字符串)

    - 对象空转时长
        ```
        typedef struct redisObject {

            // ...

            // 最后一次被访问的时间
            unsigned lru:22;

            // ...
        } robj;
        ```