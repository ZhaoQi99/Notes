1. 计算笛卡尔积
```javascript
/**
 * 计算笛卡尔积
 * @param {...Array} a - 要计算笛卡尔积的数组
 * @returns {Array} 笛卡尔积结果
 */
export const cartesian = (...a) => {
  // 使用 reduce 函数来计算笛卡尔积
  return a.reduce((a, b) => {
    // 使用 flatMap 函数来展平结果
    return a.flatMap((d) => {
      // 使用 map 函数来计算笛卡尔积
      return b.map((e) => {
        // 使用 flat 函数来合并数组
        return [d, e].flat()
      })
    })
  })
}

```