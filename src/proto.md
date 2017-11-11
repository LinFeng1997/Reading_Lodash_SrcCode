# lodash中的原型编程思想

## 原型编程简介
原型编程本来是JS的最初的编程范式，核心就在于对象间建立委托关系，形成一个有序的线性关系。

lodash虽然主要是以函数式编程为特点，但避免不了地也要使用JS原本的原型链编程思想。

## 原型链
既然说到原型就不得不说到原型链，这是组织对象的一种方法，不同于继承的派生对象(即is关系)，原型链上的对象是平等的独立个体。

根据这一点，实际上JS的对象都是靠原型链关联而不是类的派生来实现对象的，因为new其实是语法糖。
ES5提供了一个语法糖：Object.create，它的简单polyfill如下：
```
if (typeof Object.create !== "function") {
    Object.create = function (proto, propertiesObject) {
        var temp = new Object();
        temp.__proto__ = proto;
        if(typeof propertiesObject ==="object")
            Object.defineProperties(temp,propertiesObject);
        return temp;
    };
}

```
可以看出它实际上内部是创建了一个临时对象，然后把这个临时对象的原型链关联到某个原型上去。所以lodash中的baseCreate也使用了Object.create来创建一个对象。
```
var objectCreate = Object.create

var baseCreate = (function() {
function object() {}
	return function(proto) {
	  if (!isObject(proto)) {
	    return {};
	  }
	  if (objectCreate) {
	    return objectCreate(proto);
	  }
	  // 利用object这个函数，临时设置对象原型
	  object.prototype = proto;
	  // 创建一个对象，等价于result.__proto__ === prototype
	  var result = new object;
	  // 防止内存泄漏，因为闭包的原因，object常驻内存
	  object.prototype = undefined;
	  return result;
	};
}());
```
可以看出如果当前环境不支持Object.create的话lodash也会polyfill一个

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
	//检测函数是否属于String
	var func = (/^(?:replace|split)$/.test(methodName) ? String.prototype : arrayProto)[methodName],
	//检测函数是否可以链式调用
	chainName = /^(?:push|sort|unshift)$/.test(methodName) ? 'tap' : 'thru',
	//检测函数是否属于lodash未包装的
	retUnwrapped = /^(?:pop|join|replace|shift)$/.test(methodName);

	lodash.prototype[methodName] = function() {
	  var args = arguments;
	  //未包装的方法
	  if (retUnwrapped && !this.__chain__) {
	    var value = this.value();
	    return func.apply(isArray(value) ? value : [], args);
	  }
	  //支持链式调用的方法
	  return this[chainName](function(value) {
	    return func.apply(isArray(value) ? value : [], args);
	  });
	};
});
```
我们可以看出lodash主要是把它的一些方法委托给Object和Array的原型了。其中有一个一个委托的，也有批量委托的。
//Todo:具体分析批量挂载