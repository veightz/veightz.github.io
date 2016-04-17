---
layout: post
title: "AFNetworking读取cookies方法"
date: 2014-09-28 01:01:59
categories:
---

AFNetworking的Cookies处理是封装到底层的,也就是说,使用AFnetworking时,开发者不需要自行去管理的,AFNetworking会自行帮你在底层进行cookies的(响应时)读取和(之后请求时)写入.可是开发过程中有时候会需要使用cookies中一些值,所有需要我们去访问cookies的内容.我们可以通过以下方式遍历Client中共享的cookies.
```objc
NSArray *cookies = [[NSHTTPCookieStorage sharedHTTPCookieStorage] cookies];
for (NSHTTPCookie *cookie in cookies) {
        // Here I see the correct rails session cookie
        NSLog(@"cookie:\n%@", cookie);
            NSLog(@"cookie.value:\n%@",cookie.value);
}
```

我在自己当前的工程中打印了一下.

```sh
2014-09-28 01:00:07.312 Meet[36308:377650] cookie:
<NSHTTPCookie version:0 name:"JSESSIONID" value:"89791FE8C44B0104DF6F5D2A611AC09B" expiresDate:(null) created:2014-09-27 17:00:07 +0000 (4.3353e+08) sessionOnly:TRUE domain:"115.29.228.115" path:"/ichujian/" isSecure:FALSE>
```
```sh
2014-09-28 01:00:07.312 Meet[36308:377650] cookie.value:
89791FE8C44B0104DF6F5D2A611AC09B
```
以此我们就能获取cookies里面的内容了.
