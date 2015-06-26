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
