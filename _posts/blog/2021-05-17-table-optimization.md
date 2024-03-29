---
layout: post
title: react-hooks 可编辑 table 性能优化
category: blog
description: react-hooks 可编辑 table 性能优化
---

在最近项目中，自己基于 antd 开发了一个可编辑 table 的组件（双击单元格可编辑，支持单元格直接展示下拉框勾选框等组件），在正常使用，数据量少情况下
是没有问题的效果也挺好，但是在大数据量的时候，比如说 100 条数据的时候，在单元格失焦保存的时候会有明显的延迟，在首屏加载的时候也是出现了要等挺久
才能渲染出组件的问题。

#### 1. 这个组件我是用的 react-hooks 写的，我在想是不是因为 hooks 频繁渲染的原因导致了我组件的卡顿。

#### 2. 调试代码后发现，在首次加载或者筛选数据的时候，因为向 table 组件传入了我的 loading 的 state 状态，每当 loading 的状态改变，就会重新渲染一次我的 table 组件。从 false 到 true 再到得到后台数据后的渲染一共加载了 3 次我的 table 组件，而我的 table 组件因为某些单元格直接展示下拉框和勾选框的原因，100 条数据可能会渲染 500 个类似的直接展示的组件，导致卡顿。

#### 3. loading 的 state 状态有问题，那么传入 table 组件的所有 props 属性的更改都会导致我的组件的渲染。所以我将 loading 状态提出到了 table 组件之外，在外层进行 loading 状态的更新，这样，首次加载和筛选的渲染只会渲染从后台拿到数据更新的 1 次，从 3 次到 1 次渲染，明显性能好了很多。其他传入 table 组件的属性，在外层用 useMemo 对 table 组件进行缓存，添加必要的数据的依赖项，在依赖项发生更新时再去重新渲染最新的组件。这样改了之后，性能确实好了很多。但是还是操作起来还是会有延迟。

#### 4. 继续调试代码， 发现我在操作一个单元格的数据后，用 usereducer 去更新 table 的数据，是更新了整个 table 的数据，导致了整个可编辑 table 组件的渲染，500 个表单组件的渲染，肯定会造成一定的性能卡顿和延迟。所以，我想做的是在我操作单元格的数据的时候，只去渲染单元格的自定义 cell 组件，而不是 table 所有的单元格组件。

#### 5. 在看了 antd 的文档后，确实发现有一个 shouldCellUpdate 的属性，用来自己控制对应单元格是否需要更新，在我的理解应该就是每个单元格组件的 shouldComponentUpdate，根据前后两次的值是否一致来控制单元格组件的更新渲染。添加之歌属性后个，果然性能大幅提升，不再延迟卡顿了

```javascript
//缓存DataTable组件 防止重复渲染
const DataTableComponent = useMemo(
  () => (
    <DataTable
      fromPage={PageRouterConfig.ORDER_PAGE}
      noScroll
      hiddenSummary
      tableColumns={columns}
      dispatch={dispatch}
      table={state.table}
      tableKey={"table"}
      onlyCheck={onlyCheck}
      controlUpdate
    />
  ),
  [columns, state.table, onlyCheck]
);
```

```javascript
  //控制单元格是否渲染 大数据性能优化
  shouldCellUpdate(record, prevRecord) => {
  },
```
