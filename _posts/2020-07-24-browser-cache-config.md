---
layout:     post
title:      "浏览器缓存-服务端配置"
subtitle:   ""
date:       2020-07-24
author:     "pompeygao"
header-img: "img/post-bg-2015.jpg"
tags:
    - 浏览器缓存

---



#### nginx配置缓存字段

------

- **expires  cache-control**

  `nginx.conf`

  ```nginx
  # expires 图片缓存30天
  location ~ .*\.(gif|jpg|jpeg|png|flv|ico|swf)(.*) {
     expires 30d;
  }
  #js css缓存一小时
  location ~ .*\.(js|css)?$  {  
     expires 1h;  
  } 
  # cache-control:给图片设置过期时间36秒，这里也可以设置其它类型文件
  location ~ .*\.(gif|jpg|jpeg|png|flv|ico|swf)$ {
     add_header    Cache-Control  max-age=3600;
  }
  ```

- **Last-Modified  ETag**

  Last-Modified在nginx中是默认启用的，所以无需我们手动进行相关配置。这里我们直接介绍如何配置Etag。

  ```nginx
  location ~ .*\.(gif|jpg|jpeg|png)$ {
     FileETag on;
     etag_format "%X%X%X"; #这里格式化规则可以修改
   }
  ```

  

