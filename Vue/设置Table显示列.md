# ColumnOptions

## 基础用法

```html unfold
<template>
  <div>
	<column-options size="small" v-model="checkedColumns" :columns="columns" />
  </div>
</template>

<script>
import ColumnOptions from '@/components/ColumnOptions'

export default {
  components: {
	ColumnOptions,
  },
  data () {
	return {
	  checkedColumns: [],
	  columns: [],
	}
  },
  created () {
	this. columns = [
	  { label: '创建时间', prop: 'create_dt' },
	  { label: '操作', prop: 'operation' },
	]
	this. checkedColumns = []
  },
}
</script>
```


## 设置固定列 & 指定显示文本/实际值对应的字段
```html fold
<template>
  <div>
    <column-options
      size="small"
      v-model="checkedColumns"
      :columns="columns"
      :fixedColumns="['operation']"
      column-prop="value"
      column-label="text"
    />
  </div>
</template>

<script>
import ColumnOptions from '@/components/ColumnOptions'

export default {
  components: {
    ColumnOptions,
  },
  data() {
    return {
      checkedColumns: [],
      columns: [],
      fixedColumns: [],
    }
  },
  created() {
    this.columns = [
      { text: '创建时间', value: 'create_dt' },
      { text: '操作', value: 'operation' },
    ]
    this.checkedColumns = []
    this.fixedColumns = ['operation']
  },
}
</script>
```

## ColumnOptions Attributes

|      参数       |        说明        |  类型  |      可选值       | 默认值 |
|:---------------:|:------------------:|:------:|:-----------------:|:------:|
| value / v-model |       绑定值       | array  |         -         |   -    |
|     columns     |    显示列的选项    | array  |         -         |   []   |
|  fixedColumns   |       固定列的值       | array  |         -         |   []   |
|      size       |        尺寸        | string | medium/small/mini |   -    |
|   column-prop   |  实际值对应的字段  | string |         -         |  prop  |
|  column-label   | 显示文本对应的字段 | string |         -         | label  |

## ColumnOptions Events

| 事件名称 | 说明                   |       回调参数        |
| -------- | ---------------------- |:---------------------:|
| input    | 绑定值变化时触发的事件 | 选中的 Column label 值 |

## 源码

```html title:index.vue
<template>
  <div class="column-options">
    <el-popover placement="bottom" trigger="click" popper-class="column-popover">
      <el-button slot="reference" type="primary" :size="size"> 显示列设置 </el-button>
      <div class="column-display">
        <el-checkbox-group v-model="checkedColumns" @change="handleChange($event)">
          <el-checkbox
            v-for="column in columns"
            :key="column.label"
            :label="column.prop"
            style="display: block"
            :checked="value.includes(column.prop) || fixedColumns.includes(column.prop)"
            :disabled="fixedColumns.includes(column.prop)"
          >
            {{ column.label }}
          </el-checkbox>
        </el-checkbox-group>
      </div>
      <div class="column-display-button">
        <el-button type="text" size="mini" @click="handleSelectAll"> 全选 </el-button>
        <el-button type="text" size="mini" @click="handleReset" :disabled="checkedColumns.length === 0">
          重置
        </el-button>
      </div>
    </el-popover>
  </div>
</template>

<script>
export default {
  name: 'ColumnOptions',
  props: {
    value: {
      required: true,
      type: Array,
    },
    columns: {
      type: Array[Object],
      required: true,
    },
    fixedColumns: {
      type: Array[String],
      default: () => [],
    },
    size: {
      type: String,
      default: 'mini',
    },
  },
  data() {
    return {
      checkedColumns: [],
    }
  },
  watch: {
    value: {
      handler(value) {
        this.checkedColumns = value
      },
      immediate: true,
    },
  },
  mounted() {},
  methods: {
    handleSelectAll() {
      this.checkedColumns = this.columns.map((item) => item.prop)
      this.emitChange(this.checkedColumns)
    },
    handleReset() {
      this.$set(this, 'checkedColumns', this.fixedColumns)
      this.emitChange(this.checkedColumns)
    },
    emitChange() {
      this.$emit('input', this.checkedColumns)
    },
    handleChange() {
      this.emitChange()
    },
  },
}
</script>
<style lang="less">
.column-popover {
  padding: 0 0 0 0 !important;
}
</style>

<style lang="less" scoped>
.column-options {
  display: inline-block;
  margin-right: 10px;
}
.column-display {
  width: 110px;
  padding: 12px;
}
.column-display-button {
  border-top: 1px solid #ebeef5;
  padding: 5px 12px;
}
</style>
```
