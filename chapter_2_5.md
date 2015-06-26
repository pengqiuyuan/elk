elasticsearch 配置文件
-------------

####  10.0.29.107 master data
- 修改elasticsearch.yml文件elasticsearch-1.6.0/config目录下
- 添加templates模版

- 修改 elasticsearch-1.6.0/bin 目录下的 elasticsearch.in.sh

```  
  ES_MIN_MEM=256m  改为  物理内存的50%  ES_MIN_MEM=2g (机器32g内存这里分配16g)
  ES_MAX_MEM=1g    改为  物理内存的50%  ES_MIN_MEM=2g(机器32g内存这里分配16g)
```




####  10.0.29.249 node data


####  10.0.29.250  node data