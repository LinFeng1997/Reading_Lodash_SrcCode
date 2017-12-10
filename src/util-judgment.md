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
对于复杂函数，注释有助于我们充分理解它
```
/**
* The base implementation of `_.isEqual` which supports partial comparisons
* and tracks traversed objects.
*
* @private
* @param {*} value The value to compare.
* @param {*} other The other value to compare.
* @param {boolean} bitmask The bitmask flags.
*  1 - Unordered comparison
*  2 - Partial comparison
* @param {Function} [customizer] The function to customize comparisons.
* @param {Object} [stack] Tracks traversed `value` and `other` objects.
* @returns {boolean} Returns `true` if the values are equivalent, else `false`.
*/
```
这段的大意就是baseIsEqual是_.isEqual的实现基础，它支持部分的比较并且跟踪遍历对象。属于私有函数。

传的value和other就是它要比较的两个元素，至于bitmask是一个比较啥的标志位，如果是1就无序比较，如果是2就部分比较。另外两个参数customizer和stack分别是自定义比较规则和跟踪遍历对象的栈。

它是怎么工作的呢？
```
function baseIsEqual(value, other, bitmask, customizer, stack) {
	//先判断全等
	if (value === other) {
	  return true;
	}
	//非对象比较、包括NaN
	if (value == null || other == null || (!isObjectLike(value) && !isObjectLike(other))) {
	  return value !== value && other !== other;
	}
	//转交给baseIsEqualDeep去判断
	return baseIsEqualDeep(value, other, bitmask, customizer, baseIsEqual, stack);
}
```
//Todo:分析复杂判断
2. baseIsEqualDeep
这就来到了baseIsEqualDeep，等于说两个非对象类型之间的比较就在baseIsEqual里比较完了。它的注释我们不必看了，因为它就是baseIsEqual的特殊版本而已，专门比较对象类型，包括数组。

只不过工作流程比较麻烦：我们拿官方例子来下断点看流程：
```
var object = { 'a': 1 };
var other = { 'a': 1 };
debugger

_.isEqual(object, other);
```
<img src="./assets/util-baseIsEqualDeep-1.png">
先判断是不是数组并获取Tag->再判断Tag是否相等->初始化栈

<img src="./assets/util-baseIsEqualDeep-2.png">
靠栈底元素兵分两栈->总栈推入元素->相同Tag一条流程，有标志位一条流程

最终进入equalObjects(equalArrays或者equalFunc)->元素出栈 

>这里注明一下：equalFunc是用户自定义比较函数

3. equalArrays
由于数组是引用对象，所以也不能直接判断相等。

让我们先来看注释
```
/**
* A specialized version of `baseIsEqualDeep` for arrays with support for
* partial deep comparisons.
*/
```
这就是个特殊的baseIsEqualDeep版本，从上一个分析可以看出来baseIsEqualDeep兵分三路，这一路就是针对数组的。改一下例子：
```
var object =[1,2];
var other = [1,2];
_.isEqual(object, other);
```
<img src="./assets/equalArrays.png">

它首先对bitmask和1(COMPARE_PARTIAL_FLAG)做了与运算，也就是只取个位。然后获取两个数组的长度，针对部分比较进行处理后开始遍历两个数组的值。
<img src="./assets/equalArrays.png">
接着就出现了三种情况：有compared、seen为真、其他一般情况。
第一种情况如果compared的布尔运算表达式为真就continue，否则返回false，第二种情况seen是部分比较的处理，这个例子没有涉及，第三种情况就是直接比较两个值。

3. equalObjects
和equalArrays类似，是baseIsEqualDeep的另一条支线
<img src="./assets/equalObjects.png">
先通过key获取长度，比较完长度再比较值

<img src="./assets/equalObjects2.png">
这里比较了每个键值，并且会跳过constructor的比较，如果没跳过就还会比较对象的原型

3. equalByTag
这就是一个判断同一种对象是否相等的函数，不过只能判断布尔日期数字、错误、正则字符串，full版本的更加全面一些
```
function equalByTag(object, other, tag, bitmask, customizer, equalFunc, stack) {
	switch (tag) {
	  case boolTag:
	  case dateTag:
	  case numberTag:
	    // Coerce booleans to `1` or `0` and dates to milliseconds.
	    // Invalid dates are coerced to `NaN`.
	    return eq(+object, +other);
	  case errorTag:
	    return object.name == other.name && object.message == other.message;
	  case regexpTag:
	  case stringTag:
	    // Coerce regexes to strings and treat strings, primitives and objects,
	    // as equal. See http://www.ecma-international.org/ecma-262/7.0/#sec-regexp.prototype.tostring
	    // for more details.
	    return object == (other + '');
	}
	return false;
}
```