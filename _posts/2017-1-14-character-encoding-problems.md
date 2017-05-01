---
layout: post
title: 字符编码 总结整理
categories: 编程相关
: 在我的印象笔记里，有一个名为“编码问题”的文件夹，里面装满了泪水。。。<br><br><img src="http://upload-images.jianshu.io/upload_images/658453-cd6916bf7875499e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">
---

### 引言

> 没被编码问题困扰过的程序员是有问题的。                   —— xiao

### 汉字

#### 汉字的数量

> 汉字的数量并没有准确数字，大约将近**十万**个（北京国安咨询设备公司汉字字库收入有出处汉字91251个）。**日常所使用的汉字只有几千字。据统计，1000个常用字能覆盖约92%的书面资料，2000字可覆盖98%以上，3000字时已到99%**。简体与繁体的统计结果相差不大。

#### 汉字编码

##### 第一次尝试：GB 2312

> GB 2312 标准共收录**6763个汉字**，其中一级汉字3755个，二级汉字3008个；同时，GB 2312 收录了包括拉丁字母、希腊字母、日文平假名及片假名字母、俄语西里尔字母在内的682个全角字符。
GB 2312 的出现，基本满足了汉字的计算机处理需要，它所收录的汉字已经覆盖中国大陆99.75%的使用频率。
对于人名、古汉语等方面出现的罕用字，GB 2312 不能处理，这导致了后来 GBK 及GB 18030 汉字字符集的出现。

##### 第二次尝试：GBK

> GB 2312 的出现基本满足了汉字的计算机处理需要，但由于上面提到未收录繁体字和生僻字，从而不能处理人名、古汉语等方面出现的罕用字，这导致了1995年《汉字编码扩展规范》（GBK）的出现。GBK 编码是 GB 2312 编码的超集，向下完全兼容 GB 2312，兼容的含义是不仅字符兼容，而且相同字符的编码也相同，同时在字汇一级支持 ISO/IEC10646—1 和 GB 13000—1的全部中、日、韩（CJK）汉字，**共计20902字**。GBK 还收录了GB 2312 不包含的汉字部首符号、竖排标点符号等字符。CP936 和 GBK 的有些许差别，绝大多数情况下可以把 CP936 当作 GBK 的别名。

##### 第三次尝试：GB 18030

> GB 18030 编码向下兼容 GBK 和 GB 2312。GB 18030 收录了所有Unicode 3.1 中的字符，包括中国少数民族字符，GBK 不支持的韩文字符等等，也可以说是世界大多民族的文字符号都被收录在内。**GBK 和GB 2312 都是双字节等宽编码，如果算上和 ASCII 兼容所支持的单字节，也可以理解为是单字节和双字节混合的变长编码。GB 18030 编码是变长编码，有单字节、双字节和四字节三种方式。**


##### 注意点

**GB 2312、GBK 和 GB 18030 都兼容 ASCII，所以它们都有1字节（ASCII）或2字节（中文/韩文/日文/古文/少数民族文字）；其中 GB 18030 还有4字节。**

#### 缺陷

以上三个编码有两个关键的缺陷：

1. 由于低字节的编码范围和 ASCII 有重合，所以不能根据一个字节的内容判断是中文的一部分还是一个独立的英文字符。（不是**前缀编码！**）

2. 如果有两个汉字编码为 A1A2 B1B2，存在 A2B1 也是一个有效汉字编码的特殊情况。这样就不能直接使用标准的字符串匹配函数来判断一个字符串里是否包含某一个汉字，而**需要先判断字符边界然后才能进行字符匹配判断**。

### 天下归一：Unicode

> 在 Unicode 里，所有的字符被一视同仁，汉字不再使用“两个扩展ASCII”，而是使用“1个Unicode”来表示，也就是说，所有的文字都按一个字符来处理，它们都有一个唯一的 Unicode 码。Unicode 用数字**0-0x10FFFF**来映射这些字符，最多可以容纳1114112个字符，或者说有**1114112（110万+）个码位**（码位就是可以分配给字符的数字）。

---

**Unicode 总共只有110万+码位，最多需要3个字节（1677万+）来存储。**

---
在 Unicode 中：汉字“字”对应的数字是23383。在 Unicode 中，我们有很多方式将数字23383表示成程序中的数据，包括：UTF-8、UTF-16、UTF-32。UTF是“UCS Transformation Format”的缩写，可以翻译成 Unicode 字符集转换格式，即怎样将 Unicode 定义的数字转换成程序数据。例如，“汉字”对应的数字是0x6c49和0x5b57，而编码的程序数据是：

