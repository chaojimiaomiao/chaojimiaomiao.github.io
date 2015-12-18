---
layout: post
title:  "react优化编译器-JSX语法进阶"
date:   2015-12-14
tags: [react, 大前端]
comments: true
---

博主在学习react native的时候接触了JSX。JSX 是一个看起来很像 XML 的 JavaScript 语法扩展。React 可以用来做简单的 JSX 句法转换。

在我看来，前端语言纷繁复杂，Html、css、js。有没有一种统一的语法将它们整合起来，包含在一个共同的作用域中，写起来既像标签语言，又有 __namespace__ 的概念；并且当我们在学习的时候，只需学习统一的格式，类定义等等就能写好native应用。可以说，JSX 就是这样的一种语法。

如果你对 react native还不了解，想先有一个总体上的认识，可以下载我的ppt 文档《React Native全接触》 [pdf版本链接：]({{ site.url }}/downloads/react-native全接触.pdf)

###理解JSX

我们知道，JavaScript 中没有显式的命名空间，这就意味着一切都定义在全局作用域中。次引用一个变量时，JavaScript 会往上遍历整个全局作用域直到找到该变量。如果遍历完整个全局作用域仍然没有找到该变量，则抛出一个 ReferenceError 错误。

我们通常通过将JS 变量包含在一个block中来区隐式地区分不同的作用域。

然而，JSX 是有命名组件的。

###走进React的编译

[Babel](https://babeljs.io/repl/) 是一个转换编译器，内置React JSX扩展。现在的react项目用的就是Babel编译器。Babel的插件系统允许开发者自定义代码转换器并插入到编译过程。这些转换器会接收一棵抽象语法树，并在代码转换成可执行的JavaScript之前对其进行操作。这个编译器实现了
- 静态&运行时类型检查
- 闭包消除
- JavaScript“健康宏（hygienic macros）”
等一些优化插件。

针对react jsx的优化包括：
- 任意函数内联
- 函数复制
- 循环内不变代码外提
- 对element打tag


###优化编译器：像React元素一样复用常量值类型

　　自从0.14开始，facebook开始将ReactElements和它们的属性对象作为值类型来对待。举个例子，任何实例的值都相等，那实例就相等。这允许我们复用任何输入为深度不可改变或有效常值的ReactElement.

用这个方法来举例：

{% highlight javascript %}
function render() {
   return <div className="foo" />;
}
{% endhighlight %}

这个可以通过移动JSX到函数块外面来优化，因此每次调用都将返回同一个实例：

{% highlight javascript %}
var foo = <div className="foo" />;
function render() {
   return foo;
}
{% endhighlight %}

这种写法不仅帮助我们复用相同的对象，同时React将会自动帮我们协调组件之间的不一致，而无需我们手动`shouldComponentUpdate`。

####引用相等性

JavaScript中的对象拥有引用相等的性质。这意味着这种优化可以实际地改变代码的表现。任何你对render()函数的调用中使用了对象相等、或者在一个Map中将ReactElement作为键值，那么这种优化特性将会打破那个用例。所以不要依赖它。

这是ReactElements在语义上的改进对比。这很难强制，但是希望JavaScript未来的版本将有对自定义对象值相等的观念，使得这种行为被强制。

####什么是常量？

最简单的假设是整个表达式，包括所有的属性（和子属性），全都是字面量值类型的（strings, booleans, null, undefined or JSX），那么它的结果也是常量。

{% highlight javascript %}
function render() {
  return <div className="foo"><input type="checkbox" checked={true} /></div>;
}
{% endhighlight %}

如果在表达式中使用了一个变量，那么你必须首先确保它自那以后从未被改变。

{% highlight javascript %}
//方法返回方法
var Foo = require('Foo');
function createComponent(text) {
  return function render() {
    return <Foo>{text}</Foo>;
  };
}
{% endhighlight %}

假如变量从未被改变，那么将常量移动到更高一层的闭包将是安全的。你只能将它移动到一个所有变量都共享的作用域。

{% highlight javascript %}
var Foo = require('Foo');
function createComponent(text) {
  var foo = <Foo>{text}</Foo>;
  return function render() {
    return foo;
  };
}
{% endhighlight %}

####对象是一种常量吗？

任意对象并不认为是常量。转译器__永远__也不应该移动一个ReactElement的作用域，除非所有属性都是可变对象。React将会默默忽略更新，并改变其行为。

{% highlight javascript %}
function render() {
  return <div style={{ width: 100 }} />; // Not safe to reuse...
}
// ...because I might do:
render().props.style.width = 200;
expect(render().props.style.width).toBe(100);
{% endhighlight %}

假如一个对象是深度不可更改的（或者从未被改变），转译器可能只会移动它到对象产生或接收的域。

{% highlight javascript %}
function render() {
  var style = Object.freeze({ __proto__: null, width: 100 });
  return <div style={style} />; // Not safe to reuse...
}
// ...because I might do:
expect(render().props.style).not.toBe(render().props.style);

// However this is...
function createComponent(width) {
  var style = Object.freeze({ __proto__: null, width: +width });
  return function render() {
    return <div style={style} />; // ...safe to move this one level up
  };
}
{% endhighlight %}

这是因为任意对象在JavaScript中有参照的身份。然而假如一个语义上不可改变的对象被当成值相等，那么就可以把它当做一个值类型。举个例子，任何通过immutable-js创建的数据结构将被当成一个值属性，假如它深度不可改变。

####例外：ref="string"

不幸的是有一个例外。如果ref属性有一个潜在的字符串值。那么复用这个元素一直是不安全的。这是因为我们在创建阶段捕获了React的拥有者。这是一个不幸的工艺，我们在不断寻找各种改变refs语义的办法来修复它。

{% highlight javascript %}
//ref不传递
render() {
  // Neither of these...
  return <div ref="str" />;
  // ...are safe to reuse...
  return <div ref={possibleStringValue} />;
  // ...because they might contain a ref.
  return <div {...objectThatMightContainARef} />;
}
{% endhighlight %}

####Non-JSX

这可以运行在JSX上，React.createElement或者由React.createFactory创建的方法。

举个例子，假设这个方法调用将会产生一个常量ReactElement是安全的。

{% highlight javascript %}
var Foo = React.createFactory(FooClass);

function render() {
  return Foo({ bar: 1 });
}
{% endhighlight %}

因此重用是安全的：
{% highlight javascript %}
var Foo = React.createFactory(FooClass);
var foo = Foo({ bar: 1 }};
function render() {
  return foo;
}
{% endhighlight %}

####高级优化

你也可以想象更聪明的优化方案，通过缓存每个实例的元素在该实例上。这将允许视自约束方法为有效常量。

如果你跟踪纯函数，假如输入到纯函数的值为常量，你甚至可以视计算值为常量。

---------------------------------------------------------------------

上述优化所包含的几个点都在下面的例子中体现：

{% highlight javascript %}
function render() {
  return <div className="foo" />;
}
{% endhighlight %}
becomes

{% highlight javascript %}
"use strict";

var _ref = React.createElement("div", { className: "foo" });

function render() {
  return _ref;
}
{% endhighlight %}
and

{% highlight javascript %}
var Foo = require('Foo');
function createComponent(text) {
  return function render() {
    return <Foo>{text}</Foo>;
  };
}
{% endhighlight %}
becomes

{% highlight javascript %}
"use strict";

var Foo = require("Foo");
function createComponent(text) {
  var _ref = React.createElement(Foo, null, text);

  return function render() {
    return _ref;
  };
}
{% endhighlight %}
