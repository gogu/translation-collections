# 6 个引人注目的 ES6 proxies 使用案例

> 作者 Bryce Johns 原载：http://devbryce.com/use-cases-for-es6-proxies/

是不是除了我，其他人都在 ES6 功能的布道中忘却了 proxies？

这主要是因为在 Safari（没版本支持），Node（v6 才开始首次支持），和编译工具（Babel/TypeScript）中 [既慢又有限](https://kangax.github.io/compat-table/es6/#test-Proxy,_internal_'get'_calls) 的支持。

尽管如此我还是对 ES6 proxies 感到兴奋。他们提供了一种简洁而语义的方式来调节在 js 应用中对对象的访问。

在这篇文章中，我将会尽我所能来解释他们是如何工作的，并列出我认为他们发挥实用功能的许多例子。

## proxy 是什么？

真实世界中，代理是一个人给了另一个代表他的人自己的权限。举例来说有些州允许代理投票，也就是说你可以赋予另一人在一次投票中代表你投一票的权利。

Proxies 在技术领域中也是一种通用范式。你或许听过某某代理服务器获取了你的所有请求/流量，代表你把它们发送的另一个目的地，然后再把响应结果返回给你。当你不希望你的流量目标知道他们的来源是哪里（比如 [nsa.gov](http://nsa.gov/)）用代理服务器会很有效。所有的目标服务器会认为请求是来自代理服务器的。

这就接近本文观点了，proxies 也是一种在应用程序编程中常用的 [设计模式](https://en.wikipedia.org/wiki/Proxy_pattern)。这种 proxy接近 ES6 proxies 意图达到的，包括用 class B 包装 class A 以拦截/控制对 class A 的访问。

proxy 模式一般可以使用在当你想要：

- 拦截或者控制对一个对象的使用时
- 通过掩盖程序或隐藏逻辑来降低方法/类的复杂度
- 阻止没有经过验证/准备的重资源操作

## ES6 中的 Proxies

`Proxy` 构造函数可以再全局对象中直接使用。用它，你可以于对象和对其的许多操作之间手机请求信息，然后返回你想要的。在这方面，proxies 和中间件（middleware）很像。

而 proxies 特别的地方是你可以拦截到那些你普遍在一个对象和它的属性上使用的方法，也就是 `get`，`set`，`apply`（函数调用时）以及 `constructor` （在函数由 new 关键词调用时）。可以拦截到的所有方法清单，[看这里](http://www.ecma-international.org/ecma-262/6.0/#sec-proxy-object-internal-methods-and-internal-slots)。Proxies 还可以配置来在任何时刻停止接受请求，取消对他们代理的对象的所有访问。这是用 `revoke` 方法来实现的，我一会儿会说起它。

### 术语

在继续前，这里有三个你需要知道的东西：`target`，`handler` 和 `trap`。

`target` 指当前代理的对象。即是你想要调节访问的对象。它会被作为 `Proxy` 构造函数的第一个参数传入，并且也会传入每个 `trap` 中（对此更多信息在后两行）。

`handler` 是一个包含你想要拦截和处理的操作的对象。它被作为第二个参数传入。它实现了 Proxy API（比如 `get`，`set`，`apply` 等）。

`trap` 是在 `handler` 中处理指定方法的函数引用。当你定义一个 `get` trap 你就能拦截 `get` 调用，诸如此类。

还有件事：你需要看了解 `Reflect` API，它也是全局对象中可以直接使用的。我建议你在 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) 上弄懂它，因为那里讲的简明扼要，你一看就明白。相信我。

### 基础用法

在我深入我见到的那些 Proxies 的有趣案例前，先看个“hello world”吧。更多关于使用代理的思考和详情，可以去看 Nicholas Zakas 的 [理解 ES6](https://leanpub.com/understandinges6/read#leanpub-auto-proxies-and-the-reflection-api) 中的相关章节。这是部杰作，而且免费在线阅读。

```javascript
let dataStore = { 
  name: 'Billy Bob',
  age: 15
};

let handler = { 
  get(target, key, proxy) {
    const today = new Date();
    console.log(`GET request made for ${key} at ${today}`);
    return Reflect.get(target, key, proxy);
  }
}

dataStore = new Proxy(dataStore, handler);

// This will execute our handler, log the request, and set the value of the `name` variable.
const name = dataStore.name;
```

## ES6 Proxy 使用案例

你也许已经有点子要用 proxies 做什么了。来看看我做了什么吧。

### 1. 抽离校验代码

这是一个使用 proxies 做校验的简单例子（出自 Zakas 的书），用来确认一个数据存储中的属性是相同类型的。由此我们可以保证任何时候任何人试图给 numbericDataStore 设置的属性值都是数字类型。

```javascript
let numericDataStore = { 
  count: 0,
  amount: 1234,
  total: 14
};

numericDataStore = new Proxy(numericDataStore, { 
  set(target, key, value, proxy) {
    if (typeof value !== 'number') {
      throw Error("Properties in numericDataStore can only be numbers");
    }
    return Reflect.set(target, key, value, proxy);
  }
});

// this will throw an error
numericDataStore.count = "foo";

// this will set the new value as expected
numericDataStore.count = 333;
```

挺有意思吧，不过你能有多频繁地创建只有相同类型属性的对象呢？

如果你想写点自定义校验给对象上的一些或者所有属性用，代码就会变得稍复杂，但是我赞美的是 proxies 可以帮助我们把校验代码和主逻辑分离开来。我不会是唯一一个讨厌用校验代码弄脏方法或类的人吧？

```javascript
// Define a validator that takes custom validators and returns a proxy
function createValidator(target, validator) { 
  return new Proxy(target, {
    _validator: validator,
    set(target, key, value, proxy) {
      if (target.hasOwnProperty(key)) {
        let validator = this._validator[key];
        if (!!validator(value)) {
          return Reflect.set(target, key, value, proxy);
        } else {
          throw Error(`Cannot set ${key} to ${value}. Invalid.`);
        }
      } else {
        // prevent setting a property that isn't explicitly defined in the validator
        throw Error(`${key} is not a valid property`)
      }
    }
  });
}

// Now, just define validators for each property
const personValidators = { 
  name(val) {
    return typeof val === 'string';
  },
  age(val) {
    return typeof age === 'number' && age > 18;
  }
}
class Person { 
  constructor(name, age) {
    this.name = name;
    this.age = age;
    return createValidator(this, personValidators);
  }
}

const bill = new Person('Bill', 25);

// all of these throw an error
bill.name = 0; 
bill.age = 'Bill'; 
bill.age = 15;
```

这样，你就能在不改变你的类/方法的前提下无线扩展你的校验代码了。

还有一个和校验相关的点子。加入你想检查一个被传入特定方法的值，并 log 出发生错误时的有用的警告信息。你可以用 proxies 来做，这会保持类型检查代码不碍什么事。

```javascript
let obj = { 
  pickyMethodOne: function(obj, str, num) { /* ... */ },
  pickyMethodTwo: function(num, obj) { /*... */ }
};

const argTypes = { 
  pickyMethodOne: ["object", "string", "number"],
  pickyMethodTwo: ["number", "object"]
};

obj = new Proxy(obj, { 
  get: function(target, key, proxy) {
    var value = target[key];
    return function(...args) {
      var checkArgs = argChecker(key, args, argTypes[key]);
      return Reflect.apply(value, target, args);
    };
  }
});

function argChecker(name, args, checkers) { 
  for (var idx = 0; idx < args.length; idx++) {
    var arg = args[idx];
    var type = checkers[idx];
    if (!arg || typeof arg !== type) {
      console.warn(`You are incorrectly implementing the signature of ${name}. Check param ${idx + 1}`);
    }
  }
}

obj.pickyMethodOne(); 
// > You are incorrectly implementing the signature of pickyMethodOne. Check param 1
// > You are incorrectly implementing the signature of pickyMethodOne. Check param 2
// > You are incorrectly implementing the signature of pickyMethodOne. Check param 3

obj.pickyMethodTwo("wopdopadoo", {}); 
// > You are incorrectly implementing the signature of pickyMethodTwo. Check param 1

// No warnings logged
obj.pickyMethodOne({}, "a little string", 123); 
obj.pickyMethodOne(123, {});
```

### 2. JavaScript 中真正的私有

我曾和一个对 JavaScript 里没有真私有属性这件事感到相当生气的开发者一起干活。他以前写 Java，Java 里可以明确设定任何属性为私有（只能在 class 中访问）或公有（内部或外部都能访问到）。

在 JavaScript 中，大家习惯用下划线（或者其他字符）来标识一个属性只能在内部使用。但是这完全不能防止大家偷偷获取或修改它。

下面这个案例，我们想只在 api 对象中使用 `apiKey`，而非常不希望在外面能访问到它。

```javascript
var api = { 
  _apiKey: '123abc456def',
  /* mock methods that use this._apiKey */
  getUsers: function(){},
  getUser: function(userId){},
  setUser: function(userId, config){}
};

// logs '123abc456def';
console.log("An apiKey we want to keep private", api._apiKey);

// get and mutate _apiKeys as desired
var apiKey = api._apiKey; 
api._apiKey = '987654321';
```

用 ES6 proxies，你可以有两种办法实现真正且完备的私有。

首先，你可以用 proxy 拦截对特定属性的请求，然后对其进行限制或返回 undefined。

```javascript
var api = { 
  _apiKey: '123abc456def',
  /* mock methods that use this._apiKey */
  getUsers: function(){ },
  getUser: function(userId){ },
  setUser: function(userId, config){ }
};

// Add other restricted properties to this array
const RESTRICTED = ['_apiKey'];

api = new Proxy(api, { 
    get(target, key, proxy) {
        if(RESTRICTED.indexOf(key) > -1) {
            throw Error(`${key} is restricted. Please see api documentation for further info.`);
        }
        return Reflect.get(target, key, proxy);
    },
    set(target, key, value, proxy) {
        if(RESTRICTED.indexOf(key) > -1) {
            throw Error(`${key} is restricted. Please see api documentation for further info.`);
        }
        return Reflect.get(target, key, value, proxy);
    }
});


// throws an error
console.log(api._apiKey);

// throws an error
api._apiKey = '987654321';
```

你还可以用 `has` trap 来掩盖这个属性的存在。

```javascript
var api = { 
  _apiKey: '123abc456def',
  /* mock methods that use this._apiKey */
  getUsers: function(){ },
  getUser: function(userId){ },
  setUser: function(userId, config){ }
};

// Add other restricted properties to this array
const RESTRICTED = ['_apiKey'];

api = new Proxy(api, { 
  has(target, key) {
    return (RESTRICTED.indexOf(key) > -1) ?
      false :
      Reflect.has(target, key);
  }
});

// these log false, and `for in` iterators will ignore _apiKey

console.log("_apiKey" in api);

for (var key in api) { 
  if (api.hasOwnProperty(key) && key === "_apiKey") {
    console.log("This will never be logged because the proxy obscures _apiKey...")
  }
}
```

### 3. 静默对象访问日志

对于资源密集、运行缓慢或重度使用的方法和接口，你也许想要记录他们的使用情况或性能。Proxies 可以很容易的在后台默默的做这件事。

注意：不幸的是，你不能只用 `apply` trap 去拦截方法。Axel Rauschmayer 有做更多的相关描述 [在这里](http://exploringjs.com/es6/ch_proxies.html#_intercepting-method-calls)。基本的问题是，每当你调用一个方法，你都会先 `get` 这个方法。所以如果你想拦截一个方法调用，你需要拦截它的 `get` 先，然后再拦截 `apply`。

```javascript
let api = { 
  _apiKey: '123abc456def',
  getUsers: function() { /* ... */ },
  getUser: function(userId) { /* ... */ },
  setUser: function(userId, config) { /* ... */ }
};

api = new Proxy(api, { 
  get: function(target, key, proxy) {
    var value = target[key];
    return function(...arguments) {
      logMethodAsync(new Date(), key);
      return Reflect.apply(value, target, arguments);
    };
  }
});

// executes apply trap in the background
api.getUsers();

function logMethodAsync(timestamp, method) { 
  setTimeout(function() {
    console.log(`${timestamp} - Logging ${method} request asynchronously.`);
  }, 0)
}
```

这很酷，因为你可以在不弄脏你的应用代码或阻碍执行的前提下记录任何东西。跟踪方法在时间上的性能表现应该也不用太多代码。

### 4. 警告或阻止特定操作

假设你想要组织某人删除 `noDelete` 属性，想要告诉调用 `oldMethod` 的用户那已经被弃用了，或者想组织某人修改 `doNotChange` 属性，这里有个快速实现：

```javascript
let dataStore = { 
  noDelete: 1235,
  oldMethod: function() {/*...*/ },
  doNotChange: "tried and true"
};

const NODELETE = ['noDelete']; 
const DEPRECATED = ['oldMethod']; 
const NOCHANGE = ['doNotChange'];

dataStore = new Proxy(dataStore, { 
  set(target, key, value, proxy) {
    if (NOCHANGE.includes(key)) {
      throw Error(`Error! ${key} is immutable.`);
    }
    return Reflect.set(target, key, value, proxy);
  },
  deleteProperty(target, key) {
    if (NODELETE.includes(key)) {
      throw Error(`Error! ${key} cannot be deleted.`);
    }
    return Reflect.deleteProperty(target, key);

  },
  get(target, key, proxy) {
    if (DEPRECATED.includes(key)) {
      console.warn(`Warning! ${key} is deprecated.`);
    }
    var val = target[key];

    return typeof val === 'function' ?
      function(...args) {
        Reflect.apply(target[key], target, args);
      } :
      val;
  }
});

// these will throw errors or log warnings, respectively
dataStore.doNotChange = "foo"; 
delete dataStore.noDelete; 
dataStore.oldMethod();
```

### 5. 组织不必要的重资源操作

假设你有个会返回一个很大的文件的服务端，你不希望在之前的请求正在进行时，或者文件正在被下载时，或者已经下载完毕的情况下进行请求。Proxies 对调节此类访问和尽可能获取缓存值而不是让用户试图频繁调用服务端的方面来说是个美好的架构。这里我会省略一些必须的代码，但也够让你体验它是如何工作的了。

```javascript
let obj = { 
  getGiantFile: function(fileId) {/*...*/ }
};

obj = new Proxy(obj, { 
  get(target, key, proxy) {
    return function(...args) {
      const id = args[0];
      let isEnroute = checkEnroute(id);
      let isDownloading = checkStatus(id);     
      let cached = getCached(id);

      if (isEnroute || isDownloading) {
        return false;
      }
      if (cached) {
        return cached;
      }
      return Reflect.apply(target[key], target, args);
    }
  }
});
```

### 6. 即时取消对敏感数据的访问

Proxies 提供任何时候取消对目标对象访问功能。如果你想要完全封锁（为了安全性、权限、或性能原因）一些数据和 API 的访问这会很有用。这儿是个基本示例，用 `revocable ` 方法来实现。注意你在使用它的时候，并没有 new 一个 `Proxy`。

```javascript
let sensitiveData = { 
  username: 'devbryce'
};

const {sensitiveData, revokeAccess} = Proxy.revocable(sensitiveData, handler);

function handleSuspectedHack(){ 
  // Don't panic
  // Breathe
  revokeAccess();
}

// logs 'devbryce'
console.log(sensitiveData.username);

handleSuspectedHack();

// TypeError: Revoked
console.log(sensitiveData.username);
```

好了，这些是我知道的。我很乐意听听你在工作中使用 proxies 时有什么好点子。

Happy JavaScript，如果你看到有什么是我需要修改的，请让我知道。
