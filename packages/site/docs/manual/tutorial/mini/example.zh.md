---
title: 绘制 Tutorial 案例
order: 1
---

本文将进行 **Tutorial 案例** 的简单绘制和配置。通过本文，你将知道创建一般图时一些常用的配置项及其作用。

## 基础绘制

### 创建容器

为了降低小程序中绘图的难度，我们封装了小程序组件，方便用户快速上手 F6。

[支付宝小程序组件](https://www.npmjs.com/package/@antv/f6-alipay)

```bash
npm install --save @antv/f6-alipay
```

`/pages/index/index.json`

```json
{
  "defaultTitle": "f6-demo",
  "usingComponents": {
    "f6-canvas": "@antv/f6-alipay/es/container/container"
  }
}
```

`/pages/index/index.axml`

```
<f6-canvas
  width="{{width}}"
  height="{{height}}"
  forceMini="{{false}}"
  pixelRatio="{{pixelRatio}}"
  onInit="onCanvasInit"
  onTouchEvent="onTouch"
/>
```

[微信小程序组件](https://www.npmjs.com/package/@antv/f6-wx)

```bash
npm install --save @antv/f6-wx
```

`/pages/index/index.json`

```
{
  "usingComponents": {
    "f6-canvas": "@antv/f6-wx/container/container"
  }
}
```

`/pages/index.index.wxml`

```

  <f6-canvas
    width="{{canvasWidth}}"
    height="{{canvasHeight}}"
    pixelRatio="{{pixelRatio}}"
    onInit="{{onCanvasInit}}"
    onTouchEvent="{{onTouch}}" >
  </f6-canvas>

```

### 数据准备

引入 F6 的数据源为 JSON 格式的对象。该对象中需要有节点（`nodes`）和边（`edges`）字段，分别用数组表示：

```javascript
const initData = {
  // 点集
  nodes: [
    {
      id: 'node1', // 节点的唯一标识
      x: 100, // 节点横坐标
      y: 200, // 节点纵坐标
      label: '起始点', // 节点文本
    },
    {
      id: 'node2',
      x: 300,
      y: 200,
      label: '目标点',
    },
  ],
  // 边集
  edges: [
    // 表示一条从 node1 节点连接到 node2 节点的边
    {
      source: 'node1', // 起始点 id
      target: 'node2', // 目标点 id
      label: '我是连线', // 边的文本
    },
  ],
};
```

<span style="background-color: rgb(251, 233, 231); color: rgb(139, 53, 56)"><strong>⚠️ 注意:</strong></span>

- `nodes` 数组中包含节点对象，唯一的 `id` 是每个节点对象中必要的属性，`x`、 `y` 用于定位；
- `edges` 数组中包含边对象，`source` 和 `target` 是每条边的必要属性，分别代表了该边的起始点 `id` 与 目标点 `id`。
- 点和边的更多属性参见：[内置的节点](/zh/docs/manual/middle/elements/nodes/defaultNode)，[内置的边](/zh/docs/manual/middle/elements/edges/defaultEdge) 教程。

### 图实例化

图实例化时，至少需要为图设置容器、宽、高：

```javascript
const graph = new F6.Graph({
  container: null, // 如果为小程序，则传null或者不传
  renderer: 'mini', // renderer 目前支持`mini`和`mini-native`两个值，可以通过onCanvasInit回调获取
  context: ctx, // 由于小程序中无法直接获取canvas的context，所以需要从外部传入
  width: 800, // Number，必须，图的宽度
  height: 500, // Number，必须，图的高度
});
```

### 图的渲染

数据的加载和图的渲染是两个步骤，可以分开进行。

```javascript
// const initData = { ... }
//  const graph = ...
graph.data(initData); // 加载数据
graph.render(); // 渲染
```

### 绘制结果

调用 `graph.render()` 方法之后，F6 引擎会根据加载的数据进行图的绘制。结果如下：

<img src='https://gw.alipayobjects.com/mdn/rms_f8c6a0/afts/img/A*YTfpQYVGhuEAAAAAAAAAAABkARQnAQ' width=400 alt='img' />

## 真实数据加载

上文中，我们使用了仅含有两个节点和一条边的数据，直接将数据定义放在了代码中。而真实场景的数据通常是远程接口请求加载的。为了方便，我们已经给读者准备好了一份 JSON 数据文件，地址如下：<br />`https://gw.alipayobjects.com/os/basement_prod/6cae02ab-4c29-44b2-b1fd-4005688febcb.json`，小程序中请求线上数据需使用`my.request(支付宝中)`或者`wx.request(微信中)`，由于请求线上数据还需设置请求白名单，所以在示例中，我们把数据放在了本地 data.js 文件中。请求相关问题参考小程序相关文档即可。

运行后，我们得到了下图结果：

<img src='https://gw.alipayobjects.com/mdn/rms_f8c6a0/afts/img/A*KzQaSZKIsQoAAAAAAAAAAABkARQnAQ' width=550 height=350 alt='img' />

乍看之下，图像有点奇怪，实际上数据已经正确的加载了进来。由于数据量比较大，节点和边都非常的多，显得内容比较杂乱。另外由于画布大小的限制，实际的图被画布截断了，目前只能看见部分内容，这个问题后文会继续解决。

请看下面摘取自 tutorial-data.json 的部分数据，我们发现数据中节点定义了位置信息 `x` 与 `y`，并且很多节点的  `x` 和 `y` 不在图的宽高（`width: 800, height: 600`）范围内：

```json
{
  "nodes": [
    { "id": "0", "label": "n0", "class": "c0", "x": 1000, "y": -100 },
    { "id": "1", "label": "n1", "class": "c0", "x": 300, "y": -10 }
    //...
  ],
  "edges": [
    //...
  ]
}
```

由于 F6 在读取数据时，发现了数据中带有位置信息（`x` 和 `y`），就会按照该位置信息进行绘制，这是为了满足按照原始数据绘制的需求设计的。但为优化超出屏幕到问题，F6 提供了图的两个相关配置项：

- `fitView`：设置是否将图适配到画布中；
- `fitViewPadding`：画布上四周的留白宽度。

我们将实例化图的代码更改为如下形式：

```javascript
const graph = new F6.Graph({
  // ...
  fitView: true,
  fitViewPadding: [20, 40, 50, 20],
});
```

上述代码将会生成如下图： <img src='https://gw.alipayobjects.com/mdn/rms_f8c6a0/afts/img/A*wAXzRJaNTXUAAAAAAAAAAABkARQnAQ' width=850 alt='img' />

可以看到，图像已经可以自动适配画布的大小，完整的显示了出来。

## 常用配置

本文使用到的配置以及后续 Tutorial 将会使用到到常用配置如下：

| 配置项 | 类型 | 选项 / 示例 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| fitView | Boolean | true / false | false | 是否将图适配到画布大小，可以防止超出画布或留白太多。 |
| fitViewPadding | Number / Array | 20 / [ 20, 40, 50, 20 ] | 0 | 画布上的四周留白宽度。 |
| animate | Boolean | true / false | false | 是否启用图的动画。 |
| modes | Object | {<br />  default: [ 'drag-node', 'drag-canvas' ]<br />} | null | 图上行为模式的集合。由于比较复杂，按需参见：[F6 中的 Mode](/zh/docs/manual/middle/states/mode) 教程。 |
| defaultNode | Object | {<br />  type: 'circle',<br />  color: '#000',<br />  style: {<br />    ......<br />  }<br />} | null | 节点默认的属性，包括节点的一般属性和样式属性（style）。 |
| defaultEdge | Object | {<br />  type: 'polyline',<br />  color: '#000',<br />  style: {<br />    ......<br />  }<br />} | null | 边默认的属性，包括边的一般属性和样式属性（style）。 |
| nodeStateStyles | Object | {<br />  hover: {<br />    ......<br />  },<br />  select: {<br />    ......<br />  }<br />} | null | 节点在除默认状态外，其他状态下的样式属性（style）。例如鼠标放置（hover）、选中（select）等状态。 |
| edgeStateStyles | Object | {<br />  hover: {<br />    ......<br />  },<br />  select: {<br />    ......<br />  }<br />} | null | 边在除默认状态外，其他状态下的样式属性（style）。例如鼠标放置（hover）、选中（select）等状态。 |

<span style="background-color: rgb(251, 233, 231); color: rgb(139, 53, 56)"><strong>⚠️ 注意:</strong></span><br />若需更换数据，请替换  `'https://gw.alipayobjects.com/os/basement_prod/6cae02ab-4c29-44b2-b1fd-4005688febcb.json'`  为新的数据文件地址。
