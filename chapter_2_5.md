elasticsearch 配置文件
-------------

####  10.0.29.107 master data
- 修改elasticsearch.yml文件elasticsearch-1.6.0/config目录下

     ``` 
    cluster.name: guangxian_game
    node.name: node_107
    node.master: true
    node.data: true
    index.number_of_shards: 3
    index.number_of_replicas: 1
    index.cache.field.type: soft
    index.cache.field.max_size: 50000
    index.cache.field.expire: 10m

    transport.tcp.compress: true
    http.cors.allow-origin: "/.*/"
    http.cors.enabled: true
    
    bootstrap.mlockall: true
    indices.cache.filter.size: 20%
    indices.fielddata.cache.size: 30%
    indices.fielddata.cache.expire: -1
    indices.memory.index_buffer_size: 50%
    index.refresh_interval: 30s
    index.translog.flush_threshold_ops: 50000
    index.store.compress.stored: true
    
    discovery.zen.ping.multicast.enabled: false
    discovery.zen.ping.unicast.hosts: ["10.0.29.152","10.0.29.107"]
 ``` 
 > node.name 节点名字，如当前机器是1o.0.29.107 ,名字为 node_107
 > discovery.zen.ping.unicast.hosts: 为master data的IP地址，如果只有一台为当前机器的ip


- 添加templates模版
``` 
elasticsearch-1.6.0/config目录下，创建文件夹templates,
mkdir templates
cd templates
templates文件夹下创建log_*.json文件
vi log_*.json  复制参考配置，保存退出
``` 
[参考地址](https://github.com/pengqiuyuan/escloud/blob/master/es-template/log_*.json) 

    ```
{
  "template" : "log_*",
  "settings" : {
    "index.refresh_interval" : "50s"
  },
  "mappings" : {
    "_default_" : {
       "_all" : {"enabled" : true},
       "dynamic_templates" : [ {
         "string_fields" : {
           "match" : "*",
           "match_mapping_type" : "string",
           "mapping" : {
             "type" : "string", "index" : "not_analyzed", "omit_norms" : true,"ignore_unmapped" : true
           }
         }
       } ],
       "properties" : {
         "@timestamp": { "type": "date", "index": "not_analyzed", "doc_values": true, "format": "dateOptionalTime","ignore_unmapped" : true},
         "date" : { "type" : "date", "index" : "not_analyzed","ignore_unmapped" : true}
       }
    }
  }
}
    ```
- 修改 elasticsearch-1.6.0/bin 目录下的 elasticsearch.in.sh

  ```  
  ES_MIN_MEM=256m  改为  物理内存的50%  ES_MIN_MEM=2g (机器32g内存这里分配16g)
  ES_MAX_MEM=1g    改为  物理内存的50%  ES_MIN_MEM=2g(机器32g内存这里分配16g)
  ```
   ```
    ES_CLASSPATH=$ES_CLASSPATH:$ES_HOME/lib/elasticsearch-1.6.0.jar:$ES_HOME/lib/*:$ES_HOME/lib/sigar/*
    
    if [ "x$ES_MIN_MEM" = "x" ]; then
        ES_MIN_MEM=512m
    fi
    if [ "x$ES_MAX_MEM" = "x" ]; then
        ES_MAX_MEM=512m
    fi
 ```



####  10.0.29.249 node data
- 修改elasticsearch.yml文件elasticsearch-1.6.0/config目录下
- 添加templates模版
- 修改 elasticsearch-1.6.0/bin 目录下的 elasticsearch.in.sh
####  10.0.29.250  node data
- 修改elasticsearch.yml文件elasticsearch-1.6.0/config目录下
- 添加templates模版
- 修改 elasticsearch-1.6.0/bin 目录下的 elasticsearch.in.sh