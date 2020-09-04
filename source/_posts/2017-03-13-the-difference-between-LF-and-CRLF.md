---
title: LF 和 CR LF 的区别
categories:
  - linux-basics
tags:
  - null
date: 2017-03-13 12:47:35
---

{% note primary %}

由于历史原因，在不同系统中，进行文本处理时，用来**“表示下一行”**的标识字符是不同的。

| 系统      | 标识字符  |
| :---:     | :---:     |
| GNU/Linux | `LF`      |
| Windows   | `CR` `LF` |

{% endnote %}

{% note primary %}

CR 和 LF 的含义如下。

| 简称  | 英文全称       | 中文  | 符号  | ASCII码 | 十六进制码 |
| :---: | :---:          | :---: | :---: | :---:   | :---:      |
| `CR`  | CarriageReturn | 回车  | \r    | 13      | 0x0D       |
| `LF`  | LineFeed       | 换行  | \n    | 10      | 0x0A       |

{% endnote %}

{% note success %}

在 Windows 系统中，使用 **Notepad++ 编辑 -> 文档格式转换** 功能，轻松转换各种系统风格的文本。

{% endnote %}

{% note warning %}

**“表示下一行”**的说法是不确切的，标识字符可以认为是一行的结束，可以认为是两行的分隔。大部分主张是一行的结束，建议文本末行末尾加标识字符。

{% endnote %}

(END)

> 本文原创，[转载请注明出处](http://blog.zhengzhiwei.net/2017/03/13/the-difference-between-LF-and-CRLF/)
> 我是 zZhiw，[更多内容请关注](https://zhengzhiwei.net)

