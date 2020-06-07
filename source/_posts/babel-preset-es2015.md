---
title: babel-preset-es2015 详解
categories: JavaScript
tags:
	- babel
date: 2019/07/11
---



`babel-preset-es2015` 是能将 `es2015` 新特性转换成 `es3` 的  `babel plugin` 的集合，这些集合到底包含了哪些插件呢，这篇博客会一一列出。



<!--more-->

通过源码，可以直接看到引入了哪些插件，共计 24 个

1. babel-plugin-transform-es2015-template-literals
2. babel-plugin-transform-es2015-literals
3. babel-plugin-transform-es2015-function-name
4. babel-plugin-transform-es2015-arrow-functions
5. babel-plugin-transform-es2015-block-scoped-functions
6. babel-plugin-transform-es2015-classes
7. babel-plugin-transform-es2015-object-super
8. babel-plugin-transform-es2015-shorthand-properties
9. babel-plugin-transform-es2015-duplicate-keys
10. babel-plugin-transform-es2015-computed-properties
11. babel-plugin-transform-es2015-for-of
12. babel-plugin-transform-es2015-sticky-regex
13. babel-plugin-transform-es2015-unicode-regex
14. babel-plugin-check-es2015-constants
15. babel-plugin-transform-es2015-spread
16. babel-plugin-transform-es2015-parameters
17. babel-plugin-transform-es2015-destructuring
18. babel-plugin-transform-es2015-block-scoping
19. babel-plugin-transform-es2015-typeof-symbol
20. babel-plugin-transform-es2015-modules-commonjs
21. babel-plugin-transform-es2015-modules-systemjs
22. babel-plugin-transform-es2015-modules-amd
23. babel-plugin-transform-es2015-modules-umd
24. babel-plugin-transform-regenerator



这是 `babel-preset-es2015@6.24.1` 版本所引入的源码，如果使用 `babel@7.x`，这些插件也还是存在，但名字去掉了 `es2015`。下面以我认为的重要程度一一了解



## babel-plugin-transform-arrow-functions

箭头函数转换，不是单纯将箭头语法做了转换，还会处理 `this` 值。



```js
const foo = () => {};
const bar = {
    name: 'ltaoo',
    say: () => {
        console.log(this.name);
    },
    hello() {
        console.log('hello' + this.name);
    },
};
```



编译成



```js
var _this = this;

const foo = function () {};

const bar = {
  name: 'ltaoo',
  say: function () {
    console.log(_this.name);
  },

  hello() {
    console.log('hello' + this.name);
  }

};
```



箭头函数内部的 `this` 值会被替换，保证了 `this` 指向不改变。

如果有人问，箭头函数使用 `.call` 会改变 `this` 指向吗？答案是不会。



## babel-plugin-transform-block-scoping

使用 `{}` 可以形成块级作用域，块级作用域的目的是为了不影响外部作用域。



```js
let foo = 'foo';
{
    let foo = 'scope foo';
    const bar = 'bar';
}
```



块级作用域外部 `foo` 仍然是 `foo`，内部 `foo` 变成了 `_foo` ，保证了不发生覆盖。



```js
var foo = 'foo';
{
  var _foo = 'scope foo';
  var bar = 'bar';
}
```



编译后的代码似乎有问题，`bar` 仍然可以在外部访问到。这是因为外部作用域没有使用到 `bar`，如果代码是



```js
let foo = 'foo';
{
    let foo = 'scope foo';
    const bar = 'bar';
}
console.log(bar);
```



结果就和预期一致了，块级作用域内部的 `bar` 变量也被转换为 `_bar`。



```js
var foo = 'foo';
{
  var _foo = 'scope foo';
  var _bar = 'bar';
}
console.log(bar);
```



并且该插件还用于 `const`、`let` 关键字转化为 `var`，但在 `babel@6.x` 时，需要配合 `check-es2015-constants` 插件才能正确处理 `const` 关键字重新声明。而在 `babel@7.x` 就不需要 `check-es2015-constants` 插件了。

但两者还是有一些区别，`babel@6.x` 会**在编译时报错**，而 `babel@7.x` 并不会报错，而是编译成



```js
function _readOnlyError(name) { throw new Error("\"" + name + "\" is read-only"); }
var foo = 'a';
foo = (_readOnlyError("foo"), 'b')
```



运行时才会报错。



## babel-plugin-transform-spread

