---
title: ElasticSearch学习总结
date: 2018-06-30 15:10:30
categories:
   - 技术相关
tags:
   - ElasticSearch
   - 笔记
#summary_img: /images/material-xx.png
---


## 0x01 简介
	ElasticSearch是一个分布式、Restful的搜索及分析服务器,设计用于分布式计算；能够达到实时搜索,稳定,可靠,快速。
	和Apache Solr一样,它也是基于Lucence的索引服务器。2013年初,GitHub抛弃了Solr,采取ElasticSearch 来做PB级的搜索


<!-- more -->

## 0x02 概念

	Cluster和Node
		ES可以以单点或者集群方式运行,以一个整体对外提供search服务的所有节点组成cluster,组成这个cluster的各个节点叫做node。
	Index
		这是ES存储数据的地方,类似于关系数据库的database。
	Shards
		索引分片,这是ES提供分布式搜索的基础,其含义为将一个完整的index分成若干部分存储在相同或不同的节点上,这些组成index的部分就叫做shard。
	Replicas
		索引副本,ES可以设置多个索引的副本,副本的作用一是提高系统的容错性,当个某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高ES的查询效率,ES会自动对搜索请求进行负载均衡。
	Recovery
		代表数据恢复或叫数据重新分布,ES在有节点加入或退出时会根据机器的负载对索引分片进行重新分配,挂掉的节点重新启动时也会进行数据恢复。
	Gateway
		ES索引快照的存储方式,ES默认是先把索引存放到内存中,当内存满了时再持久化到本地硬盘。gateway对索引快照进行存储,当这个ES集群关闭再重新启动时就会从gateway中读取索引备份数据。
	Discovery.zen
		代表ES的自动发现节点机制,ES是一个基于p2p的系统,它先通过广播寻找存在的节点,再通过多播协议来进行节点之间的通信,同时也支持点对点的交互。
	Transport
		代表ES内部节点或集群与客户端的交互方式,默认内部是使用tcp协议进行交互,同时它支持http协议json格式。
		
## 0x03 ES索引优化
	一、索引数据过程
		ES索引的过程到相对Lucene的索引过程多了分布式数据的扩展,而这ES主要是用tranlog进行各节点之间的数据平衡
		
			index.translog.flush_threshold_ops: 500000
			index.refresh_interval: -1
		
	二、检索过程
		1、分片数
		
			分片数,与检索速度非常相关的的指标,如果分片数过少或过多都会导致检索比较慢
			分片数过多会导致检索时打开比较多的文件别外也会导致多台服务器之间通讯。而分片数过少会导致单个分片索引过大,所以检索速度慢。
			
		2、副本数
		
			如果Node在非正常挂了,经常会导致分片丢失,为了保证这些数据的完整性,可以通过副本来解决这个问题
			
