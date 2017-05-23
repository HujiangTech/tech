---   
layout: post
title: 图解 React Virtual DOM （上）
categories: [Web前端]
tags: [Web前端，CSS]
description: 了解 React 的人几乎都听过说 Virtual DOM，甚至不了解 React 的人也听过 Virtual DOM。那么 React 的 Virtual DOM 到底长什么样子呢？今天我们将一探 React 的源码来揭开 React  Virtual DOM 的神秘面纱。
---  

了解 React 的人几乎都听过说 Virtual DOM，甚至不了解 React 的人也听过 Virtual DOM。那么 React 的 Virtual DOM 到底长什么样子呢？今天我们将一探 React 的源码来揭开 React  Virtual DOM 的神秘面纱。

> 参考源码为React稳定版，版本号v15.4.1。

## 1. React

我们首先试着在控制台打印一下 `React` 看看会是什么样子:

![](http://oidv7s2r8.bkt.clouddn.com/01.png)

从控制台看来，React是一个对象，那接下来我们找到相应的源码来确认看看(src/isomorphic/React.js)：

```js
var React = {
  Children: {
    map: ReactChildren.map,
    forEach: ReactChildren.forEach,
    count: ReactChildren.count,
    toArray: ReactChildren.toArray,
    only: onlyChild,
  },
  Component: ReactComponent,
  PureComponent: ReactPureComponent,
  createElement: createElement,
  cloneElement: cloneElement,
  isValidElement: ReactElement.isValidElement,
  PropTypes: ReactPropTypes,
  createClass: ReactClass.createClass,
  createFactory: createFactory,
  createMixin: function(mixin) {
    return mixin;
  },
  DOM: ReactDOMFactories,
  version: ReactVersion,
  __spread: __spread,
};
```

可以了解到，React 确实是一个 Object ，我们可以把 React 对象画成下图的形式，方便大家直观的观察：

![](http://oidv7s2r8.bkt.clouddn.com/02.png)

React 是一个对象，里面包含了许多方法和属性，有最新的 v15 版本的方法，也有些以前的 API 和一些已经废弃不建议使用的 API。

* `Component` 用来创建 React 组件类。
* `PureComponent` 用来创建 React 纯组件类。
* `createElement` 创建 React 元素。
* `cloneElement` 拷贝 React 元素。
* `isValidElement` 判断是否是有效的 React 元素。
* `PropTypes` 定义 React props 类型。(过时的API)
* `createClass` 创建 React 组件类（过时的API）。
* `createFactory` 创建 React 工厂函数。（不建议使用）。
* `createMixin` 创建 Mixin。
* `DOM` 主要和同构相关。
* `version` 当前使用的 React 版本号。
* `__spread` **已废弃**，直接用 `Object.assign()` 代替

`__spread` 方法已经**废弃**，不再建议使用。在作者写这篇文章的时候，React 又发布了 v15.5.0 版本，在这个版本里，`createClass` 和 `PropTypes` 也已经被标记为**过时**的 API，会提示 warning。

* 对于原来的旧 API `React.createClass`，现在推荐开发者用 class 的方式继承 `Component` 或者 `PureComponent`。
* 对于 `PropTypes` 的引入方式也不是原来的 `import { PropTypes } from 'react'`，而变成了 `import PropTypes from 'prop-types'`。

其他属性和方法我们暂且就不详细的讲述了，这篇文章就只详细的研究一下和创建 React Virtual DOM 最紧密相关的方法——`React.createElement`。

> `React.createElement` 方法其实是调用的ReactElement模块的 `ReactElement.createElement` 方法。

## 2. React Element
Virtual DOM 是真实 DOM 的模拟，真实 DOM 是由真实的 DOM 元素构成，Virtual DOM 也是由虚拟的 DOM 元素构成。真实 DOM 元素我们已经很熟悉了，它们都是 HTML 元素（HTML Element）。那虚拟 DOM 元素是什么呢？React 给虚拟 DOM 元素取名叫 React 元素（React Element）。

![](http://oidv7s2r8.bkt.clouddn.com/06.png)

我们知道，React 可以通过组合一些 HTML 原生元素形成组件，然后组件又可以被其他的组件复用。所以，原生元素和组件其实在概念上都是一致的，都是具有特定功能和 UI 的可复用的元素。因此，React 把这些元素抽象成了 React Element。不论是 HTML 原生元素，例如：`<p></p>`，`<a></a>`，等。或者这些原生元素的组合（组件），例如 `<Message />` 等。它们都是 React Element，而创建这些 Element 的方法就是 `React.createElement`。

> **React Virtual DOM 就是由 React Element 构成的一棵树**。

接下来我们就探究下 React Element 到底长什么样以及 React 是如何创建这些 React Element 的。

### 2.1 ReactElement 模块

我们在控制台里直接打印出 `<h1>hello</h1>`：

![](http://oidv7s2r8.bkt.clouddn.com/03.png)

我们再打印出 `<App />`，App 组件的结构如下：

```html
<div>
	<h1>App</h1>
	<p>Hello world!</p>
</div>
```
打印出的结果如下：

![](http://oidv7s2r8.bkt.clouddn.com/04.png)

可以很直观的发现，打印的 HTML 元素并不是真实的 DOM 元素，打印的组件也不是 DOM 元素的集合，所有打印出来的元素都是一个对象，而且它们长的非常相似，那其实这些对象都是 React Element 对象。

然后我们再看看源码部分：

```js
var ReactElement = function(type, key, ref, self, source, owner, props) {
  var element = {
    $$typeof: REACT_ELEMENT_TYPE,
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
  };
  if (__DEV__) {
    // ...
  }
  return element;
};
```

ReactElement其实是一个工厂函数，接受7个参数，最终返回一个React Element对象。

* `$$type` React Element 的标志，是一个Symbol类型。
* `type` React 元素的类型。
* `key` React 元素的 key，diff 算法会用到。
* `ref` React 元素的 ref 属性，当 React 元素生成实际 DOM 后，返回 DOM 的引用。
* `props` React 元素的属性，是一个对象。
* `_owner` 负责创建这个 React 元素的组件。

参数中的 `self` 和 `source` 都是只供开发环境下用的参数。从上面的例子我们可以发现唯一不同的就是`type` 了，对于原生元素，`type` 是一个字符串类型，记录了原生元素的类型；对于 react 组件来说呢，`type` 是一个构造函数，或者说它是一个类，记录了这个 react 组件的是哪一个类的实例。所以`<App/>.type === App` 的。

所以，每一个包装过后的React元素都是这样的对象:

```js
{
    $$typeof: REACT_ELEMENT_TYPE,
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner,
}
```

用图片表示 React Element，就是下图这样：

![](http://oidv7s2r8.bkt.clouddn.com/05.png)

### 2.2 ReactElement.createElement 方法
在此之前，可能有人会问，我们开发当中似乎没有用到 React.createElement 方法呀。其实不然，看下面的示例：

```js
class OriginalElement extends Component {
  render() {
    return (
      <div>Original Element div</div>
    );
  }
}
```

经过babel转译之后是这样的

```js
_createClass(OriginalElement, [{
    key: "render",
    value: function render() {
      return React.createElement(
        "div",
        null,
        "Original Element div"
      );
    }
  }]);
```

可以看到，所有的 JSX 都会被编译成 React.createElement 方法，所以这个方法可能是我们在使用React用的最多的方法。

接下来我们看看 React.createElement 方法是怎样的，前面说过了 React.createElement 方法其实就是 ReactElement.createElement 方法。

```js
ReactElement.createElement = function(type, config, children) {
  var propName;
  var props = {};
  var key = null;
  var ref = null;
  var self = null;
  var source = null;
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;

    for (propName in config) {
      if (hasOwnProperty.call(config, propName) &&
          !RESERVED_PROPS.hasOwnProperty(propName)) {
        props[propName] = config[propName];
      }
    }
  }
  var childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    var childArray = Array(childrenLength);
    for (var i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      // ...
    }
    props.children = childArray;
  }
  if (type && type.defaultProps) {
    var defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  if (__DEV__) {
    // ...
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props
  );
};
```

reactElement.createElement大致做了2件事。

第一件是初始化 React Element 里的各种参数，例如 `type`，`props` 和 `children` 等。在初始化的时候，会提取出 `key`，`ref` 这两个属性，然后 \_\_self，\_\_source 这两个属性也是仅开发用。所以如果你在组件里定义了 `key`，`ref`，`__self`，`__source` 这4个属性中的任何一个，都是不能在 `this.props` 里访问到的。从第三个参数开始，传入的参数都会合并为 `children` 属性，如果只有一个，那么 `children` 就是第三个元素，如果超过一个，那么这些元素就会合并成一个 `children` 数组。

第二件是初始化 `defaultProps`，我们可以发现，`defaultProps` 是通过 `type` 来初始化的，我们在上面也说过，对于 `react` 组件来说，`type` 是 React Element 所属的类，所以可以通过 `type` 取到该类的 `defaultProps`（默认属性）。这里还有一点需要注意，如果我们把某个属性的值定义成 `undefined`，那么这个属性也会使用默认属性，但是定义成 `null` 就不会使用默认属性。

下面是图解：

![](http://oidv7s2r8.bkt.clouddn.com/07.png)

## 4. 创建Virtual DOM树

有了上面的作为基础，那创建 Virtual DOM 就很简单了。整个 Virtual DOM 就是一个巨大的对象。

比如我们有这么一个 `App`：

```html
App:
<div>
  <Header />
  <List />
</div>

Header:
<div>
  <Logo />
  <button>菜单</button>
</div>

List:
<ul>
  <li>text 1</li>
  <li>text 2</li>
  <li>text 3</li>
</ul>

Logo:
<div>
  <img src="./foo.png" alt="logo" />
  <p>text logo</p>
</div>

ReactDOM.render(<App />, document.getElementById('root'))

```

通过上面的了解到的 React Element 创建方式，我们不难知道，生成的对应的 Virtual DOM 应该是类似于这样的：

![](http://oidv7s2r8.bkt.clouddn.com/08.png)

需要注意的是，这些元素**并不是**真实的 DOM 元素， 它们只是一些对象，而且我们可以看到 React 组件实际上是概念上的形态，最终还是会生成原生的虚拟 DOM 对象。当这些对象上的数据发生变化时，通过打 patch 把变化同步到真实的 DOM 上去。

目前我们可以认为 Virtual DOM 就是这样的一种形态，但是实际上，并没有这么简单，这只是最基本的样子，在后续的文章中我会带大家一起看看更高级的形态。
