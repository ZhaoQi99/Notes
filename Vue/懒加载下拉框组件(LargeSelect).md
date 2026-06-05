
> 基于 Element UI `el-select` 封装，解决大数据量下拉列表渲染性能问题，避免一次性渲染全部选项。
---

# Usage

## Basic Usage

```html
<template>
  <div>
    <LargeSelect
      v-model="value"
      :options="options"
      @change="change"
      placeholder="请选择"
      clearable
    />
  </div>
</template>

<script>
import LargeSelect from '@/components/LargeSelect'

export default {
  components: { LargeSelect },
  data() {
    return {
      value: null,
      options: [],
    }
  },
  methods: {
    change(val) {
      console.log('选中值：', val)
    },
  },
}
</script>
```

## Custom Option Rendering

```html
<LargeSelect v-model="value" :options="options">
  <template #default="{ options }">
    <el-option
      v-for="item in options"
      :key="item.id"
      :value="item.username"
    >
      <span style="float: left">{{ item.username }}（{{ item.real_name }}）</span>
    </el-option>
  </template>
</LargeSelect>
```

# API

## Attributes

|       参数        |        说明         |       类型        |        可选值        |   默认值   |
| :-------------: | :---------------: | :-------------: | :---------------: | :-----: |
| value / v-model |        绑定值        | string / number |         -         |    -    |
|     options     |      完整选项数据       |      Array      |         -         |    -    |
|    increment    |     每次滚动加载的条数     |     Number      |         -         |  `10`   |
|   placeholder   |        占位符        |     String      |         -         |  `请选择`  |
|    multiple     |       是否多选        |     Boolean     |         -         | `false` |
|    clearable    |     是否显示清空按钮      |     Boolean     |         -         | `false` |
|    valueKey     |    选项数据中值的字段名     |     String      |         -         | `value` |
|    labelKey     |    选项数据中标签的字段名    |     String      |         -         | `text`  |
|   selectStyle   | 传递给 el-select 的样式 |     Object      |         -         |  `{}`   |
|      size       |       输入框尺寸       |     String      | medium/small/mini |    -    |

## Events

| 事件名称 |       说明       |                     回调参数                      |
| :------: | :--------------: | :-----------------------------------------------: |
| `change` | 选中值变化时触发 | 目前的选中值 `(value: string \| number \| array)` |

## Slots

|   名称    |      说明      |    作用域参数     |
| :-------: | :------------: | :---------------: |
| `default` | 自定义选项内容 | `{ options: [] }` |

# 实现
```html title:src/components/LargeSelect/index.vue
<template>
  <el-select
    :popper-append-to-body="false"
    :clearable="clearable"
    filterable
    :placeholder="placeholder"
    v-model="value_"
    :multiple="multiple"
    :filter-method="filterMethod"
    @change="change"
    @visible-change="visibleChange"
    v-el-select-loadmore="loadMore()"
    :style="selectStyle"
    :size="size"
    :value-key="valueKey"
  >
    <slot :options="this.options_.slice(0, rangeNumber)">
      <el-option
        v-for="(item, i) in options_.slice(0, rangeNumber)"
        :key="item[valueKey] + i"
        :label="item[labelKey]"
        :value="item[valueKey]"
      >
      </el-option>
    </slot>
  </el-select>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'LargeSelect',
  props: {
    value: {
      required: true,
    },
    options: {
      type: Array,
      default: () => [],
      required: true,
    },
    increment: {
      type: Number,
      default: 10,
    },
    placeholder: {
      type: String,
      default: '请选择',
    },
    multiple: {
      type: Boolean,
      default: false,
    },
    clearable: {
      type: Boolean,
      default: false,
    },
    valueKey: {
      type: String,
      default: 'value',
    },
    labelKey: {
      type: String,
      default: 'text',
    },
    selectStyle: {
      type: Object,
      default: () => {},
    },
    size: {
      type: String,
      default: 'small',
    },
  },
  watch: {
    value: {
      handler(val) {
        this.value_ = val
      },
      immediate: true,
    },
  },
  data() {
    return {
      value_: null,
      options_: [],
      rangeNumber: this.increment,
    }
  },

  mounted() {},
  directives: {
    'el-select-loadmore': (el, binding) => {
      const selectDropdownWrap = el.querySelector('.el-select-dropdown .el-select-dropdown__wrap')
      if (selectDropdownWrap) {
        selectDropdownWrap.addEventListener('scroll', function () {
          const condition = this.scrollHeight - this.scrollTop <= this.clientHeight
          if (condition) binding.value()
        })
      }
    },
  },

  methods: {
    change(val) {
      this.$emit('change', val)
    },
    loadMore() {
      return () => (this.rangeNumber += this.increment)
    },
    visibleChange(val) {
      if (val) {
        this.options_ = this.options
        // this.filterMethod()
      }
    },
    filterMethod: _.debounce(function (value) {
      console.log(value)
      if (value) {
        this.options_ = this.options.filter((item) => {
          return item[this.labelKey].toLowerCase().includes(value.toLowerCase())
        })
      } else {
        this.options_ = this.options
      }
    }, 500),
  },
}
</>

<style scoped lang="less">
/deep/ .el-select-dropdown.el-popper {
  top: auto !important;
}
</style>

```