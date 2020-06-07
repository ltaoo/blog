---
title: 模仿 velocity.js 实现 DOM 动画类库（三）
categories: JavaScript
tags:
- 动画
date: 2017/04/12
---

先来解决开始值与结束值单位不一致的问题。

## 单位转换

### 原理分析

假设动画元素初始宽度为`300px`，现在需要将其改变为`50%`，这就肯定需要将单位与值进行转换了，计算`300px`是百分之多少，或者计算`50%`是多少像素，这都和父容器宽度有关，所以首先是要获取到父容器了。

但是，`velocity`并没有获取父容器的宽度，而是将动画元素的宽度设为`10%`，再获取到宽度则为父容器宽度的`10%`，再除以`10`就得到比率。

用实际例子来说明，假设父容器宽度为`1314px`，动画元素宽度为`300px`，首先是将动画元素宽度设为`10%`后使用`getComputedStyle`得到宽度为`131.4px`，再除以`10`就得到`13.14`，最后`300/1/13.14`得到`300px`占父容器的百分比。

而这个值，恰恰是`300/1314/100`也能够计算得到的结果。

<!--more-->

### 实际代码

OK，现在开始写代码，上面的原理对应的代码就是：


```javascript
// 计算换算比率
function calculateUnitRatios (element, property) {
    const measurement = 10
    setPropertyValue(element, property, measurement + '%')
    return (parseFloat(getPropertyValue(element, property)) || 1) / measurement
}
```

在什么时候调用这个函数呢？

在获取到开始值与结束值的单位后，就可以判断两个单位是否相同，如果不同则计算换算比率：

```javascript
// 保存要改变的属性集合
let propertiesContainer = {}
for(let property in propertiesMap) {
    // 拿到开始值
    const startSeparatedValue = separateValue(property, getPropertyValue(element, property))
    let startValue = parseFloat(startSeparatedValue[0]) || 0
    const startValueUnitType = startSeparatedValue[1]
    // 结束值
    const endSeparatedValue = separateValue(property, propertiesMap[property])
    const endValue = parseFloat(endSeparatedValue[0]) || 0
    const endValueUnitType = endSeparatedValue[1]
	// 判断两个单位是否一致
    if (startValueUnitType !== endValueUnitType) {
        const ratios = calculateUnitRatios(element, property)
        startValue *= 1 / ratios
    }

    propertiesContainer[property] = {
        startValue,
        endValue,
        // 单位用结束值的单位，因为将开始值转换为了结束值相同单位的数值
        unitType: endValueUnitType
    }
}
```

## 生命周期

由于颜色动画比较复杂，所以暂时不实现颜色动画，先来增加生命周期，也就是回调函数，当动画开始前、执行中、结束后分别调用用户自定义的函数。

### 动画开始前
只要在`tick()`前（步骤4）判断配置项中是否存在`begin`字段，如果存在就调用。

动画开始中与动画结束都是同理，只是调用位置不同。

## 常用动画选项

`jQuery`存在预先设置好的动画函数，比如滑动`slideDown`显示被隐藏的元素，`slideUp`隐藏显示的元素等。

其实原理很简单，就是调用我们已经声明好的`animation`函数，将高度变为`0`或者恢复高度即可。


