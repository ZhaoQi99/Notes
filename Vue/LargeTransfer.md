# 改造
1. 从  [GitHub ElemeFE v2.15.14 ](https://github.com/ElemeFE/element/blob/v2.15.14/packages/transfer/src/main.vue) 上拷贝源码 `packages/transfer/`
2. 修改 `transfer-panel.vue`
	1. 添加 `v-infinite-scroll="loadMore"` 和 `:infinite-scroll-distance="10"`
	2. 修改 `v-for="item in filteredData.slice(0, rangeNumber)"`
	3. 添加 `increment` prop 和 `loadMore` 函数
3. 修改 `index.js` 为 `LargerTransfer`
## 文件结构
```shell
src/components/LargeTransfer

├──      index.js
├──    ﵂  main.vue  
└──    ﵂  transfer-panel.vue 
```
## 问题
可能遇到 `Support for the experimental syntax 'jsx' isn't currently enabled (86:13):` 报错，改为如下写法即可
```javascript title:transfer-panel.vue
// <span>{ this.option[panel.labelProp] || this.option[panel.keyProp] }</span>
h('span', this.option[panel.labelProp] || this.option[panel.keyProp])
```
## 源码

```javascript title:index.js
import LargerTransfer from './main';

/* istanbul ignore next */
LargerTransfer.install = function(Vue) {
  Vue.component(LargerTransfer.name, LargerTransfer);
};

export default LargerTransfer;
```
```javascript title:transfer-panel.vue
<template>
	<el-checkbox-group
	  v-infinite-scroll="loadMore"
	  :infinite-scroll-distance="10"
	  v-model="checked"
	  v-show="!hasNoMatch && data.length > 0"
	  :class="{ 'is-filterable': filterable }"
	  class="el-transfer-panel__list"
  >	
	  <el-checkbox
	    class="el-transfer-panel__item"
	    :label="item[keyProp]"
	    :disabled="item[disabledProp]"
	    :key="item[keyProp]"
	    v-for="item in filteredData.slice(0, rangeNumber)"
    >	
	    <option-content :option="item"></option-content>
	  </el-checkbox>
	</el-checkbox-group>
<template>
<script>
export default {
  ...
	props: {
	  ...
	  increment: {
	    type: Number,
	    default: 10,
	  },
	},
	methods: {
	  ...
		loadMore() {
		  this.rangeNumber += this.increment
		},
	}
</script>
```

## 参考
* [(Transfer)解决：Element-ui 中 Transfer 穿梭框因数据量过大而渲染卡顿问题的三种方法\_el-transfer数据量大-CSDN博客](https://blog.csdn.net/weixin_43405300/article/details/137335363)
* [虚拟列表优化：el-transfer穿梭器，在数据量大的情况下加载卡顿，全选卡顿的问题el-transfer虚拟列表优化 - 掘金](https://juejin.cn/post/7406147685303009316)