## 0x04 ElasticSearch常用配置及性能参数

	bootstrap.memory_lock: true # 	锁定内存,不让JVM写入swapping,避免降低ES的性能 。
	http.cors.enabled: true		# 	使用监控 例如head
    http.cors.allow-origin: "*" #   和上面那个一起开启,用于插件head
	cluster.name: new-elk   	#	集群名称, 如果在同一网段下有多个集群,就可以用这个属性来区分不同的集群。
	node.name: "test"  			#	节点名称
	node.master: true  			#	是否主节点
	node.data: true   			#	是否存储数据
	
	index.number_of_shards: 5			#	索引分片数,默认为5片
	index.number_of_replicas: 1			#	索引副本数,默认为1个副本
	path.data: /data/elasticsearch/data	#	数据目录存放位置
	path.logs: /data/elasticsearch/log	#	日志数据存放位置
	index.cache.field.max_size: 500000	#	索引缓存
	index.cache.field.expire: 5m		#	索引缓存过期时间
	
	
	一个Elasticsearch节点会有多个线程池,但重要的是下面四个(默认都是cached类型):
		索引(index) 	:	主要是索引数据和删除数据操作
		搜索(search)	:	主要是获取,统计和搜索操作
		批量操作(bulk)	:	主要是对索引的批量操作
		更新(refresh)	:	主要是更新操作
		
	thread pool setting
	threadpool.index.type: fixed 		#	写索引线程池类型 
	threadpool.index.size: 64 			#	线程池大小(建议2~3倍cpu数) 
	threadpool.index.queue_size: 1000	#	 队列大小

	threadpool.search.size: 64 			#	搜索线程池大小 
	threadpool.search.type: fixed 		#	搜索线程池类型 
	threadpool.search.queue_size: 1000 	#	队列大小

	threadpool.get.type: fixed 			#	取数据线程池类型 
	threadpool.get.size: 32 			#	取数据线程池大小 
	threadpool.get.queue_size: 1000 	#	队列大小

	threadpool.bulk.type: fixed 		#	批量请求线程池类型 
	threadpool.bulk.size: 32 			#	批量请求线程池大小 
	threadpool.bulk.queue_size: 1000 	#	队列大小

	threadpool.flush.type: fixed 		#	刷磁盘线程池类型 
	threadpool.flush.size: 32 			#	刷磁盘线程池大小 
	threadpool.flush.queue_size: 1000 	#	队列大小
	
	indices.store.throttle.type: merge 
	indices.store.throttle.type: none 				#	写磁盘类型 
	indices.store.throttle.max_bytes_per_sec:500mb 	#	写磁盘最大带宽

	index.merge.scheduler.max_thread_count: 8 		#	索引merge最大线程数 
	index.translog.flush_threshold_size:600MB 		#	刷新translog文件阀值

	cluster.routing.allocation.node_initial_primaries_recoveries:8 	#	并发恢复分片数 
	cluster.routing.allocation.node_concurrent_recoveries:2 		#	同时recovery并发数

	使用bulk API 增加入库速度 
	初次索引的时候,把 replica 设置为 0 
	增大 threadpool.index.queue_size 1000 
	增大 indices.memory.index_buffer_size: 20% 
	index.translog.durability: async –这个可以异步写硬盘,增大写的速度 
	增大 index.translog.flush_threshold_size: 600MB 
	增大 index.translog.flush_threshold_ops: 500000
	
	change java lvm (jvm.options):
		Xms8g
		Xmx8g

	elasticsearch内存设置:
	
		export ES_HEAP_SIZE=30g   (linux)
		env  ES_HEAP_SIZE 30g   (windows)    ===  Xms30g Xmx30g
		
	官方建议
	
		a. heap size不要超过系统可用内存的一半,默认 1G,不要超过32GB
		b. 将xms和xmx设置成和heap一样大小,避免动态分配heap size
	
	ES对于内存的消耗,和很多因素相关,诸如数据总量、mapping设置、查询方式、查询频度等。
	ES是JAVA应用,作为一个JAVA应用,就脱离不开JVM和GC经常出现关于这两者的错误,
	上手ES的时候,对GC一点概念都没有就去网上抄各种JVM"优化"参数,却仍然被heap不够用,内存溢出这样的问题
	  
## 0x05 Elasticsearch线程池介绍
	curl -XGET 'http://localhost:9200/_nodes/stats?pretty'
	......

## 0x06 ElasticSearch 我的小結

### 命令行模式导入elastic
	命令行导入elastic(因为PDI(kettle)本身有900M,打开需要一定的内存,而且导入的时候还卡,所以使用命令行模式,速度快了10倍以上):

	a. 先在PDI里面设置好ktr文件
	b. 创建bat文件
	
### csv文件导入到elastic
	因为kettle的原因,直接通过csv文件导入到elastic里面是报错的(id是必须提供),所以要加一个字段ID,而且不能重复(自增)。
	其实kettle里面有个插件ADD QEUEUxx,在传输的时候通过这个插件就可以了,之后在右边选择valuename(插件默认名字)
	
### 日期/string类型
	因为某个库里面含有string类型的字段(生日-BIRTHDATE : 2011-03-22),这使得在PDI(kettle)里面映射为字符串类型,
