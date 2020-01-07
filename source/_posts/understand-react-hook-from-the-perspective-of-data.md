---
title: 从数据的角度看React Hooks
tags:
  - React Hooks
categories:
  - Programming
date: 2020-01-08 01:04:42
---


[React Hooks](https://reactjs.org/docs/hooks-intro.html)在 2020 年已经不是前沿的概念了。
Hooks 已经可以很自然地融入日常工作中。在绝大部分情况下，
Hooks + Function Component 完全可以满足需求。
Hooks 的设计是非常有想象力的，说实话，在 Hooks 出现之前，
我完全想不出来有那么优雅的方式来将逻辑和 UI 组件融合在一起。
赞美之词先到这里，很多介绍 Hooks 的材料，包括官方文档都会以 Hooks
与其相对应的 Class-based Component 的功能的比较来入门 Hooks 的核心概念。
这令人看起来，Hooks 像是 Class-based Component 的`setState`与生命周期的延伸。
其实不然，Hooks 创造出了一种更加声明式的编程范式。

<!--more-->

## 从一个带建议的输入框说起

带建议的输入框是非常常见的需求，例如 Google 的搜索建议。

![a input with suggestion](suggestion.gif)

在这种需求中，一般而言后台会提供一个 API，
我们以用户输入作为关键字调用这个 API 来获取备选词的列表。
我们一起来使用 Class-based Component 来实现这个组件。为了简单起见，
我们实现输入框和建议列表，点击建议项回填到输入框暂时不实现。

首先，我们需要一个输入框和一个显示建议项的列表。

```jsx
class AutoComplete extends React.Component {
  render() {
    return (
      <div>
        <input />
        <ul></ul>
      </div>
    );
  }
}
```

![demo](2020-01-07-23-06-04.png)

然后，由于需要以近乎实时的方式给用户提供建议，
用受控组件的方式来管理用户输入是更好的选择。这里我们新增加了一个状态`keyword`
来保存用户的输入，增加了一个事件处理函数 `handleKeywordInput`。
将 `handleKeywordInput` 绑定在 `onChange` 事件之后，
我们的状态 `keyword` 就与用户输入同步了。

```jsx
class AutoComplete extends React.Component {
  constructor() {
    super();
    this.state = {
      keyword: ""
    };

    this.handleKeywordInput = this.handleKeywordInput.bind(this);
  }
  handleKeywordInput(keyword) {
    this.setState({
      keyword
    });
  }
  render() {
    return (
      <div>
        <input
          value={this.state.input}
          onChange={event => this.handleKeywordInput(event.target.value)}
        />
        <ul></ul>
      </div>
    );
  }
}

ReactDOM.render(<AutoComplete />, document.getElementById("root"));
```

接下来，我们需要获取建议项的列表。显然，最好的时机是`componentDidUpdate`生命周期。
同样为了简单期间，我们不调用真正的 API，用下面一个根据关键词来生成一个列表的函数来替代。

```javascript
function fetchSuggestions(keyword) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(
        [1, 2, 3].map(repeatTime => keyword.repeat(repeatTime)),
        2000
      );
    });
  });
}
```

有了这个~~伪~~API 之后，我们需要一个新的状态`suggestions`来保存获取到的建议项，
并且使用`map`将这些建议项渲染出来。最终的代码就像这样。

```jsx
class AutoComplete extends React.Component {
  constructor() {
    super();
    this.state = {
      keyword: "",
      suggestions: []
    };

    this.handleKeywordInput = this.handleKeywordInput.bind(this);
  }
  handleKeywordInput(keyword) {
    this.setState({
      keyword
    });
  }

  async componentDidUpdate(prevProps, prevState) {
    if (prevState.keyword !== this.state.keyword) {
      const newSuggestions = await fetchSuggestions(this.state.keyword);
      this.setState({
        suggestions: newSuggestions
      });
    }
  }
  render() {
    return (
      <div>
        <input
          value={this.state.keyword}
          onChange={event => this.handleKeywordInput(event.target.value)}
        />
        <ul>
          {this.state.suggestions.map(suggestion => (
            <li key={suggestion}>{suggestion}</li>
          ))}
        </ul>
      </div>
    );
  }
}
```

Demo 如下:

<p class="codepen" data-height="265" data-theme-id="default" data-default-tab="result" data-user="ayase-252" data-slug-hash="wvByMqP" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Suggestion CBC">
  <span>See the Pen <a href="https://codepen.io/ayase-252/pen/wvByMqP">
  Suggestion CBC</a> by Qingyu Deng (<a href="https://codepen.io/ayase-252">@ayase-252</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

让我们回想一下设计组件的整个思考流程：

- 首先，我们用了两个标签将组件的整体轮廓给画了出来；
- 然后，我们思考用户动作对数据的影响，因此，
  我们新建了状态`keyword`与事件处理函数`handleKeywordInput`来实现受控组件；
- 接下来，我们**为了获取**建议列表，选择在 `componentDidUpdate`这个生命周期中调用 API。
- 最后，我们渲染建议列表。

可以看到，我们的思路仍然是命令式的。因为我们需要获取建议列表，
所以我们要在组件的某个生命周期里去做这件事情。
这样的思路对于熟悉命令式的开发者而言是很自然的。但是，
如果我们从另外一个角度来看，是不是会有一些新的想法呢?

## 数据依赖

从需求来看，建议列表`suggestions`是我们通过向一个 API 输入关键词`keyword`获取的。从数据的关系来看，
建议列表`suggestions`就像是关键词`keyword`的[卫星数据](https://stackoverflow.com/questions/14551845/what-is-satellite-information-in-data-structures)。
换句话说，`suggestions`是依赖的`keyword`。想起数据依赖，
熟悉 Hooks API 的同学可能会想起`useEffect`等 API 中的依赖数组。
没有错，这些 API 就是我们刻画数据依赖关系的核心。

## 用 Hooks 重写带建议的输入框

在上一节中，我们发现了`suggestions`依赖于`keyword`。
我们可以很方便地用`useEffect`来刻画这样的数据依赖关系。
现在，让我们用 Hooks 重写这个组件。

首先，我们仍然会有一个`keyword`状态与一个`suggestions`状态。然后，
为了表现两者的依赖关系，我们使用以`[keyword]`为依赖数组的 `useEffect`来**自动地**
在`keyword`改变的时候重新获取相对应的`suggestions`。

```jsx
function AutoComplete() {
  // ----- 数据逻辑
  const [keyword, setKeyword] = useState("");
  const [suggestions, setSuggestions] = useState([]);
  useEffect(() => {
    async function _fetchSuggestions() {
      const newSuggestions = await fetchSuggestions(keyword);
      setSuggestions(newSuggestions);
    }
    _fetchSuggestions();
  }, [keyword]);

  // -----交互逻辑
  function handleKeywordInput(keyword) {
    setKeyword(keyword);
  }

  return (
    <div>
      <input
        value={keyword}
        onChange={event => handleKeywordInput(event.target.value)}
      />
      <ul>
        {suggestions.map((suggestion, idx) => (
          <li key={idx}>{suggestion}</li>
        ))}
      </ul>
    </div>
  );
}
```

Demo 如下：

<p class="codepen" data-height="265" data-theme-id="default" data-default-tab="result" data-user="ayase-252" data-slug-hash="ZEYrWwp" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Sugguestion with Hooks">
  <span>See the Pen <a href="https://codepen.io/ayase-252/pen/ZEYrWwp">
  Sugguestion with Hooks</a> by Qingyu Deng (<a href="https://codepen.io/ayase-252">@ayase-252</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

我们回顾使用 Hooks 实现的思路。我们不再是*为了获取某个数据，然后在某个生命周期里执行某些操作去考虑。*
而是变成了，_因为某个数据 A 是依赖某个数据 B 的，所以在被依赖数据 B 改变的时候，我们也应该通过某些手段让数据 A 与数据 B 同步。_
我们不再刻意地去挑选同步的时机，刚渲染完也好（`componentDidMount`）、
刚更新完也好（`compnentDidUpdate`）甚至是任意时候，
只要框架能够保证这两个数据是同步的就行。正因为如此，在 Hooks 的作用下，
React 的数据管理能够变得更加声明化。

由于我们不必要去挑选数据同步时机，组件的生命周期这一概念消失了。
数据之间的交互与组件彻底解耦，因此，Hooks 带来了第二个好处——逻辑重用。
我们可以很轻松地将通过关键词来获取建议的逻辑封装起来变成一个自定义 Hook——`useSuggestions`。

```jsx
function useSuggestions {
  const [keyword, setKeyword] = useState('')
  const [suggestions, setSuggestions] = useState([])
  useEffect(() => {
    async function _fetchSuggestions() {
      const newSuggestions = await fetchSuggestions(keyword)
      setSuggestions(newSuggestions)
    }
    _fetchSuggestions()
  }, [keyword])

  return {
    setKeyword,
    keyword,
    suggestions
  }
}

function AutoComplete () {
  const {keyword, setKeyword, suggestions} = useSuggestions()

  function handleKeywordInput(keyword) {
    setKeyword(keyword)
  }

  return (
      <div>
        <input
          value={keyword}
          onChange={event => handleKeywordInput(event.target.value)}
        />
        <ul>
           {suggestions.map((suggestion, idx) => <li key={idx}>{suggestion}</li>)}
        </ul>
      </div>
  )
}
```

现在我们可以在其他组件上也可以用到根据关键词提供的建议列表了。甚至，
通过参数化请求方法，我们可以创造出更加通用的 Hook。
具体的组件不需要知道里面的数据之间的交互逻辑。
因此，Hooks 开创了可以将一些业务中典型的数据交互逻辑抽象出来的方法。这一方面，
无论是之前的 Class-based Component 或者 Vue 的响应式系统都是没能做到的。

## 总结

Hooks 强调数据之间的依赖关系，在 Hooks 的框架下：

- 相关联的数据可以自动地同步；
- Hooks 是独立于组件的；

前者免除了我们需要手动同步相关联数据的需要，抹除了组件生命周期的存在意义；
后者可以将业务中典型的数据交互逻辑抽象出来应用于其他组件中。通过 Hooks，
我们实现组件的思路得以更加向声明式与数据驱动靠近。
