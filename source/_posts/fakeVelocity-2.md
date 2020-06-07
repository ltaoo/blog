---
title: 模仿 velocity.js 实现 DOM 动画类库（二）
categories: JavaScript
tags:
- 动画
date: 2017/04/11
---

## separateValue

这次我们先来讨论如何正确分离属性值与单位，假设要实现下面的动画，
点击动画按钮，同时改变透明度、宽度以及旋转角度。
<!--more-->

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
    <script src="./src/fakeVelocity.js"></script>
    <style>
        #demo {
            width: 300px;
            height: 180px;
            background-color: red;
            transform: rotate(30deg)
        }
    </style>
</head>
<body>
    <div id="demo"></div>
    <button id="run">点击执行动画</button>
    <script>
        window.onload = function () {
            document.querySelector('#run').onclick = function () {
                const animateEl = new Animation(document.querySelector('#demo'))
                animateEl.animation({
                    opacity: 0.5,
                    width: '300px',
                    rotateZ: '90deg'
                })
            }
        }
    </script>
</body>
</html>
```


> 由于在`velocity`中旋转不是使用`transform`而必须使用`rotateZ`，少了`Z`也不行。

由于`rotateZ`这样的动画存在，所以不能单纯从值中提取出单位，还要根据属性名来获取，比如`rotateZ`是无法获取到开始值的，所以就额外处理一次，使用`getUnitType`返回预期的值。


```javascript
function separateValue (property, value) {
    let unitType,
        numericValue
    // replace 是字符串的方法，如果是数值类型则没有 replace 方法，所以先将 value 转为字符串
    numericValue = value.toString().replace(/[%A-z]+$/, function(match) {
    	// 将匹配到的字母作为单位
        unitType = match
        // 将属性值中的字母都去掉，保留数字
        return ""
    })

    // 如果没有获取到单位，就根据属性来获取
    function getUnitType (property) {
        if (/^(rotate|skew)/i.test(property)) {
            // 这两个属性值单位是 deg ，有点特殊
            return "deg"
        } else if (/(^(scale|scaleX|scaleY|scaleZ|opacity|alpha|fillOpacity|flexGrow|flexHeight|zIndex|fontWeight)$)|color/i.test(property)) {
            // 这些属性值都是没有单位的
            return ""
        } else {
            // 如果都没有匹配到，就默认是 px
            return "px"
        }
    }

    if (!unitType) {
        unitType = getUnitType(property)
    }

    return [ numericValue, unitType ]
}
```

## 多个属性值变化

之前使用全局变量保存`property`、`startValue`、`endValue`和`unitType`，如果要改变多个属性，就必须使用到对象了，对象上每一个属性都有这些值。

```javascript
let propertiesContainer = {}
for(let property in propertiesMap) {
    // 拿到开始值与开始单位
    const startSeparatedValue = separateValue(property, getPropertyValue(element, property))
    const startValue = parseFloat(startSeparatedValue[0])
    const startValueUnitType = startSeparatedValue[1]
    // 结束值与结束单位
    const endSeparatedValue = separateValue(property, propertiesMap[property])
    const endValue = parseFloat(endSeparatedValue[0]) || 0
    const endValueUnitType = endSeparatedValue[1]
	// 将结果保存到对象中
	propertiesContainer[property] = {
        startValue,
        endValue,
        unitType: endValueUnitType
    }
}
```


接下来就简单了，只要在`tick`函数内遍历`propertiesContainer`获取不同属性的开始值与结束值计算得到当前值即可。

```javascript
// 核心动画函数
function tick () {
    // 当前时间
    let timeCurrent = (new Date).getTime()
    // 遍历要执行动画的 element 元素，这里暂时只支持一个元素
    // 当前值
    // 如果 timeStart 是 undefined ，表示这是动画的第一次执行
    if (!timeStart) {
        timeStart = timeCurrent - 16
    }
    // 检测动画是否执行完毕
    const percentComplete = Math.min((timeCurrent - timeStart) / opts.duration, 1) 

    // 遍历要改变的属性值并一一改变
    for(let property in propertiesContainer) {
        // 拿到该属性当前值，一开始是 startValue
        const tween = propertiesContainer[property]
        // 如果动画执行完成
        if (percentComplete === 1) {
            currentValue = tween.endValue
        } else {
            currentValue = parseFloat(tween.startValue) + ((tween.endValue - tween.startValue) * Animation.easing['swing'](percentComplete))
            tween.currentValue = currentValue
        }
        // 改变 dom 的属性值
        setPropertyValue(element, property, currentValue + tween.unitType)
        // 终止调用 tick
        if (percentComplete === 1) {
            isTicking = false
        }

        if (isTicking) {
            requestAnimationFrame(tick)
        }
    }
}
```

透明度与宽度能够正确处理，但是角度却没有正确处理，因为并没有对`rotateZ`做特殊处理，实际并不能够直接给 DOM 设置`rotateZ`属性而需要设置`transform`属性。

### 改变角度

在调用`setPropertyValue`时，传入了`(element, 'rotateZ', 'xxdeg')`，为了职责分明，不在调用该函数前将`rotateZ`改变为`transform`，而是在`setPropertyValue`函数内部根据属性来判断究竟该怎么设置元素的属性值。

```javascript
function setPropertyValue(element, property, value) {
	let propertyName = property
	if (normalization[property]) {
		// 如果在 normalization 这个对象内，就表示这个属性是需要经过处理的
		propertyName = 'transform'
	}
}
```

有哪些属性是使用`transform`设置的呢，在源码 499 行附近

- rotate(X|Y|Z)
- scale(X|Y|Z)
- skew(X|Y)
- translate(X|Y|Z)

```javascript
function setPropertyValue (element, property, value) {
    let propertyName = property
    /********************
      声明需要额外处理的属性
    *********************/
    const transformProperties = [ "translateX", "translateY", "translateZ", "scale", "scaleX", "scaleY", "scaleZ", "skewX", "skewY", "rotateX", "rotateY", "rotateZ" ]
    const Normalizations = {
        registered: {}
    }
    for(let i = 0, len = transformProperties.length; i < len; i++) {
        const transformName = transformProperties[i]
        Normalizations.registered[transformName] = function (propertyValue) {
            return transformName + '(' + propertyValue + ')'
        }
    }
    let propertyValue = value
    // 判断是否需要额外处理
    if (Normalizations.registered[property]) {
        propertyName = 'transform'
        propertyValue = Normalizations.registered[property](value)
    }
    console.log(propertyName, propertyValue)
    element.style[propertyName] = propertyValue
}
```

能够正确动画，但是却很卡。。不过只需要将判断是否终止动画`tick`的判断拿到`for..in`循环外即可。

## 最终代码

```javascript
;(function (window) {
    /********************
      声明需要额外处理的属性
    *********************/
    const transformProperties = [ "translateX", "translateY", "translateZ", "scale", "scaleX", "scaleY", "scaleZ", "skewX", "skewY", "rotateX", "rotateY", "rotateZ" ]
    const Normalizations = {
        registered: {}
    }
    // 如果这个属性是需要额外处理的
    for(let i = 0, len = transformProperties.length; i < len; i++) {
        const transformName = transformProperties[i]
        Normalizations.registered[transformName] = function (propertyValue) {
            return transformName + '(' + propertyValue + ')'
        }
    }
    // 获取指定 dom 的指定属性值
    function getPropertyValue (element, property) {
        return window.getComputedStyle(element, null).getPropertyValue(property)
    }
    // 给指定 dom 设置值
    function setPropertyValue (element, property, value) {
        let propertyName = property
        let propertyValue = value
        // 判断是否需要额外处理
        if (Normalizations.registered[property]) {
            propertyName = 'transform'
            propertyValue = Normalizations.registered[property](value)
        }
        element.style[propertyName] = propertyValue
    }
    // 分割值与单位
    function separateValue (property, value) {
        // 只处理两种简单的情况，没有单位和单位为 px
        let unitType,
            numericValue
        // replace 是字符串的方法，如果是数值类型则没有 replace 方法
        numericValue = value.toString().replace(/[%A-z]+$/, function(match) {
            unitType = match
            return ""
        })
        // 如果没有获取到单位，就根据属性来获取
        function getUnitType (property) {
            if (/^(rotate|skew)/i.test(property)) {
                // 这两个属性值单位是 deg ，有点特殊
                return "deg"
            } else if (/(^(scale|scaleX|scaleY|scaleZ|opacity|alpha|fillOpacity|flexGrow|flexHeight|zIndex|fontWeight)$)|color/i.test(property)) {
                // 这些属性值都是没有单位的
                return ""
            } else {
                // 如果都没有匹配到，就默认是 px
                return "px"
            }
        }
        if (!unitType) {
            unitType = getUnitType(property)
        }
        return [ numericValue, unitType ]
    }
    /* ========================
     * 构造函数
    =========================*/
    function Animation (element) {
        this.element = element
    }
    // easing 缓动函数
    Animation.easing = {
        swing: function (a) {
            return .5 - Math.cos(a * Math.PI) / 2
        }
    }
    // 暴露的动画接口
    Animation.prototype.animation = function (propertiesMap) {
        const element = this.element
        // 默认参数
        const opts = {
            duration: 400
        }
        // 保存要改变的属性集合
        let propertiesContainer = {}
        for(let property in propertiesMap) {
            // 拿到开始值
            const startSeparatedValue = separateValue(property, getPropertyValue(element, property))
            const startValue = parseFloat(startSeparatedValue[0]) || 0
            const startValueUnitType = startSeparatedValue[1]
            // 结束值
            const endSeparatedValue = separateValue(property, propertiesMap[property])
            const endValue = parseFloat(endSeparatedValue[0]) || 0
            const endValueUnitType = endSeparatedValue[1]

            propertiesContainer[property] = {
                startValue,
                endValue,
                unitType: endValueUnitType
            }
        }
        let timeStart
        // 终止动画标志
        let isTicking = true
        // 核心动画函数
        function tick () {
            // 当前时间
            let timeCurrent = (new Date).getTime()
            // 遍历要执行动画的 element 元素，这里暂时只支持一个元素
            // 当前值
            // 如果 timeStart 是 undefined ，表示这是动画的第一次执行
            if (!timeStart) {
                timeStart = timeCurrent - 16
            }
            // 检测动画是否执行完毕
            const percentComplete = Math.min((timeCurrent - timeStart) / opts.duration, 1) 
            // 遍历要改变的属性值并一一改变
            for(let property in propertiesContainer) {
                // 拿到该属性当前值，一开始是 startValue
                const tween = propertiesContainer[property]
                // 如果动画执行完成
                if (percentComplete === 1) {
                    currentValue = tween.endValue
                } else {
                    currentValue = parseFloat(tween.startValue) + ((tween.endValue - tween.startValue) * Animation.easing['swing'](percentComplete))
                    tween.currentValue = currentValue
                }
                // 改变 dom 的属性值
                setPropertyValue(element, property, currentValue + tween.unitType)
            }
            // 终止调用 tick
            if (percentComplete === 1) {
                isTicking = false
            }
            if (isTicking) {
                requestAnimationFrame(tick)
            }
        }
        tick()
    }
    // 暴露至全局
    window.Animation = Animation
})(window)
```

## 总结

这次主要是实现了分割值与单位，同时简单的实现了同时改变多个属性的动画。仍存在很大缺陷，下篇笔记主要解决颜色值的改变与开始值结束值单位不一致这两个问题。

