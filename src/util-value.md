## 值相关工具

1. Id值访问器函数
```
function uniqueId(prefix) {
	var id = ++idCounter
	return toString(prefix) + id
}
```