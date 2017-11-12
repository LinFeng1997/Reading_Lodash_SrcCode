## 判断工具

### 判断数据类型
lodash首先声明了十三种对象的标签变量：(其实这块我觉得用常量更好)
1. Number、String、Boolean、Object基本类型
2. Array、Function、Date、Error、RegExp常用内置对象
3. Arguments、AsyncFunction、GeneratorFunction、Proxy不常用内置对象
```
var argsTag = '[object Arguments]',
	arrayTag = '[object Array]',
	asyncTag = '[object AsyncFunction]',
	boolTag = '[object Boolean]',
	dateTag = '[object Date]',
	errorTag = '[object Error]',
	funcTag = '[object Function]',
	genTag = '[object GeneratorFunction]',
	numberTag = '[object Number]',
	objectTag = '[object Object]',
	proxyTag = '[object Proxy]',
	regexpTag = '[object RegExp]',
	stringTag = '[object String]';
```
这些变量都是给baseGetTag函数获取一个对象类型后对比用的
```
function baseGetTag(value) {
	return objectToString(value)
}
//委托Object.proto.toString
var nativeObjectToString = objectProto.toString
function objectToString(value) {
	return nativeObjectToString.call(value)
}
```
之后一些常见的类型就拿来抽象成函数：
```
//数字
function isNumber(value) {
	return typeof value == 'number' ||
		(isObjectLike(value) && baseGetTag(value) == numberTag)
}
//字符串
function isString(value) {
	return typeof value == 'string' ||
		(!isArray(value) && isObjectLike(value) && baseGetTag(value) == stringTag)
}
//布尔
function isBoolean(value) {
	return value === true || value === false ||
		(isObjectLike(value) && baseGetTag(value) == boolTag)
}
//对象
function isObject(value) {
	var type = typeof value
	return value != null && (type == 'object' || type == 'function')
}

//数组
var isArray = Array.isArray
//日期
function baseIsDate(value) {
	return isObjectLike(value) && baseGetTag(value) == dateTag
}
//函数
function isFunction(value) {
	if (!isObject(value)) {
		return false
	}
	var tag = baseGetTag(value)
	return tag == funcTag || tag == genTag || tag == asyncTag || tag == proxyTag
}
//正则表达式
function baseIsRegExp(value) {
	return isObjectLike(value) && baseGetTag(value) == regexpTag
}
//arguments **来自于full版本**
function baseIsArguments(value) {
  return isObjectLike(value) && baseGetTag(value) == argsTag;
}
```
我们发现数组比较特殊，它是直接引用了Array的isArray方法。

这里我们发现其实还有两个判断类似类型的函数：isArrayLike和isObjectLike，即类数组和类对象类型，比如DOM对象就是一个类数组类型
```
function isArrayLike(value) {
	return value != null && isLength(value.length) && !isFunction(value);
}
function isObjectLike(value) {
	return value != null && typeof value == 'object';
}
```
isObjectLike和isObject的主要区别就在于isObjectLike不认函数。
另外，判断null和undefined也是单独抽象出函数的：
```
function isNull(value) {
	return value === null
}
function isUndefined(value) {
	return value === undefined
}
```
有人可能说这样多麻烦，但是这样写才是优雅的工程代码，万一以后有一天js判断数据类型的方式改变了，我们只需要修改函数内部的逻辑就好了，而不是到处去修改。

### 判断某个简单逻辑
1. 比较大小，如baseGt、baseLt、eq
```
function baseGt(value, other) {
	return value > other
}
function baseLt(value, other) {
    return value < other
}
//考虑NaN不等于自身的情况
function eq(value, other) {
	return value === other || (value !== value && other !== other);
}
```
2. 可扁平化
```
function isFlattenable(value) {
	return isArray(value) || isArguments(value);
}
```
3. 空数组或者空对象
```
function isEmpty(value) {
	if (isArrayLike(value) &&
	  (isArray(value) || isString(value) ||
	    isFunction(value.splice) || isArguments(value))) {
	  return !value.length;
	}
	return !nativeKeys(value).length;
}
```
4. 有穷数值
```
var nativeIsFinite = root.isFinite
function isFinite(value) {
	return typeof value == 'number' && nativeIsFinite(value);
}
```
5. 是坏掉的数值NaN
```
function isNaN(value) {
	return isNumber(value) && value != +value;
}
```
6. 合法的类数组长度
```
function isLength(value) {
	return typeof value == 'number' &&
	  value > -1 && value % 1 == 0 && value <= MAX_SAFE_INTEGER;
}
```
6. 检查对象上有某个属性
```
function has(object, path) {
	return object != null && hasOwnProperty.call(object, path);
}
```

### 复杂的判断逻辑
//Todo:复杂的判断逻辑
1. baseIsEqual、isEqual

2. baseIsEqualDeep

3. equalArrays

3. equalObjects

3. equalByTag