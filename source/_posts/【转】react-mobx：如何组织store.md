---
title: 【转】Mobx React  最佳实践
date: 2019-10-23 15:14:48
tags: React
categories: React
---

> 【转】Mobx React  最佳实践
> 转载自 https://juejin.im/post/5a3b1a88f265da431440dc4a

在这一篇文章里，将展示一些使用了mobx的React的最佳实践方式，并按照一条一条的规则来展示。在你遇到问题的时候，可以依照着这些规则来解决。
这篇文章要求你对于mobx的stores有基本的理解，如果没有的话请先阅读[官方文档](https://mobx.js.org/best/store.html)。

<!-- more -->

### stores 代表着UI状态
永远记住，你的stores代表着你的UI状态，这就意味着，当你将你的stores储存下来后，就算你关了网页，再次打开，载入这个stores，你得到的网页也应该是相同的。虽然stores并不是一个本地数据库的角色，但是他依然存储着一些类似于按钮是否可见，input里面的内容之类的UI状态。

```
class SearchStore {
  @observable searchText;

  @action
  setSearchText = (searchText) => {
    this.searchText = searchText
  }
}

@observer
class SearchInput extends React.Component {

  handleInputChanged = (event) => {
    const { searchStore } = this.props;
    searchStore.setSearchText(event.target.value);
  }

  render() {
    const { searchStore } = this.props;
    return (
      <input
        value={searchStore.searchText}
        onChange={this.handleInputChanged}
      />
    );
  }
}
```


### 将你的REST API请求和store的action分离

不建议将REST API请求的函数放在stores里面，因为这样以来这些请求代码很难测试。你可以尝试把这些请求函数放在一个类里面，把这个类的代码和store放在一起，在store创建时，这个类也相应创建。然后当你测试时，你也可以优雅的把数据从这些类里面mock上去。

```
class TodoApi {

  fetchTodos = () => request.get('/todos')
}

class TodoStore {

  @observable todos = [];

  constructor(todoApi) {
    this.todoApi = todoApi;
  }

  fetchTodos = async () => {
    const todos = await this.todoApi.fetchTodos();

    runInAction(() => {
      this.todos = todos;
    });
  }
}

// 在你的主要函数里面
const todoApi = new TodoApi();
const todoStore = new TodoStore(todoApi);
```

### 把你的业务逻辑放在stores里面

尽量不要把业务逻辑写在你的组件里面。当你把业务逻辑写在组件里面的时候，你是没有办法来及时定位错误的，因为你的业务逻辑分散在各种不同的组件里面，让你很难来通过行为来定义到底是哪些代码涉及的这个错误。最好就把业务逻辑放在stores的方法里面，从组件里面调用。
避免使用全局的store实例
请尽量避免使用全局的store实例，因为这样你很难写出有条理而可靠的组件测试。取而代之的是，你可以使用Provider来把你的store inject到你的component实例的props里面。这样你就可以轻松的mock这些store来测试了。

```
const searchStore = new SearchStore();

const app = (
  <Provider searchStore={searchStore}>
    <SearchInput />
  </Provider>
);

ReactDom.render(app, container);
```

[mobxjs/mobx-react](https://github.com/mobxjs/mobx-react#provider-and-inject)

### 只有在store里面才允许改变属性
请不要直接在组件里面直接操作store的属性值。因为只有store才能够来修改自己的属性。当你要改变属性的时候，请使用相应的store方法。不然的话你的属性修改会散落在各处不受控制，这是很难debug的。

### 时刻记得在组件声明 @observer
在每个组件声明的时候使用@observer来更新组件的状态。不然在嵌套组件里面，子组件没有声明的话，每次状态更新涉及到的都是父组件级的重新渲染。当你都使用了@observer时，重新渲染的组件数量会大大降低。

### 使用 @computed
就像下面代码的例子，使用@computed属性来处理一些涉及多个属性的逻辑。使用@computed可以减少这样的判断类业务逻辑在组件里面出现的频率。

```
class ApplicationStore {

  @observable loggedInUser;

  @observable isInAdminMode;

  @computed isAdminButtonEnabled = () => {
    return this.loggedInUser.role === 'admin' && this.isInAdminMode;
  }
}
```

### 你不需要 react router 来管理状态
你不需要使用react router管理状态。就像我前面所说的，你的store就代表了应用的状态。当你让router来管理部份应用状态的时候，这部分状态就从store里面剥离开来。所以尽量使用store来储存所有的UI状态，这样store的属性就是你的界面所得。

### 倾向于编写可控组件
多编写可控组件，这样会大大降低你的测试复杂度，也让你的组件易于管理。
[Forms – React](https://zh-hans.reactjs.org/docs/forms.html)


作者：Daniel Bischoff
原文：Mobx React — Best Practices
翻译：Dominic Ming
