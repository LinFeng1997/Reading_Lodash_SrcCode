## 转换工具

1. 类数组转数组
```
function toArray(value) {
	if (!isArrayLike(value)) {
		return values(value)
	}
	return value.length ? copyArray(value) : []
}
```
2. 取整 **来自于full版本**
```
function toInteger(value) {
  var result = toFinite(value),
    remainder = result % 1;

  return result === result ? (remainder ? result - remainder : result) : 0;
}
```
3. 转成数值
```
var toNumber = Number;
```
3. 转字符串
```
function toString(value) {
	if (typeof value == 'string') {
	  return value;
	}
	return value == null ? '' : (value + '');
}
```
3. 剩余(rest)参数转化
```
function overArg(func, transform) {
	return function(arg) {
		return func(transform(arg))
	}
}
```
3. 包装值函数
```
function identity(value) {
	return value
}
//可以实现深复制一个数组
var copyArr = [1,2,3,4,5].map(_.identity);
```

3. 转义
```
var htmlEscapes = {
'&': '&amp;',
'<': '&lt;',
'>': '&gt;',
'"': '&quot;',
"'": '&#39;'
};

var escapeHtmlChar = basePropertyOf(htmlEscapes);
var reUnescapedHtml = /[&<>"']/g,
reHasUnescapedHtml = RegExp(reUnescapedHtml.source);

function escape(string) {
	string = toString(string)
	return (string && reHasUnescapedHtml.test(string)) ? string.replace(reUnescapedHtml, escapeHtmlChar) : string
}
```