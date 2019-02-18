---
title: css3(选择器，文本，盒子，背景，渐变)
date: 2017-08-12 15:32:18
description: 文章访问需要密码
password: llz721097
categories: 
    - CSS
    - CSS3
tags: 
    - CSS
    - CSS3 
---
# 选择器
## + 相邻兄弟选择器
匹配的是紧跟在后面的第一个兄弟元素
## ~ 通用兄弟选择器
匹配同级的后面的所有兄弟元素
## > 子元素选择器
只能匹配直接后代
## 属性选择器
- [attr]：包含attr属性的所有元素，不管attr的值
- [attr=val]：包含attr属性值为val的元素
- [attr~=val]：匹配包含attr属性，且attr属性值中包含有val，attr的属性值为空格分割的数组，如下面代码匹配前2个div
    ```html
    [name~="test"]
    <div name="test aa"></div>
    <div name="test bb"></div>
    <div name="cc"></div>
    ```
- [attr|=val]：匹配attr属性值以val-开头和属性值为val的元素
- [attr^=val]：匹配attr属性值以val开头的元素
- [attr$=val]：匹配attr属性值以val结尾的元素
- [attr*=val]：匹配attr属性值包含val的元素

## 伪类和伪元素选择器
给a标签设置时按如下顺序L V H A,即link，visited，hover，active，因为link和visited分别表示链接访问前和访问后的状态，已经包括了hover和active，如果放在后面，会覆盖掉hover和active的样式
### 链接伪类
链接伪类作用于链接元素
- :link     表示作为超链接，并指向一个未访问的链接 
- :visited      表示作为超链接，并指向一个已访问的链接，visited中只有3个属性能起作用：color,background-color,border-color
- :target       表示一个特殊的元素，其id是URI的片段

demo:点击某个链接下面显示某个div
```css
div{
   width:200px;
   height:200px;
   background-color:pink;
   display:none;
   text-align:center;
   font:50px/200px "微软雅黑"
}
:target{
    display:block;
}
```
```html
<a href="#div1">div1</a>
<a href="#div2">div2</a>
<a href="#div3">div3</a>
<div id="div1">div1</div>
<div id="div2">div2</div>
<div id="div3">div3</div>
```
### 动态伪类
- :hover：鼠标浮上去
- :active：鼠标点击

### 表单伪类
- :enabled      匹配可编辑的表单     
- :disabled     匹配被禁用的表单
- :checked      匹配被选中的表单
- :focus        匹配获取焦点的表单

### 结构性伪类
- index的值从1开始计数
- index可以为变量n且只能为n
- index可以为odd，even


#### nth-child(index)系列
```#wrap li:nth-child(1)``` ： 找到wrap下第一个子元素，且该子元素为li

- :first-child      等价于:nth-child(1)
- :last-child       等价于nth-last-child(1)
- :nth-last-child(index)    index从后往前数，即index=1等价于选中最后一个元素
- :only-child       既是第一个元素也是最后一个

#### nth-of-type(index)系列
```#wrap li:nth-of-type(1)```   ：  找到wrap下第一个li元素

- :first-of-type    等价于:nth-of-type(1)
- :last-of-type     等价于:nth-last-of-type(1)
- :nth-last-of-type(index)      index从后往前数，即index=1等价于选中最后一个元素
- :only-of-type     既是第一个元素也是最后一个

<font color="red">注意：:nth-of-type以元素为中心</font>
```css
#wrap .inner:nth-of-type(1){
    border:1px solid;
}
```
```html
<div id="wrap">
    <div class="inner">div</div>
    <p class="inner">p</p>
    <h1 class="inner">h1</h1>
</div>
```
上述css选择器会将3个class为inner的元素都选中，即等同于如下写法：
```css
#wrap div:nth-of-type(1){}
#wrap p:nth-of-type(1){}
#wrap h1:nth-of-type(1){}
```

#### not
```css
div > a:not(:last-of-type)
```
上述css表示div下选中除最后一个的所有a标签
#### empty
```css
div:empty{}
```
上述css表示选中div中内容为空的div（有空格都不行）

### 伪元素选择器
- ::before
- ::after
- ::firstLetter      选中文本的第一个字
- ::firstLine        选中文本的第一行
- ::selection        鼠标选中的文本

## css声明的优先级
选择器的特殊性由选择器本身的组件确定，特殊性值表述为4个部分，如0，0，0，0
- 对于id，加0，1，0，0
- 对于class，属性选择器，伪类选择器，加0，0，1，0
- 元素和伪元素选择器，加0，0，0，1
- 通配符选择器，为0，0，0，0
- 内联声明，加1，0，0，0

# 自定义字体图标
- @font-face：允许开发者为网页指定在线字体，通过这种自备字体的方式可以消除对用户电脑字体的依赖
- font-family：所指定的字体名字将会被用于font或font-family属性
- src：指定字体资源
- 不能在一个css选择器中定义@font-face
```css
@font-face{
    font-family:
    src:
}
```

# 文本新增样式
## text-shadow
```css
text-shadow: h-shadow v-shadow blur color;
```
- h-shadow	必需,水平阴影的位置,允许负值
- v-shadow	必需,垂直阴影的位置,允许负值
- blur	可选,模糊的距离
- color	可选,阴影的颜色