展开运算符，但在 `es2015` 规范中，只能用于展开数组，对象的展开还处于草案阶段，需要使用 `babel-plugin-transform-object-rest-spread` 插件。



```js
// 数组展开
const foo = [...[1, 2, 3]];
// 数组收集
const [a, ...rest] = foo;
// 对象展开
const bar = {
    ...{
        a: 'a',
        b: 'b',
    },
};
// 对象收集
const { b, ...restProps } = bar;
```



这里有四个语法，但只有数组展开能被该插件处理，编译成



```js
// 数组展开
const foo = [1, 2, 3].concat();
// 数组收集
const [a, ...rest] = foo;
// 对象展开
const bar = { ...{
    a: 'a',
    b: 'b'
  }
};
// 对象收集
const {
  b,
  ...restProps
} = bar;
```



## babel-plugin-transform-destructuring

解构语法支持，日常开发中最常用的语法，同样是上面的例子，能够正确处理数组收集和对象收集。



```js
function _objectWithoutProperties(source, excluded) {
    if (source == null) return {};
    var target = _objectWithoutPropertiesLoose(source, excluded);
    var key, i;
    if (Object.getOwnPropertySymbols) {
        var sourceSymbolKeys = Object.getOwnPropertySymbols(source);
        for (i = 0; i < sourceSymbolKeys.length; i++) {
            key = sourceSymbolKeys[i];
            if (excluded.indexOf(key) >= 0) continue;
            if (!Object.prototype.propertyIsEnumerable.call(source, key)) continue;
            target[key] = source[key];
        }
    }
    return target;
}

function _objectWithoutPropertiesLoose(source, excluded) {
    if (source == null) return {};
    var target = {};
    var sourceKeys = Object.keys(source);
    var key, i;
    for (i = 0; i < sourceKeys.length; i++) {
        key = sourceKeys[i];
        if (excluded.indexOf(key) >= 0) continue;
        target[key] = source[key];
    }
    return target;
}

// 数组展开
const foo = [...[1, 2, 3]];
// 数组收集(解构)
const a = foo[0], rest = foo.slice(1);
// 对象展开
const bar = { ...{
    a: 'a',
    b: 'b'
  }
};
// 对象收集(解构)
const b = bar.b, restProps = _objectWithoutProperties(bar, ["b"]);
```



但是对象展开语法需要 `@babel/plugin-proposal-object-rest-spread` 支持。



> 还有个 @babel/plugin-syntax-object-rest-spread 插件，名字看起来差不多，但该插件无法处理对象展开。查看源码发现 proposal-object-rest-spread 引用了 syntax-object-rest-spread。



## babel-plugin-transform-computed-properties

对象属性支持变量



```js
const foo = 'a';
const bar = {
    [foo]: 'b',
};
```



编译成



```js
function _defineProperty(obj, key, value) { if (key in obj) { Object.defineProperty(obj, key, { value: value, enumerable: true, configurable: true, writable: true }); } else { obj[key] = value; } return obj; }

const foo = 'a';

const bar = _defineProperty({}, foo, 'b');
```



## babel-plugin-transform-shorthand-properties

对象属性简写



```js
const name = 'Hello';
const obj = {
	name,
	foo() {
		console.log(this.name);
	},
};
```



编译成



```js
const name = 'Hello';
const obj = {
	name: name,
	foo: function () {
		console.log(this.name);
	}
};
```



## babel-plugin-transform-classes

类语法，更简单的方式写继承



```js
class Animal {

}
class Cat extends Animal {
	constructor() {
        super();
		this.name = 'Hello World';
	}
}
```



编译成



```js
function _possibleConstructorReturn(self, call) {
	if (!self) {
		throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
	}
	return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

function _inherits(subClass, superClass) {
	if (typeof superClass !== "function" && superClass !== null) {
		throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
	}
	subClass.prototype = Object.create(superClass && superClass.prototype, {
		constructor: {
			value: subClass,
			enumerable: false,
			writable: true,
			configurable: true,
		},
	}); 
	if (superClass) {
		Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
	}
}

function _classCallCheck(instance, Constructor) {
	if (!(instance instanceof Constructor)) {
		throw new TypeError("Cannot call a class as a function");
	}
}

let Animal = function Animal() {
	_classCallCheck(this, Animal);
};

let Cat = function (_Animal) {
	_inherits(Cat, _Animal);

	function Cat() {
		_classCallCheck(this, Cat);

		var _this = _possibleConstructorReturn(this, (Cat.__proto__ || Object.getPrototypeOf(Cat)).call(this));

		_this.name = 'Hello World';
		return _this;
	}

	return Cat;
}(Animal);
```

