logstash template 模版
-------------
```
修改elasticsearch-template.json文件
logstash-1.4.2/lib/logstash/outputs/elasticsearch目录下
复制参考配置，保存退出
```
```
{
  "template" : "logstash-*",
  "settings" : {
    "index.refresh_interval" : "15s"
  },
  "mappings" : {
    "_default_" : {
       "_all" : {"enabled" : true},
       "dynamic_templates" : [ {
         "string_fields" : {
           "match" : "*",
           "match_mapping_type" : "string",
           "mapping" : {
             "type" : "string", "index" : "not_analyzed", "omit_norms" : true,"doc_values": true,"ignore_unmapped" : true 
           }
         }
       } ],
       "properties" : {
         "@version": { "type": "string", "index": "not_analyzed" },
         "@timestamp": { "type": "date", "index": "not_analyzed", "doc_values": true, "format": "dateOptionalTime" },
         "geoip"  : {
           "type" : "object",
             "dynamic": true,
             "path": "full",
             "properties" : {
               "location" : { "type" : "geo_point" }
             }
             ,"doc_values": true,"ignore_unmapped" : true 
         },
         "当前在线人数" : { "type" : "integer" ,"doc_values": true,"ignore_unmapped" : true },
         "增加的体力数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "消耗的体力数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "增加的道具数量": { "type": "integer", "doc_values": true ,"ignore_unmapped" : true }, 
         "消耗的道具数量": { "type": "integer", "doc_values": true ,"ignore_unmapped" : true }, 
         "增加的充值币数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "减少的充值币数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "购买的商品数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "增加的虚拟币数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "减少的虚拟币数量" : { "type" : "integer" ,"doc_values": true,"ignore_unmapped" : true },
         "增加的游戏币数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true },
         "减少的游戏币数量" : { "type" : "integer", "doc_values": true,"ignore_unmapped" : true }
       }
    }
  }
}
```