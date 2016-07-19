# 六个俏皮的 ES6 技巧
Six nifty ES6 tricks 转载自 http://www.2ality.com/2016/05/six-nifty-es6-tricks.html

在这篇的 blog 中，博主 Dr. Axel Rauschmayer 展示了六个使用 ES6 新功能的技巧。在每小结的最后有提示其与他的 “[Exploring ES6](http://exploringjs.com/es6/)”（在线可免费阅读） 一书中关联的部分。

## 通过参数缺省值强制使用参数

ES6 参数缺省值是仅在他们实际被使用时才会执行的。这可以让你强制要求提供所需参数：

```javascript
/**
* Called if a parameter is missing and
* the default value is evaluated.
*/
function mandatory() {
    throw new Error('Missing parameter');
}
function foo(mustBeProvided = mandatory()) {
    return mustBeProvided;
}
```

这个函数只有在参数 mustBeProvided 缺失时才执行 mandatory() 。

来看看交互演示：

    > foo()
    Error: Missing parameter
    > foo(123)
    123

More information:

- Sect. “[Required parameters](http://exploringjs.com/es6/ch_parameter-handling.html#_required-parameters)” in “Exploring ES6”

## 通过 for-of 循环来遍历数组的索引和元素

forEach() 方法使你可以遍历数组的元素。它也给了你每个元素的索引值，要是你想要的话：

```javascript
var arr = ['a', 'b', 'c'];
arr.forEach(function (elem, index) {
    console.log('index = '+index+', elem = '+elem);
});
// Output:
// index = 0, elem = a
// index = 1, elem = b
// index = 2, elem = c
```

ES6 for-of 循环支持 ES6 迭代（通过 iterables 和 iterators）和解构。如果你将解构和新的数组方法 entries() 结合起来，你可以：

```javascript
const arr = ['a', 'b', 'c'];
for (const [index, elem] of arr.entries()) {
    console.log(`index = ${index}, elem = ${elem}`);
}
```

arr.entires() 返回一个包括一对索引和值的 iterable 对象。[ index, elem ] 的解构方式让我们能直接访问其每一对的组成部分。console.log() 的参数就是个所谓的模板文本，他让字符串插入 JavaScript 语句。

More information:

- Chap. “[Destructuring](http://exploringjs.com/es6/ch_destructuring.html)” in “Exploring ES6”
- Chap. “[Iterables and iterators](http://exploringjs.com/es6/ch_iteration.html)” in “Exploring ES6”
- Sect. “[Iterating with a destructuring pattern](http://exploringjs.com/es6/ch_for-of.html#_iterating-with-a-destructuring-pattern)” in “Exploring ES6”
- Chap. “[Template literals](http://exploringjs.com/es6/ch_template-literals.html)” in “Exploring ES6”

## 迭代 unicode 编码文本

一些 Unicode 编码文本（简单说是字符）包含两个 JavaScript 字符。举例说，表情文字：

字符串是一种 ES6 iteration 的实现。如果你迭代他们，你会得到编码后的字符（一或两个 JavaScript 字符）。举个例子：

```javascript
for (const ch of 'x\uD83D\uDE80y') {
    console.log(ch.length);
}
// Output:
// 1
// 2
// 1
```

这里有个办法可以数清楚字符串里的文本个数。

```javascript
> [...'x\uD83D\uDE80y'].length
3
```

展开操作符（ … ）把他们放入了数组中的一个操作对象里。

More information:

- Chap. “[Unicode in ES6](http://exploringjs.com/es6/ch_unicode.html)” in “Exploring ES6”
- Sect. “[The spread operator (...)](http://exploringjs.com/es6/ch_parameter-handling.html#sec_spread-operator)” in “Exploring ES6”

## 通过解构交换两个变量值

如果你把两个变量放进一个数组，然后解构这个数组再到这两个变量，你可以省去一个没必要的变量来交换他们的值。

```javascript
[a, b] = [b, a];
```

这个技巧在未来有可能得到 JavaScript 引擎的优化，来让其中过程并无数组创建。

More information:

- Chap. “[Destructuring](http://exploringjs.com/es6/ch_destructuring.html)” in “Exploring ES6”

## 通过模板文本来创建简单的模板

ES6 模板文本比起传统的文本模板更接近字符串模板。不过如果你使用函数来返回他们，你可以把它们用作文本模板。

```javascript
const tmpl = addrs => `
    <table>
    ${addrs.map(addr => `
        <tr><td>${addr.first}</td></tr>
        <tr><td>${addr.last}</td></tr>
    `).join('')}
    </table>
  `;
```

函数 temple() 映射一个 address 数组成为一个字符串。让我们来用 tmpl() 在数组数据上来试试：

```javascript
const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];
console.log(tmpl(data));
// Output:
// <table>
//
//    <tr><td><Jane></td></tr>
//    <tr><td>Bond</td></tr>
//
//    <tr><td>Lars</td></tr>
//    <tr><td><Croft></td></tr>
//
// </table>
```

More information:

- Blog post “[Handling whitespace in ES6 template literals](http://www.2ality.com/2016/05/template-literal-whitespace.html)”
- Sect. “[Text templating via untagged template literals](http://exploringjs.com/es6/ch_template-literals.html#_text-templating-via-untagged-template-literals)” in “Exploring ES6”
- Chap. “[Arrow functions](http://exploringjs.com/es6/ch_arrow-functions.html)” in “[Exploring ES6]()”

## 通过子类工厂来实现 mixins

如果一个 ES6 类继承自另一个类，而这个类被一个表达式动态指定的话（不是由标识符静态指定）：

```javascript
// Function id() simply returns its parameter
const id = x => x;

class Foo extends id(Object) {}
```

这允许你能用函数实现一个 mixin ，他映射一个 C 类到一个新的带着 mixin 方法的 C 的子类。举例，下面的两个函数 Storage 和 Validation 是 mixins：

```javascript
const Storage = Sup => class extends Sup {
    save(database) { ··· }
};
const Validation = Sup => class extends Sup {
    validate(schema) { ··· }
};
```

你可以用他们来构成一个 Employee 类如下：

```javascript
class Person { ··· }
class Employee extends Storage(Validation(Person)) { ··· }
```



More information:

- Sect. “[Simple mixins](http://exploringjs.com/es6/ch_classes.html#_simple-mixins)” in “Exploring ES6”

Further reading

Two chapters of “Exploring ES6” give a good overview of ECMAScript 6:

- [An overview of what’s new in ES6](http://exploringjs.com/es6/ch_overviews.html)
- [First steps with ECMAScript 6](http://exploringjs.com/es6/ch_first-steps.html) [features that are easy to adopt]
