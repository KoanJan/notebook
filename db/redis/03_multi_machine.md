### 多机数据库的实现

#### Replication
- 相关命令
    1. SLAVEOF $m_host $m_port
    2. PING
    3. AUTH $pass
    4. REPLCONF listening-port $s_port
    5. PSYNC
    6. (命令传播期间心跳检测)
- 模式(完整重同步/部分重同步)
- 相关概念
    - 复制偏移量
    - 复制积压缓冲区(固定长度1MB的FIFO队列)
    - 服务器运行ID
- 心跳检测
    - 命令: REPLCONF ACK <replication_offset>
    - 用途
        1. 检测主从服务器的网络连接状态
        2. 辅助实现min-slaves选项
        3. 检测命令丢失

#### Sentinel
- 主逻辑
    - 当master节点下线超过一定时间时, sentinel节点提升该master下的某一slave为新的master
- 判断
    - 主观下线
    - 客观下线
- 选举领头Sentinel
    - 先到先得
    - 局部领头/超过半数
- 故障转移
    1. 删除slave列表中网络通信不佳的slave
    2. 删除slave列表中数据过旧的slave
    3. 按照服务器优先级排序, 选取最高优先级的slave作为新master
    4. (旧master若恢复则作为新master的slave)

#### Cluster
- 相关概念
    - slot
    - 重新分片
    - ASK错误/MOVED错误