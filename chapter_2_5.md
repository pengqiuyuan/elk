elasticsearch 配置文件
-------------

####  10.0.29.107 master data
- 修改elasticsearch.yml文件elasticsearch-1.6.0/config目录下
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


####  10.0.29.250  node data