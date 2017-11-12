## 值相关工具

具体情况具体分析

1. compareAscending

2. nativeKeysIn

3. size

3. Id值访问器函数
```
function uniqueId(prefix) {
	var id = ++idCounter
	return toString(prefix) + id
}
```