1. BYTE data_utf8[] = {0xE6, 0xB1, 0x89, 0xE5, 0xAD, 0x97}; // UTF-8编码

2. WORD data_utf16[] = {0x6c49, 0x5b57}; // UTF-16编码

3. DWORD data_utf32[] = {0x6c49, 0x5b57}; // UTF-32编码

这里用BYTE、WORD、DWORD分别表示无符号8位整数，无符号16位整数和无符号32位整数。UTF-8、UTF-16、**UTF-32分别以BYTE、WORD、DWORD作为 【 编 码 单 位 】**。“汉字”的UTF-8编码需要6个字节。“汉字”的UTF-16编码需要两个WORD，大小是4个字节。“汉字”的UTF-32编码需要两个DWORD，大小是8个字节。根据字节序的不同，UTF-16可以被实现为UTF-16LE或UTF-16BE，UTF-32可以被实现为UTF-32LE或UTF-32BE。

#### UTF-8

![UTF-8](http://upload-images.jianshu.io/upload_images/658453-5f7aa0fce81c8709.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

UTF-8 的第一个字节开始的1的个数代表了总的编码字节数，后续字节都是以10开始。由上面的规则可以清晰的看出 UTF-8 编码**克服了中文编码的两个问题（中英文字符编码严格区分，且中文字符定界清晰）。**

#### UTF-16

UTF-16 编码以16位无符号整数为单位。我们把 Unicode 编码记作 U。编码规则如下：
如果 U<0×10000，U 的 UTF-16 编码就是 U 对应的16位无符号整数（为书写简便，下文将16位无符号整数记作WORD）。中文范围 4E00-9FBF，所以**在 UTF-16 编码里中文2个字节编码**。如果 U≥0×10000，我们先计算 U’=U-0×10000，然后将 U’ 写成二进制形式：yyyy yyyy yyxx xxxx xxxx，U 的 UTF-16 编码（二进制）就是：110110yyyyyyyyyy 110111xxxxxxxxxx（2个双字节）。

#### UTF-32

UTF-32 编码以32位无符号整数为单位。Unicode 的 UTF-32 编码就是其对应的32位无符号整数。

---

> **UTF-8 以单字节为*编码单位*，有1字节、2字节、3字节、4字节四种；UTF-16 以双字节为编码单位，有2字节、4字节两种；UTF-32以4字节为编码单位，只有4字节（3字节就够了）。**

---

#### 字节序

多字节的编码就有字节序的问题。

根据字节序的不同，UTF-16 可以被实现为 UTF-16LE（Little Endian）或UTF-16BE（Big Endian），UTF-32 可以被实现为 UTF-32LE 或 UTF-32BE。

那么，怎么判断字节流的字节序呢？Unicode 标准建议用 BOM（Byte Order Mark）来区分字节序，即在传输字节流前，先传输被作为 BOM 的字符”零宽无中断空格”。这个字符的编码是 FEFF，而反过来的FFFE（UTF-16）和 FFFE0000（UTF-32）在 Unicode 中都是未定义的码位，不应该出现在实际传输中。下表是各种 UTF 编码的 BOM：

![BOM](http://upload-images.jianshu.io/upload_images/658453-ce099bbbf208c1a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 虽然 UTF-8 这种变长的编码方式不存在大端小端的问题，但是还是指定了 BOM 来表示这个文件是 UTF-8。在实际的使用过程当中尽量不要对 UTF-8 添加 BOM，否则很多软件解析都会出现问题。

### 其他

Java 中使用 UTF-16 为字符编码格式，中文英文都占2字节。

**注意不要把字符和char等同起来；**Java中char类型固定占2字节，但一个char不一定能表示一个字符；码值大于0xFFFF的字符就需要两个char来存储；

> **Unicode Character Representations**
> 
> The char data type (and therefore the value that a Character object encapsulates) are based on the original Unicode specification, which defined characters as fixed-width 16-bit entities. The Unicode Standard has since been changed to allow for characters whose representation requires more than 16 bits. The range of legal code points is now U+0000 to U+10FFFF, known as Unicode scalar value. (Refer to the definition of the U+n notation in the Unicode Standard.)
> 
> The set of characters from U+0000 to U+FFFF is sometimes referred to as the Basic Multilingual Plane (BMP). Characters whose code points are greater than U+FFFF are called supplementary characters. The Java platform uses the UTF-16 representation in char arrays and in the String and StringBuffer classes. In this representation, supplementary characters are represented as a pair of char values, the first from the high-surrogates range, (\uD800-\uDBFF), the second from the low-surrogates range (\uDC00-\uDFFF).

> 暂时先整理这些，以后碰到合适的内容再加。