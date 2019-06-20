---
title: http cache
date: 2018-12-05 17:39:03
tags: http
---

### 1.强缓存 

#### 强缓存就是向浏览器缓存来查询文件是否存在以及是否可用

http response header 中的expires 和 cache-control 字段,cache-control的优先级更高

expires 给的是具体的时间，这个有个明显的问题就是存在服务端和客户端时间不一致的情况，因此用的不是很多

cache-control 可能的情况有
    
 1. public 所有内容都缓存，包括客户端和代理服务端
 2. private 只有客户端可以缓存，这个也是cache-control的默认值
 3. no-cache 客户端会缓存内容，但是是否使用，需要协商缓存来决定
 4. no-store 所有内容都不缓存，包括协商缓存也不使用
 5. max-age=time 缓存会在time时间后失效，相对值

比如：

```javascript
Cache-Control:public, max-age=31536000
```


### 2.协商缓存
#### 协商缓存就是指强缓存失效，浏览器待着缓存标识到服务器上去查看缓存是否可用，它与强缓存最大的区别在于需要发起真正的请求

常用的有两对：

```javascript
Last-Modified / If-Modified-Since
Etag / If-None-Match // 优先级更高
```

```last-modified``` 是服务器设置的文件最后被修改的时间。
浏览器拿到这个时间后，会再下一次发送请求的时候带上字段If-Modified-Since字段，其中放的就是上次浏览器给的最后修改的时间，服务器拿到这个时间后可以判断文件是否被更新过了从而决定是否使用缓存，如果可以使用缓存就会返回304

```Etag / If-None-Match``` 和上面的原理基本一样，服务器在第一次返回的时候会携带Etag字段用来标识文件的唯一标识，类似于hash，之后浏览器在第二次请求的过程中会带上If-None-Match字段，服务器通过这个子段来判断缓存是否失效，从而返回304或者200


#### 协商缓存和强缓存的共同点是，大家都是从使用客户端的缓存，不同点在于由谁来决定是否使用缓存。


