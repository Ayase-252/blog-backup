---
title: 新感觉前端组件测试库——React Testing Library
categories: Technology
tags:
  - Test
  - TDD
---

## Why use RTL

在前端自动化测试中一般是围绕 DOM 元素进行的。例如，我们想验证一个按钮，
它应该有“提交”字样，点击这个按钮之后将输入框中的数据发送给某个接口。
一个很自然的想法是，我们找到这个按钮的 DOM 元素，
检查 DOM 节点的 `innerHtml` 是否等于 `提交`。然后，
我们手动在这个节点上触发 `click` 事件，
看 `fetch` 或者其他网络请求方法是否按照预期被调用。

但是，这样做最大的困难在于如何定位这个节点？读者也许会说，
我们通过 [CSS 选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Selectors)来定位不就好了吗？
确实，使用 CSS 选择器来定位一个元素是非常自然的想法。
流行的 React 测试框架 [enzyme](https://github.com/enzymejs/enzyme)就是这么做的。

然而，CSS 选择器来定位元素意味着被测应用(Application Under Testing, AUT)中的与选择器相关的属性，
包括并不限于`class`、`id`、各种 attribute 等可能会与测试代码产生紧耦合。
假如，我们移除了某个元素中无用的 class，但是测试用例因无法找到这个元素导致失败。
此时，测试用例并不是因为*功能缺失*而失败，而是仅仅因为无法找到元素而失败，
这样，测试用例不但会变得难以维护，而且会由于虚警率高造成测试不可靠。

针对这个问题，RTL 探索了另外一种思路——**使用更贴近用户的方法**来定位元素。

![github login page](github-login.png)

用一个例子来说，看到上面的界面，我们可以很轻松找到输入用户名或密码的输入框。
这个寻找的过程可能太过直接以致于我们都没有思考一个问题——我们是如何找到对应的输入框。
仔细回想一下，我们看到 _Username or email address_，它下面有一个输入框，
这个框就是输入用户名的地方，不可能是其他的输入框。
用这种方法我们也可以唯一定位一个元素，而且比使用 CSS 选择器更有优势的地方在于，
我们并没有用到任何`class`，`id`等。作为用户而言，
我们并关心用户名输入框的`class`是 `username` 还是 `abdesdf-23afed`。

RTL 便利用了这种直觉来定位元素。但是，测试代码终究是一些计算机程序代码，
为了实现这种直觉，RTL 大量使用了一些提高 可访问性（Accessibility） 的实践，
如通过图片的 `alt` 属性来定位图片，
以及 [Accessible Rich Internet Application(ARIA)](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/ARIA)
中定义的一些属性。在方便测试的同时，也提高了应用的可访问性，可谓一举两得。

总结起来，RTL 的优势有两点:

- 使用更加贴近用户的方法来定位元素，解决了使用 CSS 定位元素使测试代码与业务代码产生紧耦合，
  测试用例不便维护的痛点。
- 使用大量提高可访问性的实践来定位元素，使应用在便于测试的同时提高了可访问性。

## RTL 的元素获取方法

RTL 的杀手特性就是其贴近用户的元素定位方法。这一节，我们来看一下这些 API。

一个通常的定位元素的 API 如 `getByText('username')`，其命名由两部分组成——
*动词*与*定位方法*。*动词*是指 `get` 这个部分， `get` 表示获取由方法定位到的元素中的第一个，
如果没有定位到任何元素，这个 API 会抛出一个异常。*定位方法*是指`ByText`，
表示匹配元素的 `textContent` 应该等于给定的字符串。

### 动词

动词部分包含 3 个系列，每个系列有两种变体。不同系列在行为上有一些差别:

- `get`： 获取由方法定位到的元素中的第一个，如果没有定位到任何元素，抛出一个异常；
- `getAll`：获取方法定位到的所有元素，因此返回值是一个数组。如果没有定位到任何元素，抛出一个异常；
- `query`：获取由方法定位到的元素中的第一个，如果没有定位到任何元素，返回 `null`，常用于检查元素是否存在；
- `queryAll` :获取方法定位到的所有元素，如果没有定位到任何元素，返回 `[]`。
- `find`：返回一个 `promise`。当在确定时间段（默认 1000ms）内，方法匹配到了元素， 那么 resolve 成这些元素中的第一个，
  如果未能够匹配到任何元素，则 reject；
- `findAll`：返回一个 `promise`。当在确定时间段（默认 1000ms）内，方法匹配到了元素， 那么 resolve 成这些元素，
  如果未能够匹配到任何元素，则 reject；

### 定位方法

定位方法一共有八种：`ByLabelText`、`ByPlaceHolderText`、`ByText`、`ByAltText`、`ByTitle`、`ByDisplayValue`、`ByRole`以及`ByTestId`。

#### `ByLabelText`

`ByLabelText` 可以通过定位 textContent 是给定字符串的 label 元素**对应**的元素。比如：

```html
<!-- label 与 表单元素通过 for/id 属性关联 -->
<label for="username">username</label>
<input id="username" />

<!-- label 与 表单元素通过 aria-labelledby 属性关联 -->
<label id="username-label">username</label>
<input aria-labelledby="username-label" />

<!-- 非表单元素间通过 aria-labelledby 属性关联 -->
<section aria-labelledby="username-title">
  <h3 id="username-title">username</h3>
  <p>content</p>
</section>

<!-- 表单元素内嵌在 label 元素里面 -->
<label>username <input /></label>

<!-- 表单元素内嵌在 label 元素里面（文本被子元素包裹） -->
<label><span>username</span> <input /></label>

<!-- 通过 aria-label 属性打标签（该标签对用户不可见）  -->
<input aria-label="username" />
```

均可以通过 `getByLabelText('username')` 匹配各个情形下对应的元素。
可以看到 RTL 综合应用了很多与可访问性相关的实践。
读屏器就是倚赖这些技术来服务视力障碍人士的。

#### `ByPlaceholderText`

在一些表单的设计中，输入框有时不会对应一个 Label，
而是用一个占位文字来提示这个字段应该填什么，如 YouTube 的视频搜索框。

![YouTube视频搜索框](youtube-search.png)

```html
<input placeholder="Search" />
```

像这种例子，我们就可以轻松地使用 `getByPlaceholderText('Search')` 匹配。

#### `ByText`

`ByText`直观上可以看作匹配屏幕上文本是给定值的元素。更加精确而言，
这个方法匹配元素 `textContent` 时给定值的元素。如，我们有一个提交按钮：

```html
<button>submit</button>
```

我们可以通过 `getByText('submit')` 轻松匹配。

由于使用 `input[type=submit]` 与 `input[type=button]` 作为按钮的实践也相当普遍，
针对这两种元素，其 `value` 属性被作为 `textContent` 看待。如

```html
<input type="submit" value="submit" /> <input type="button" value="submit" />
```

均可以用 `getByText('submit')` 匹配。

#### `ByAltText`

`ByAltText` 使用元素的 `alt` 属性来匹配元素。
`alt` 属性通常用在 `img` 标签上作为图片载入失败时显示的替代文本。

```html
<img alt="logo" />
```

上述的 `img` 标签可以使用 `getByAltText('logo')` 匹配。

#### `ByTitle`

`ByTitle` 使用元素的 `title` 属性来匹配元素。对于 `svg` 元素，
它也会使用 `svg` 元素内嵌的 `title` 标签来匹配。

```html
<span title="open"></span>

<svg>
  <title>open</title>
</svg>
```

`getByTitle("open")` 可以匹配上面的两个元素。

#### `ByDisplayValue`

`ByDisplayValue` 匹配**显示为给定字符串**的 `input`、`textarea`与`select`标签。如

```html
<input value="alaska" />
<textarea value="alaska"></textarea>

<select>
  <option value="AL">Alabama</option>
  <option value="AK" selected>Alaska</option>
</select>
```

`getByDisplay("alaska")`可以匹配上面的元素。注意对于 `select` 元素是以显示值作为判断依据，
而不是以元素的 `value` 作为判断依据。

#### `ByRole`

`ByRole` 使用元素的 ARIA Role 来匹配标签。这个方法与 ARIA Role 中的一些规定相关。
ARIA Role 这个主题本身就可以扩展成一篇文章，这里我们不展开。

#### `ByTestId`

`ByTestId` 使用元素的 `data-testid`标签来匹配元素。

```html
<div data-testid="component"></div>
```

这个 `div` 元素可以使用 `getByTestId("component")` 来匹配。

`ByTestId` 是上述方法均无法使用时的最后手段。从本质上而言，
`ByTestId` 使用的是业务代码的实现细节，
即元素的 `data-testid` 属性的字符串去匹配。
更改 `data-testid` 一般不会影响应用的功能，s
但是测试用例会因此失败，因此会造成与使用 CSS 选择器同样的问题。

## 实践：以注册表单为例

在本节中，我们通过一个实际的例子来演示如何使用 RTL 对 React 组件进行测试。
在 create-react-app CLI 工具中已经内置了对 RTL 的支持。
为了方便起见，我们直接使用 create-react-app 创建一个 React 项目 `registration-form`。

```bash
npx create-react-app registration-form
```

在这个小需求中，我们要实现一个注册表单，具体需求如下:

1. 表单需要有 3 个字段，_用户名_、*密码*以及*性别*。其中用户名与密码是一个输入框，性别是一个下拉选择框。可选项有男、女以及保密；
2. 表单有一个提交按钮，点击提交按钮，将用户名，
   密码以及性别通过 `POST` 方法上传到 `http://awesome-site.com/register` 接口中；
3. 用户名，密码与性别均为必选字段，当有任意字段为空，阻止提交，并且通过 `alert` 提示用户有字段为空。

事不宜迟，我们这里使用 TDD 的方法来开发这个组件。从第一个需求中，
我们可以看到界面需要展示 3 个字段。据此，我们可以写出一个测试用例。在 `src/components`
文件夹下创建一个 `__test__`文件夹来存放测试用例，然后在该文件夹下创建一个测试用例文件
`form.spec.tsx`。其中 `**.spec.tsx`是测试用例文件的命名惯例，这些文件会被 jest
测试框架自动发现并且执行。

创建好之后，我们开始写第一个测试用例：

```jsx
// src/components/__test__/form.spec.jsx
import React from "react";
import { render } from "@testing-library/react";
import Form from "../form";

describe("Form", () => {
  it("should have 3 fields, which are username, password and gender", () => {
    const { getByLabelText } = render(<Form></Form>);

    expect(getByLabelText("用户名")).toBeInTheDocument();
    expect(getByLabelText("密码")).toBeInTheDocument();
    expect(getByLabelText("性别")).toBeInTheDocument();
  });
});
```

这里我们使用 `getByLabelText` 方法来定位元素。注意到我们的断言使用了 `toBeInTheDocument`
表示期待元素出现在文档中。其实这种写法与 `getByLabelText('用户名')` 等价，
后者在无法找到元素时也会抛出异常。但是显式的使用断言会使测试用例更加清晰。

在命令行中执行 `yarn test` 或者 `npm test`（如果使用 npm 的话），
此时会以 `watch` 模式启动 jest。项目中的代码改动会自动触发相关的测试用例，
不需要手动调用。对于这个测试用例，我们的期望是一盏<span styles="color: red">红灯</span> FAILED。
因为我们连业务代码的文件都没有创建，会出现找不到模组错误。果然，
console 中的结果没有使我们失望。

```plaintext
FAIL  src/components/__test__/form.spec.jsx
  ● Test suite failed to run

    Cannot find module '../form' from 'form.spec.jsx'

      1 | import React from 'react'
      2 | import { render } from '@testing-library/react'
    > 3 | import Form from '../form'
        | ^
      4 |
      5 | describe('Form', () => {
      6 |   it('should have 3 fields, which are username, password and gender', () => {

      at Resolver.resolveModule (node_modules/jest-resolve/build/index.js:259:17)
      at Object.<anonymous> (src/components/__test__/form.spec.jsx:3:1)

Test Suites: 1 failed, 1 total
Tests:       0 total
Snapshots:   0 total
Time:        3.883s, estimated 4s
Ran all test suites related to changed files.

Watch Usage: Press w to show more.
```

为了纠正这个错误，我们在 `src/components` 文件夹下创建一个 `form.jsx` 文件，
并且 `export default` 一个组件：

```jsx
// src/components/form.jsx
import React from "react";

function Form() {
  return <></>;
}

export default Form;
```

按下保存，jest 会自动执行先前的测试用例。这里我们的期望仍然是一盏<span styles="color: red">红灯</span>，
因为这个组件没有做任何事情。测试结果验证了我们的期望：

```plaintext
FAIL  src/components/__test__/form.spec.jsx
  Form
    × should have 3 fields, which are username, password and gender (35ms)

  ● Form › should have 3 fields, which are username, password and gender

    Unable to find a label with the text of: 用户名

    <body>
      <div />
    </body>

       7 |     const { getByLabelText } = render(<Form></Form>)
       8 |
    >  9 |     expect(getByLabelText('用户名')).toBeInDocument()
         |            ^
      10 |     expect(getByLabelText('密码')).toBeInDocument()
      11 |     expect(getByLabelText('性别')).toBeInDocument()
      12 |   })

      at Object.getElementError (node_modules/@testing-library/dom/dist/config.js:34:12)
      at getAllByLabelText (node_modules/@testing-library/dom/dist/queries/label-text.js:121:38)
      at getByLabelText (node_modules/@testing-library/dom/dist/query-helpers.js:54:17)
      at Object.it (src/components/__test__/form.spec.jsx:9:12)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        3.688s
Ran all test suites related to changed files.

```

要实现好业务代码才能够纠正这个错误，因此我们简单地向组件中添加相关元素。

```jsx
// src/components/form.jsx
import React from "react";

function Form() {
  return (
    <form>
      <label htmlFor="username">用户名</label>
      <input id="username"></input>
      <label htmlFor="password">密码</label>
      <input id="password"></input>
      <label htmlFor="gender">性别</label>
      <select id="gender"></select>
    </form>
  );
}

export default Form;
```

此时我们期望测试结果是一盏 <span styles="color:green">绿灯</span>，
如果此时测试仍然是失败的话，说明写出 bug 了。幸好测试结果说明了我们的努力没有全部白费。

```plaintext
PASS  src/components/__test__/form.spec.jsx
  Form
    √ should have 3 fields, which are username, password and gender (52ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
```

但是，对细节敏感的我们发现了一个问题，密码字段的输入框应该是一个 `type="password"` 的
`input`，不然用户在输入的时候，密码就会大公开。emm，这里我们需要更新测试用例来验证这个需求。

```jsx
it("should have 3 fields, which are username, password and gender", () => {
  const { getByLabelText } = render(<Form></Form>);

  expect(getByLabelText("用户名")).toBeInTheDocument();
  expect(getByLabelText("密码")).toBeInTheDocument();
  expect(getByLabelText("密码")).toHaveAttribute("type", "password");
  expect(getByLabelText("性别")).toBeInTheDocument();
});
```

保存，测试结果应该是一盏红灯。

```plaintext
 FAIL  src/components/__test__/form.spec.jsx
  Form
    × should have 3 fields, which are username, password and gender (80ms)

  ● Form › should have 3 fields, which are username, password and gender

    expect(element).toHaveAttribute("type", "password") // element.getAttribute("type") === "password"

    Expected the element to have attribute:
      type="password"
    Received:
      null

       9 |     expect(getByLabelText('用户名')).toBeInTheDocument()
      10 |     expect(getByLabelText('密码')).toBeInTheDocument()
    > 11 |     expect(getByLabelText('密码')).toHaveAttribute('type', 'password')
         |                                  ^
      12 |     expect(getByLabelText('性别')).toBeInTheDocument()
      13 |   })
      14 | })

      at Object.it (src/components/__test__/form.spec.jsx:11:34)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        5.631s
Ran all test suites related to changed files.
```

## 总结