text-shadow 属性向文本添加一个或多个阴影,多个阴影之间逗号隔开

### 浮雕文字
```css
h1{
    text-align: center;
    font:100px/200px '微软雅黑';
    color:#FFF;
    text-shadow: #000 1px 1px 100px;
}
```
```html
<h1>浮雕文字</h1>
```
![浮雕文字效果](https://note.youdao.com/yws/api/personal/file/FDDFDA1C19D8482EA3142D9922052C70?method=download&shareKey=6c930703a8d85b9f4e6f169504d7f6f6)

## 文字排版direction
取值为ltr(从左到右)或rtl(从右到左)，当值为rtl时，还需要配合```unicode-bidi:bidi-override```才能实现文字从右到左

## 文本溢出text-overflow
```css
white-space:nowrap;         //文本不换行
overflow:hidden;
text-overflow:ellipsis;     //溢出文本用省略号表示
```

## 文字描边-webkit-text-stroke
<font color="red">只能在webkit内核才能使用，即移动端和pc的chrome，safari，opera下可以使用</font>
```css
-webkit-text-stroke:width color;
```
- width:描边的线的宽度
- color：描边的线的颜色

# 盒模型新增样式
## box-shadow
可以设定多组效果，每组参数值以逗号分隔
```css
box-shadow: h-shadow v-shadow blur spread color inset;
```
- h-shadow	必需，水平阴影的位置。允许负值	
- v-shadow	必需，垂直阴影的位置。允许负值	
- blur	可选，模糊距离	
- spread	可选，阴影的尺寸，省略时阴影和原盒子一样大，为正值时比盒子大，负值时比盒子小
- color	可选，阴影的颜色
- inset	可选，将外部阴影 (outset) 改为内部阴影

## 盒子倒影-webkit-box-reflect
<font color="red">只能在webkit内核才能使用，即移动端和pc的chrome，safari，opera下可以使用</font>
```css
-webkit-box-reflect：direction offset;
```
- direction取值(必需)：
    - above：倒影在盒子上方
    - below：倒影在盒子下方
    - left：倒影在盒子左方
    - right：倒影在盒子右方
- offset设置倒影和原盒子距离

## 拖动改变盒子大小resize
必需与```overflow:auto```配合使用
```css
overflow:auto;
resize:both;
```
resize取值：
- none：不允许用户调整元素大小
- both：用户可以调节元素的宽度和高度
- horizontal：用户可以调节元素的宽度
- vertical：用户可以调节元素的高度

# 背景
## background-image
```css
background-image:url(),url()...
```
支持多背景，用逗号隔开，多背景从z轴方向堆叠，先指定的背景在上层

## background-position
```css
background-position:offsetX offsetY
```
- offsetX为px时，正值向右移动，offsetY为px时，正值向下移动
- offsetX为%时，%是相对于(图片所在div宽度-图片原本宽度)，即差值为负值时，%正值换算成px为负值，所以向左移动

## background-origin修改绘制起始区域
修改背景图片的绘制起始区域，默认背景图是从padding box开始绘制，从border box开始裁剪(图片大于绘制区域时)

取值：
- content-box
- padding-box (默认值)
- border-box

## background-clip修改裁剪区域
修改背景图片裁剪区域

取值：
- content-box
- padding-box
- border-box (默认值)
- text：按文字剪切背景，即只有文字区域内有背景，使用该值时，需加-webkit前缀

## background-size设置背景图片大小
- 当只有一个值时，是指定图片宽度，高度隐式为auto
- 取值为%时，是相对于背景图所在div区域的，
- 取值为auto，为背景图真实大小
- 取值为cover，背景图等比缩放至铺满容器，背景图可能超出容器
- 取值为contain，背景图等比缩放至宽度或者高度与容器相同，背景图始终在容器内

# 渐变
## 线性渐变
```css
background-image:linear-gradient(color1,color2...)
```
默认线性渐变是从上到下渐变的，改变渐变方向使用下面写法：
```css
background-image:linear-gradient(to top|left|right|bottom,color1,color2...)
```
或者使用角度来控制：
```css
background-image:linear-gradient(角度,color1,color2...)
```
其中0°是12点钟方向，顺时针为正值

控制渐变颜色位置，使用下面写法：
```css
background-image:linear-gradient(角度,color1 10%,color2 20%,color3 30%)
```
上面表示0~ 10%是color1的纯色，10%~ 20%是color1到color2的渐变，20%~30%是color2到color3的渐变，30%后是color3的纯色

当要使渐变重复时，使用下面写法
```css
background-image:repeating-linear-gradient(角度,color1 10%,color2 20%)
```

## 径向渐变
```css
background-image:radial-gradient(color1,color2...)
```
默认径向渐变从内到外，且形状为圆形，若要修改形状，使用下面写法：
```css
background-image:radial-gradient(shape,color1,color2...)
```
其中shape取值可以为：
- circle （默认值）
- ellipse

修改径向渐变的大小，使用如下写法：
```css
background-image:radial-gradient(size [shape],color1,color2...)
```
其中size取值为：
- closest-side：按最近的边
- farthest-side：按最远的边
- closest-corner：按最近的角
- farthest-corner：按最远的角(默认值)
 
设置圆心位置，采用下面写法：
```css
background-image:radial-gradient(size circle at 10px 10px,color1,color2...)
```






