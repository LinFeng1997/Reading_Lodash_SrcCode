# lodash中的函数式编程思想

## 函数式编程简介
这是lodash最突出的编程范式，为什么这么说呢？首先让我们看看函数式编程有什么特点：函数是一等公民，即可以当参数传递也可以当结果返回，它和一个对象是等价的，只不过可以执行。

而函数式编程思想在lodash里具体的体现主要有两点：使得集合可迭代和可链式调用无副作用的函数。

## 迭代者iteratee
lodash**(5.0版本以前)**能够让迭代函数同时拥有遍历数组和对象的能力，这一切来源于一个函数：baseIteratee	
```
function baseIteratee(func) {
	if (typeof func == 'function') {
	  return func;
	}
	if (func == null) {
	  return identity;
	}
	return (typeof func == 'object' ? baseMatches : baseProperty)(func);
}
//返回一个可以匹配对象的函数
function baseMatches(source) {
    var props = nativeKeys(source);
    return function(object) {
      var length = props.length;
      if (object == null) {
        return !length;
      }
      object = Object(object);
      while (length--) {
        var key = props[length];
        if (!(key in object &&
            baseIsEqual(source[key], object[key], COMPARE_PARTIAL_FLAG | COMPARE_UNORDERED_FLAG)
          )) {
          return false;
        }
      }
      return true;
    };
}
//根据键返回一个可以得到对象值的函数
function baseProperty(key) {
	return function(object) {
	  return object == null ? undefined : object[key];
	};
}
}
```
这个函数的目的只有一个：包装一个func成为一个迭代者iteratee，如果本身就是函数则返回自身，如果是对象则返回匹配的，如果是字符串则返回对象对应的值。

它所用到的两个工具函数-baseMatches和baseProperty分别是针对func是对象和字符串的形式，也就是说，当func是对象的时候，迭代者解释为匹配对象的函数。

当func为字符串的时候，迭代者就解释为可以根据键返回一个可以得到对象值的函数。

让我们看看它具体是怎么用的：以\_.filter为例，baseFilter的细节我们先不必关注
```
function filter(collection, predicate) {
	return baseFilter(collection, baseIteratee(predicate));
}

var users = [
  { 'user': 'barney', 'age': 36, 'active': true },
  { 'user': 'fred',   'age': 40, 'active': false }
];

// func是函数
_.filter(users, function(o) { return !o.active; });
// => objects for ['fred']

// func是对象
_.filter(users, { 'age': 36, 'active': true });
// => objects for ['barney']

// func是数组(还是对象)
_.filter(users, ['active', false]);
// => objects for ['fred']

// func是字符串
_.filter(users, 'active');
// => objects for ['barney']
```

every/some/forEach/reduce/sortBy等函数同样使用这种方法包装迭代者。

## 链式调用
函数式编程另一大特点：函数都是无副作用的

关键函数：chain
```
function chain(value) {
	var result = lodash(value);
	result.__chain__ = true;
	return result;
}
```
它把一个普通的值包装成lodash对象，从而拥有lodash的各种方法，自然就可以链式调用了，这和jq等库的链式调用有异曲同工之妙。

这个包装函数在函数式编程里称为Monad。ES6中的Promise其实也是一种Monad。
