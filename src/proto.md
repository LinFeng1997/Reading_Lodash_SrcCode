# lodash中的原型编程思想

## 原型编程简介
原型编程本来是JS的最初的编程范式，核心就在于对象间建立委托关系，形成一个有序的线性关系。

lodash虽然主要是以函数式编程为特点，但避免不了地也要使用JS原本的原型链编程思想。

## 原型链
既然说到原型就不得不说到原型链，这是组织对象的一种方法，不同于继承的派生对象(即is关系)，原型链上的对象是平等的独立个体

//Todo:创建对象的方式

## 委托
很多lodash上的方法其实是委托JS原生API来做

```
var arrayProto = Array.prototype,
    objectProto = Object.prototype;

var hasOwnProperty = objectProto.hasOwnProperty;
var nativeObjectToString = objectProto.toString;
var propertyIsEnumerable = objectProto.propertyIsEnumerable;

// 批量委托
baseEach(['pop', 'join', 'replace', 'reverse', 'split', 'push', 'shift', 'sort', 'splice', 'unshift'], function(methodName) {
	var func = (/^(?:replace|split)$/.test(methodName) ? String.prototype : arrayProto)[methodName],
	  chainName = /^(?:push|sort|unshift)$/.test(methodName) ? 'tap' : 'thru',
	  retUnwrapped = /^(?:pop|join|replace|shift)$/.test(methodName);

	lodash.prototype[methodName] = function() {
	  var args = arguments;
	  if (retUnwrapped && !this.__chain__) {
	    var value = this.value();
	    return func.apply(isArray(value) ? value : [], args);
	  }
	  return this[chainName](function(value) {
	    return func.apply(isArray(value) ? value : [], args);
	  });
	};
});
```
