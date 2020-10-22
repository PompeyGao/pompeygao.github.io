---
layout:     post
title:      "浏览器缓存机制"
subtitle:   ""
date:       2020-07-18
author:     "pompeygao"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - 浏览器缓存
---

<!-- TOC -->

- [#### 二、强缓存](#h4-id二强缓存-33二强缓存h4)
    - [三、协商缓存](#三协商缓存)
    - [四、缓存机制](#四缓存机制)
    - [参考文章](#参考文章)

<!-- /TOC -->

#### 一、缓存作用

---

一个优秀的缓存策略可以缩短网页请求资源的距离，减少延迟，并且由于缓存文件可以重复利用，还可以减少带宽，降低网络负荷。


#### 二、强缓存
---

**强缓存：不会向服务器发送请求，直接从缓存中读取资源，在chrome控制台的Network选项中可以看到该请求返回200的状态码，并且Size显示from disk cache或from memory cache。强缓存可以通过设置两种 HTTP Header 实现：`Expires` 和 `Cache-Control`。**

- `Expires`

  Expires是HTTP/1.0时代的产物，缓存过期时间，用来指定资源到期的时间，它描述的是一个绝对时间，由服务端返回。**受限于本地时间，如果修改了本地时间，可能会造成缓存失效**。

- `Cache-Control`

  Cache-Control出现在HTTP/1.1中，**优先级高于 Expires** ，表示的相对时间。

  > Cache-Control: max-age=300     //表示在该请求正确返回时间的 5分钟内再次加载资源，就会命中强缓存。

  Cache-Control 可以在请求头或者响应头中设置，并且可以组合使用多种指令：
  
  ![cache-control](/img/in-post/browser-cache/cache-control.png)
  
  **public**：**所有内容都将被缓存（客户端和代理服务器都可缓存）**。具体来说响应可被任何中间节点缓存，如 Browser <-- proxy1 <-- proxy2 <-- Server，中间的proxy可以缓存资源，比如下次再请求同一资源proxy1直接把自己缓存的东西给 Browser 而不再向proxy2要。
  
  **private**：**所有内容只有客户端可以缓存**，Cache-Control的默认取值。具体来说，表示中间节点不允许缓存，对于Browser <-- proxy1 <-- proxy2 <-- Server，proxy 会老老实实把Server 返回的数据发送给proxy1,自己不缓存任何数据。当下次Browser再次请求时proxy会做好请求转发而不是自作主张给自己缓存的数据。
  
  **no-cache**：客户端缓存内容，是否使用缓存则需要经过协商缓存来验证决定。表示不使用 Cache-Control的缓存控制方式做前置验证，而是使用 Etag 或者Last-Modified字段来控制缓存。**需要注意的是，no-cache这个名字有一点误导。设置了no-cache之后，并不是说浏览器就不再缓存数据，只是浏览器在使用缓存数据时，需要先确认一下数据是否还跟服务器保持一致。**
  
  **no-store**：所有内容都不会被缓存，即不使用强制缓存，也不使用协商缓存
  
  **max-age**：max-age=xxx (xxx is numeric)表示缓存内容将在xxx秒后失效
  
  **s-maxage**（单位为s)：同max-age作用一样，只在代理服务器中生效（比如CDN缓存）。比如当s-maxage=60时，在这60秒中，即使更新了CDN的内容，浏览器也不会进行请求。max-age用于普通缓存，而s-maxage用于代理缓存。**s-maxage的优先级高于max-age**。如果存在s-maxage，则会覆盖掉max-age和Expires header。
  
  **max-stale**：能容忍的最大过期时间。max-stale指令标示了客户端愿意接收一个已经过期了的响应。如果指定了max-stale的值，则最大容忍时间为对应的秒数。如果没有指定，那么说明浏览器愿意接收任何age的响应（age表示响应由源站生成或确认的时间与当前时间的差值）。
  
  **min-fresh**：能够容忍的最小新鲜度。min-fresh标示了客户端不愿意接受新鲜度不多于当前的age加上min-fresh设定的时间之和的响应。

#### 三、协商缓存

---

**协商缓存：强制缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存的过程**

- **Last-Modified / If-Modified-Since**

  `Last-Modified`从字面意思就可以看出是最后一次的修改时间，同样它也是由服务端放到Response Headers返回。

  如果有设置协商缓存，我们在首次请求的时候，返回的Response Headers会带有Last-Modified。当再次请求没有命中强制缓存的时候，这个时候我们的Request Headers就会携带If-Modified-Since字段，它的值就是我们第一次请求返回给我们的Last-Modified值。服务端接收到资源请求之后，根据If-Modified-Since的字段值和服务端资源最后的修改时间是否一致来判断资源是否有修改。

  **如果时间一致，协商缓存生效，则返回的状态码为304 Not Modified，直接从缓存读取数据**

  **如果时间不同，协商缓存失效，则返回新的资源，状态码为200**。

  

  **缺点** 

  1. 服务端对Last-Modified标注的最后修改时间只能精确到秒级，如果某些文件在1秒钟以内被修改多次的话，这个时候服务端无法准确标注文件的修改时间。
  2. 服务端有时候会定期生成一些文件，有时候文件的内容并没有任何变化，但这个时候Last-Modified会发生改变，导致文件无法使用缓存。
  3. 如果本地打开缓存文件，即使没有对文件进行修改，但还是会造成 Last-Modified 被修改，服务端不能命中缓存导致发送相同的资源。

  

- **ETag / If-None-Macth**

  既然根据文件修改时间来决定是否缓存尚有不足，能否可以直接根据文件内容是否修改来决定缓存策略？所以在 HTTP / 1.1 出现了 `ETag` 和`If-None-Match。`

  `Etag`就像一个指纹，资源变化都会导致ETag变化，跟最后修改时间没有关系，`ETag`可以保证每一个资源是唯一的。

  Etag/If-None-Match的用法这里对应Last-Modified/If-Modified-Since。

  **`ETag`的优先级比`Last-Modified`高**

#### 四、缓存机制

---

**强制缓存优先于协商缓存进行，若强制缓存(Expires和Cache-Control)生效则直接使用缓存，若不生效则进行协商缓存(Last-Modified / If-Modified-Since和Etag / If-None-Match)，协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，返回200，重新返回资源和缓存标识，再存入浏览器缓存中；生效则返回304，继续使用缓存**。具体流程图如下： 

![cache](/img/in-post/browser-cache/cache.png)

> 强缓存不发请求到服务器，协商缓存会发请求到服务器。



#### 参考文章

- [深入理解浏览器的缓存机制](https://segmentfault.com/a/1190000017004307)
- [缓存（二）——浏览器缓存机制：强缓存、协商缓存](https://github.com/amandakelake/blog/issues/41)