---
layout: post
title: ObjectiveC 生成 MD5
date: 2014-09-28 01:19:00
categories:
---

在开发时处于安全考虑, 会经常使用到MD5.由于比较常用, 就把现在用的代码记一下, 权当笔记.

```objc
#import <CommonCrypto/CommonCryptor.h>
#import <CommonCrypto/CommonDigest.h>
+ (NSString *)MD5: (NSString *) inPutText
{
    const char *cStr = [inPutText UTF8String];
    unsigned char result[CC_MD5_DIGEST_LENGTH];
    CC_MD5(cStr, (CC_LONG)strlen(cStr), result);

    return [[NSString stringWithFormat:@"%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X",
             result[0], result[1], result[2], result[3],
             result[4], result[5], result[6], result[7],
             result[8], result[9], result[10], result[11],
             result[12], result[13], result[14], result[15]
             ] uppercaseString];
}
```
当然, 如果需要MD5值为小写字母表达的, 只需要将`uppercaseString`替换为`lowercaseString`就行了.
