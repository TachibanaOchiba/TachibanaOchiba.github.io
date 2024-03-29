---
title: MS excel 常用的文本操作函数
tags:
  - Excel函数
  - 文本处理
categories:
  - 财务狗的日常
date: 2019-12-18 12:45:35
---

<!--在此处插入头图-->
{%asset_img header.png "Cover"%}

<!--在此处插入概述-->
在日常工作中，经常会会遇到在excel中处理文本的情况。熟悉文本操作函数可以使工作事半功倍。特此将其中常用的整理如下，以备日常参考。

<!--more-->

<!--以下为正文-->

## 本文语法
- **`函数(必要参数，[可选参数])`**: 函数的意义 : `输入示例` -> `输出` (必要注释)

## 文本格式化
> 将文本格式化再处理可以避免因为文本格式问题分析数据时产生错误。

- **`T(item)`**: 如果`item`是文本，返回`item`本身，否则返回空文本:
  - `T("text")` -> `"text"`
  -  `T(123)` -> `""`
- **`TRIM(text)`**: 删除多余的空格:`TRIM("I come from China ")` -> `"I come from China"`
- **`ASC(text)`**: 将双字节字符改为单字节字符: `ASC("Yes， I will come")` -> `"Yes, I will come"`
- **`UPPER(text)`**: 所有字母大写: `UPPER("shepherd")` -> `"SHEPHERD"`
- **`LOWER(text)`**: 所有字母小写:`LOWER("SHePherd")` -> `"shepherd"`
- **`PROPER(text)`**: 首字母大写: `PROPER("davros knows. Davros remembers")` -> `"Davros Knows. Davros Remembers"`

## 文本操作
> 可以用于从一个单元格中提取需要的信息，如从地址从提取邮编。
### 子串
- **`LEFT(text,sequence)`**: 返回文本的前`sequence`个字的子串: `LEFT("shepherd",2)` -> `"sh"`
- **`RiGHT(text,sequence)`**: 返回文本的后`sequence`个字的子串: `RIGHT("shepherd",2)` -> `"rd"`
- **`MID(text,begin,len)`**: 返回文本从`begin`开始`len`长的子串: `MID("shepherd",2,3)` -> `"hep"`
- **`LEFTB(text,sequence)`**: 同`LEFT`，但是`sequence`的单位为字节数
- **`RiGHTB(text,sequence)`**: 同`RIGHT`，但是`sequence`的单位为字节数
- **`MIDB(text,begin,len)`**: 同`MID`，但是`sequence`的单位为字节数

### 编辑
- **`REPLACE(og_str,replace_from,replace_to,replace_str)`**: 将`og_str`从`replace_from`到`replace_to`的文字替换成`replace_str`: `REPLACE("awsl",3,4,"tmdyhl") `-> `"awtmdyhl"`
- **`SUBSTITUTE(og_str,find_str,subs_str,[seq])`**: 将`og_str`中的`seq`个`find_str`替换为`subs_str`（不指定`seq`则全部替换）: 
   - `SUBSTITUTE("niconiconi","n","w",2)` -> `"nicowiconi"`
   - `SUBSTITUTE("niconiconi","n","w")` -> `"wicowicowi"`

### 和其他文本
- **`CONCAT(item1,item2,[...])`**:合并所有`item`,生成字符串(不限数据类型):
   - `CONCAT("cat"," dog")` -> `"cat dog"`
   - `CONCAT(1,"2",3)` -> `"123"`
- **`SEARCH(find,source)`**: 在`source`中找到第一个`find`的位置: `SEARCH("日","白日依山尽")`-> `2`
  
## 文本和其他数据类型

### 文本和数值
- **`NUMBERVALUE(Text, [Decimal_separator], [Group_separator])`**: 将`Text`转化为数字。如果不可能，返回`#VALUE!`: `NUMBERVALUE("1,348,960")` -> `1348960`。
- **`TEXT(value, [format])`**: 将`value`转化为`format`形式的文本: `=TEXT(1.14514,"0.0")` -> `"1.1"`。
### 财会/金融狗专区
- **`DOLLAR(number,dec)`**: 将`number`转化为美元形式的文本，保留`dec`位小数: `DOLLAR(1348960,2)` -> `"$1,348,960.00"`
- **`RMB(number,dec)`**: 将`number`转化为人民币形式的文本，保留`dec`位小数: `RMB(1348960,2)` -> `"¥1,348,960.00"`
- **`DOLLARFR(number,frac)`**: 将`number`转化为分母为`frac`的金额数字: `DOLLARFR(1.125,16)` -> `1.02` (即一又十六分之二)
- **`DOLLARDE(number,frac)`**: `DOLLARFR`的逆运算: `DOLLARFR(1.02,16)` -> `1.125` 

## 应用题
- 问：找出将下列对账单明细中的外币数字：
    - `A1` = `Hands Chopped at LLC GROUP Spent Amount: 114,514 JAPANESE YEN`
    - `A2` = `114514`
    
- 答： 
    - `A2` =`SUBSTITUTE(MID(A1,FIND(":",A1)+1,LEN(A1)-FIND("JAPANESE YEN",A1)-2),",","") `
    - 思路：
    ```excel
    A2 = FIND(":",A1) = 39
    A3 = FIND("JAPANESE YEN",A1) = 49
    A4 = LEN(A1) = 60
    A5 = MID(A1,A2+1,A4-A3-2) = 114,514
    A6 = SUBSTITUTE(A5,",","")=114514  
    ```

## P.S
- 配图是excel 95的彩蛋"Hall of Tortured Souls"，有兴趣可以移步敖厂长视频 [【敖厂长】EXCEL暗藏恐怖游戏](https://www.bilibili.com/video/av3446850)

## 参考资料
- [MS excel帮助](https://support.office.com/zh-cn/article/dollarfr-%e5%87%bd%e6%95%b0-0835d163-3023-4a33-9824-3042c5d4f495?NS=EXCEL&Version=90&SysLcid=2052&UiLcid=2052&AppVer=ZXL900&HelpId=xlmain11.chm60492&ui=zh-CN&rs=zh-CN&ad=CN)
