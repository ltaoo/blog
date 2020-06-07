---
title: 写给 JSer 的 python 学习手册 - 数据类型篇
categories: Python
tags:
- python
date: 2018/1/21
---

基础语法之后，就需要了解数据类型了，大部分语言的数据类型都是相似的，字符串、数字、布尔值、数组对象等等，虽然不同语言可能本质上的实现不同，但在编程时的写法是相似的，只有在本质上有什么不同，就可以作为深入学习去了解了。

`JavaScript`中有七种数据类型

- String
- Number
- Boolean
- null
- undefined
- Symbol
- Object

`Python3`中有六个标准的数据类型

- String
- Number
- List
- Tuple
- Sets
- Dictionary

下面进行一一对比。

<!--more-->


## 字符串

和`JavaScript`一样，可以用单引号，也可以用双引号表示一个字符串。

```python
# python
foo = 'bar'
bar = "foo"
```

```javascript
// javascript
let foo = 'bar';
let bar = "foo";
```

### 模板字符串

```
name = 'ltaoo'
age = 25
print("My name is %s and age is %d"%(name, age))
```

对应

```javascript
const name = 'ltaoo';
const age = 25;
console.log(`My name is ${name} and age is ${age}`);
```

### 方法

同样的，`python`中的字符串也有方法可以进行调用。


