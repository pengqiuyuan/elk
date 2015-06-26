运行logstash
------
**logstash shipper => redis日志收集，运行在日志所在服务器** 

```
以fb项目为例，进入 logstash-1.4.2目录
启动命令 nohup bin/logstash agent -f logstash-shipper /agent_fb.conf	&
```
```
input {   
 file {   
   type => "fb_game_log"   
   path => ["/Users/apple/Desktop/log/*/*user_*.log","/Users/apple/Desktop/log/*/*ap*.log","/Users/apple/Desktop/log/*/*coin*.log","/Users/apple/Desktop/log/*/*item*.log","/Users/apple/Desktop/log/*/*money*.log","/Users/apple/Desktop/log/*/*dummy*.log","/Users/apple/Desktop/log/*/*arenacoin*.log","/Users/apple/Desktop/log/*/*expeditioncoin*.log"]
   exclude => ["*.gz"]
   tags => ["fb"]   
      start_position => beginning
 }   
}
output {   
 redis {   
   host => "10.0.29.251"   
   data_type => "list"   
   key => "fb"
   workers => 6   
 }   
}
```

```
path ：为日志所在目录 ＊是通配符（这里启动的时候需要确定路径是否正确）
host ：为安装redis的服务器ip
查看shipper实例是否启动成功： tail －200f nohup.out
```


**logstash indexer => elasticsearch , 可以运行在redis所在服务器** 

```
以fb项目为例，进入 logstash-1.4.2目录
启动命令 nohup bin/logstash agent -f logstash- indexer /index_fb.conf	&
查看indexer实例是否启动成功： tail －200f nohup.out

```

