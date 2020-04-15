---
title: 【问题梳理】AmazonS3文件名包含英文括号导致401
categories: S3
tags: bug
date: 2020-04-15 11:22:58
---

# 遇到的问题
通过S3上传文件，如果存储文件名中有英文括号，则S3服务端会返回鉴权失败【401】。出现问题的代码如下所示：
{% asset_img 1.png %}

<!-- more -->

# S3文件上传流程
{% asset_img S3-framework.png %}

# 出现问题的地方
org.apache.http.client.utils.URIUtils#rewriteURI

| 阶段 | URL |
|---|---|
| S3生成权限校验签名 | ``` http://s3/bk/sam%28t%29hh.txt ``` |
| Request请求 | ``` http://s3/bk/sam(t)hh.txt ``` |

导致鉴权失败，返回401。

# 细节分析
## 本来Request中的uri是``` http://s3/bk/sam%28t%29hh.txt ```，为什么apache会进行rewrite？
{% asset_img 2.png %}
rewrite的逻辑如上图所示，主要目的是去除url中的无效路径。
同时，我们通过代码可以看出，无论是否有空节点，最终都会执行uribuilder.setPathSegment(pathSegments);  
在4.5.10中已经解决了这个问题，如果发现并没有进行空节点的remove操作，这里就不会执行uribuilder.setPathSegment(pathSegments);  

## rewrite的具体流程是什么
可以详细看看【图3】中的setPathSegment干了什么：将encodedPath设置为null;
{% asset_img 3.png %}

在进行【图3】中所示的uribuilder.build操作的时候，如下图所示，会对pathSetment重新进行encodePath。
["bk","sam(t)hh.txt"] ["bk","sam(t)hh.txt"]
本例中，pathSetment包含两个元素 ["bk","sam(t)hh.txt"]
{% asset_img 5.png %}

encodePath最终调用的是org.apache.http.client.utils.URLEncodedUtils#urlEncode方法
这里就很明显了，如果是在safechars集合中的字符，则不需要进行转码。
这里的safechars = PATHSAFE如，PATHSAFE 包含 UNRESERVED
根据{% asset_link https://www.ietf.org/rfc/rfc2396.txt RFC-2396 %}标准UNRESERVED应该包含英文括号，即英文括号不需要被转码。
如下图所示

| PATHSAFE | UNRESERVED |
|---|---|
| {% asset_img 6.png %} | {% asset_img 7.png %} |
| {% asset_img 8.png %} | {% asset_img 9.png %} |

所以进行uribuilder.build之后，pathSetment包含的两个元素 ["bk","sam(t)hh.txt"]，所有字符都不会被转码，生成的最终uri就成了：``` http://s3/bk/sam(t)hh.txt ```。

## S3在创建HttpRequest的时候，英文括号为什么会被转码：``` http://s3/bk/sam%28t%29hh.txt ```
S3生成的URI，是经过Java原生的UrlEncoder进行encode的：
{% asset_img 11.png %}

这里不需要转码的集合【dontNeedEncoding】如下所示：
{% asset_img 12.png %}

其中不包含英文括号，所以英文括号也会被转码，导致生成的URI就是：``` http://s3/bk/sam%28t%29hh.txt ```
然后S3-Client认为该URI【``` http://s3/bk/sam%28t%29hh.txt ```】就是需要访问的资源，通过该URI生成权限签名Sign-Auth。
但是最终Apache-client经过rewrite之后的Uri却是：【``` http://s3/bk/sam(t)hh.txt ```】, 所以S3的会报权限异常401【Sign-Auth与真实访问的资源URI不匹配】。

# 解决
这个问题在apache的httpclient 4.5.10版本中已经没有出现了。原因如下：
{% asset_img 13.png %}

# 后续
其实这个问题的本质在于：
## S3生成url鉴权的时候，用的Java-UrlEncoder。对英文括号要进行转码。
## 底层执行http请求的时候用的 Apache-httpclient，会对uri进行rewrite，不会对英文括号进行转码。
导致异常。
apache 4.5.10以前，不论是否有空节点，都会进行setPathSegments，从而导致重新encode。【对英文括号不转码】
apache 4.5.10及以后，如果路径中没有空节点，就不会进行setPathSegments，直接沿用原来的encodePath。
