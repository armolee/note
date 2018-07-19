###Kafka集群的迁移
* 从业务代码中提取所有在用的的topic  
* 将所有在用topic生成迁移json，包含kafka偏移量topic：__consumer_offsets  ，保存在move_topic.json中
```bash 
{"topics": [ {"topic": "__consumer_offsets"},
            {"topic": "1000"},
            {"topic":"1001"},
            ...
            {"topic":"29002"}],
"version":1
}
```
* 使用move_topic.json生成迁移计划
```bash
bin/kafka-reassign-partitions.sh --zookeeper zknode1:2181,zknodeq:2181,zknodew:2181 --topics-to-move-json-file move_topic.json --broker-list "1,2,3,4" --generate
#broker-list "1,2,3,4" 为迁移完分配的broker id
#将该命令输出的结果Proposed partition reassignment configuration后的输出保存到result.json
```
* 使用result.json执行迁移动作
```bash
bin/kafka-reassign-partitions.sh --zookeeper  zknode1:2181,zknodeq:2181,zknodew:2181  --reassignment-json-file result.json --execute
```
* 查看迁移进度
```bash
bin/kafka-reassign-partitions.sh --zookeeper  zknode1:2181,zknodeq:2181,zknodew:2181  --reassignment-json-file result.json --verify
```
* 平衡leader
```bash
bin/kafka-preferred-replica-election.sh --zookeeper   zknode1:2181,zknodeq:2181,zknodew:2181 
```
* 检查所有topic是否正常工作
* 修改php程序控制连接kafka broker的ip地址列表
* 下线旧broker，直接停止kafka进程