## babel-plugin-transform-parameters

函数参数支持默认值、收集运算符



```js
function foo(a, b = 'b', { c }, ...rest) {
    console.log(a, b, c, rest);
}
```



编译成



```js
function foo(a) {
  let b = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 'b';
  let { // 这里还是解构语法
    c
  } = arguments.length > 2 ? arguments[2] : undefined;

  for (var _len = arguments.length, rest = new Array(_len > 3 ? _len - 3 : 0), _key = 3; _key < _len; _key++) {
    rest[_key - 3] = arguments[_key];
  }

  console.log(a, b, c, rest);
}
```



可以发现，只有结构语法不被转换，加上 `@babel/plugin-transform-destructuring` 插件后，编译成



```js
function foo(a) {
  let b = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 'b';

  let _ref = arguments.length > 2 ? arguments[2] : undefined,
      c = _ref.c; // 能够处理解构

  for (var _len = arguments.length, rest = new Array(_len > 3 ? _len - 3 : 0), _key = 3; _key < _len; _key++) {
    rest[_key - 3] = arguments[_key];
  }

  console.log(a, b, c, rest);
}

```

## babel-plugin-transform-template-literals

模板字符串



```js
const foo = 'world';
const bar = `Hello ${foo}`;
const str = `
	Hello World;
`;
```

编译成



```js
const foo = 'world';
const bar = 'Hello ' + foo;
const str = '\n\tHello World;\n'
```



## babel-plugin-transform-modules-xxx

将 `export default` 这种语法(怎么称呼？标准模块机制？)转换为其他模块化代码



```js
import foo from './foo';
export default 1;
```



### amd



```js
define(['exports', './foo'], function (exports, _foo) {
	'use strict';

	Object.defineProperty(exports, "__esModule", {
		value: true
	});
    
    var _foo2 = _interopRequireDefault(_foo);

	function _interopRequireDefault(obj) {
		return obj && obj.__esModule ? obj : {
			default: obj
		};
	}
    
    exports.default = 1;
});
```



### umd



```js
(function (global, factory) {
	if (typeof define === "function" && define.amd) {
		define(['exports', './foo'], factory);
	} else if (typeof exports !== "undefined") {
		factory(exports, require('./foo'));
	} else {
		var mod = {
			exports: {}
		};
		factory(mod.exports, global.foo);
		global.test = mod.exports;
	}
})(this, function (exports, _foo) {
	'use strict';

	Object.defineProperty(exports, "__esModule", {
		value: true
	});

	var _foo2 = _interopRequireDefault(_foo);

	function _interopRequireDefault(obj) {
		return obj && obj.__esModule ? obj : {
			default: obj
		};
	}
    
    exports.default = 1;
});
```



### commonjs



```js
'use strict';

Object.defineProperty(exports, "__esModule", {
	value: true
});


var _foo = require('./foo');

var _foo2 = _interopRequireDefault(_foo);

function _interopRequireDefault(obj) {
    return obj && obj.__esModule ? obj : { default: obj };
}

exports.default = 1
```



### systemjs



```js
System.register(['./foo'], function (_export, _context) {
	"use strict";
    
    var foo;
    return {
		setters: [function (_foo) {
			foo = _foo.default;
		}],
        execute: function () {
            _export('default', 1);
        },
    };
});
```



## babel-plugin-transform-regenerator

支持 `generator` 语法



```js
function* foo() {
    yield 1;
}
```



编译成



```js
var _marked = /*#__PURE__*/regeneratorRuntime.mark(foo);

function foo() {
	return regeneratorRuntime.wrap(function foo$(_context) {
		while (1) switch (_context.prev = _context.next) {
			case 0:
				_context.next = 2;
				return 1;

			case 2:
			case 'end':
				return _context.stop();
		}
	}, _marked, this);
}
```





## babel-plugin-transform-for-of

这里开始是一些比较少用的语法



```js
const obj = {
	a: 'a',
	b: 'b',
};
for (const key of obj) {
	console.log(key);
}
```



编译成



