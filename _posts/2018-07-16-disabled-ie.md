---
layout:     post
title:      "a标签的diabled属性在IE仍可点击解决办法"
subtitle:   "a标签的diabled属性在IE仍可点击解决办法"
date:       2018-07-16 14:12:06
author:     "pompeygao"
header-img: "img/post-bg-js-version.jpg"
tags:
    - 前端开发 
    - react
---

## 问题描述

如下代码，将`<a/>`赋予`disabled`，在chrome浏览器中，`<a/>`样式也变成了灰色，点击`<a/>`无任何反应，这是我们想要的效果；
在IE11浏览器中，`<a/>`样式也变成了灰色，但是点击仍然可以进行操作，说明在IE11下面是有问题的。

![disabled-in-ie11](/img/in-post/disabled-ie/IE1.png)
<small class="img-hint">图</small>
```js
columns = [
    {
        title: '操作',
        render: (text, role) => {
            return (
                <div>
                    <a disabled
                        onClick={this.edit.bind(this, role)}
                    >编辑</a>
                    <Divider type="vertical" />
                    <Popconfirm
                        title="确认删除吗?"
                        onConfirm={this.remove.bind(this, role)}
                    >
                        <a disabled>删除</a>
                    </Popconfirm>
                </div>
            );
        }
    }
];
```

**解决方法**
使用 `pointer-events` 属性：
- auto——效果和没有定义pointer-events属性相同，鼠标不会穿透当前层。在SVG中，该值和visiblePainted的效果相同。
- none——元素不再是鼠标事件的目标，鼠标不再监听当前层而去监听下面的层中的元素。但是如果它的子元素设置了pointer-events为其它值，比如auto，鼠标还是会监听这个子元素的。

`style.less`
```less
.disabled {
    pointer-events: none;
    cursor: default;
}
```

```js
columns = [
    {
        title: '操作',
        render: (text, role) => {
            return (
                <div>
                    <a disabled className='disabled'
                        onClick={this.edit.bind(this, role)}
                    >编辑</a>
                    <Divider type="vertical" />
                    <Popconfirm
                        title="确认删除吗?"
                        onConfirm={this.remove.bind(this, role)}
                    >
                        <a disabled className='disabled'>删除</a>
                    </Popconfirm>
                </div>
            );
        }
    }
];
```
这样就解决了在IE11下面出现的问题。