# JavaScript 插件开源项目 API 设计

作者 BRYAN BRAUN 原载 https://seesparkbox.com/foundry/api_patterns_for_your_open_source_javascript_plugin

我最近刚为我的 JavaScript 开源项目 1.0 版本重构了代码。这些代码一开始挤在一个全局函数里，但过不了多久就已经显得不够我添加功能了。我需要搞得好点。

在准备重构时，我寻找了许多成功的 JavaScript 插件，来看他们的 API 时如何设计的，做笔记。在这篇文章中我想要展示给你我发现的这些模式，以及每种模式的优缺点。

## Getting Started

开始之前，这里有些前提：

1. 这些 API 模式仅支持前端 JavaScript。Node 项目是另一回事。
2. 必须是“插件”。我们讨论的开源项目是 一些功能代码库。我没有收集完整应用的设计模式（如果你感兴趣， [这里有些不错的](http://addyosmani.com/resources/essentialjsdesignpatterns/book/) ）

我没有去找一定规模的项目，而是去注意各种项目规格：大型，中型还是小型。我对应对这些情况要用什么解决方案很感兴趣。

为了对比这些方案，我会使用一个虚构项目 `rainbowSparkles.js`。

还有，这是一次对 API 的探索，并不是设计模式的详细列表。我并不会提到设计模式产生的相似的 API。

## All right. Ready? Here we go.

### 1.Include and Go

有时最好的 API 是没有 API ！JS 里这意味着简单引入资源（通过 script 标签，模块加载器或者构建过程）就好，不用你去多写。这会对 polyfills 或其他没有配置的工具引用很有用。

```html
<script src=”/rainbowSparkles.js”></script>
```

一种实现的通用方式是在[立即调用函数表达式](http://benalman.com/news/2010/11/immediately-invoked-function-expression/)中运行你所有的代码。这样执行时会保证你的函数和变量在不在全局作用域中。

```javascript
// inside rainbowSparkles.js
(function() {
  // Run code here.
})();
```

This API format is used in:
这种 API 格式被用在了：

- [classLIst.js](https://github.com/eligrey/classList.js/)

### 2. Global Function

为用户提供一个可以简单运行的全局函数。你可以通过传递函数来提供少许定制功能。这很好很简单，不过如果你需要一大堆配置它就不太合适。

```javascript
// Add magic sparkles to all h1’s on the page.
rainbowSparkles(‘h1’);

// or…
// Crank the level of sparkles up to 11.
rainbowSparkles(11);
```

这种 API 格式用在了：

- [colorify.js](http://colorify.rocks/)
- [QS](https://github.com/ljharb/qs)
- [mediaCheck](https://github.com/sparkbox/mediaCheck)
- [jQuery](https://jquery.com/)

### 3. Constructor Interface

这种 API 提供一个用户可以构造新对象的全局函数。实例通过 new 关键词来创建并从同一个原型继承功能。

我发现到这种模式对 UI 组件（比如 tooltips，modals 之类）相当通用，因为它刚好满足开发者在页面上放置多个同样功能的组件的需求。

```javascript
var miniSparkles = new RainbowSparkles({ size: 2 });
var megaSparkles = new RainbowSparkles({ size: 9 });

miniSparkles.shower(1000);
megaSparkles.radiate(‘.call-to-action’);
```

用在了：

- [Awesomeplete](https://leaverou.github.io/awesomplete/)
- [roll.js](https://github.com/williamngan/roll)

### 4. Factory Interface

这种模式设置一个供用户运行并返回一个对象的函数。这个对象包含所有你需要的功能，比如配置和方法。

作为一个 API ，这看起来很像构造函数接口：

```javascript
// constructor
var mySparkles = new RainbowSparkles({ size: 2 });

// factory
var mySparkles = rainbowSparkles({ size: 2 });
```

他们都返回对象，但是工厂模式是一个函数，它在返回给你对象前可以运行任意代码（比如逻辑和条件分支）。这使得你可以返回以来上下文或配置的依赖根本上不同的对象。例如：

```javascript
var mySparkles = rainbowSparkles({
  object: ‘sparkles’,
  size: 6,
  shimmer: 8
});

var myRainbow = rainbowSparkles({
  object: ‘rainbow’,
  type: ‘double’
});

myRainbow.appear();
mySparkles.shower(1000);
```

用在了：

- [Dragula](http://bevacqua.github.io/dragula/)

### 5. Singleton Interface

这种方案是让你的插件提供一个包含所有功能、配置和数据的单独的全局对象。

单例通常用在 [给予一大堆功能于命名空间](http://yuiblog.com/blog/2007/06/12/module-pattern/) ——甚至整个应用。尽管只暴露了一个全局对象，这也并没得到太多限制（比如和一个全局函数比）。

```javascript
rainbowSparkles.settings = { size: 5, shimmer: 8 };
rainbowSparkles.shower(1000);
```

用在了：

- [Accounting.js](http://openexchangerates.github.io/accounting.js/)

### 6. Constructor and Instance

简单设置一个构造和提前创建的单个实例给用户。如果你期望大多数人只需要一个实例，不过要是还想留一个开放配置给他们，这会很有意义。

唯一的缺点在于我只能想到这种最终暴露两个全局对象的办法。

```javascript
// It has the simplicity of a singleton...
rainbowSparkles.settings = { size: 5, shimmer: 8 };
rainbowSparkles.shower(1000);

// ...but users can also create new instances if needed.
var altSparkles = new RainbowSparkles({ size: 5, shimmer: 8 });
altSparkles.shower(1000);
```

用在了：

- [Archor.js](http://bryanbraun.github.io/anchorjs/)

### 7. Method Chaining

这种 API 模式允许你在队列中运行多个函数，每个运行结果会传递给下一个。你很容易在 JQuery 里看到函数链，效果不错：

```javascript
// Jquery does it.
$(‘#message’).text("Hello!").css("color", "red");

// We can do it too.
rainbowSparkles.shower(500).shimmer().fadeOut();
```

这种 API 模式可以实际应用在和任一种模式组合。任何时候在你执行一个函数是，你就可以返回上下文去链接额外的方法。

这种 API 模式被用在：

- [jQuery](https://jquery.com/)

### 8. Framework-Specific Plugin

要不考虑把你的项目搞成一个不错的 jQuery 插件？或者 React 组件？

这里是我的观点：如果你想让你的项目走得长远，不要从某个框架特定的插件开始。这样做这锁定应用环境的生态系统，减少用户数量。相反，无依赖的 JavaScript 插件用一点点代码就能集成在各种各样的项目里。

当然一旦你的无依赖插件在原始环境下成功了，你就能为 jQuery、Angular、React 等等这些框架写集成，集成代码只是原生插件的简单封装。这么做会让你集中精力对应需求，而不是把时间浪费在集成框架却还无人问津。

## Plan to Plan

收集这些方案帮助我找到了一种适合升级工作的 API，不过最好是在开始的时候做这件事。一开始就用了正确的 API，你就能提供丝滑无破坏性的升级给用户。