但是导入到elastic时,会默认自动转为日期类型,这使得库里面字段(生日-BIRTHDATE )还有其他不是日期日期类型的(XXX字符串),转换不了日期类型,从而报错。
所以不能使用kettle默认创建索引,而是先使用curl创建索引及字段 : 

	curl -XPUT "http://9.9.9.10:8200/sbbb-typejp'
	curl -XPOST "http://9.9.9.10:8200/sbbb-typejp/sbbb-typejp/_mapping?pretty" -d '
	{
		"sbbb-typejp":{
			"properties":{
				"邮箱-EMAIL":{
					"type":"string"
				},
				"用户名-USERNAME":{
					"type":"string"
				},
				"mwmm-ps":{
					"type":"string"
				},
				"生日-BIRTHDATE":{
					"type":"string"
				}
			}
		}
	}'



即:
	
	curl -XPOST "http://9.9.9.10:8200/sbbb-typejp/sbbb-typejp/_mapping?pretty" -d '{"sbbb-test-2021dlh":{"properties":{"邮箱-EMAIL":{"type":"string"},"用户名-USERNAME":{"type":"string"},"mwmm-ps":{"type":"string"},"生日-BIRTHDATE":{"type":"string"}}}}'

之后在双击bat文件就好

	curl -XGET "http://9.9.9.10:8200/sbbb-typejp/_mapping?pretty" 查看mapping

当然可以关闭该功能:修改mapping设置  'date-detection' => false 即可

	curl -XPUT "http://9.9.9.10:8200/sbbb-typejp
	{
		"mappings":{
			"sbbb-typejp":{
				"date-detection":false
			}
		}
	}
	
