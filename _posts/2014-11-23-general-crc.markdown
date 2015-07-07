---
layout: post
title: CRC冗余生成及校验
date: 2014-11-23 17:55:12
categories:
---

前阵子写的一个CRC冗余计算的代码.
- 不同类型的 CRC 矩阵直接丢二进制了
- 由于奇偶校验是一种特殊的 CRC, 直接一起写了

{% highlight cpp %}
#include <iostream>
#include <bitset>
enum CRCType {CRC1, CRC4, CRC16, CRC32};
unsigned long leftMove(CRCType type) {
    switch (type) {
        case CRC1:
            return 1;
            break;
        case CRC4:
            return 4;
            break;
        case CRC16:
            return 16;
            break;
        case CRC32:
            return 32;
        default:
            return 0;
            break;
	    }
	}
unsigned long generalPolynomial(CRCType type) {
    switch (type) {
        case CRC1:
            return 0x2;
        case CRC4:
            return 0x13;
            break;
        case CRC16:
            return 0x18005;
            break;
        case CRC32:
            return 0x104800007;
            break;
        default:
            break;
	    }
	}
	/**
 *  自用转成二进制输出
	 *
 *  @param value 输出值
	 */
void log(unsigned long value) {
    std::cout << std::bitset<sizeof(unsigned long)*8>(value) << std::endl;
	}


	/**
 *  寻找1出现的最高位
	 *
 *  @param crc 现在的冗余值
	 *
 *  @return 返回位数
	 */
unsigned long leftOffset(unsigned long crc) {
    unsigned long length = sizeof(unsigned long)*8;
    for (unsigned long i = length - 1; i < length; i--) {
        unsigned long key = crc >> i;
        if (key == 0x01) { return i;}
	    }
    return 0;
	}
	/**
 *  生成CRC
	 *
 *  @param originValue 原值
 *  @param type        CRC类型
	 *
 *  @return 返回拼接后的编码值
	 */
unsigned long generalCRC(unsigned long originValue, CRCType type) {
    // 原编码移位
    originValue = originValue << leftMove(type);
    unsigned long CRCValue = originValue;

    // 获取表达式
    unsigned long polynomial = generalPolynomial(type);
    unsigned long offset;
    while ( (offset = leftOffset(CRCValue))  >=  leftMove(type) ) {
//        log(CRCValue);
//        log(polynomial << (offset - (unsigned long)leftMove(type)));
        CRCValue = CRCValue ^ (polynomial << (offset - (unsigned long)leftMove(type)));
//        log(CRCValue);
//        std::cout << std::endl;
	    }

    std::cout << "冗余码计算结果:" << std::endl;
    log(CRCValue);
    std::cout << "待添加的编码:" << std::endl;
    log(originValue);
    originValue = originValue | CRCValue;
    std::cout << "生成码计算结果:" << std::endl;
    log(originValue);
    std::cout << "Over" << std::endl << std::endl;
    return originValue;
	}

	/**
 *  检查CRC
	 *
 *  @param receivedValue 接受的值
 *  @param type          CRC类型
	 *
 *  @return 返回判断结果
	 */
bool checkCRC(unsigned long receivedValue, CRCType type) {
    // 用户匹配出生成的冗余值的掩码
//    unsigned long maskValue = (0x1 << leftMove(type)) - 1;
    // 提取的冗余值
//    unsigned long checkValue = receivedValue & maskValue ;
//    log(checkValue);

	    /**
     *  貌似上面这两货算出来也没什么用诶+_+ 真是醉了
	     */


    // 获取表达式
    unsigned long polynomial = generalPolynomial(type);
    unsigned long offset;
    while ( (offset = leftOffset(receivedValue))  >=  leftMove(type) ) {
//                log(receivedValue);
//                log(polynomial << (offset - (unsigned long)leftMove(type)));
        receivedValue = receivedValue ^ (polynomial << (offset - (unsigned long)leftMove(type)));
//                log(receivedValue);
//                std::cout << std::endl;
	    }

    if (receivedValue == 0x0) {
        return true;
	    }
    return false;
	}


	/**
 *  用于随机翻转几位可怜的编码
	 *
 *  @param thePoorvalue 待破坏的值
 *  @param type         CRC类型
 *  @param fuckTimes    破坏的位数
	 *
 *  @return 翻转后的值
	 */
unsigned long randomDestory(unsigned long thePoorvalue, CRCType type, unsigned int fuckTimes) {
    unsigned long resultValue = thePoorvalue;

    srand((int)time(0));

    unsigned int changedPosition = rand() % 32;
    std::cout << "原值:\n";
    log(resultValue);

    for (int i = 0; i < fuckTimes; i++) {

        unsigned long maskValue = 0x1 << changedPosition;
        std::cout << "改变了倒数第" << changedPosition + 1 << "位\n";
        resultValue = resultValue ^ maskValue;
        log(resultValue);
        changedPosition = changedPosition * changedPosition % 32;
	    }
    return resultValue;
	}

int main(int argc, const char * argv[]) {
    unsigned long checkedValue;
    checkedValue = 0x123456790;
    CRCType valueCRCType = CRC4;

    std::cout << "第一次直接判断\n\n";
    unsigned long resultValue = generalCRC(checkedValue, valueCRCType);
    if (checkCRC(resultValue, valueCRCType)) {
        std::cout << "校验正确\n\n\n";
    } else {
        std::cout << "校验错误\n\n\n";
	    }
	    /**
     *  破坏一下下, 生成新的resultValue
	     */
    std::cout << "进行破坏\n";
    resultValue = randomDestory(checkedValue, valueCRCType, 2);

	    /**
     *  再次检查
	     */
    std::cout << "第二次判断\n\n";
    if (checkCRC(resultValue, valueCRCType)) {
        std::cout << "校验正确\n\n\n";
    } else {
        std::cout << "校验错误\n\n\n";
	    }


//    generalCRC(0x1234, CRC16);
    return 0;
	}
{% endhighlight %}
