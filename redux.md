# Redux

> 介绍

## Redux 入门

Redux 是 JavaScript 应用的可预测状态容器.

它帮助你编写行为一致的应用程序，运行在不同的环境（客户端、服务器和原生），并且它是容易测试的。最重要的是，它提供了极好的的开发体验，例如结合时间旅行调试器（开发者工具）的动态代码编辑。

你可以将 Redux 结合 React 使用，或者搭配别的视图库，它非常的小（2kB，包括依赖），但它有庞大的生态系统和可用的插件库。

### 安装

#### Redux Toolkit

Redux Toolkit 是我们官方推荐的编写 Redux 逻辑的方法。它包含 Redux 核心，并包含了我们认为对构建 Redux 应用必要的软件包和功能。Redux Toolkit 建立在我们建议的最佳实践的前提下，简化了大多数 Redux 任务，避免了常见的错误，使编写 Redux 应用程序更简单。

RTK（Redux Toolkit）包括有助于简化许多常见用例的实用程序，例如：建立仓库、创建 reducers 和编写不可变的更新逻辑，甚至立即创建整个状态的切片。

无论你是用 Redux 建立你第一个项目的小白，还是为了简化已有项目的老司机，Redux Toolkit 都有助于你使你的 Redux 代码更棒。

Redux Toolkit 是 NPM 上的软件包，可与模块管理器或 Node 应用程序一起使用。

```shell
# NPM
npm install @reduxjs/toolkit

# Yarn
yarn add @reduxjs/toolkit
```

#### 创建 React Redux 应用

推荐使用 React 和 Redux 创建新应用的方法是使用用于 Create React App 的官方 Redux + JS 模板，它利用 Redux Toolkit 和 React Redux 与 React 组件集成。

```shell
npx create-react-app my-app --template redux
```

#### Redux Core

Redux 核心库是 NPM 上的软件包，可与模块管理器或 Node 应用程序一起使用。

```shell
# NPM
npm install redux

# Yarn
yarn add redux
```

它也可以作为预编译的 UMD 软件包来使用，该软件包定义了一个`window.Redux`的全局变量。UMD软件包可以通过`<script>`标签直接使用。

### 基础示例

应用程序的整个状态存储在单个仓库内的对象树中。修改状态树唯一的方法是发起action，对象描述发生了什么。要指定actions如何转换状态树，你可以编写纯reducers。

```javascript
import { createStore } from 'redux'

/**
 * 这是一个 reducer，具有 (state, action) => state 函数签名的纯函数。
 * 它描述了 action 如何将状态转换为下一个状态。
 *
 * 状态的数据类型由你决定: 它可以是基础数据类型，数组，对象，甚至是 Immutable.js（不可变数据集合） 数据结构。
 * 最重要的部分是你不应该改变状态对象，而是在状态更改时返回一个新对象。
 *
 * 在这个例子中，我们使用`switch`语句和字符串，但是如果对你的项目有意义，你可以使用一个遵循不同约定（例如 function maps）的helper
 */
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}

// 创建一个保存应用程序状态的 Redux 仓库
// 它的 API 是 { subscribe, dispatch, getState }.
let store = createStore(counter)

// 你可以使用 subscribe() 去更新 UI 以响应状态更改。
// 通常你会使用一个视图绑定库 (e.g. React Redux) 而不是直接使用 subscribe()。
// 但是，把当前状态保存在 localStorage 也很方便。

store.subscribe(() => console.log(store.getState()))

// 改变内部状态的唯一方法是 dispatch an action.
// The actions 可以被序列化，记录或存储然后重播.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

你可以使用被称为 actions 的状态对象指定要发生的改变，而不是直接改变状态。然后编写一个称为 reducer 的特殊的函数去决定每个 action 如何转换整个应用程序的状态。

在一个典型的 Redux 应用程序中，只有一个具有根 reducing 函数的仓库。随着应用程序的增长，你可以将根 reducer 拆分为更小的 reducers，分别在状态树的不同部分上运行。这就像在 React 应用中只有一个根组件，但它是由很多小的组件组成的。

对于一个计数器应用来说，这种架构似乎有些夸张了，但是这种模式的优点在于它可以很好的扩展到大型和复杂的应用。它还启用了非常强大的开发者工具，因为它可以追溯每个改变到造成改变的 action 上。你可以记录用户会话（user sessions）并仅通过重播每个 action 来重现它们。