```js
const obj = {
	a: 'a',
	b: 'b'
};
var _iteratorNormalCompletion = true;
var _didIteratorError = false;
var _iteratorError = undefined;

try {
	for (var _iterator = obj[Symbol.iterator](), _step; !(_iteratorNormalCompletion = (_step = _iterator.next()).done); _iteratorNormalCompletion = true) {
		const key = _step.value;

		console.log(key);
	}
} catch (err) {
	_didIteratorError = true;
	_iteratorError = err;
} finally {
	try {
		if (!_iteratorNormalCompletion && _iterator.return) {
			_iterator.return();
		}
	} finally {
		if (_didIteratorError) {
			throw _iteratorError;
		}
	}
}
```



## babel-plugin-transform-object-super

支持调用超类



```js
const foo = {
	say() {
		return super.say() + 'World';
	},
};
```



编译成



```js
var _get = function get(object, property, receiver) {
	if (object === null) {
		object = Function.prototype;
		var desc = Object.getOwnPropertyDescriptor(object, property);
		if (desc === undefined) {
			var parent = Object.getPrototypeOf(object);
			if (parent === null) {
				return undefined;
			} else {
				return get(parent, property, receiver);
			}
		} else if ("value" in desc) {
			return desc.value;
		} else {
			var getter = desc.get;
			if (getter === undefined) {
				return undefined;
			}
			return getter.call(receiver); 
		}
	}
};

const foo = _obj = {
  say() {
    return _get(_obj.__proto__ || Object.getPrototypeOf(_obj), 'say', this).call(this) + 'World';
  }
};
```



## babel-plugin-transform-duplicate-keys

支持写重复的键，忘记了在旧版本中如果对象有重复的键会怎么样了，但使用 `eslint` 的情况下，有重复键会报错的。



```js
const obj = {
	a: 'a',
	a: 'b',
	a: 'c',
};
```



编译成



```js
const obj = {
	a: 'a',
	['a']: 'b',
	['a']: 'c'
};
```



 `obj.a === 'c'` ，支持重复的 `key` 有什么意义呢？



## babel-plugin-transform-sticky-regex



```js
const regexp = /foo/y;
```



编译成



```js
const regexp = new RegExp("foo", "y");
```

## babel-plugin-transform-unicode-regex



```js
const string = "foo💩bar";
const match = string.match(/foo(.)bar/u);
```



编译成



```js
const string = "foo💩bar";
const match = string.match(/foo((?:[\0-\t\x0B\f\x0E-\u2027\u202A-\uD7FF\uE000-\uFFFF]|[\uD800-\uDBFF][\uDC00-\uDFFF]|[\uD800-\uDBFF](?![\uDC00-\uDFFF])|(?:[^\uD800-\uDBFF]|^)[\uDC00-\uDFFF]))bar/);
```



## babel-plugin-transform-typeof-symbol



```js
typeof Symbol() === 'symbol';
```



编译成



```js
var _typeof = typeof Symbol === "function"
	&& (
		typeof Symbol.iterator === "symbol"
			? function (obj) { return typeof obj; }
			: function (obj) {
				return obj
					&& typeof Symbol === "function"
					&& obj.constructor === Symbol
					&& (
						obj !== Symbol.prototype
							? "symbol"
							: typeof obj
					);
			}
	);
_typeof(Symbol()) === 'symbol';
```



## babel-plugin-transform-literals

编码转换



```js
const u = 'Hello\u{000A}\u{0009}!';
```



编译成



```js
const u = 'Hello\n\t!';
```



## babel-plugin-transform-block-scoped-functions

从名字来看，是块级作用域函数，是指什么呢？

是让块级作用域内声明的函数只作用域块级作用域。



```js
{
  function name (n) {
    return n;
  }
}

name("Steve");
```



编译成



```js
{
  let name = function (n) {
    return n;
  };
}
name("Steve");
```



感觉和 `block-scoping` 有点类似，块级作用域内的变量不被外部使用？如果不用 `block-scoped-functions` 而仅用 `block-scoping` 会怎么样呢？



```js
{
  function _name(n) {
    return n;
  }
}
name("Steve");
```



`_name` 函数对外部作用域还是可知的，有一些区别。



## babel-plugin-transform-function-name

再是一些不算新特性的插件吧，比如这个给匿名函数增加函数名



```js
const foo = function () {}
```



编译成



```js
const foo = function foo() {};
```



## 总结

可以发现，真正有用的插件大概有 10 个，但每次都会安装全部的，即使使用 `@babel/preset-env`，它也仅仅是根据浏览器可以去掉一些插件，但如果我们能根据项目中实际使用的语法，只配置 `plugin`，才是最精简的方式。

但首先要求对项目中所使用的语法，以及新特性对应的插件非常了解，才能做到有的放矢。