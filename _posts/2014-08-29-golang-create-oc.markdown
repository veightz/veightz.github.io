---
layout: post
title: Golang生成ObjectiveC接口
date: 2014-08-29 22:40:22
categories:
---

之前写ObjectiveC的接口写哭了，因为接口数量实在是太多了，所以写了个Golang的小工具打辅助。

{% highlight go %}
package main

import "fmt"

func main() {
    fName := "clientupdateVersion"
    pName := "version"
    otherParameters := [] string {}
    printFuncHeader(fName, pName)
    printFuncParameter(otherParameters)
}

func printFuncHeader(fName string, pName string) {
    fmt.Printf("+ (NSDictionary *)%s:(NSString *)%s", fName, pName)
    return
}

func printFuncParameter(otherParameters [] string) {
    for _, value := range otherParameters {
        fmt.Printf("\n%s:(NSString *)%s", value, value)
    }
    fmt.Println(";")
    return
}
{% endhighlight %}

> 好吧, 的确很low, 有空研究下如何自动写入到系统剪贴板, 这样就可以偷懒不去复制了...