### 安装plugin

	a. elasticdump
        Linux : 
            wget http://nodejs.org/dist/v9.2.0/node-v9.2.0-linux-x64.tar.gz
            tar -zxvf node-v9.2.0-linux-x64.tar.gz
            sudo mv node-v9.2.0-linux-x64 /opt/
            sudo ln -s /opt/node-v9.2.0-linux-x64/bin/node /usr/local/bin/node		
            sudo ln -s /opt/node-v9.2.0-linux-x64/bin/npm /usr/local/bin/npm

        Windows : 
            https://nodejs.org/en/download/
            node -v
            v6.10.3
            npm -v
            3.10.10
		Install:
            npm install elasticdump  (在外面安装好elasticdump,之后打包到没网的linux/windows上(node_modules文件夹下),运行 根目录运行npm link)
	
	b. elasticsearch-head
		https://github.com/mobz/elasticsearch-head
		cd elasticsearch-head
		npm install (在外面安装好,之后打包到没网的linux/windows上(任意目录))
		npm run start
		open http://localhost:9100/
	c. x-pack
        kibana x-pack :
            https://artifacts.elastic.co/downloads/kibana-plugins/x-pack/x-pack-5.6.3.zip
            
        elasticsearch x-pack :
            https://artifacts.elastic.co/downloads/elasticsearch-plugins/x-pack/x-pack-5.6.3.zip

        C:\kibana-5.6.3-windows-x86> bin\kibana-plugin.bat install file:///C:\Users\TuT\Desktop\x-pack-1years\kibana__x-pack-5.6.3.zip
        C:\elasticsearch-5.6.3> bin\elasticsearch-plugin.bat install file:///C:\Users\TuT\Desktop\x-pack-1years\elasticsearch__x-pack-5.6.3.zip

        但是免费的无法使用Graph (https://www.elastic.co/subscriptions)
        所以得破解X-Pack 
        1. 反编译 X-Pack : 
            使用 [Luyten](https://github.com/deathmarine/Luyten/releases/download/v0.5.3/luyten-0.5.3.jar) 进行反编译
        将org.elasticsearch/license/LicenseVerifier.class反编译并保存出来
        这个类是检查license完整性的类,我们使其始终返回true,就可以任意修改license并导入。将其改为:
            package org.elasticsearch.license;
            import java.nio.*;
            import java.util.*;
            import java.security.*;
            import org.elasticsearch.common.xcontent.*;
            import org.apache.lucene.util.*;
            import org.elasticsearch.common.io.*;
            import java.io.*;

            public class LicenseVerifier
            {
                public static boolean verifyLicense(final License license, final byte[] encryptedPublicKeyData) {
                    return true;
                }
                public static boolean verifyLicense(final License license) {
                    return true;
                }
            }
        然后需要重新编译class文件。注意这里我们无需编译整个工程,将原来的x-pack-5.6.3.jar和依赖包加入CLASSPATH,即可完成单个文件的编译。实际上只用到了3个依赖包
            javac -cp "C:\Users\TuT\Desktop\elasticsearch-5.6.3\lib\elasticsearch-5.6.3.jar:C:\Users\TuT\Desktop\elasticsearch-5.6.3\lib\lucene-core-6.6.1.jar:C:\Users\TuT\Desktop\elasticsearch-5.6.3\plugins\x-pack\x-pack-5.6.3.jar" LicenseVerifier.java
        Windows 无法编译因为无法识别两个冒号:C:,确定不了那个是路径(我猜)
        放到kali里 运行 : 
            javac -cp "/root/Desktop/license/elasticsearch-5.6.3.jar:/root/Desktop/license/lucene-core-6.6.1.jar:/root/Desktop/license/x-pack-5.6.3.jar" LicenseVerifier.java
        ok 生成了LicenseVerifier.class
        
        把x-pack-5.6.3.jar用压缩文件管理器打开,将里面的LicenseVerifier.class替换掉。
        
        卸载 x-pack
            bin\elasticsearch-plugin.bat remove x-pack --purge
        重装破解版的x-pack
            bin\elasticsearch-plugin.bat install file:///C:\Users\TuT\Desktop\x-pack-1years\elasticsearch__x-pack-5.6.3_pojie.zip
        
        1个小时后我认识到我错了,不要卸载,不能再次重装,不然会报错的(fatal error on the network layer)。
        
        应该先把之前的x-pack-5.6.3.jar 删掉 (elasticsearch-5.6.3\plugins\x-pack\目录下)
        然后把替换后的x-pack-5.6.3.jar 复制到elasticsearch-5.6.3\plugins\x-pack 下面
        
        ok 
        
        申请免费的[license](https://license.elastic.co/registration) : 
        license.json : 
            {"license":{"uid":"54d70d4d-38fb-411a-b377-446414e271da","type":"basic","issue_date_in_millis":1522281600000,"expiry_date_in_millis":1553903999999,"max_nodes":100,"issued_to":"hello1 world1 (heoool1)","issuer":"Web Form","signature":"AAAAAwAAAA0W8lk7zcfzpqVHNtbBAAABmC9ZN0hjZDBGYnVyRXpCOW5Bb3FjZDAxOWpSbTVoMVZwUzRxVk1PSmkxaktJRVl5MUYvUWh3bHZVUTllbXNPbzBUemtnbWpBbmlWRmRZb25KNFlBR2x0TXc2K2p1Y1VtMG1UQU9TRGZVSGRwaEJGUjE3bXd3LzRqZ05iLzRteWFNekdxRGpIYlFwYkJiNUs0U1hTVlJKNVlXekMrSlVUdFIvV0FNeWdOYnlESDc3MWhlY3hSQmdKSjJ2ZTcvYlBFOHhPQlV3ZHdDQ0tHcG5uOElCaDJ4K1hob29xSG85N0kvTWV3THhlQk9NL01VMFRjNDZpZEVXeUtUMXIyMlIveFpJUkk2WUdveEZaME9XWitGUi9WNTZVQW1FMG1DenhZU0ZmeXlZakVEMjZFT2NvOWxpZGlqVmlHNC8rWVVUYzMwRGVySHpIdURzKzFiRDl4TmM1TUp2VTBOUlJZUlAyV0ZVL2kvVk10L0NsbXNFYVZwT3NSU082dFNNa2prQ0ZsclZ4NTltbU1CVE5lR09Bck93V2J1Y3c9PQAAAQC0AXW9YHTryY9FB7+gOag2KBJNcb0JL/SRL8u4uaL+JDM5nQBKlnBVfTnrTEy0EVyB13xZuShDR7EKOz5UfMI5i8/JCmUBFlut1wPT6JO9vWoCRrgbfHthErCE0PM1V5YUAKfDgjDx8+3RBeEHgct4ZsO3w1RvXlHdDYlTzTj5loV+4nsp2XbXc2Oc+hjm0+xLmOgwNytdX2ndFilH1aoD4TDiLZEzX9iXi1Vkm7bwNtVdFzjNK3O/m9G1NxhWYYjA+syEGM5GNp0m7Ue0E5WXHufXZv1WK3t2Utyhfuke1UsqwFYzHMrAvvB3IKSfgNGdMLq8KUshksjBoQZeZ4dF","start_date_in_millis":1522281600000}}
        重启 elasticsearch 
        > curl -XGET -u elastic:changeme 'localhost:9200/_license?pretty'
        {
          "license" : {
            "status" : "active",
            "uid" : "31e436af-dbd5-456b-bf1f-e48642056bc9",
            "type" : "trial",
            "issue_date" : "2018-03-29T05:09:56.897Z",
            "issue_date_in_millis" : 1522300196897,
            "expiry_date" : "2018-04-28T05:09:56.897Z",             ###one month 
            "expiry_date_in_millis" : 1524892196897,
            "max_nodes" : 1000,
            "issued_to" : "my-test",
            "issuer" : "elasticsearch",
            "start_date_in_millis" : -1
          }
        }
        
        将 上面的修改:
            "type":"platinum"
            "expiry_date_in_millis":253395907200000
           ==> 
           为:
            {"license":{"uid":"54d70d4d-38fb-411a-b377-446414e271da","type":"platinum","issue_date_in_millis":1522281600000,"expiry_date_in_millis":253395907200000,"max_nodes":100,"issued_to":"hello1 world1 (heoool1)","issuer":"Web Form","signature":"AAAAAwAAAA0W8lk7zcfzpqVHNtbBAAABmC9ZN0hjZDBGYnVyRXpCOW5Bb3FjZDAxOWpSbTVoMVZwUzRxVk1PSmkxaktJRVl5MUYvUWh3bHZVUTllbXNPbzBUemtnbWpBbmlWRmRZb25KNFlBR2x0TXc2K2p1Y1VtMG1UQU9TRGZVSGRwaEJGUjE3bXd3LzRqZ05iLzRteWFNekdxRGpIYlFwYkJiNUs0U1hTVlJKNVlXekMrSlVUdFIvV0FNeWdOYnlESDc3MWhlY3hSQmdKSjJ2ZTcvYlBFOHhPQlV3ZHdDQ0tHcG5uOElCaDJ4K1hob29xSG85N0kvTWV3THhlQk9NL01VMFRjNDZpZEVXeUtUMXIyMlIveFpJUkk2WUdveEZaME9XWitGUi9WNTZVQW1FMG1DenhZU0ZmeXlZakVEMjZFT2NvOWxpZGlqVmlHNC8rWVVUYzMwRGVySHpIdURzKzFiRDl4TmM1TUp2VTBOUlJZUlAyV0ZVL2kvVk10L0NsbXNFYVZwT3NSU082dFNNa2prQ0ZsclZ4NTltbU1CVE5lR09Bck93V2J1Y3c9PQAAAQC0AXW9YHTryY9FB7+gOag2KBJNcb0JL/SRL8u4uaL+JDM5nQBKlnBVfTnrTEy0EVyB13xZuShDR7EKOz5UfMI5i8/JCmUBFlut1wPT6JO9vWoCRrgbfHthErCE0PM1V5YUAKfDgjDx8+3RBeEHgct4ZsO3w1RvXlHdDYlTzTj5loV+4nsp2XbXc2Oc+hjm0+xLmOgwNytdX2ndFilH1aoD4TDiLZEzX9iXi1Vkm7bwNtVdFzjNK3O/m9G1NxhWYYjA+syEGM5GNp0m7Ue0E5WXHufXZv1WK3t2Utyhfuke1UsqwFYzHMrAvvB3IKSfgNGdMLq8KUshksjBoQZeZ4dF","start_date_in_millis":1522281600000}}
            
        
        PS > gc .\license.json | Invoke-WebRequest -uri 'http://localhost:9200/_xpack/license?acknowledge=true' -Credential elastic -Method Put -UseBasicParsing

        (Linux) > curl -XPUT -u elastic:changeme 'http://localhost:9200/_xpack/license?acknowledge=true' -H "Content-Type: application/json" -d @license.json
        
        再次重启 elasticsearch
        > curl -XGET -u elastic:changeme 'localhost:9200/_license?pretty'
            ...
            "expiry_date" : "9999-10-19T00:00:00.000Z",    # 9999 year
        
        使用Graph,报错....
        
        需要修改 fielddata=true , 但是elasticsearch 中文社区说 keyword类型也可以 ,然后发现我之前 PDI自动创建的索引字段类型都自带keyword了,
        但是 自己主动生成的倒是没有 ,需要每个字段都加上 "fields": {"keyword": {"type": "keyword","ignore_above": 256}}
        即 :
            "properties": {
               "Name": {
                  "type": "text",
                  "fields": {
                     "keyword": {
                        "type": "keyword",
                        "ignore_above": 256
                     }
                  }
               }
        
        
        开启 [fielddata](https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html)
            PUT my_index/_mapping/_doc
            {
              "properties": {
                "my_field": { 
                  "type":     "text",
                  "fielddata": true
                }
              }
            }
        
        

    d. 因为索引建立好之后,使用 elasticdump 转移数据时
	
            elasticdump --limit 50000 --bulk=true --input=http://localhost:9201/2021xxx --output=http://9.9.9.10:8200/sbbb-2021xxx --type=mapping
            elasticdump --limit 50000 --bulk=true --input=http://localhost:9201/2021xxx --output=http://9.9.9.10:8200/sbbb-2021xxx --type=data
            字段是无法修改的。
			
    e. v5.6.3 沒有csv-export 
	
            补丁 : https://github.com/fbaligand/kibana/releases/tag/v5.6.3-csv-export

### 常用搜索

	> curl -XDELETE "http://localhost:9200/twitter1?pretty"
	> curl -XPUT "localhost:9200/twitter1"
	> curl -XPOST "http://9.9.9.10:8200/sbbb-in/sbbb-in/_mapping?pretty" -d '{"sbbb-nnnnn":{"properties":{"性别-SEX":{"type":"string"},"生日-BIRTHDATE":{"type":"string"},"姓名-FULLNAME":{"type":"string"},"电话-PHONE":{"type":"string"},"账户号-ACCT_NB":{"type":"string"},"ID卡-IDCARD":{"type":"string"},"地址-ADDRESS":{"type":"string"}}}}'
	> curl -XPOST "http://9.9.9.10:9200/sbbb-unlimit/sbbb-unlimit/_mapping?pretty" -d '{"sbbb-unlimit":{"properties":{"邮箱-EMAIL":{"type":"string"},"mwmm-ps":{"type":"string"},"注册来源-SOURCE":{"type":"string"},"注册时间-TIMESTAMP":{"type":"string"}}}}'
	> curl -XGET "http://localhost:9200/gg/_count?pretty" -d'{"query":{"match_all":{}}}'
	> curl -XGET 'http://localhost:9200/_cat/nodes?v'
	> curl -X GET 'http://localhost:9200/_cat/indices/booking?pri

    	yellow open booking xeKdxcBZTyeXXKFL0_8eMQ 5 1 9999999 10401 5.4gb 5.4gb
	
	> curl -XGET 'http://localhost:9200/_cat/indices?v'

	    health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
	    yellow open   vv   aivl6uRxTVuu2x9ghhlbyg   5   1     470873            0    496.1mb        496.1mb
	    yellow open   .kibana Q2a4JlacR2-qIMAC0WNu7A   1   1          4            1     21.4kb         21.4kb
	    yellow open   bb    xtXjkuNsRBCVeUI7iaJIbw   5   1    4244901           79      3.8gb          3.8gb
	
	> curl -XGET "http://localhost:9200/gg/sdsd/100?pretty"

	    {
	      "_index" : "gg",
	      "_type" : "sdsd",
	      "_id" : "100",
	      "_version" : 2,
	      "found" : true,
	      "_source" : {
		"ACCOUNT_ID" : "1450",
		"PASSWORD" : "t+PhTvdySshVoOKNa7biHAY6HNURc+9cIfzDA6xxxxx",
		"COUNTRY_CODE_ADDRESS_LINE" : "GB__15 Anroyd Street xxxxxx",
		"ADDRESS_POSTCODE" : "W F 1 3 4 L T",
		"EMAIL" : "bb.vv@vvv",
		"NAME" : "Mrs xxxx",
		"HOME_PHONE" : "01924510306",
		"MOBILE_PHONE" : "null",
		"GENDER" : "null",
		"DATE_OF_BIRTH" : "null",
		"CRAD_AND_NAME" : "412983xxxx07__SP RILEY"
	      }
	    }

通配符搜索

	> GET /my_index/my_type/_search
	{
	  "query": {
	    "wildcard": {
	      "title": {
		"value": "C3?*"
	      }
	    }
	  }
	}

正则搜索

	GET /my_index/my_type/_search
	{
	  "query": {
	    "regexp" : {
	      "title" : "C[0-9].+345"
	    }
	  }
	}

	curl -XPOST "http://9.9.9.10:8200/sbbb-test-2021dlh/sbbb-test-2021dlh/_mapping?pretty" -d '{"sbbb-test-2021dlh":{"properties":{"邮箱-EMAIL":{"type":"string"},"用户名-USERNAME":{"type":"string"},"mwmm-ps":{"type":"string"}}}}'
	curl -XPOST "http://9.9.9.10:8200/sbbb-test-2021dlh/sbbb-test-2021dlh/_mapping?pretty" -d '
	{
		"sbbb-test-2021dlh":{
			"properties":{
				"邮箱-EMAIL":{
					"type":"string"
				},
				"用户名-USERNAME":{
					"type":"string"
				},
				"mwmm-ps":{
					"type":"string"
				}
			}
		}
	}'


	curl -XGET http://9.9.9.10:8200/sbbb-2021dlh/_mapping?pretty

	{
	  "sbbb-2021dlh" : {
		"mappings" : {
		  "jdbc" : {
			"properties" : {
			  "email" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  },
			  "password" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  },
			  "username" : {
				"type" : "text",
				"fields" : {
				  "keyword" : {
					"type" : "keyword",
					"ignore_above" : 256
				  }
				}
			  }
			}
		  }
		}
	  }
	}



	https://stackoverflow.com/questions/36856849/elastic-search-document-count
	docs.count in _cat/indices returns the count of all documents, including artificial documents that have been created for nested fields.
	GET /index/type/_count count will tell you how many Elasticsearch documents are in your index, i.e. how many you have indexed.
	GET /_cat/indices?v' will tell you how many Lucene documents are in your index.






	https://discuss.elastic.co/t/elasticsearch-health-status-is-yellow-how-to-up-green/81765
	Elasticsearch health status is yellow. How to up Green?
	==> 
	The Yellow status means that there is a risk of losing data if something goes wrong with your shards.
	To get it green you should at least have another node. I mean you should create a cluster.
	Kab,

集群状态为yellow 
由于我们是单节点部署elasticsearch,而默认的分片副本数目配置为1,而相同的分片不能在一个节点上,
所以就存在副本分片指定不明确的问题,所以显示为yellow,我们可以通过在elasticsearch集群上添加一个节点来解决问题,
如果你不想这么做,你可以删除那些指定不明确的副本分片(当然这不是一个好办法)但是作为测试和解决办法还是可以尝试的,
下面我们试一下删除副本分片的办法
解决办法

	root@c2dbf1f9da5d:/# curl -XPUT "http://localhost:9200/_settings" -d' { "number_of_replicas" : 0 } '
	{"acknowledged":true}



## 0x07 Reference

	http://www.cnblogs.com/kaynet/p/5861926.html
	https://blog.csdn.net/mvpboss1004/article/details/65445023
	http://blog.csdn.net/xf_87/article/details/51026197
