​	在编写虚拟DOM (Virtual DOM)之前，你需要知道两件事情。你并不需要对 React的源码或者其他实现虚拟DOM的源码有深入的了解。它们非常的巨大而且复杂，实际上，你只需要编写不到50行的代码即可实现虚拟DOM的主要部分。只需要不到50行代码哦！！！



​	这里有两个概念：

- 虚拟DOM是真实DOM（Real DOM）的一种表示。

- 当我们在虚拟DOM树（Virtual DOM Tree）上修改一些东西，我们会得到一个信息虚拟树（Virtual Tree）。两棵树（新的和旧的）通过算法来进行比较，找到不同的地方，然后只对真实的DOM进行最少的必要操作。

  现在让我们开始深入探讨这些概念。

> 第二部分关于在虚拟DOM中设置 props 和 events 的文章在[这里](https://medium.com/@deathmood/write-your-virtual-dom-2-props-events-a957608f5c76) 



### 表示我们的 DOM 树

​	首先，我们需要一个地方来存储我们的DOM树。我们可以用传统的JS对象来实现。假设我们有一棵如下的树：



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

​	这看上去非常不错，让我们将节点的**props**暂时放在一边。我们稍后再讨论它们。我们理解虚拟DOM的基本概念不需要**props**，它们会徒增不少的复杂度。



### 处理改变

​	好了，现在我们已经可以将虚拟DOM转换为真实DOM了，是时候开始考虑如何去比较我们的虚拟DOM树了。首先，我们需要编写一个比较算法，它会比较新的树和旧的树之间的区别，然后只对真实DOM树进行最少的必要操作。

​	如何去比较两棵树？我们需要处理下一个场景。

- 添加了一个节点, 我们需要通过`appendChild(...)`来添加

  ![img](https://cdn-images-1.medium.com/max/800/1*GFUWrX6pBgiDQ5Z-IvzjUw.png)

- 节点被移除了，我们需要通过`removeChild(...)`来处理

  ![img](https://cdn-images-1.medium.com/max/800/1*VRoYwAeWPF0jbiWXsKb2HA.png) 

- 节点被替换了，我们需要通过`replaceChild(...)`来处理

  ![img](https://cdn-images-1.medium.com/max/800/1*6iQYEH0APjbuPvYmnD7Qlw.png) 

- 我们需要去比较node树的子节点

  ![img](https://cdn-images-1.medium.com/max/800/1*x1Eq-uuqgL0z9d9qn_opww.png) 

  让我们编写一个叫做`updateElement(...)`的函数，这个函数可以接受3个变量`$parent, newNode 和 oldNode`, `$parent是我们虚拟DOM的真实DOM中的父元素，相比较。现在我们看看如何去处理上面描述的所有情况。



### 新增节点

​	这里挺简单的，我就不仔细说明了：

```javascript
function updateElement($parent, newNode, oldNode) {
    if(!oldNode) {
        $parent.appendChild(
        	createElement(newNode)
        );
    }
}
```



### 节点被删除了

​	这种情况下，我们有一个问题，如果在虚拟DOM中的没有真实DOM中的某个节点，我们需要在真实DOM中移除它，我们应该怎么做呢？因为我们有了父元素（被传递给函数的），从而我们去引用真实的DOM去执行`$parent.removeChild(...)`。但是我们不知道。如果我们知道节点相对于他的父元素的位置，我们可以用`$parent.childNodes[index]`来引用它，`index`是需要删除的节点相对于它的父元素的位置。

​	现在index应该要传给我们的函数了，所以我们的代码应该是：

```javascript
function updateElement($parent, newNode, oldNode, index = 0) {
    if(!oldNode) {
        $parent.appendChild(
        	createElement(newNode)
        );
    } else if (!newNode) {
        $parent.removeChild(
        	$parent.childNodes(index)
        )
    }
}
```



### 节点改变

首先我们需要去编写一个比较两个节点的函数，告诉我们节点是否真的被改变了。我们需要考虑它可能是元素，也可能是文本。

```javascript
function changed(node1, node2) {
    return typeof node1 !== typeof node2 ||
           typeof node1 === 'string' && node1 !== node2 ||
           node1.type !== node2.type
}
```

现在，我们有了当前节点相对于父元素的索引，我们可以简单的替换成一个新创建的节点

```javascript
function updateElement($parent, newNode, oldNode, index = 0) {
    if(!oldNode) {
        $parent.appendChild(
        	createElement(newNode)
        );
    } else if (!newNode) {
        $parent.removeChild(
        	$parent.childNodes[index]
        );
    } else if (changed(newNode, oldNode)) {
        $parent.replaceChild(
        	createElement(newNode),
            $parent.childNodes[index]
        );
    }
}
```



### 遍历比较子元素

​	最后，我们需要通过调用`updateElement(...)`去遍历每一个节点的子元素并进行比较。是的，又要使用递归了。

​	但是我们在写代码之前需要考虑一下几件事：

- 我们只需要比较元素节点的子元素（文本节点不能有子元素）
- 我们需要传递当前节点，作为`parent` 参数
- 我们需要一个一个的去比较子元素，即使一些子元素是'undefined'，这个没问题，我们的函数可以处理这种情况。
- 最后，***index***，它是子元素节点的索引相对于`children`数组。

```javascript
function updateElement($parent, newNode, oldNode, index = 0) {
    if(!oldNode) {
        $parent.appendChild(
        	createElement(newNode)
        );
    } else if (!newNode) {
        $parent.removeChild(
        	$parent.childNode[index]
        );
    } else if (changed(newNode, oldNode)) {
        $parent.replaceChild(
        	createElement(newNode),
            $parent.chaildNodes[index]
        );
    } else if (newNode.type) {
        const newLength = newNode.children.length;
        const oldLength = oldNode.children.length;
        for(let i = 0; i < newLength || i < oldLength; i++) {
            updateElement(
            	$parent.childNodes[index], 
                newNode.children[i],
                oldNode.children[i],
                i
            );
        }
    }
}
```



### 将它们整合在一起

```javascript
/** @jsx h */

function h(type, props, ...children) {
  return { type, props, children };
}

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

function changed(node1, node2) {
  return typeof node1 !== typeof node2 ||
         typeof node1 === 'string' && node1 !== node2 ||
         node1.type !== node2.type
}

function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(
      createElement(newNode)
    );
  } else if (!newNode) {
    $parent.removeChild(
      $parent.childNodes[index]
    );
  } else if (changed(newNode, oldNode)) {
    $parent.replaceChild(
      createElement(newNode),
      $parent.childNodes[index]
    );
  } else if (newNode.type) {
    const newLength = newNode.children.length;
    const oldLength = oldNode.children.length;
    for (let i = 0; i < newLength || i < oldLength; i++) {
      updateElement(
        $parent.childNodes[index],
        newNode.children[i],
        oldNode.children[i],
        i
      );
    }
  }
}

// ---------------------------------------------------------------------

const a = (
  <ul>
    <li>item 1</li>
    <li>item 2</li>
  </ul>
);

const b = (
  <ul>
    <li>item 1</li>
    <li>hello!</li>
  </ul>
);

const $root = document.getElementById('root');
const $reload = document.getElementById('reload');

updateElement($root, a);
$reload.addEventListener('click', () => {
  updateElement($root, b, a);
});
```

### 总结

​	恭喜，我们已经完成了。我们就实现了一个虚拟DOM，并且它是好使的。我希望你在阅读完这篇文章以后能够理解虚拟DOM的基本概念，和react背后的工作原理。

 	然而，我这里还有一些没有提到的东西，我会在将来的文章中尽量讲到：

- 设定元素属性（props），然后遍历比对/更新它们
- 处理事件，给我们的元素添加事件监听器
- 让我们的虚拟DOM能够在组件中工作，像react
- 获得真实DOM节点的引用
- 虚拟DOM与JS库相结合去操作真实DOM，像jQuery和它的插件。
- 等等...

> 更新：第二篇文章，关于如何给我们的虚拟DOM设置属性和事件的[地址](https://medium.com/@deathmood/write-your-virtual-dom-2-props-events-a957608f5c76) 