```
input {   
 redis {   
   host => "10.0.29.251"   
   port => "6379"    
   data_type => "list"   
   key => "fb"
   codec => "json"
   threads => "6"
 }   
}

filter {

        mutate {
            split => ["message", "|"]
        }
	json {
            source => "[message][9]"
	}
	#userLog
	if "login" in [message][0]{
	        mutate {
		        replace => { "type" => "fb_user.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "isfirst", "是否首次登录" ]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "logintype", "登录设备系统" ]
			rename => [ "create_time", "注册时间"]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
		}
		geoip {
			source => "玩家登陆ip"
			fields => ["country_name", "region_name", "city_name", "real_region_name", "latitude", "longitude"]
			remove_field => [ "[geoip][longitude]", "[geoip][latitude]" ]
		}
	}else if "create" in [message][0]{
	        mutate {
		        replace => { "type" => "fb_user.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "createuser", "索引" ]
			rename => [ "role_id", "创建角色游戏ID" ]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "from", "注册渠道" ]

		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
		}
		geoip {
			source => "玩家登陆ip"
			fields => ["country_name", "region_name", "city_name", "real_region_name", "latitude", "longitude"]
			remove_field => [ "[geoip][longitude]", "[geoip][latitude]" ]
		}
	}else if "online" in [message][0]{
       		json {
        	    source => "[message][6]"
	        }
		mutate {
		    replace => { "type" => "fb_user.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			rename => [ "number", "当前在线人数"]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["当前在线人数","integer"]
		}
	}else if "newbieguide" in [message][0]{
               mutate {
                        replace => { "type" => "fb_user.log" }
                        add_field => [ "日志分类关键字", "%{[message][0]}"]
                        add_field => [ "日期", "%{[message][1]}"]
                        add_field => [ "游戏ID", "%{[message][2]}"]
                        add_field => [ "运营大区ID", "%{[message][3]}"]
                        add_field => [ "渠道ID", "%{[message][4]}"]
                        add_field => [ "服务器ID", "%{[message][5]}"]
                        add_field => [ "玩家平台ID", "%{[message][6]}"]
                        add_field => [ "玩家GUID", "%{[message][7]}"]
                        add_field => [ "玩家名字", "%{[message][8]}"]
						rename => [ "role_id", "角色ID" ]
                        rename => [ "role_level", "角色等级" ]
                        rename => [ "guide_id", "功能编号" ]
              }
	      mutate {
			remove_field => [ "message","host","path","tags" ]
	      }
        }
	#apLog
	else if "ap_get" in [message][0]{
		mutate {
		    replace => { "type" => "fb_ap.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "item_id", "日志道具id" ]
			rename => [ "get_path", "获得途径" ]
			rename => [ "get_point", "二级获得途径" ]
			rename => [ "ap_before", "增加前体力数量" ]
			rename => [ "ap_after", "增加后体力数量" ]
			rename => [ "ap_mount", "增加的体力数量" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
			convert => ["增加的体力数量","integer"]
		}
	}else if "ap_cost" in [message][0]{
		mutate {
		    replace => { "type" => "fb_ap.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "item_id", "日志道具id" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "ap_before", "消耗前体力数量" ]
			rename => [ "ap_after", "消耗后体力数量" ]
			rename => [ "ap_reduce", "消耗的体力数量" ]	
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]	
			convert => ["消耗的体力数量","integer"]
		}

	}
        #itemLog
	else if "item_get" in [message][0]{
		mutate {
		    replace => { "type" => "fb_item.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "item_id", "日志道具id" ]
			rename => [ "get_path", "获得途径" ]
			rename => [ "get_point", "二级获得途径" ]
			rename => [ "item_before", "增加前道具数量" ]
			rename => [ "item_after", "增加后道具数量" ]
			rename => [ "item_mount", "增加的道具数量" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
			convert => ["增加的道具数量","integer"]
		}
	}else if "item_cost" in [message][0]{
		mutate {
		        replace => { "type" => "fb_item.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "item_id", "日志道具id" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "item_before", "消耗前道具数量" ]
			rename => [ "item_after", "消耗后道具数量" ]
			rename => [ "item_reduce", "消耗的道具数量" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
			convert => ["消耗的道具数量","integer"]
		}
	}

	#moneyLog
	else if "money_get" in [message][0]{
		mutate {
		        replace => { "type" => "fb_money.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "order_id", "订单号" ]
			rename => [ "pay_count", "支付金额" ]
			rename => [ "currency", "支付货币" ]
			rename => [ "money_before", "增加前充值币数量" ]
			rename => [ "money_after", "增加后充值币数量" ]
			rename => [ "money_mount", "增加的充值币数量" ]
			rename => [ "vip_level", "充值后的vip等级" ]
			rename => [ "vip_exp", "充值后的vip经验" ]
			rename => [ "create_time", "注册时间"]
			rename => [ "is_first", "是否首次充值"]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["支付金额","float","增加的充值币数量","integer"]
		}
                geoip {
                        source => "玩家登陆ip"
                        fields => ["country_name", "region_name", "city_name", "real_region_name", "latitude", "longitude"]
                        remove_field => [ "[geoip][longitude]", "[geoip][latitude]" ]
                }
	}else if "money_cost" in [message][0]{
		mutate {
		        replace => { "type" => "fb_money.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "money_before", "减少前充值币数量" ]
			rename => [ "money_after", "减少后充值币数量" ]
			rename => [ "money_reduce", "减少的充值币数量" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "goods_sum", "购买的商品数量" ]
			rename => [ "goods_price", "商品打折前单价" ]
			rename => [ "discount_price", "商品打折后单价" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["减少的充值币数量","integer","购买的商品数量","integer"]
		}
		                geoip {
                        source => "玩家登陆ip"
                        fields => ["country_name", "region_name", "city_name", "real_region_name", "latitude", "longitude"]
                        remove_field => [ "[geoip][longitude]", "[geoip][latitude]" ]
                }
	}
    #dummyLog
	else if "dummy_get" in [message][0]{
		mutate {
		    replace => { "type" => "fb_dummy.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "dummy_before", "增加前虚拟币数量" ]
			rename => [ "dummy_after", "增加后虚拟币数量" ]
			rename => [ "dummy_mount", "增加的虚拟币数量" ]
			rename => [ "get_path", "获得途径" ]
			rename => [ "get_point", "二级获得途径" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["增加的虚拟币数量","integer"]
		}

	}else if "dummy_cost" in [message][0]{
		mutate {
		    replace => { "type" => "fb_dummy.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "dummy_before", "减少前虚拟币数量" ]
			rename => [ "dummy_after", "减少后虚拟币数量" ]
			rename => [ "dummy_reduce", "减少的虚拟币数量" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "goods_sum", "购买的商品数量" ]
			rename => [ "goods_price", "商品打折前单价" ]
			rename => [ "discount_price", "商品打折后单价" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["减少的虚拟币数量","integer","购买的商品数量","integer"]
		}
	}
	#arenacoin
	else if "arenacoin_get" in [message][0]{
		mutate {
		    replace => { "type" => "fb_arenacoin.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "coin_before", "增加前游戏币数量" ]
			rename => [ "coin_after", "增加后游戏币数量" ]
			rename => [ "coin_mount", "增加的游戏币数量" ]
			rename => [ "get_path", "获得途径" ]
			rename => [ "get_point", "二级获得途径" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["增加的游戏币数量","integer"]
		}
	}else if "arenacoin_cost" in [message][0]{
		mutate {
		    replace => { "type" => "fb_arenacoin.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "coin_before", "减少前游戏币数量" ]
			rename => [ "coin_after", "减少后游戏币数量" ]
			rename => [ "coin_reduce", "减少的游戏币数量" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "goods_sum", "购买的商品数量" ]
			rename => [ "goods_price", "商品打折前单价" ]
			rename => [ "discount_price", "商品打折后单价" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["减少的游戏币数量","integer","购买的商品数量","integer"]
		}
	}
	#expeditioncoin
	else if "expeditioncoin_get" in [message][0]{
		mutate {
		    replace => { "type" => "fb_expeditioncoin.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "coin_before", "增加前游戏币数量" ]
			rename => [ "coin_after", "增加后游戏币数量" ]
			rename => [ "coin_mount", "增加的游戏币数量" ]
			rename => [ "get_path", "获得途径" ]
			rename => [ "get_point", "二级获得途径" ]
		}
		mutate {
			remove_field => ["message","host","path","tags" ]
            convert => ["增加的游戏币数量","integer"]
		}
	}else if  "expeditioncoin_cost" in [message][0]{
		mutate {
		    replace => { "type" => "fb_expeditioncoin.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "coin_before", "减少前游戏币数量" ]
			rename => [ "coin_after", "减少后游戏币数量" ]
			rename => [ "coin_reduce", "减少的游戏币数量" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "goods_sum", "购买的商品数量" ]
			rename => [ "goods_price", "商品打折前单价" ]
			rename => [ "discount_price", "商品打折后单价" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["减少的游戏币数量","integer","购买的商品数量","integer"]
		}
	}
	#coinLog
	else if "coin_get" in [message][0] and !("arenacoin_get" in [message][0]) and !("expeditioncoin_get" in [message][0]){
		mutate {
		        replace => { "type" => "fb_coin.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "coin_before", "增加前游戏币数量" ]
			rename => [ "coin_after", "增加后游戏币数量" ]
			rename => [ "coin_mount", "增加的游戏币数量" ]
			rename => [ "get_path", "获得途径" ]
			rename => [ "get_point", "二级获得途径" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["增加的游戏币数量","integer"]
		}
	}else if "coin_cost" in [message][0] and !("arenacoin_cost" in [message][0]) and !("expeditioncoin_cost" in [message][0]) {
		mutate {
		        replace => { "type" => "fb_coin.log" }
			add_field => [ "日志分类关键字", "%{[message][0]}"]
			add_field => [ "日期", "%{[message][1]}"]
			add_field => [ "游戏ID", "%{[message][2]}"]
			add_field => [ "运营大区ID", "%{[message][3]}"]
			add_field => [ "渠道ID", "%{[message][4]}"]
			add_field => [ "服务器ID", "%{[message][5]}"]
			add_field => [ "玩家平台ID", "%{[message][6]}"]
			add_field => [ "玩家GUID", "%{[message][7]}"]
			add_field => [ "玩家名字", "%{[message][8]}"]
			rename => [ "level", "人物等级" ]
			rename => [ "ip", "玩家登陆ip" ]
			rename => [ "coin_before", "减少前游戏币数量" ]
			rename => [ "coin_after", "减少后游戏币数量" ]
			rename => [ "coin_reduce", "减少的游戏币数量" ]
			rename => [ "cost_path", "消耗途径" ]
			rename => [ "cost_point", "二级消耗途径" ]
			rename => [ "goods_sum", "购买的商品数量" ]
			rename => [ "goods_price", "商品打折前单价" ]
			rename => [ "discount_price", "商品打折后单价" ]
		}
		mutate {
			remove_field => [ "message","host","path","tags" ]
            convert => ["减少的游戏币数量","integer","购买的商品数量","integer"]
		}
	}	

}


output {
	if [type] == "fb_user.log"{
		  elasticsearch {
		    port => 9300
		    cluster => "guangxian_game"
		    host => ["10.0.29.152"]
		    index => "logstash-fb-user-%{+YYYY.MM}"
		    workers => 1
		  }
	}
	else if [type] == "fb_money.log"{
		  elasticsearch {
		    port => 9300
		    cluster => "guangxian_game"
		    host => ["10.0.29.152"]
		    index => "logstash-fb-money-%{+YYYY.MM}"
		    workers => 1
		  }
	}
	elasticsearch {
		port => 9300
		cluster => "guangxian_game"
		host => ["10.0.29.152"]
		index => "logstash-fb-all-%{+YYYY.MM}"
		workers => 4
	}
}
```



```
redis host : 安装redis，收集日志的服务器ip
cluster ： elasticsearch 集群名称，与elasticsearch.yml中配置的匹配，不用动
host ：elasticsearch master data 实例，所在服务器的ip
```