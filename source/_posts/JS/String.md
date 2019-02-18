---
title: JS常用API之----String
date: 2018-04-16 10:05:48
description: 文章访问需要密码
password: llz721097
categories: 
    - JS
    - String
tags: 
    - JS
    - String
---
# String
## includes(查找的字符串，[起始位置])
返回布尔值，表示是否找到了参数字符串
## startsWith(查找的字符串，[起始位置])
返回布尔值，表示参数字符串是否在原字符串的头部
## endsWith(查找的字符串，[截止位置])
返回布尔值，表示参数字符串是否在原字符串的尾部
## repeat(重复次数n)
返回一个新字符串，表示将原字符串重复`n`次
- 参数如果是小数，会被取整(舍去小数部分)
- 参数是负数或者`Infinity`，会报错  
- 参数是 0 到-1 之间的小数或者NaN，则等同于 0
- 参数是字符串，则会先转换成数字
## padStart(字符串最小长度，[用什么字符串补全])
头部补全，如： `'x'.padStart(5, 'ab')     // 'ababx'`
- 若原字符串的长度，等于或大于指定的最小长度，则返回原字符串
- 用来补全的字符串与原字符串，两者的长度之和超过了指定的最小长度，则会截去超出位数的补全字符串,如： `'abc'.padStart(10, '0123456789')    // '0123456abc'`
- 若省略第二个参数，默认使用空格补全长度
## padEnd(字符串最小长度，[用什么字符串补全])
尾部补全，其他同上
## substring(开始位置，[结束位置])
## substr(开始位置，[长度])  <font color="red">ECMAscript 没有对该方法进行标准化，因此反对使用它</font>
## slice(开始位置，结束位置)
## split(分隔符)
## match(searchvalue | regexp)
- 当参数为regexp时，返回存放匹配结果的数组。该数组的内容依赖于 regexp 是否具有全局标志 g
- 如果 regexp 没有标志 g，那么 match() 方法就只能在 stringObject 中执行一次匹配。如果没有找到任何匹配的文本， match() 将返回 null。
- 否则，它将返回一个数组，其中存放了与它找到的匹配文本有关的信息。该数组的第 0 个元素存放的是匹配文本，而其余的元素存放的是与正则表达式的子表达式匹配的文本。
- 除了这些常规的数组元素之外，返回的数组还含有两个对象属性。index属性声明的是匹配文本的起始字符在 stringObject 中的位置，input属性声明的是对 stringObject 的引用。
```javascript
//不带修饰符g
var url = 'http://www.baidu.com?a=1&b=2&c=3';
var reg = /([^?&=]+)=([^?&=])*/;
var result = url.match(reg);
console.log(result); 
//["a=1", "a", "1", index: 21, input: "http://www.baidu.com?a=1&b=2&c=3"]

console.log(result.index); //21

console.log(result.input); //http://www.baidu.com?a=1&b=2&c=3
```
-  如果 regexp 具有标志 g，则 match() 方法将执行全局检索，找到 stringObject 中的所有匹配子字符串。
-  若没有找到任何匹配的子串，则返回 null。
-  如果找到了一个或多个匹配子串，则返回一个数组。不过全局匹配返回的数组的内容与前者大不相同，它的数组元素中存放的是 stringObject 中所有的匹配子串，而且也没有 index 属性或 input 属性。
```javascript
//带修饰符g
var url = 'http://www.baidu.com?a=1&b=2&c=3';
var reg = /([^?&=]+)=([^?&=])*/g;
var result = url.match(reg);
console.log(result); //["a=1", "b=2", "c=3"]

console.log(result.index); //undefined

console.log(result.input); //undefined
```
## replace(regexp/substr,replacement)
返回一个新的字符串，是用 replacement 替换了 regexp 的第一次匹配或所有匹配之后得到的
- 字符串 stringObject 的 replace() 方法执行的是查找并替换的操作。它将在 stringObject 中查找与 regexp 相匹配的子字符串，然后用 replacement 来替换这些子串。如果 regexp 具有全局标志 g，那么 replace() 方法将替换所有匹配的子串。否则，它只替换第一个匹配子串
-  replacement 可以是字符串，也可以是函数。如果它是字符串，那么每个匹配都将由字符串替换。但是 replacement 中的 $ 字符具有特定的含义。如下表所示，它说明从模式匹配得到的字符串将用于替换

|字符|替换文本|
|:-:|:-:|
|$1,$2,...,$99|与regexp中第1个到第99个子表达式相匹配的文本|
|$&|与regexp相匹配的子串|
|$`|位于匹配子串左侧的文本|
|$'|位于匹配子串右侧的文本|
|$$|直接量符号(要替换为$符号时，就写2个$)|

```javascript
//不带修饰符g
var url = 'http://www.baidu.com?a=1&b=2&c=3';
var reg = /([^?&=]+)=([^?&=])*/;
var url1 = url.replace(reg,function(a,b,c,d,e){
console.log(a,b,c,d,e); //a=1, a, 1, 21, http://www.baidu.com?a=1&b=2&c=3
return 'ok';
})
console.log(url1); //http://www.baidu.com?ok&b=2&c=3


//带修饰符g
var url = 'http://www.baidu.com?a=1&b=2&c=3';
var reg = /([^?&=]+)=([^?&=])*/g;
var url1 = url.replace(reg,function(a,b,c,d,e){
console.log(a,b,c,d,e); 
//执行3次，分别输出为：
a=1, a, 1, 21, http://www.baidu.com?a=1&b=2&c=3 和 
b=2, b, 2, 25, http://www.baidu.com?a=1&b=2&c=3 和 
c=3, c, 3, 29, http://www.baidu.com?a=1&b=2&c=3
return 'ok';
})
console.log(url1); //http://www.baidu.com?ok&ok&ok

//第二个参数为字符串时
var url = 'http://www.baidu.com?a=1&b=2&c=3';
var reg = /([^?&=]+)=([^?&=])*/; //不带修饰符g
var url1 = url.replace(reg,"$&")
console.log(url1); //http://www.baidu.com?a=1&b=2&c=3
var url1 = url.replace(reg,"$1")
console.log(url1); //http://www.baidu.com?a&b=2&c=3
var url1 = url.replace(reg,"$2")
console.log(url1); //http://www.baidu.com?1&b=2&c=3
var url1 = url.replace(reg,"$'")
console.log(url1); //http://www.baidu.com?&b=2&c=3&b=2&c=3

var reg = /([^?&=]+)=([^?&=])*/g; //带修饰符g
var url1 = url.replace(reg,"$&")
console.log(url1); //http://www.baidu.com?a=1&b=2&c=3
var url1 = url.replace(reg,"$1")
console.log(url1); //http://www.baidu.com?a&b&c
var url1 = url.replace(reg,"$2")
console.log(url1); //http://www.baidu.com?1&2&3
var url1 = url.replace(reg,"$'")
console.log(url1); //http://www.baidu.com?&b=2&c=3&&c=3&
```






