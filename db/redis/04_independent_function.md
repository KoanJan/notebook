### 独立功能的实现

#### Pub/Sub
- 相关命令
    - PUBLISH 发布
    - SUBSCRIBE/UNSUBSCRIBE 订阅/退订
    - PSUBSCRIBE/PUNSUBSCRIBE 订阅/退订(模式)
    - PUBSUB 查看相关信息

#### Transaction
- 相关命令(MULTI/EXEC/WATCH)
- 实现
    1. 事务开始
    2. 命令入队
    3. 事务执行
- ACID

#### Lua Script
- 相关命令(EVAL/EVALSHA)

#### Sort
- 原理
    1. 创建redisSortObject结构数组, 保存原各元素指针 
    2. 排序
    3. 遍历数组, 将各数组项的obj指针所指向的列表项作为排序结果返回给客户端
- 子选项的实现
    - ALPHA
    - BY \<pattern\>
    - ASC/DESC
    - LIMIT \<how-many-skip\> \<how-many-return\>
    - GET \<pattern\>
    - STORE \<list\>
- 子选项的执行与摆放顺序(待完善)    

#### Bit Array
- 相关命令(GETBIT/SETBIT/BITCOUNT/BITOP)
- 数据结构(利用SDS逆序存储数据实现)
- BITCOUNT实现
    - 查表法
    - variable-precision SWAR算法

#### Slow Log

#### Monitor