运行elasticsearch
----------
- 1、集群 先启动master节点 ，其他节点逐步启动并查看

```
cd elasticsearch-1.6.0 目录下执行
nohup bin/elasticsearch & 
查看es实例是否启动成功
tail －200f nohup.out
```

- 2、单台机器 启动master节点

```
cd elasticsearch-1.6.0 目录下执行
nohup bin/elasticsearch & 
查看es实例是否启动成功
tail －200f nohup.out
```