| JavaScript | Python | 描述 |
| --- | --- | --- |
| charAt |  | 返回特定位置的字符 |
| charCodeAt |  | 返回表示给定索引的字符的 Unicode 的值 |
| codePointAt |  | 返回使用UTF-16编码的给定位置的值的非负整数 |
| concat |  | 连接两个字符串文本，并返回一个新的字符串 |
| includes |  | 判断一个字符串里是否包含其他字符串 |
| endWith | endswith | 判断一个字符串的结尾是否包含其他字符串中的字符 |
| indexOf | find | 从字符串对象中返回首个被发现的给定值的索引值，如果没有找到则返回-1 |
| lastIndexOf | rfind(str, beg=0,end=len(string)) | 从字符串对象中返回最后一个被发现的给定值的索引值，如果没有找到则返回-1 |
| localeCompare |  | 返回一个数字表示是否引用字符串在排序中位于比较字符串的前面，后面，或者二者相同 |
| match |  | 使用正则表达式与字符串相比较 |
| normalize |  | 返回调用字符串值的Unicode标准化形式 |
| padEnd | ljust(width) | 在当前字符串尾部填充指定的字符串， 直到达到指定的长度。 返回一个新的字符串 |
| padStart | rjust(width) | 在当前字符串头部填充指定的字符串， 直到达到指定的长度。 返回一个新的字符串 |
| repeat |str*n| 返回指定重复次数的由元素组成的字符串对象 |
| replace | replace(str1, str2,  num=string.count(str1)) | 被用来在正则表达式和字符串直接比较，然后用新的子串来替换被匹配的子串 |
| search |  | 对正则表达式和指定字符串进行匹配搜索，返回第一个出现的匹配项的下标 |
| slice |str[x:n]| 摘取一个字符串区域，返回一个新的字符串；`python`中虽然没有该方法，但有更简单的方式。 |
| split | split(str="", num=string.count(str)) | 通过分离字符串成字串，将字符串对象分割成字符串数组 |
| startWith | startswith(obj, beg=0,end=len(string)) | 判断字符串的起始位置是否匹配其他字符串中的字符 |
| substr |  | 通过指定字符数返回在指定位置开始的字符串中的字符 |
| substring |  | 返回在字符串中指定两个下标之间的字符 |
| toLocaleLowerCase |  | 根据当前区域设置，将符串中的字符转换成小写。对于大多数语言来说，toLowerCase的返回值是一致的 |
| toLocaleUpperCase |  | 根据当前区域设置，将字符串中的字符转换成大写，对于大多数语言来说，toUpperCase的返回值是一致的 |
| toLowerCase | lower | 将字符串转换成小写并返回 |
| toString |  | 返回用字符串表示的特定对象 |
| toUpperCase | upper | 将字符串转换成大写并返回 |
| trim | strip | 从字符串的开始和结尾去除空格；strip 实际是执行了 lstrip 和 rstrip，即去除左侧的空格与右侧的空格。 |
| valueOf |  | 返回特定对象的原始值 |
|  | capitalize | 把字符串的第一个字符大写 |
|  | center(width) | 返回一个原字符串居中,并使用空格填充至长度 width 的新字符串 |
|  | count(str, beg=0, end=len(string)) | 返回 str 在 string 里面出现的次数，如果 beg 或者 end 指定则返回指定范围内 str 出现的次数 |
|  | decode(encoding='UTF-8', errors='strict') | 以 encoding 指定的编码格式解码 string，如果出错默认报一个 ValueError 的 异 常 ， 除非 errors 指 定 的 是 'ignore' 或 者'replace' |
|  | encode(encoding='UTF-8', errors='strict') | 以 encoding 指定的编码格式编码 string，如果出错默认报一个ValueError 的异常，除非 errors 指定的是'ignore'或者'replace' |
|  | expandtabs(tabsize=8) | 把字符串 string 中的 tab 符号转为空格，tab 符号默认的空格数是 8 |
|  | format() | 格式化字符串 |
|  | index(str, beg=0, end=len(string)) | 跟find()方法一样，只不过如果str不在 string中会报一个异常 |
|  | isalnum() |<div>如果 string 至少有一个字符并且所有字符都是字母或数字则返回 True,否则返回 False</div>|
|  | isalpha() |<div>如果 string 至少有一个字符并且所有字符都是字母则返回 True,否则返回 False</div>|
|  | isdecimal() | 如果 string 只包含十进制数字则返回 True 否则返回 False |
|  | isdigit() | 如果 string 只包含数字则返回 True 否则返回 False |
|  | islower() | 如果 string 中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True，否则返回 False |
|  | isnumeric() | 如果 string 中只包含数字字符，则返回 True，否则返回 False |
|  | isspace() | 如果 string 中只包含空格，则返回 True，否则返回 False |
|  | istitle() | 如果 string 是标题化的(见 title())则返回 True，否则返回 False |
|  | isupper() | 如果 string 中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是大写，则返回 True，否则返回 False |
|  | join(seq) | 以 string 作为分隔符，将 seq 中所有的元素(的字符串表示)合并为一个新的字符串 |
|  | lstrip() | 截掉 string 左边的空格；JavaScript 中有该方法但不属于标准 |
|  | maketrans(intab, outtab]) | maketrans() 方法用于创建字符映射的转换表，对于接受两个参数的最简单的调用方式，第一个参数是字符串，表示需要转换的字符，第二个参数也是字符串表示转换的目标。 |
|  | partition(str) | 有点像 find()和 split()的结合体,从 str 出现的第一个位置起,把字符串 string 分成一个3元素的元组(string_pre_str,str,string_post_str),如果 string 中不包含str 则 string_pre_str == string. |
|  | rindex( str, beg=0,end=len(string)) | 类似于 index()，不过是从右边开始 |
|  | rpartition(str) | 类似于 partition()函数,不过是从右边开始查找 |
|  | rstrip() | 删除 string 字符串末尾的空格 |
|  | splitlines([keepends]) | 按照行('\r', '\r\n', \n')分隔，返回一个包含各行作为元素的列表，如果参数 keepends 为 False，不包含换行符，如果为 True，则保留换行符 |
|  | swapcase() | 翻转 string 中的大小写 |
|  | title() | 返回"标题化"的 string,就是说所有单词都是以大写开始，其余字母均为小写(见 istitle()) |
|  | translate(str, del="") |<div>根据 str 给出的表(包含 256 个字符)转换 string 的字符,</div><div>要过滤掉的字符放到 del 参数中</div>|
|  | zfill(width) | 返回长度为 width 的字符串，原字符串 string 右对齐，前面填充0 |
|  | isdecimal() | isdecimal()方法检查字符串是否只包含十进制字符。这种方法只存在于unicode对象 |

可以看到`python`中内置的方法数远大于`JavaScript`拥有的，不过常用的还是相同。从这里可以看出`Python`的`slogon`，内置更多的方法避免自己花时间实现。


## 布尔值

```python
# python
a = True
b = False
```

```javascript
// javascript
let a = true;
let b = false;
```

`python`中的布尔值是大写开头的，其他方面并没有太大的区别。

## 数字

```python
# python
var1 = 1
var2 = 1.0
var3 = -1
```


```javascript
// javascript

let var1 = 1;
let var2 = 1.0;
let var3 = -1;
```

在`python`中，数字分为

- 整型(int)
- 长整型(long integers)
- 浮点型(floating point real values)
- 复数(complex numbers)


相比之下`JavaScript`只有「数字」，且并不区分浮点型和整型。那`Python`这四种类型有明确的区别吗？

> `Python`和`JavaScript`都有（其实是所有语言）一个共同的问题，即`0.2 + 0.1`结果并非`0.3`。


### 方法

`Python`中的数字类型没有方法，而`JavaScript`有一些不常用的：

| JavaScript | Python | 描述 |
| --- | --- | --- |
| toFixed() |  | 用来去除指定数量的小数位数 |
| toLocaleString([locales [, options]]) |  |  |

貌似只有这个是常用的。


### 整数相除

```python
# python
var1 = 3/2
```

在`python2`中，得到的结果是`1`，如果希望得到`1.5`，两个数字中有一个需要是浮点类型。

```javascript
// javascript
let var1 = 3/2; // 1.5 正确!
```

## 数组

`JavaScript`中的数组在`Python`中称为「列表」。

```python
# python

list = ['a', 123, True];
```

对应的`JavaScript`版则是

```javascript
// javascript

let list = ['a', 123, true];
```

完全一毛一样啊！似不似，元素都可以是任意类型。

### 访问数组中的值

```python
# python

list = [1, 2, 3]
firstItem = list[0]
```


```javascript
// javascript

const list = [1, 2, 3];
const firstItem = list[0];
```

### 更新数组中的值

```python
# python

list = [1, 2, 3, 4]
list[1] = 5

# [1, 5, 3, 4]
```

```javascript
// javascript

const list = [1, 2, 3, 4];
list[1] = 5;

// [1, 5, 3, 4]
```


### 删除数组中的值

```python
# python

list = [1, 2, 3, 4]
del list[0]

# [2, 3, 4]
```

`JavaScript`中数组并没有该方法，但可以使用`delete`实现，而该关键字是用在对象上的，由于数组就是对象，所以也可以用在数组上。

```javascript
// javascript

const list = [1, 2, 3, 4];
delete list[0];

// [empty, 2, 3, 4]
```

结果和`Python`的并不一样，只是将值删掉了，但还有个「坑」在，其实就是`undefined`；事实上如果要实现和`Python`一样的效果，往往是使用`splice`方法。

```javascript
list.splice(0, 1);

// [2, 3, 4]
```

### 截取

也可以称为「切片」

```python
# python

list = [1, 2, 3, 4]

list2 = list[1:]
list3 = list[1:2]

# [2, 3, 4]
# [2]
```

`JavaScript`中则需要使用方法实现：

```javascript
// javascript

const list = [1, 2, 3, 4];
const list2 = list.slice(1);
const list3 = list.slice(1, 2);

// [2, 3, 4]
// [2]
```

同样是「左闭右开区间」，即包含左边的下标，不包含右边的下标，所以得到的值有`2`没有`3`。

### 操作符

#### 合并

和`JavaScript`很大不同，`Python`中的「列表」，是可以使用一些操作符的，比如`+`用来连接两个列表：

```python
list1 = [1]
list2 = [2]

list3 = list1 + list2

# [1, 2]
```

而`JavaScript`这么做，实际会将两个数组的字符串相加，得到`[object Array][object Array]`；如果要实现元素相加，则需要使用`concat`方法，或者解构：

```javascript
const list1 = [1];
const list2 = [2];
const list3 = list1.concat(list2);

// or
const list3 = [...list1, ...list2];
```

#### 重复

```python
# python

list = [1]

list2 = list * 4

# [1, 1, 1, 1]
```

`JavaScript`就没有这么方便了，甚至可以说很麻烦，需要自己实现函数。

### 方法


| JavaScript | Python | 描述 |
| --- | --- | --- |
| 修改器方法 |  | JavaScript 中下面这些方法会改变原数组 |
| pop() | pop(obj=list[-1]) | 删除数组的最后一个元素，并返回这个元素；python 可以删除任意下标的元素，默认最后一个。 |
| push() | append(obj) | 在数组的末尾增加一个或多个元素，并返回数组的新长度 |
| reverse() |reverse()| 颠倒数组中元素的排列顺序，即原先的第一个变为最后一个，原先的最后一个变为第一个 |
| shift() |  | 删除数组的第一个元素，并返回这个元素 |
| sort() |sort(func)| 对数组元素进行排序，并返回当前数组 |
| splice() |  | 在任意的位置给数组添加或删除任意个元素 |
| unshift() |  | 在数组的开头增加一个或多个元素，并返回数组的新长度 |
| 访问方法 |  | 不会改变原数组的方法 |
| concat() | extend(seq) | 返回一个由当前数组和其它若干个数组或者若干个非数组值组合而成的新数组 |
| join() |  | 连接所有数组元素组成一个字符串 |
| slice() |  | 抽取当前数组中的一段元素组合成一个新数组  |
| toString() |  | 返回一个由所有数组元素组合而成的字符串 |
| toLocaleString() |  | 返回一个由所有数组元素组合而成的本地化后的字符串 |
| indexOf() | index(obj) | 返回数组中第一个与指定值相等的元素的索引，如果找不到这样的元素，则返回 -1 |
| lastIndexOf() |  | 返回数组中最后一个（从右边数第一个）与指定值相等的元素的索引，如果找不到这样的元素，则返回 -1 |
| 迭代方法 |  |  |
| forEach() |  | 为数组中的每个元素执行一次回调函数 |
| every() |  | 如果数组中的每个元素都满足测试函数，则返回 true，否则返回 false |
| some() |  | 如果数组中至少有一个元素满足测试函数，则返回 true，否则返回 false |
| filter() |  | 将所有在过滤函数中返回 true 的数组元素放进一个新数组中并返回 |
| map() |  | 返回一个由回调函数的返回值组成的新数组 |
| reduce() |  | 从左到右为每个数组元素执行一次回调函数，并把上次回调函数的返回值放在一个暂存器中传给下次回调函数，并返回最后一次回调函数的返回值 |
| reduceRight() |  | 从右到左为每个数组元素执行一次回调函数，并把上次回调函数的返回值放在一个暂存器中传给下次回调函数，并返回最后一次回调函数的返回值 |
||count(obj)|统计某个元素在列表中出现的次数|
||insert(index, obj)|将对象插入列表；JavaScript可以使用 splice 模拟|
||remove(obj)|移除列表中某个值的第一个匹配项；JavaScript可以使用 splice 模拟|


相比`JavaScript`，`Python`中数组的方法就比较少了。

## 对象

> 字典是另一种可变容器模型，且可存储任意类型对象。字典的每个键值(key=>value)对用冒号(:)分割，每个对之间用逗号(,)分割，整个字典包括在花括号({})中

`Python`中的字典，即`JavaScript`中的对象。

```python
# python

dict = {'name': 'ltaoo', 'age': 7}
```

对比

```javascript
// javascript

const dict = {name: 'ltaoo', age: 7};
```

有一个很大不同，`Python`中的键也需要用单引号或双引号包裹，而`JavaScript`不需要。

如果不用引号包裹，表示这是一个变量，如：

```python
# python
key = 'name'
dict = { key: 'ltaoo' }
```

而`JavaScript`默认将键作为字符串处理，即使有同名的变量，仍然是字符串：

```javascript
// javascript

const key = 'name';
const dict = { key: 'ltaoo' };

// 这样写才是和 python 效果相同
dict = { [key]: 'ltaoo' };
```

### 访问对象中的值

```python
# python

dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'};
name = dict['Name']

# Zara
```

```javascript
const dict = {
    Name: 'Zara',
    Age: 7,
    Class: 'First',
};
const name = dict['Name'];
// 更常用的
name = dict.Name; 
```

如果访问一个不存在的键，比如`dict['Hello']`，`Python`会报`KeyError`，而`JavaScript`则返回`undefined`，并不会报错。


### 修改字典

#### 更新已存在的值

```python
# python

dict = { 'name': 'ltaoo', 'age', 18 }
dict['name'] = 'wuya'
```

```javascript
const dict = { name: 'ltaoo', age: 18 };
dict['name'] = 'wuya';
dict.name = 'wuya';
```

#### 增加键值对

```python
# python

dict['tel'] = '123123123'
```

```javascript
// javascript

dict.tel = '123123123';
```

#### 删除

```python
# python

dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'};
 
del dict['Name']; # 删除键是'Name'的条目
dict.clear();     # 清空词典所有条目
del dict;        # 删除词典
```

`JavaScript`没有直接的方法清空所有条目。

```javascript
// javascript

const dict = { Name: 'Zara', Age: 7, Class: 'First' };

delete dict.Name;
delete dict;
```

可以给`dict`重新赋值为`{}`等于变相实现清空所有条目，而`delete dict`本质上是`delete window.dict`，也是删除键值对。

### 方法


| JavaScript | Python | 描述 |
| --- | --- | --- |
| hasOwnProperty() |  | 返回一个布尔值 ，表示某个对象是否含有指定的属性，而且此属性非原型链继承的 |
| isPrototypeOf() |  | 返回一个布尔值，表示指定的对象是否在本对象的原型链中 |
| propertyIsEnumerable() |  | 判断指定属性是否可枚举 |
|  | clear() | 删除字典内所有元素 |
| {…obj} | copy() | 返回一个字典的浅复制 |
|  | fromkeys(seq[, val]) | 创建一个新字典，以序列 seq 中元素做字典的键，val 为字典所有键对应的初始值 |
|  | get(key, default=None) | 返回指定键的值，如果值不在字典中返回default值 |
|  | has_key(key) | 如果键在字典dict里返回true，否则返回false |
|  | items() | 以列表返回可遍历的(键, 值) 元组数组 |
| Object.keys(obj) | keys() | 以列表返回一个字典所有的键 |
|  | setdefault(key, default=None) | 和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default |
| Object.assign(obj1, obj2) | update(dict2) | 把字典dict2的键/值对更新到dict里 |
| Object.values() | values() | 以列表返回字典中的所有值 |

`Python`中字典的方法数远大于`JavaScript`的，但很多方法在`JavaScript`不是实例方法而是「静态方法」。

## 元组

`Python`中存在一种数据类型是`JavaScript`中没有的，但可以模拟，即「元组」。

> Python的元组与列表类似，不同之处在于元组的元素不能修改。元组使用小括号，列表使用方括号。


```python
# python

tup1 = ('physics', 'chemistry', 1997, 2000)
tup3 = "a", "b", "c", "d"

tup = (1,) # 只包含一个元素时，需要在元素后面添加逗号
```

在`JavaScript`使用`Object.freeze()`可以「冻结」一个对象，不能修改、删除该对象的属性，而数组其实也是对象，所有可以用该函数模拟元组。

```javascript
// javascript

const ary1 = [1, 2, 3];
const ary2 = Object.freeze(ary1);
ary2[0] = 4;

// ary2 === [1, 2, 3];
```

## Sets

> 集合（set）是一个无序不重复元素的序列。


```python
# python

sets = { '1', '2', '3' }
```

该数据类型可以进行集合运算，即交集、并集差集等。

`JavaScript`中新增的`Set`和该数据类型相同：

```javascript
// javascript

const set1 = new Set(['1', '2', '3']);
```

只能通过构造函数得到，没有字面量。

## 空

`Python`中有两个特殊的值

- Null
- None

使用`type(None)`返回`<class 'NoneType'>`，而`type(Null)`会报错`Null is not defined`（`null`也一样）。

`Null`表示「空值」，即这里没有值，查找了一些资料，提到从数据库中读取数据，如果没有值，要区分是「空字符串」还是`Null`。

也还是蛮复杂的。


## 类型判断

```python
# python
print(type(vari))
```

```javascript
// javascript
console.log(typeof vari);
```

## 类型转换

`python`中不存在自动转换，所以：

```python
# python
a = 1
b = '1'
print(a + b) # TypeError: must be str, not int
```

字符串与数值相加，会报错，所以需要手动转换。

### 其他类型转数字

```python
# python
a = '1'
b = int(a)
```

### 其他类型转布尔

```python
# python

a = 'a'
b = bool(a)
```

### 其他类型转字符

```python
# python
a = 1
b = str(a)
```

### 列表转元组

```python
# python

tup = tuple(list)
```

### 元组转为列表

```python
# python

list1 = list(tup)
```

## 总结

`Python`与`JavaScript`同样是脚本语言，在很多方面都是类似的，比如「万物皆对象」，`JavaScript`中适用，`Python`也是一样。

除去类似的部分，两者仍有很多不同的地方，例如`JavaScript`的异步编程`Python`似乎没有，而在元编程方面`Python`早已经非常成熟而`JavaScript`这方面属于刚起步。

在未来各语言是否会取长补短而变得越来越相似呢，有趣。

