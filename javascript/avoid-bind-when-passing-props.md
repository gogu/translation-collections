# 传递 props 时别用 bind
> Don't Use Bind When Passing Props 转载自 https://daveceddia.com/avoid-bind-when-passing-props/ 作者 DAVE CEDDIA

在写 React 项目时会有很多需要给 prop 传递函数的场景。通常是为了给子组件传递一个回调，这样子组件就能通知父组件一些事件。

要考虑到这些函数的绑定是很重要的，即被调用的时候 this 对象指向什么。

不少方式可以确定绑定的正确性，但良莠不齐，本文将复习这些方式。

## Way #1: Autobinding (good, only with React.createClass)

如果你在用 `React.createClass`，组件内的成员函数会自动地绑定到组件实例上。你可以不通过调用 `bind` 来传递他们，你传递的绝对总是那个相同的函数。

```javascript
var Button = React.createClass({
  handleClick: function() {
    console.log('clickity');
  },
  render: function() {
    return (
      <button onClick={this.handleClick}/>
    );
  }
});
```

## Way #2: Calling .bind Within render (bad, ES6)

而当使用 ES6 classes 时，React 不会自动绑定成员函数。

像这样在最后时刻绑定它可以保证工作正常，但这将轻度损耗性能，因为每次一个 re-render 都会创建一个新函数（re-render 可真的是很常发生呢）。

真正的问题不在于创建函数是个昂贵的操作。而是每次创建一个新函数的话，你传递给的那个组件将每次都在 prop 上看到一个新值。等到了通过 `shouldComponentUpdate` 接口来调整性能的时候，不断变化的 prop 会让这看起来有东西在变化，尽管实际上和之前是相同的。

```javascript
class Button extends React.Component {
  handleClick() {
    console.log('clickity');
  }

  render() {
    return (
      <button onClick={this.handleClick.bind(this)}/>
    );
  }
}
```

下面的变体会做同样的事，每次 render 被调用都创建新函数：

```javascript
class Button extends React.Component {
  handleClick() {
    console.log('clickity');
  }

  render() {
    var handleClick = this.handleClick.bind(this);
    return (
      <button onClick={handleClick}/>
    );
  }
}
```

## Way #3: Arrow Function in render (bad, ES6)

这个例子出了使用了箭头函数代替 bind 外和之前的一样。看起来好点，但还是会每次 render 调用都创建新函数！这并不好。

```javascript
class Button extends React.Component {
  handleClick() {
    console.log('clickity');
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}/>
    );
  }
}
```

## Way #4: Class Instance Field With an Arrow Function (good, ES8+)

这个方法通过在组件创建时一次性把 `handleClick` 设置成一个箭头函数来实现。在 `render` 和其他函数里，`this.handleClick` 可以直接传递，因为箭头函数保持了 this 绑定。

这个方法被打了个 ES8 的标签，因为它并不属于 ES6 或 ES7（ES2016）的实验性功能。[ES2016 已经终稿](http://www.2ality.com/2016/01/ecmascript-2016.html)，只多了 `Array.prototype.includes` 和求幂操作符，所以当它被纳入标准，就该到 ES2017 或者更往后了。

尽管现在他可以由 Babel提供，这个功能还是会有点被踢出标准规格或被要求重构的小危险。不过有一大堆人已经在用了，感觉它还是会留下的。

```javascript
class Button extends React.Component {
  // Use an arrow function here:
  handleClick = () => {
    console.log('clickity');
  }

  render() {
    return (
      <button onClick={this.handleClick}/>
    );
  }
}
```

## Way #5: Binding in the Constructor (good, ES6)

你可以在构造函数中一次性设置绑定，然后怎么用他们都行。就是别忘了调用 `super` 方法。

```javascript
class Button extends React.Component {
  constructor(props) {
    super(props);

    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    console.log('clickity');
  }

  render() {
    return (
      <button onClick={this.handleClick}/>
    );
  }
}
```

## Way #6: Using Decorators (good, ES8+)

这儿有个很棒的库叫 [autobind-decorator](https://github.com/andreypopp/autobind-decorator)，可以做到这样：

```javascript
import autobind from 'autobind-decorator';

class Button extends React.Component {
  @autobind
  handleClick() {
    console.log('clickity');
  }

  render() {
    return (
      <button onClick={this.handleClick}/>
    );
  }
}
```

修饰器 `@autobind` 绑好了 `handleClick` 和其他你设置过得函数。如果你比较懒，还可以给这个 class 都用上：

```javascript
import autobind from 'autobind-decorator';

@autobind
class Button extends React.Component {
  handleClick() {
    console.log('clickity');
  }

  handleOtherStuff() {
    console.log('also bound');
  }

  render() {
    return (
      <button onClick={this.handleClick}/>
    );
  }
}
```

同样的，[ES2106/ES7 不包含这个功能]()，所以你得接受你的代码里有一点小危险，即使 Babel 提供了此功能。

## Bonus: Passing Arguments Without Bind

如 Marc 在评论里所说，使用 `.bind` 来预设函数的调用参数其实是相当通用的，尤其在列表中，如下：

```javascript
var List = React.createClass({
  render() {
    let { handleClick } = this.props;
    return (
      <ul>
        {this.props.items.map(item =>
          <li key={item.id} onClick={handleClick.bind(this, item.id)}>
            {item.name}
          </li>
        )}
      </ul>
    );
  }
});
```

如 [此处](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md#protips) 所解释，有个修复并避免 bind 的方法是把 `<li>` 抽象成单独的组件，然后就能带着每个 id 来调用你传入的点击处理事件：

```javascript
var List = React.createClass({
  render() {
    let { handleClick } = this.props;
    // handleClick still expects an id, but we don't need to worry
    // about that here. Just pass the function itself and ListItem
    // will call it with the id.
    return (
      <ul>
        {this.props.items.map(item =>
          <ListItem key={item.id} item={item} onItemClick={handleClick} />
        )}
      </ul>
    );
  }
});

var ListItem = React.createClass({
  render() {
    // Don't need a bind here, since it's just calling
    // our own click handler
    return (
      <li onClick={this.handleClick}>
        {this.props.item.name}
      </li>
    );
  },

  handleClick() {
    // Our click handler knows the item's id, so it
    // can just pass it along.
    this.props.onItemClick(this.props.item.id);
  }
});
```

## 性能上的提醒

在这些方法中间的权衡中一个规律：更多（也更复杂）的代码可以换取一些理论上的性能优势。

“过早优化是罪恶之源”，Donald Knuth 如是说。于是……在你为了减少一些周期而分离或复杂化你的代码时，还是要实际测量它会造成的影响：点开 devtools，用 [React performance tools](https://facebook.github.io/react/docs/perf.html) 来剖析你的代码。

## 总结

本文是关于如何绑定你传递给 props 的函数的几种方法的解释。如果知道其他方法？或是特别偏好哪个？让我们在评论里讨论吧。
