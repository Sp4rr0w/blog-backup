---
title: Blog Install
date: 2017-11-25 00:00:00
categories:
   - 其他
tags:
---
<!-- more -->

```
wget http://nodejs.org/dist/v9.2.0/node-v9.2.0-linux-x64.tar.gz
tar -zxvf node-v9.2.0-linux-x64.tar.gz
sudo mv node-v9.2.0-linux-x64 /opt/

sudo ln -s /opt/node-v9.2.0-linux-x64/bin/node /usr/local/bin/node		
sudo ln -s /opt/node-v9.2.0-linux-x64/bin/npm /usr/local/bin/npm

sudo npm install hexo -g 
/opt/node-v9.2.0-linux-x64/lib/node_modules/hexo/bin/hexo -v
sudo ln -s /opt/node-v9.2.0-linux-x64/lib/node_modules/hexo/bin/hexo /usr/local/bin/hexo
hexo init
hexo server

current folder
https://www.jianshu.com/p/53f37196f6e7
sudo npm install hexo-deployer-git --save
sudo npm install hexo-generator-feed --save
sudo npm install hexo-generator-sitemap --save
sudo npm install hexo-generator-search --save
sudo npm install hexo-front-matter-excerpt --save
sudo npm install hexo-git-backup --save
sudo npm install shelljs --save

_config.yml.
deploy:
  type: git
  repo: git@github.com:Sp4rr0w/sp4rr0w.github.io.git
  branch: master

backup:
    type: git
    theme: landscape
    repository:
       github: git@github.com:Sp4rr0w/blog-backup.git,master
       
git config --global user.name "sp4rr0w"
git config --global user.email "zbxzyzbx@163.com"
ssh-keygen -t rsa -C "zbxzyzbx@163.com"
 ==> id_rsa.pub 
  profile -> Settings -> SSH and GPG Keys -> Add new SSH key
  
hexo clean && hexo g && hexo d
git clone https://github.com/viosey/hexo-theme-material.git themes/material

安装访客量
http://ibruce.info/2015/04/04/busuanzi/#more
没加载出来前就显示旋转效果 ：
打开themes/你的主题/layout/_partial/footer.ejs添加
<script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
Total <span id="busuanzi_value_site_pv"><i class="fa fa-spinner fa-spin"></i></span> views.
您是Sparrow博客的第<span id="busuanzi_value_site_uv"><i class="fa fa-spinner fa-spin"></i></span>个小伙伴.


换个简洁评论valine
https://github.com/litten/hexo-theme-yilia/pull/646
_config.yml : 
valine: 
 appid:  #Leancloud应用的appId   XzzzFqxY7zSnAm9am6FHg6do-gzGzoHsz
 appkey:  #Leancloud应用的appKey   aaBboz6UbNeJnPowxKNNDTm2
 
https://material.viosey.com/docs
 
language: zh-CN

search:
    path: search.xml
    field: all
baidu_site_id: 


categories:
- 技术相关
tags:
- PS3
- Games

pages:
	标签云: 
	    link: "/tags"
            icon: person
            divider: false

            
hexo-front-matter-excerpt 这个插件，默认是读取 <!-- more -->

```
