​	在编写虚拟DOM (Virtual DOM)之前，你需要知道两件事情。你并不需要对 React的源码或者其他实现虚拟DOM的源码有深入的了解。它们非常的巨大而且复杂，实际上，你只需要编写不到50行的代码即可实现虚拟DOM的主要部分。只需要不到50行代码哦！！！



​	这里有两个概念：

- 虚拟DOM是真实DOM（Real DOM）的一种表示。

- 当我们在虚拟DOM树（Virtual DOM Tree）上修改一些东西，我们会得到一个信息虚拟树（Virtual Tree）。两棵树（新的和旧的）通过算法来进行比较，找到不同的地方，然后支队真实的DOM进行最少的必要操作。

  现在让我们开始深入探讨这些概念。

> 第二部分关于在虚拟DOM中设置 props 和 events 的文章在这里



### 表示我们的 DOM 树

​	首先，我们需要一个地方来存储我们的DOM树。我们可以用老式的JS对象来实现。假设我们有一棵如下的树：



```html
<ul class='list'>
    <li>item 1</li>
    <li>item 2</li>
</ul>
```

​	看上去还是挺简单的，我们如何用 JS 的对象来表示呢？

```json
{
    type: 'l',
    props: {
        'class': 'list'
    },
    children: [
        {
            type: 'li',
            props: {},
            children: ['item1'],
        },
        {
            type: 'li',
            props: {},
            children: ['item2']
        }
    ]
}
```

​	

​	在这你可能会注意到两件事情：

- 我们使用对象来表示DOM元素，就像：

  ```json
  {
      type: '...',
      props: { ... },
      children: [...]
  }
  ```

  

- 我们使用 JS 的字符串类型来表示DOM的文本节点

  ​	但是我们用这种方法来描述一棵巨大的树的时候就会变得很困难。为了让我们能够更简单的去理解结构，我们需要写一个助手函数。

  ```javascript
  function h (type, props, ...children) {
      return {type, props, children};
  }
  ```

  ​        我们现在可以这样来写 DOM 树：

  ```
  h('ul', {'class': 'list'},
  	h('li', {}, 'item 1'),
  	h('li', {}, 'item 2'),
  )
  ```

  ​	这样看上去更干净，不是吗？让我们更进一步，你曾经听过 JSX 吗？ 我希望在这里能够使用JSX， 所以他是如何工作的？

  ​	如果你阅读 Babel 关于 JSX 的文档， 你就会知道，Babel 将这些代码：

  ```html
  <ul className='list'>
  	<li>item 1</li>
  	<li>item 2</li>
  </ul>
  ```

  ​	编译成

  ```javascript
  React.createElement('ul', { className: 'list' },
  	React.createElement('li', {}, 'item 1'),
  	React.createElement('li', {}, 'item 2'),
  );
  ```

  ​	注意到了和之前相似的地方了吗？如果我们将 React.createElement(...) 替换成 h(...) ，这就证明我们的方法是可行的，通过使用叫做 jsx pragma的东西。我们在源文件顶部加上一句注释

  ```
  /** @jsx h */
  
  <ul className='list'>
  	<li>item 1</li>
  	<li>item 2</li>
  </ul>
  ```

  ​	这是用来告诉 Babel '使用 h 来转译这段JSX 而不是 React.createElement'。在这里你可以用任何东西来替代 'h'。然后都会被编译。

  ​	总的来说，我们将会使用这种方法来写我们的DOM：

  ```
  /** @jsx h */
  
  const a = (
  	<ul className='list'>
  		<li>item 1</li>
  		<li>item 2</li>
  	</ul>
  );
  ```

  ​	当函数 ‘h’执行时， 它会返回一个 JS 的对象， 表示我们的虚拟DOM：

  ```javascript
  const a = {
      {
      	type: 'ul',
      	props: {
              className: 'list',
      	},
      	children: [
              {
                  type: 'li',
                  props: {},
                  children: ['item 1'],
              },
              {
                  type: 'li',
                  props: {},
                  children: ['item 2'],
              }
      	]
      }
  }
  ```

  

### 应用我们的 DOM 表达式

​	我们现在有了可以表示我们 DOM 树和 DOM 树结构的 JS 对象。我们现在需要DOM树的表达式来创建我们真实的DOM。因为我们无法直接append我们的表达式到DOM中去。

​	首先，让我们做一些假设和术语约定：

- 我会将用'$'开头来表示真实的DOM节点。所以**\$parent**是一个真实的节点元素。
- 虚拟DOM表达式会是一个命名为**node**的变量。
- 就像React，你只能够拥有一个根节点，其他的node都会在其中。

​        让我们编写一个函数 **createElement(...)**可以通过输入一个虚拟DOM节点返回一个真实的DOM节点。暂时忘记 'props'和 'children' 吧，我们将会在之后讲到。

```
function createElement(node) {
    if (typeof node === 'string') {
        return document.createTextNode(node);
    }
    return document.createElement(node.type);
}
```

​	所以，因为我们有两种文本节点，一种是普通的 JS 字符串，一种是元素。JS 对象看上去像

```
{
    type: '...',
    props:  { ... },
    children: [...]
}
```

​	从而，我们可以处理文本节点和虚拟元素节点。

​	现在，让我们考虑考虑children，它们可能会是一个文本节点或者元素节点。所以他们也会通过我们的**createElement(...)** 函数。就像递归一样，我们可以执行 **createElemnt(...)**对于每一个元素的children，然后我们再使用**appendChild()**将它们append进我们的元素，就像这样：

```
function createElement(node) {
    if (typeof node === 'string') {
        return document.createTextNode(node);
    }
    const $el = document.createElement(node.type);
    node.children
    .map(createElement)
    .forEach($el.appendChild.bind($el));
    return $el;
}
```

这看上去非常不错，让我们将节点的**props**暂时放在一边。我们稍后再讨论他们。我们理解虚拟DOM的基本概念不需要**props**，它们会徒增不少的复杂度。

