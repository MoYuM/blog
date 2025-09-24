---
title: '为什么这样写——classnames'
pubDate: '2025-05-01'
description: '暂无摘要'
---

# 为什么这样写——classnames


# 简介

classnames 能帮助我们快速且灵活的构建出一个className字符串。

其实同类的库还有 [Classcat](https://github.com/jorgebucaran/classcat)，[clsx](https://github.com/lukeed/clsx)

# 核心逻辑

利用 `arguments` 的特性——所有参数存放在一个类数组里，循环遍历每个参数。

不同的参数类型对应不同的动作。

- 数字/字符串 ⇒ 保留 `truthy` 的值
- 数组 ⇒ 再次调用 `classnames()` 做递归
- 对象 ⇒ 保留所有值为 `truthy` 的值，忽略其余

上面保留下来的值都会存入一个数组中。

最后会把这个数组转换成字符串作为最终结果返回。

# 一些问题

### 为什么不能 if (module && module.exports)

*小知识点*

因为这样会引用报错

需要这样写

```jsx
if (typeof module !== 'undefined' && module.exports) {
	...
}
```

### 为什么要检查hasOwnProperty

*小知识点*

检查一个对象中的属性是不是他自己的，而不是他继承来的，很容易理解。

例如有人在 `Object.prototype` 上挂载了自己的属性

但为什么要从别处引用 `hasOwnProperty` ?

```jsx
var hasOwn = {}.hasOwnProperty;

...

if (hasOwn.call(arg, key) && arg[key]) {
		classes.push(key);
}
```

因为这个对象有可能没有 `hasOwnProperty` 

[Why use Object.prototype.hasOwnProperty.call(myObj, prop) instead of myObj.hasOwnProperty(prop)?](https://stackoverflow.com/questions/12017693/why-use-object-prototype-hasownproperty-callmyobj-prop-instead-of-myobj-hasow/12017703#12017703)

### 为什么支持数字？

*小知识点*

很奇怪的功能，毕竟没人会把数字当作class名。也确实有人提了一个[issue](https://github.com/JedWatson/classnames/issues/239)询问此事。

最初版本中确实是不支持数字的，直到这个[issue](https://github.com/JedWatson/classnames/pull/8)。

这里就说明了为什么加上对数字的支持，仅仅因为readme中写的是"falsy keys won't be included"。

但类似数字 `1` 这种非 `falsy` 值却会被忽略。加入对数字的支持就是为了解决这个问题。

再说加上这个支持也不难，也没有额外的副作用，就一直留着了。

但现在有个问题是：真的不能拿数字当作class名么？

答案是肯定的。

首先在 html 中，是可以给class一个数字的，不会报错。

在 CSS 中虽然允许类名中存在数字，但不允许以数字开头。

必须使用转译符 `\` 。

最终代码这样写

```css
.\36\36\36 {
	color: red;
}
```

[CSS标准](https://www.w3.org/TR/CSS2/syndata.html#value-def-identifier)

### 为什么要返回undefined？

*为什么不接受这个PR*

[return undefined for empty class list by jakubsadura · Pull Request #183 · JedWatson/classnames](https://github.com/JedWatson/classnames/pull/183)

考虑下面一种情况

```jsx
const classname = classnames({'foo': false})

return (
	<div className={classname}>Hello,world</div>
)
```

结果为

```html
<div class>Hello,world</div>
```

class 属性依然存在，因为这时 classnames 返回的是 `''` 。

而如果返回的是 `undefined` ，则结果会变成。

```html
<div>Hello,world</div>
```

这样的 html 没有了多余的属性，看起来更加简洁了

那要不要提个 PR 改进一下呢？只要在最终返回结果的时候过滤一下就行了，很简单。

但真的要这样改么？

作者最终给出的回复是：[不要改](https://github.com/JedWatson/classnames/pull/183#issuecomment-470597872)。主要原因在于：**要尽量避免预期外的行为**

而对于classnames这样一个已经被大量使用的库，稳定性和安全性是优先考虑的。这样一个 **breaking change** 显然是不可接受的。

`classnames()` 会从**始终**返回一个字符串，变成**可能**返回一个字符串。

如果真的出现 `undefined` 就将它强制转换为字符串，也就是 `'undefined'` ，是一个更好的方式

如果真的不想要一个空字符串，也可以自己把 `classnames` 包裹一下

```jsx
const cx = args => classnames(...args) || undefined;
```

### 为什么要检查 toString()

*为什么接受这个PR*

[Ability to handle object with own `.toString()` method by resetko · Pull Request #170 · JedWatson/classnames](https://github.com/JedWatson/classnames/pull/170)

在最新版本的 classnames 中的我们可以看见这样的代码

```jsx
if (argType === 'object') {

		// 这一层判断是做什么的？
		if (arg.toString === Object.prototype.toString) {
				for (var key in arg) {
						if (hasOwn.call(arg, key) && arg[key]) {
							classes.push(key);
						}
				}
		} else {
					classes.push(arg.toString());
		}
}
```

显然这里的意思是：如果传入的对象身上有一个自定义的 `toString()` 则自动调用 `toString()`

但为什么要这样写？有什么用？

想象这样一个场景：我在使用一个可以生成类名的库，类似于这样

```tsx
class MyClassName {
    constructor(mainName) {
      this.name = mainName;
    }

    el(element) {
      this.name = this.name + '_' + element;
      return this
    }

    toString(){
      return this.name;
    }
  }

const className = new MyClassName('hbc');

return (
    <button className={className.el('btn')}>
      hello,world
    </button>
);
```

这样的代码结果是

```jsx
<button class="hbc_btn">hello,world</button>
```

我并没有显式的调用 `toString()` ，但我却拿到了预期的结果。

那是因为 React 在这里做了一次隐式转换

```tsx
// `setAttribute` with objects becomes only `[object]` in IE8/9,
// ('' + value) makes it output the correct toString()-value
attributeValue = '' + value;
```

JS 会自动调用 `toString()` 以求得到一个字符串的结果

所以在React中如果传一个对象到 className 会得到什么？

```tsx
const obj = {}

return (
	<div className={obj}>Hello,world</div>
)
```

得到

```tsx
<div class="[object Object]">Hello,world</div>
```

不会报错

但最初的 classnames 并没有考虑这一点。如果使用 classnames 的话，必须显式的调用 `toString()` 。

很多的 UI 库都依赖于 classnames ，比如 ant-design、Semantic UI...

所以，当你期望像使用 React 原生组件那样，使用这类 UI 库组件的时候，你就得不到想要的结果。

[className with object containing toString() method isn't working as expected · Issue #2599 · Semantic-Org/Semantic-UI-React](https://github.com/Semantic-Org/Semantic-UI-React/issues/2599)

所以，这样的改变能让各类使用了 classnames 的UI库，让其组件的行为能和 React 原生组件保持一致。

### 为什么需要 `dedupe.js`

*为什么接受这个PR*

[Allow replacing of previous class names · Issue #18 · JedWatson/classnames](https://github.com/JedWatson/classnames/issues/18)

想象这样一个场景

```tsx
classNames('foo', 
	{ 
		foo: false,
		bar: true 
	}
)
```

输出的结果会是 `'foo bar'` ，对象中的 `foo: false` 没有覆盖掉前面的 `'foo'` 。

想要实现这个功能，必须采用新的思路，但新思路的运行速度没法像原来一样快。最初版本的 `dedupe.js` 比原始函数**慢10倍**。

而这种性能上的差异对于 classnames 来说是不可接受的。

我们的工具函数需要一个新功能，但想实现新功能需要重构这个函数，而且还会降低性能。

我们又不想影响原本的函数，该如何是好？

解决方法其实很简单，写一个新的函数就行了。

classnames 加入了一个新的导出选项，如果有去重需要的使用者，自行导出 `dedupe.js` 就行了。这样就**不会影响原始函数**的性能表现，还能给新用户提供去重功能。

```tsx
import classNames from 'classnames/dedupe';
```

# 性能优化

性能优化主要集中在 `dedupe.js` 中，毕竟这个函数是最慢的

最初的 `dedupe.js` 是这样的

```tsx
function () {
	'use strict';

	function _parseArray (resultSet, array) {
		var length = array.length;

		for (var i = 0; i < length; ++i) {
			_parse(resultSet, array[i]);
		}
	}

	function _parseObject (resultSet, object) {
		for (var k in object) {
			if (object.hasOwnProperty(k)) {
				if (object[k]) {
					resultSet[k] = true;
				} else {
					delete resultSet[k]
				}
			}
		}
	}

	function _parseNumber (resultSet, num) {
		resultSet[num] = true;
	}

	var SPACE = /\s+/;
	function _parseString (resultSet, str) {
		var array = str.split(SPACE);
		var length = array.length;

		for (var i = 0; i < length; ++i) {
			resultSet[array[i]] = true;
		}
	}
const argType = typeof arg

	function _parse (resultSet, arg) {
		// 'foo bar'
		if ('string' === argType) {
			_parseString(resultSet, arg)

		// ['foo', 'bar', ...]
		} else if (Array.isArray(arg)) {
			_parseArray(resultSet, arg)

		// { 'foo': true, ... }
		} else if ('object' === typeof arg) {
			_parseObject(resultSet, arg)

		// '130'
		} else if ('number' === typeof arg) {
			_parseNumber(resultSet, arg)
		}
	}

	function _classNames () {
		function add () {}
		add.prototype = Object.create(null);
		const obj = new add();
		var classes = '';
		var argLength = arguments.length;

		for (var i = 0; i < argLength; ++i) {
			_parse(classSet, arguments[i]);
		}

		for (var k in classSet) {
			if (classSet.hasOwnProperty(k) && classSet[k]) {
				classes += ' ' + k;
			}
		}

		return classes.substr(1);
	}

	return _classNames;
}
```

整体思路和现在一致，都是用一个对象当作缓存，再遍历每个参数，值为 truthy 则对应的键设置为 `true` 

- 删除值为 falsy 的键
    
    ```tsx
    if ('object' === typeof arg) {
    	for (var k in arg) {
    		if (arg.hasOwnProperty(k)) {
    			result[k] = arg[k];
    		} else {
    			
    			// 删掉不需要的值，这样在最后遍历缓存对象的时候就不用判断了
    			// 遍历次数也能少一些
    			delete result[k];
    		}
    	}
    }
    ```
    
    但对一个对象做删除操作会降低这个对象的性能
    
    ### hidden class
    
    在JS引擎中，引擎使用了hidden class优化对象，使其能快速的访问属性
    
    [JavaScript 引擎基础：Shapes 和 Inline Caches](https://zhuanlan.zhihu.com/p/38202123)
    
    在 JS 中，不同的对象可能拥有相同的**形状**
    
    ```tsx
    const object1 = { x: 1, y: 2 };
    const object2 = { x: 3, y: 4 };
    ```
    
    ![](/为什么这样写——classnames-4.png)
    
    如果为每个对象都完整的储存他们的属性名和属性值，那无疑是浪费空间的。
    
    为此，引擎将对象的**形状**单独储存
    
    ![](/为什么这样写——classnames-1.png)
    
    `shape` 包含除 `[[Value]]` 之外的所有属性名和其余特性。用对象值的偏移量 `Offest` 代替 `[[Value]]` 。这样引擎就知道去哪里查找具体值了。
    
    每个具有相同形状的对象都指向这个 `Shape` 事例。每个对象只需要存储对这个对象来说唯一的那些值。
    
    ![](/为什么这样写——classnames-2.png)
    
    当我们为对象添加新的属性时
    
    引擎会生成一个新的 `Shape` ，而对象也会转而指向这个新的 `Shape` 
    
    ![](/为什么这样写——classnames-3.png)
    
    每个 `Shape` 都与之前的 `Shape` 相连，这样，新的 `Shape` 就只需要保存新的属性值即可。旧属性可以沿着 `Shape` 链往回找就行了。
    
    `delete` 操作可能会导致引擎改变对象的结构，降级为哈希表储存方式。应尽量避免使用，就算没有改变对象结构，也会让引擎调用 `check` 方法，检查是否应该将对象结构降级。同样会影响性能。
    
    [Slow delete of object properties in JS in V8](https://stackoverflow.com/questions/43594092/slow-delete-of-object-properties-in-js-in-v8)
    
    [V8 是怎么跑起来的 -- V8 中的对象表示](https://juejin.cn/post/6844903833571688462)
    
    所以现在我们知道不应该使用 `delete` 了。所以目前最新的版本是这样的。
    
    ```tsx
    function _parseObject (resultSet, object) {
    	for (var k in object) {
    		if (hasOwn.call(object, k)) {
    			resultSet[k] = !!object[k];
    		}
    	}
    }
    ```
    
- 这里其实就是一次 `parseArray()`
    
    ```tsx
    var argLength = arguments.length;
    
    for (var i = 0; i < argLength; ++i) {
    	_parse(classSet, arguments[i]);
    }
    ```
    
    所以改为
    
    ```tsx
    _parseArray(classSet, arguments);
    ```
    
- 用字符串拼接还是数组？
    
    我们最终返回的其实就是一个字符串，完全可以用字符串拼接代替数组的操作
    
    [use [].join over concatenation by dcousens · Pull Request #50 · JedWatson/classnames](https://github.com/JedWatson/classnames/pull/50)
    
    这个PR中，作者做了性能上的测试，结果是：
    
    两者各有优劣
    
    作者拿不准主意，又找来 @jdalton ，他也拿不准主意，于是又找来一个大佬 [@mraleph](https://github.com/mraleph)。
    
    经过一番讨论，认为用 `join()` 和字符串拼接在性能上的差别取决于你的应用场景
    
    两者的差别其实不大，但使用 `join()` 更符合直觉，可读性上也更好一些。
    
    于是最终决定还是使用 `join()` 
    
    （其实在最开始的版本中就是用的 `join` 因为这确实更符合程序猿的习惯）
    
- 缓存 `argType`
- 更安全的 `hasOwnProperty` 检查
- 参数泄漏
    
    [](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers#32-leaking-arguments)
    
    不要将 `arguments` 直接传到另一个函数中，这会导致参数泄漏，让JS引擎放弃对函数的优化。
    
    所以需要自行构建一个数组出来
    
    ```tsx
    var len = arguments.length;
    var args = Array(len);
    for (var i = 0; i < len; i++) {
    	args[i] = arguments[i];
    }
    
    var classSet = {};
    _parseArray(classSet, args);
    ```
    
- 跳过 `hasOwnProperty` 检查
    
    `hasOwnProperty` 无非就是检查对象的上的属性到底是自己的还是，继承来的。那如果我们在开始创建对象的时候就不继承任何东西，那也就不用再做检查了
    
    [Is creating JS object with Object.create(null) the same as {}?](https://stackoverflow.com/questions/15518328/is-creating-js-object-with-object-createnull-the-same-as#answer-21079232)
    
    ```tsx
    var classSet = Object.create(null);
    ```
    
    这里还可以继续优化
    
    要注意的是，classnames 是一个递归函数，所以使用 `new` 创建一个继承自 `Object.create(null)` 的实例，要比一遍又一遍的调用 `Object.create(null)` 快
    

# 最后

最近的 classnames 已经很少更新了，让人怀疑这个仓库是不是已经 "死" 了

[Is this project dead? · Issue #220 · JedWatson/classnames](https://github.com/JedWatson/classnames/issues/220)

事实是，确实已经有后起之秀了，比如 classcat , clsx ，其中 clsx 的速度比 classnames 快差不多三倍，体积也更小。

那现在如果你是 antd 的作者，你可以一键将 antd 中所有依赖的 classnames 换成 clsx ，你换么？

![image.png](/为什么这样写——classnames-5.png)
