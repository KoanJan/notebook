### 单机数据库实现原理

#### 过期键删除原理
- 存储
    - 过期字典, 其中key是对象指针, value是ms精度的UNIX时间戳
- 策略
    - 惰性删除
    - 定期删除

#### RDB持久化
- 通过保存键值对数据(全量备份)
- 策略
    - 手动执行命令(SAVE/BGSAVE)
    - 配置自动执行

- RDB文件结构

    | 部分 | 含义 |
    |---|---|
    | REDIS | REDIS |
    | db_version | 文件版本号 |
    | databases |  |
    | EOF | 正文内容的结束 |
    | check_sum | 校验和 |

    | databases内各部分 | 含义 |
    |---|---|
    | SELECTDB | SELECTDB常量 |
    | db_number | 数据库号码 |
    | key_value_pairs | 键值对 |

    | key_value_pairs内各部分 | 含义 |
    |---|---|
    | TYPE | 记录value类型 |
    | key | 键对象, REDIS_RDB_TYPE_STRING编码 |
    | value | 值对象, 结构各不相同 |

#### AOF持久化
- 通过保存执行命令(增量备份)
- 相关命令(BGREWRITEAOF)
- AOF重写

#### 事件(待完善)