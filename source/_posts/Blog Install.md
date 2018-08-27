---
title: Blog Install
date: 2017-11-25 00:00:00
categories:
   - 其他
tags:
summary_img: /images/favicon.png
---
.....
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
sudo npm install hexo-deployer-git --save
sudo npm install hexo-generator-feed --save
sudo npm install hexo-generator-sitemap --save
sudo npm install hexo-generator-search --save
sudo npm install hexo-front-matter-excerpt --save
sudo npm install hexo-generator-index --save
sudo npm install hexo-generator-archive --save 
sudo npm install hexo-generator-tag --save
sudo npm install hexo-asset-image --save
sudo npm install hexo-git-backup --save
sudo npm install shelljs --save

_config.yml.
deploy:
  type: git
  repo: git@github.com:yourgithubname/yourgithubname.github.io.git
  branch: master

backup:
    type: git
    theme: landscape,next
    repository:
       github: git@github.com:yourgithubname/blog-backup.git,master
       
git config --global user.name "yourgithubname"
git config --global user.email "z....x@xxx.com"
ssh-keygen -t rsa -C "z....x@xxx.com"
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


hexo b
            
hexo-front-matter-excerpt 这个插件，默认是读取 <!-- more -->

sparrow logo : https://i.loli.net/2018/03/26/5ab8be73b5b56.png

sudo npm install hexo-asset-image --save   图片引用
post_asset_folder: true
hexo n "new artitle"
![xxx](new artitle/img.jpg)



换成 next 主题：

git clone https://github.com/iissnan/hexo-theme-next.git themes/next
好用多了
容易配置


站点配置文件
archive_generator:
  per_page: 20
  yearly: true
  monthly: true

tag_generator:
  per_page: 10

  
  
  about ：
   没什么写的，有事留言吧!

```
文章封面 

    https://neveryu.github.io/2017/07/15/hexo-next-five/
    修改 \themes\next\layout\_macro\post.swing 文件

```
{% if is_index %}
<!-- 文章摘要图片 -->
{% if post.summary_img  %}
<d1v class="out-img-topic">
<1mg src={{ post.summary_img }} class="img-topic">
</d1v>
{% endif %} 
<!-- 文章摘要图片 -->
```
  
之后： 

    ---
    title: Blog Install
    date: 2017-11-25 00:00:00
    categories:
       - 其他
    tags:
    summary_img: /images/avatar.png
    ---



```


https://github.com/Neilpang/acme.sh/wiki/dns-manual-mode
申请SSL(Secure socket layer) Server Certificate证书
https://github.com/Neilpang/acme.sh
    acme.sh 实现了 acme 协议, 可以从 letsencrypt 生成免费的证书.
    一般来说是收费的，只有letsencrypt免费，谷歌浏览器登陆有letsencrypt免费证书的网页也会显示绿色的图标，因为也是安全的SSL
    dns 方式, 在域名上添加一条 txt 解析记录, 验证域名所有权
    你不需要任何服务器, 不需要任何公网 ip, 只需要 dns 的解析记录即可完成验证. 
    坏处是，如果不同时配置 Automatic DNS API，使用这种方式 acme.sh 将无法自动更新证书，每次都需要手动再次重新解析验证域名所有权。
1. DNS manual mode
acme.sh --issue -d example.com --dns --yes-I-know-dns-manual-mode-enough-go-ahead-please

2. Please add the TXT record to your DNS records. 
   Domain: '_acme-challenge.1sparrow.com'
   TXT value: 'OX5btHmMS1GSWrN01hFomW6l-pSRFccccccw44RCQk'
3. nslookup -q=txt _acme-challenge.1sparrow.com
Server:  google-public-dns-a.google.com
Address:  8.8.8.8
Non-authoritative answer:
_acme-challenge.1sparrow.com    text =
        "OX5btHmMS1GSWrN01hFomW6l-pSRFWXC62bywcccccck"
        
4. Now retry with --renew command.
acme.sh --renew -d example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
[Tue Apr 17 21:23:00 EDT 2018] Renew: '1sparrow.com'
[Tue Apr 17 21:23:06 EDT 2018] Single domain='1sparrow.com'
[Tue Apr 17 21:23:06 EDT 2018] Getting domain auth token for each domain
[Tue Apr 17 21:23:06 EDT 2018] Verifying:1sparrow.com
[Tue Apr 17 21:23:24 EDT 2018] Success
[Tue Apr 17 21:23:24 EDT 2018] Verify finished, start to sign.
[Tue Apr 17 21:23:30 EDT 2018] Cert success.

==> 
but github can't enforce https for custom domain
所以使用netlify 
走亚马逊的CDN
国内访问比github快
支持自定义域名和证书
免费


```
