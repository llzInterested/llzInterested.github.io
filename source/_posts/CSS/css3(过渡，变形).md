---
title: css3(过渡，变形)
date: 2018-12-10 10:23:48
description: 文章访问需要密码
password: llz721097
categories: 
    - CSS
    - CSS3
tags: 
    - CSS
    - CSS3 
---


# 过渡
- 过渡结束时可以监听`transitionend`事件,在webkit下是`webkitTransitionEnd`事件
- 每一个拥有过渡的属性在完成过渡时都会触发一次`transitionend`事件
- 在transition完成前设置`display:none`，过渡事件就不会被触发
- 在元素首次渲染还未完成时不会触发过渡
- 在绝大部分变换样式切换中，如果变换函数位置，个数不相同也不会触发过渡


# transform变形
## 前置属性
###  transform-origin指定变形中心点
- 指定变形的中心点，默认为元素正中心，正值表示正向位移，负值负向位移(X,Y,Z轴正向分别为向右，向下，靠近用户眼睛)
- 二维的x-offset/y-offset可以设px，%，关键字(top,center等)，- 三维z-offset只能为px
- 该属性对translate没有影响，translate始终相对于元素正中心位移

### transform-style指定舞台为2D或3D
取值：
- flat(2D)
- preserve-3d(3D)

需在变形元素父级上（舞台上）设置该属性。若同时设置transform-style:preserve-3d和overflow:hidden(包括祖先元素)，3D效果将失效，当设置为preserve-3d时，当元素进入舞台背面后会消失

### perspective指定3D的视距
即人眼距显示器的距离，只能设px，该属性设置在舞台元素上，不是进行3D变换的元素上

### perspective-origin设置视距的基点
即设置人眼的位置，默认在50%，50%，即中心处，需和perspective一起使用     

###  backface-visibility设置是否看见3D舞台的背面
默认值visible可见，可设hidden隐藏

## 2D变形transform
只对块级元素有效
### 旋转rotate
```css
transform:rotate()
```

只能设单值，正数表示顺时针旋转(注：2D层面无rotateX()和rotateY())

### 平移translate()
```css
transform:translate()
```
可设单值或双值，设单值表示只x轴位移，y轴不变,有translateX()，translateY()

### 倾斜skew()
```css
transform:skew()
```
可设单值或双值，设单值表示只x轴倾斜，y轴不变，有skewX()和skewY()

### 缩放scale()
```css
transform:scale()
```
- 可设单值或双值，设单值表示x，y轴等值缩放。可设负数，负数会先将元素反转再缩放
- 只想x轴缩放scaleX(.5)  <=====>    scale(.5,1)    
- 只想y轴缩放scaleY(.5)  <=====>    scale(1,.5) 

## 3D变形transform
### 位移translate3d(x,y,z)
其中z只能为px，z越大离眼睛越近，但大于perspective视距时元素会消失，有translateX(),translateY(),translateZ()

### 缩放scale3d(x,y,z)        
还可单独设置z轴缩放scaleZ(),只在组合变换时有用，单独scaleZ()没有效果
```css
transform:scaleZ(2) translateZ(100px)   //向前平移200px
transform:translateZ(100px) scaleZ(2)   //向前平移100px
```


###  旋转rotate3d(x,y,z,a)
a表示旋转角度，可单独设置某个轴，rotateX(),rotateY(),rotateZ()，其中x，y，z为1时，表示按该轴旋转，为0时表示不按该轴旋转

# 动画
- 动画内属性
    - animation-name：设置对象所应用的动画名称
    - animation-duration：设置对象动画的持续时间
    - animation-timing-function：设置对象动画的过渡类型
- 动画外属性
    - animation-delay：设置对象动画延迟的时间
     
- animation-iteration-count：设置对象动画的循环次数，只作用于动画内属性，循环的是关键帧
- animation-direction：设置对象动画的运动方向，反转的是关键帧和animation-timing-function
- animation-fill-mode：设置对象动画时间之外的状态,即from之前和to之后的状态，取值如下：
    - none
    - backwards：from前的状态和from一致
    - forwards：to之后的状态和to一致
    - both：包含backwards和forwards

- animation-play-state：控制动画运动或暂停

## 关键帧定义
```css
@keyframes 动画名称{
    动画持续时间百分比{
        
    }
    动画持续时间百分比{
        
    }
}
```
动画持续时间百分比使用from表示0%，to表示100%






