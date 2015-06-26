### 服务器调整

**1. Linux调整文件数**

> /etc/security/limits.conf 在文件中增加
* soft nofile 65536
* hard nofile 65536

[参考文档](https://www.elastic.co/guide/en/elasticsearch/guide/current/_file_descriptors_and_mmap.html)


**2. 开放80端口**

> vi /etc/sysconfig/iptables
> - 添加： -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
> - 重启服务： service iptables restart

**3. nginx 配置 ，使用80端口代理到9200**

> 修改配置文件/usr/local/nginx/conf  
> - 使用vi编辑 vi nginx.conf 添加

```
server {
  listen       80;
  server_name  www.pengqiuy.com;
  access_log logs/es.access.log;
  error_log  logs/es.error.log info;
  location /port/{
      proxy_set_header    Host $http_host;
      proxy_set_header    X-Real-IP   $remote_addr;
      proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_redirect off;
      proxy_pass  http://127.0.0.1:9200/;
  }
  location /head/{
      proxy_set_header    Host $http_host;
      proxy_set_header    X-Real-IP   $remote_addr;
      proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_redirect off;
      proxy_pass  http://127.0.0.1:9200/_plugin/head/;
  }
  location /kopf/{
      proxy_set_header    Host $http_host;
      proxy_set_header    X-Real-IP   $remote_addr;
      proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_redirect off;
      proxy_pass  http://127.0.0.1:9200/_plugin/kopf/;
  }
  location /bigdesk/{
      proxy_set_header    Host $http_host;
      proxy_set_header    X-Real-IP   $remote_addr;
      proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_redirect off;
      proxy_pass  http://127.0.0.1:9200/_plugin/bigdesk/;
  }

}
```
>- sbin/nginx –t 测试
>- sbin/nginx –s reload





