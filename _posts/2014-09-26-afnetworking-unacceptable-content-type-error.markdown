---
layout: post
title: "AFNetworking中unacceptable content type错误"
date: 2014-09-26 01:04:24
categories:
---

刚开始用AFNetworking的人总会遇到如下这个错误.

{% highlight sh %}
Error Domain=com.alamofire.error.serialization.response Code=-1016 "Request failed: unacceptable content-type: text/plain" UserInfo=0x7a9308f0 {NSErrorFailingURLKey=http://115.29.228.115/ichujian/user/getauthcode?ucoqHKpz0eNTLX8%2FhqK5DFABbackKl0vwW8zP%2FCXOde8IZna3lcunfdPkH7l2IBP, com.alamofire.serialization.response.error.response=<NSHTTPURLResponse: 0x79779eb0> { URL: http://115.29.228.115/ichujian/user/getauthcode?ucoqHKpz0eNTLX8%2FhqK5DFABbackKl0vwW8zP%2FCXOde8IZna3lcunfdPkH7l2IBP } { status code: 200, headers {
        "Content-Length" = 18;
            Date = "Thu, 25 Sep 2014 16:48:39 GMT";
                Server = "Apache-Coyote/1.1";
                    "Set-Cookie" = "JSESSIONID=2BA83A8F01E99FC14224F66E13E09A83; Path=/ichujian/; HttpOnly";
} }, NSLocalizedDescription=Request failed: unacceptable content-type: text/plain}
{% endhighlight %}

# 简谈原因
其实请求还是成功的,响应也是正常返回的.只是因为AFNetworking抬严谨才报的错.看最后一句描述,其实说的很清楚:

{% highlight sh %}
failed: unacceptable content-type: text/plain
{% endhighlight %}

因为`缺省`的content-type是严格的JSON,具体是什么我也不清楚=-=.虽然我猜以mattt的高逼格,大概会写`application/Json`.好像关于JSON的标准content-type写法比较混乱,我没做过后端,也不知道业内的是怎么个通用处理方法.

# 解决方法
{% highlight objc %}
manager.responseSerializer = [AFHTTPResponseSerializer serializer];
{% endhighlight %}

更改响应序列化类型后,content-type就变为`text/plain`了.
祝大家写的开心呢..
