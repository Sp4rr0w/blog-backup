---
title: 批量导出ElasticSearch查询的结果
date: 2018-03-28 16:09:00
categories:
   - 技术相关
tags:
   - ElasticSearch
   - 笔记
#summary_img: /images/pic8.jpg
---
.....
<!-- more -->

工具 ： [es2csv](https://github.com/taraslayshchuk/es2csv)

## 安装python3
后修改 

    C:\python3\python.exe --> C:\python3\python3.exe

## 添加环境变量

    C:\python3\

## 安装es2csv所使用的插件

[python-progressbar](https://github.com/niltonvolpato/python-progressbar)
    
[unicodecsv-0.14.1](https://pypi.python.org/pypi/unicodecsv/0.14.1)
    
[urllib3-1.22](https://pypi.python.org/pypi/urllib3)
    
[elasticsearch-5.2.0](https://pypi.python.org/pypi/elasticsearch/5.2.0)
    
进入到各自的文件夹下 运行：
    
	python3 setup.py install

## 英文系统下(打不了中文)

进入[ConEmuPack](https://sourceforge.net/projects/conemu/files/OldReleases/ConEmuPack/)文件夹下，双击ConEmu64.exe运行：

	搜索字符串 ： -q '00'
	搜索全部索引 ： -i _all
	搜索需要的字段 ： -f 邮箱-EMAIL 明文密码-PASSWORD
	保存成csv文件 ： -o file.csv
	
cd es2csv文件夾 运行： 

	python3 es2csv_new.py -u http://9.9.9.10:9200 -q 'gov.vn' -i _all -e -f 邮箱-EMAIL 明文密码-PASSWORD -o file-new.csv -k
		Found 1846 results
		Run query [#####] [1846/1846] [100%] [0:00:00] [Time: 0:00:00] [ 3.92 kdocs/s]
		To csv [######] [614/614] [100%] [0:00:00] [Time: 0:00:00] [ 15.35 klines/s]
		DELETE http://9.9.9.10:9200/_search/scroll [status:400 request:0.000s]


导出数目多于官方规定死的500条，无限制
