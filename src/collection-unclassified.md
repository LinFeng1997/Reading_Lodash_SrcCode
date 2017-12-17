## 集合未分类函数

### flat系列
有flatMap、flatMapDeep、flatMapDepth这三个函数，都是full版本才有的。

其实它们就是flat和map的结合体，至于flat在数组章再分析。

### flatMap
看看用例：
```
function duplicate(n) {
  return [n, n];
}
 
_.flatMap([1, 2], duplicate);
// => [1, 1, 2, 2]
```
原先应该是[[1,2],[1,2]],经过flatten之后就成了一维数组。
flatMap实现：
```
function flatMap(collection, iteratee) {
  return baseFlatten(map(collection, iteratee), 1);
}
```
可以看出，集合是先经过了map，然后flatten了1层。

#### flatMapDeep和flatMapDepth
直接看实现：
```
function flatMapDeep(collection, iteratee) {
  return baseFlatten(map(collection, iteratee), INFINITY);
}

function flatMapDepth(collection, iteratee, depth) {
  depth = depth === undefined ? 1 : toInteger(depth);
  return baseFlatten(map(collection, iteratee), depth);
}
```
和flatMap唯一的区别就是一个flatten了无穷层，一个是flatten了指定层

### 随机系列

#### sample
它也是full版本的函数，看看用例，它的功能是从集合中随机取一个元素：
```
_.sample([1, 2, 3, 4]);
// => 2
```
实现：
```
function sample(collection) {
  var func = isArray(collection) ? arraySample : baseSample;
  return func(collection);
}
```
我们发现关键函数是arraySample和baseSample：
```
function arraySample(array) {
  var length = array.length;
  return length ? array[baseRandom(0, length - 1)] : undefined;
}

function baseSample(collection) {
  return arraySample(values(collection));
}
```
原来是arraySample生成了一个随机索引，然后去数组里取，而baseSample是先把对象转成数组再借用arraySample实现的。

还可以看看baseRandom：
```
nativeFloor = Math.floor
nativeRandom = Math.random

function baseRandom(lower, upper) {
  return lower + nativeFloor(nativeRandom() * (upper - lower + 1));
}
```
一个很简单实用的生成区间内随机数的函数。

#### sampleSize
它同样是full版本的函数，先看用例，发现它比sample高级一些，因为可以指定结果数组的元素个数：
```
_.sampleSize([1, 2, 3], 2);
// => [3, 1]
_.sampleSize([1, 2, 3], 4);
// => [2, 3, 1]
```
实现：
```
function sampleSize(collection, n, guard) {
  if ((guard ? isIterateeCall(collection, n, guard) : n === undefined)) {
    n = 1;
  } else {
    n = toInteger(n);
  }
  var func = isArray(collection) ? arraySampleSize : baseSampleSize;
  return func(collection, n);
}
```
其实关键也是arraySampleSize和baseSampleSize：
```
function arraySampleSize(array, n) {
  return shuffleSelf(copyArray(array), baseClamp(n, 0, array.length));
}

function baseSampleSize(collection, n) {
  var array = values(collection);
  return shuffleSelf(array, baseClamp(n, 0, array.length));
}
```
我们发现这里用到了shuffleSelf和baseClamp 
//Todo:继续分析shuffleSelf和baseClamp
#### shuffle
洗牌算法，看用例，其实就是个打乱集合的函数：
```
_.shuffle([1, 2, 3, 4]);
// => [4, 1, 3, 2]
```
直接看实现:
```
function shuffle(collection) {
  var func = isArray(collection) ? arrayShuffle : baseShuffle;
  return func(collection);
}
```
关键点在于arrayShuffle和baseShuffle：
```
function arrayShuffle(array) {
  return shuffleSelf(copyArray(array));
}
function baseShuffle(collection) {
  return shuffleSelf(values(collection));
}
```
很明显，最后的归宿都是shuffleSelf，只不过对象会被转成一个值数组
```
function shuffleSelf(array, size) {
  var index = -1,
    length = array.length,
    lastIndex = length - 1;

  size = size === undefined ? length : size;
  while (++index < size) {
    var rand = baseRandom(index, lastIndex),
      value = array[rand];

    array[rand] = array[index];
    array[index] = value;
  }
  array.length = size;
  return array;
}
```
这个函数很简单，就是生成一个随机的索引，然后和数组原本索引对应的值顺序交换。

### 其他

#### invokeMap
看用例，它是个invoke和map的结合函数：
```
_.invokeMap([[5, 1, 7], [3, 2, 1]], 'sort');
// => [[1, 5, 7], [1, 2, 3]]
```
具体过程大家应该可以猜到，那就是遍历集合然后调用相关函数，实现的话留给大家自己看了~

#### partition
看用例，它是个一分集合为二的函数
```
var users = [
  { 'user': 'barney',  'age': 36, 'active': false },
  { 'user': 'fred',    'age': 40, 'active': true },
  { 'user': 'pebbles', 'age': 1,  'active': false }
];
 
_.partition(users, function(o) { return o.active; });
// => objects for [['fred'], ['barney', 'pebbles']]
```
一个集合是符合条件的，另一个是不符合条件的，不知道大家有没有想起filter和reject？
我们来看看实现：
```
var partition = createAggregator(function(result, value, key) {
  result[key ? 0 : 1].push(value);
}, function() {
  return [
    [],
    []
  ];
});
```
可以发现它和sql系列的函数用了同一个辅助函数：createAggregator，而这个函数是负责将元素分类的，自然就将一个集合一分为二了。

#### size
看用例，它是个查看集合元素个数的函数：
```
_.size({ 'a': 1, 'b': 2 });
// => 2
```
实现：
```
function size(collection) {
	if (collection == null) {
	  return 0;
	}
	collection = isArrayLike(collection) ? collection : nativeKeys(collection);
	return collection.length;
}
```
其实就是数组直接访问length属性，对象把值都转成数组再访问length属性