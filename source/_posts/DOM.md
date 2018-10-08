title: DOM操作
date: 2015-03-01 15:59:37
tags: Js
---

DOM操作的简单总结。
<!-- more -->

#### 获取:

- getElementById();
- getElementsByTagName();
- getElementsByClassName(); //h5新方法
- querySelector(selectors)
- querySelectorAll(selectors)

#### 获取/设置属性:

    getAttribute(atribute);
    setAttribute(attribute,value);

#### childNodes属性:用于获取任意一个元素的所有子元素.

`element.childNodes`

#### nodeType属性:用于区分不同的节点类型.

    nodeType共有12种可取值，但只有三种具有实用价值.
    nodeType = 1 //元素节点
    nodeType = 2 //属性节点
    nodeType = 3 //文本节点

#### nodeValue属性:用于获取或改变一个**文本**节点的值.

#### firstChild,lastChild

    var nodes = element.childNodes;
    nodes.firstChild = nodes[0];
    nodes.lastChild = nodes[nodes.length-1];

创建节点：
    
    //该方法只能创建元素节点
    var e = document.createElement(element);
    //创建文本节点
    var text = document.createTextNode(value);
    //将节点插入DOM中:
    target.appendChild(e);
    target.appendChild(text);

style属性，操作DOM样式:

    element.style.property
    
    Eg:element.style.color;
    //对于含有"-"的属性，如"font-family"
    element.style.fontFamily