---
title: JS常用API之----Array
date: 2018-04-13 14:25:43
description: 文章访问需要密码
password: llz721097
categories: 
    - JS
    - Array
tags: 
    - JS
    - Array
---
# Array
## Array.from(要转为数组的对象，[function]，[绑定this])
- 参数2的function类似`map()`,用来对每个元素进行处理，将处理后的值放入返回的数组
- 只能将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）
## Array.of()
构造数组实例，`of()`可以解决`new Array()`构造器因参数个数不同，导致的行为有差异的问题(`new Array()`当参数只有一个数值时，构造函数会把它当成数组的长度)。
    如：`Array.of(1,2,3) ==> [1,2,3]`
## copyWithin(target,[start],[end])     <font color="red">会改变原数组</font>
浅复制数组的一部分到同一数组中的另一个位置，并返回它，而不修改其大小
- target（必需）：复制序列到该位置。如果为负值，表示倒数。如果 target ≥arr.length，将会不发生拷贝。如果 target< start，复制的序列将被修改以符合 arr.length
- start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示倒数。如果 start 被忽略，copyWithin 将会从0开始复制
- end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数
```javascript
[1, 2, 3, 4, 5].copyWithin(-2);
// [1, 2, 3, 1, 2]
[1, 2, 3, 4, 5].copyWithin(0, 3);
// [4, 5, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(0, 3, 4);
// [4, 2, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(-2, -3, -1);
// [1, 2, 3, 3, 4]
```
## find(回调函数)
找出第一个符合条件的数组成员，没有返回undefined
## findIndex(回调函数)
返回第一个符合条件的数组成员的位置，没有返回-1
## fill(填充的值，[填充起始位置]，[填充结束位置])   <font color="red">会改变原数组</font>
使用给定的值填充数组，常用于初始化数组，数组中已有的元素，会被全部抹去
## 遍历array
### keys()
### values()
### entries()
```javascript
for (let [index, elem] of ['a','b'].entries()) {                           
 console.log(index,elem);  
 //  0  'a'
 //  1  'b'
}

let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```
## includes(在数组中查询的值，[起始位置]，[结束位置])
返回一个布尔值，表示某个数组是否包含给定的值
## reduce(function(prev,cur,index,arr),设置prev的初始类型和初始值)
- prev: 第一项的值或者上一次叠加的结果值
- cur: 当前会参与叠加的项
- index： 当前值的索引
- arr: 数组本身
## filter(function(currentValue,index,arr), thisValue)
创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素
- currentValue：当前元素的值，必须
- index：当前元素索引，可选
- arr：当前元素所属数组对象，可选
- thisValue：执行该函数的this，可选
```javascript
const studentsAge = [17, 16, 18, 19, 21, 17];
const ableToDrink = studentsAge.filter( age => age > 18 );        
//  [19,21]
```
## map(function(currentValue，index，arr),thisValue)
返回一个新数组，数组中的元素为原始数组元素调用函数处理的后值，参数含义同上
```javascript
const numbers = [2, 3, 4, 5];
const dollars = numbers.map( number => '$' + number);        
//['$2', '$3', '$4', '$5']
```
## forEach(function(currentValue，index，arr),thisValue)
## some(function(currentValue,index,arr),thisValue)    <font color="red">不改变原数组</font>
检测数组中的元素是否满足指定条件
- 如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测
- 如果没有满足条件的元素，则返回false
## every(function(currentValue,index,arr),thisValue)    <font color="red">不改变原数组</font>
检测数组所有元素是否都符合指定条件
- 如果数组中检测到有一个元素不满足，则整个表达式返回 false ，且剩余的元素不会再进行检测
- 如果所有元素都满足条件，则返回 true
## slice(start,end)     <font color="red">不改变原数组</font>
## splice(start[, deleteCount[, item1[, item2[, ...]]]])    <font color="red">会改变原数组</font>
- start：指定修改的开始位置（从0计数）。如果超出了数组的长度，则从数组末尾开始添加内容；如果是负值，则表示从数组末位开始的第几位（从-1计数）；若只使用start参数而不使用deleteCount、item，表示删除[start，end]的元素
- deleteCount (可选)：整数，表示要移除的数组元素的个数。如果 deleteCount 是 0，则不移除元素。这种情况下，至少应添加一个新元素。如果 deleteCount 大于start 之后的元素的总数，则从 start 后面的元素都将被删除（含第 start 位）。如果deleteCount被省略，则其相当于(arr.length - start)
- item1, item2, ... (可选)：要添加进数组的元素,从start 位置开始。如果不指定，则 splice() 将只删除数组元素
-  返回值： 由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组
## sort()    <font color="red">会改变原数组</font>
## reverse()    <font color="red">会改变原数组</font>
## join(分隔符)    <font color="red">不改变原数组</font>
将数组（或一个类数组对象）中所有元素都转化为字符串并使用指定的分隔符(默认分隔符为逗号)连接在一起，返回最后生成的字符串
## concat()     <font color="red">不改变原数组</font>
## isArray()
判定对象是否为数组，还能通过以下方式判断：
`Object.prototype.toString.call(arg) === '[object Array]'`