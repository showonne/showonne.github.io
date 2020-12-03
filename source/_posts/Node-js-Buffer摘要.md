title: Node.js Buffer摘要
date: 2015-07-20 22:41:10
tags: [Node.js]
---

纯JavaScript对Unicode友好但是无法很好地处理二进制数据.当我们处理类似TCP流或文件系统时，是需要处理八位流的.Node有几种操作、创建以及消费(consuming)八位流的策略。

Buffer像一个整数数组，但对应着 V8 堆之外的一个原始内存分配区域.Buffer不能被resize.
Buffer类是一个直接处理二进制数据的全局类型，它可以被多种方式构造.
<!-- more -->

## 构建
**new Buffer(size)**
分配一个新的 buffer 大小是size的8位字节.
**new Buffer(array)**
分配一个新的 buffer 使用一个8位字节array数组.
**new buffer(str [,encoding])**
分配一个buffer，包含着指定的str，默认编码utf8

## 类方法
**Buffer.byteLength(str, [encoding])**
将会返回这个字符串`真实byte长度`,`encoding`默认`utf8`

    console.log(Buffer.byteLength("呵呵")); //返回值为6,常用中文字符用utf-8编码占用3个字节

**Buffer.concat(list, [totalLength]) **
返回一个保存着将传入buffer数组中所有buffer对象拼接在一起的buffer对象.list为用于被拼接的buffer数组.totalLength:上述Buffer数组的所有Buffer的总大小。

    var buf1 = new Buffer([1,2,3]); 
    console.log(buf1);  //<Buffer 01 02 03>
    var buf2 = new Buffer([4,5,6]);
    console.log(buf2);  //<Buffer 04 05 06>
    buf2 = Buffer.concat([buf1,buf2]);
    console.log(buf2);  //<Buffer 01 02 03 04 05 06>


## 实例属性
**buf[index]**
**buf.length** 返回这个buffer的bytes大小.这未必是这buffer里面内容的大小，而是依据是buffer对象所分配的内存数值，它不会随着这个buffer对象内容的改变而改变。

    var buf1 = new Buffer(20);
    console.log(buf1.length);  //20

## 实例方法
**buf.write(str, [offset], [length], [encoding])**
按照指定参数将str写入buffer，str为指要写入的字符串，offset偏移量，默认0，length写入长度，默认`buffer.length-offset`，encoding默认utf8.返回值为number类型，表示多少8位字节流被写入了，这个方法不会出现写入部分字符。
   
    var buf = new Buffer(10);
    var len = buf.write("今天天不错");
    console.log(len + "bytes:" + buf.toString());//9 bytes:今天天

**buf.toString([encoding], [start], [end])**将Buffer转换为字符串，默认值分别为`utf8`,`0`,`buffer.length`

**buf.toJSON()** 返回一个JSON表示的Buffer实例。`JSON.stringify`将会默认调用来字符串序列化这个Buffer实例。

**buf.slice([start], [end])** 返回一个新的buffer，这个buffer将会和老的buffer引用相同的内存地址，只是根据start(默认是0) 和end(默认buffer.length) 偏移和裁剪了索引.

    var buf1 = new Buffer("123");
    console.log(buf1); // <Buffer 31 32 33>
    var buf2 = buf1.slice(1,2);
    console.log(buf2); // <Buffer 32>
    buf2[0] = 0x33;
    console.log(buf1); // <Buffer 31 33 33>

**buf.fill(value, [offset], [end])** 使用指定的value来填充这个buffer。

    var b = new Buffer(50);b.fill("h");





















