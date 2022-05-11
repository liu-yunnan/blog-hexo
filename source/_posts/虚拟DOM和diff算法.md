---
title: 虚拟DOM和diff算法
date: 2022-04-12 19:20:21
tags: [vue]
cover: https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204131005909.png
---
# 虚拟DOM和diff算法

### 前言

- vdom是实现vue和React的重要基石
- diff算法是vdom中最核心、最关键的部分

- vdom就是创建js对象，用这个对象结构代表真实的dom结构。这个对象上通常含有标签名，标签上的属性，事件监听和子元素们，以及其他属性
- 例如：

![image-20220412160432957](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204121920703.png)

**用js模拟DOM结构：**

![image-20220412160458479](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204121920041.png)

### 为什么使用虚拟DOM，有什么好处

DOM是很慢的。如果我们把一个简单的div元素的属性都打印出来，你会看到：

![img](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204121920511.jpeg)![img](https://pic1.zhimg.com/80/d5cda33e28d83ba12368202645f9e35b_1440w.jpg?source=1940ef5c)

而这仅仅是第一层。真正的 DOM 元素非常庞大，这是因为标准就是这么设计的。而且操作它们的时候你要小心翼翼，轻微的触碰可能就会导致页面重排，这可是杀死性能的罪魁祸首。

相对于 DOM 对象，原生的 **JavaScript 对象处理起来更快，而且更简单**。DOM 树上的结构、属性信息我们都可以很容易地用 JavaScript 对象表示出来：

```js
var element = {
  tagName: 'ul', // 节点标签名
  props: { // DOM的属性，用一个对象存储键值对
    id: 'list'
  },
  children: [ // 该节点的子节点
    {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
  ]
}
```

上面对应的HTML写法是：

```html
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
```

既然原来 DOM 树的信息都可以用 JavaScript 对象来表示，反过来，你就可以根据这个用 JavaScript 对象表示的树结构来构建一棵真正的DOM树。用新渲染的对象树去和旧的树进行对比，记录这两棵树差异。记录下来的不同就是我们需要对页面真正的 DOM 操作，然后把它们应用在真正的 DOM 树上，页面就变更了。这样就可以做到：视图的结构确实是整个全新渲染了，但是也避免了没有必要的dom操作，从而提高性能。

这就是所谓的 Virtual DOM 算法。具体实现步骤：

1. 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
2. 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
3. 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了

Virtual DOM 本质上就是在 **JS 和 DOM 之间做了一个缓存**。可以类比 CPU 和硬盘，既然硬盘这么慢，我们就在它们之间加个缓存：既然 DOM 这么慢，我们就在它们 JS 和 DOM 之间加个缓存。CPU（JS）只操作内存（Virtual DOM），最后的时候再把变更写入硬盘（DOM）。

### 虚拟DOM

**用算法实现虚拟DOM**

```javascript
function VDOM(tagName, props, children) {
    this.tagName = tagName;//元素标签名
    this.props = props;//元素所包含的属性
    this.children = children;//元素的子标签（数组）
}
VDOM.prototype.render = function() {
    // 建立一个真实元素
    var el = document.createElement(this.tagName);
    
    // 往该元素上添加属性
    for (var name in this.props) {
        el.setAttribute(name, this.props[name]);
    }
    
    // children是一个数组
    var children = this.children || [];
    for (var child of children) {
        var childEl = (child instanceof VDOM) ? child.render() //如果子节点是虚拟DOM，递归构建DOM节点
     :document.createTextNode(child);//如果字符串，只构建文本节点
        el.appendChild(childEl);
    }
    return el;
}
```

### diff算法

比较两棵DOM树的差异是 Virtual DOM 算法最核心的部分，这也是所谓的 Virtual DOM 的 diff 算法。**两个树的完全的 diff 算法是一个时间复杂度为 O(n^3) 的问题**。但是在前端当中，你很少会跨越层级地移动DOM元素。所以 Virtual DOM 只会**对同一个层级的元素进行对比**：

![image-20220412172905079](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204121921513.png)

上图中div只会和同一层级的div对比，第二层的只会跟第二层级对比。这样算法复杂度就到了O(n)

![image-20220412184025887](https://liu-yunnan-hexo-blog.oss-cn-beijing.aliyuncs.com/img/202204121921232.png)

在深度优先遍历的时候，每遍历到一个节点就把该节点和新的的树进行对比。如果有差异的话就记录到一个对象里面。

**diff.js 的简单代码实现**：

在同层进行比较时候会出现四种情况：

1、此节点是否被移除 -> 添加新的节点
 2、属性是否被改变 -> 旧属性改为新属性
 3、文本内容被改变-> 旧内容改为新内容
 4、节点要被整个替换 -> 结构完全不相同 移除整个替换

最后返回一个patch对象用来应用到实际的DOM tree更新，它的结构是这样的：

```js
// index记录是哪一层的改变，type表示是哪种变化，第二个属性对应着变化存储相应的内容
patches = {index:[{type: utils.REMOVE/utils.TEXT/utils.ATTRS/utils.REPLACE, index/content/attrs/node: }, ...], ...}
```

```javascript
let utils = require('./utils');

let keyIndex = 0;
function diff(oldTree, newTree) {
    //记录差异的空对象。key就是老节点在原来虚拟DOM树中的序号，值就是一个差异对象数组
    let patches = {};
    keyIndex = 0; // 儿子要起另外一个标识
    let index = 0; // 父亲的表示 1 儿子的标识就是1.1 1.2
    walk(oldTree, newTree, index, patches);
    return patches;
}
//遍历
function walk(oldNode, newNode, index, patches) {
    let currentPatches = []; //这个数组里记录了所有的oldNode的变化
    if (!newNode) { //如果新节点没有了，则认为此节点被删除了
        currentPatches.push({type: utils.REMOVE, index});
        //如果说老节点的新的节点都是文本节点的话
    } else if (utils.isString(oldNode) && utils.isString(newNode)) {
        //如果新的字符符值和旧的不一样
        if (oldNode != newNode) {
            ///文本改变
            currentPatches.push({type: utils.TEXT, content: newNode});
        }
    } else if (oldNode.tagName == newNode.tagName) {
        //比较新旧元素的属性对象
        let attrsPatch = diffAttr(oldNode.attrs, newNode.attrs);
        //如果新旧元素有差异 的属性的话
        if (Object.keys(attrsPatch).length > 0) {
            //添加到差异数组中去
            currentPatches.push({type: utils.ATTRS, attrs: attrsPatch});
        }
        //自己比完后再比自己的儿子们
        diffChildren(oldNode.children, newNode.children, index, patches, currentPatches);
    } else {
        currentPatches.push({type: utils.REPLACE, node: newNode});
    }
    if (currentPatches.length > 0) {
        patches[index] = currentPatches;
    }
}
//老的节点的儿子们 新节点的儿子们 父节点的序号 完整补丁对象 当前旧节点的补丁对象
function diffChildren(oldChildren, newChildren, index, patches, currentPatches) {
    oldChildren.forEach((child, idx) => {
        walk(child, newChildren[idx], ++keyIndex, patches);
    });
}
function diffAttr(oldAttrs, newAttrs) {
    let attrsPatch = {};
    for (let attr in oldAttrs) {
        //如果说老的属性和新属性不一样。一种是值改变 ，一种是属性被删除 了
        if (oldAttrs[attr] != newAttrs[attr]) {
            attrsPatch[attr] = newAttrs[attr];
        }
    }

    // 对比旧节点新增的属性
    for (let attr in newAttrs) {
        if (!oldAttrs.hasOwnProperty(attr)) {
            attrsPatch[attr] = newAttrs[attr];
        }
    }
    return attrsPatch;
}
module.exports = diff;
```

### patch.js的简单实现

把差异应用到真正的DOM树上

```javascript
let keyIndex = 0;
let utils = require('./utils');
let allPatches;//这里就是完整的补丁包
function patch(root, patches) {
    allPatches = patches;
    walk(root);
}
function walk(node) {
    //从patches拿出当前节点的差异
    let currentPatches = allPatches[keyIndex++];
    (node.childNodes || []).forEach(child => walk(child));
    if (currentPatches) {
        doPatch(node, currentPatches);
    }
}
function doPatch(node, currentPatches) {
    currentPatches.forEach(patch => {
        switch (patch.type) {
            case utils.ATTRS:
                for (let attr in patch.attrs) {
                    let value = patch.attrs[attr];
                    if (value) {
                        utils.setAttr(node, attr, value);
                    } else {
                        node.removeAttribute(attr);
                    }
                }
                break;
            case utils.TEXT:
                node.textContent = patch.content;
                break;
            case utils.REPLACE:
                let newNode = (patch.node instanceof Element) ? path.node.render() : document.createTextNode(path.node);
                node.parentNode.replaceChild(newNode, node);
                break;
            case utils.REMOVE:
                node.parentNode.removeChild(node);
                break;
        }
    });
}
module.exports = patch;
```

### 结语

Virtual DOM 算法主要是实现上面步骤的三个函数：VDOM、diff、patch。然后就可以实际的进行使用：

```js
// 1. 构建虚拟DOM
var tree = el('div', {'id': 'container'}, [
    el('h1', {style: 'color: blue'}, ['simple virtal dom']),
    el('p', ['Hello, virtual-dom']),
    el('ul', [el('li')])
])

// 2. 通过虚拟DOM构建真正的DOM
var root = tree.render()
document.body.appendChild(root)

// 3. 生成新的虚拟DOM
var newTree = el('div', {'id': 'container'}, [
    el('h1', {style: 'color: red'}, ['simple virtal dom']),
    el('p', ['Hello, virtual-dom']),
    el('ul', [el('li'), el('li')])
])

// 4. 比较两棵虚拟DOM树的不同
var patches = diff(tree, newTree)

// 5. 在真正的DOM元素上应用变更
patch(root, patches)
```

最后，推荐一个vdom库：[snabbdom](https://github.com/snabbdom/snabbdom)，vue参考它实现的vdom和diff

> 参考文章：https://www.zhihu.com/question/29504639?sort=created


