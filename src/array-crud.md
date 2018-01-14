## 数组系列之增删改查方法

### 查

#### first(head)和take
得头，获得array中第一个元素：
```
_.head([1, 2, 3]);
// => 1
```
实现：
```
function head(array) {
  return (array != null && array.length)
    ? array[0]
    : undefined
}
```
实现大概可以猜到的，核心就是array[0]，但是对于数组是否存在的判断使得这个函数很健壮。

take是变异版本，可以从头拿多个元素：
```
_.take([1, 2, 3]);
// => [1]
 
_.take([1, 2, 3], 2);
// => [1, 2]
```
实现：
```
function take(array, n=1) {
  if (!(array != null && array.length)) {
    return []
  }
  return slice(array, 0, n < 0 ? 0 : n)
}
```
它和first的区别就在于n < 0 ? 0 : n这个表达式。由此衍生的还有takeRight、takeRightWhile、takeWhile三个函数，它们的分别是取尾、高级取尾、高级取头。大家可以留作作业自己探究实现方式。

### 增
暂无

### 删

#### initial
去尾，获得array中除最后1个元素以外的元素，参数为array：
```
_.initial([1, 2, 3]);
// => [1, 2]
```
来看实现：
```
function initial(array) {
  const length = array == null ? 0 : array.length
  return length ? slice(array, 0, -1) : []
}
```
其实就是借用了slice函数，需要注意的点是slice的最后一个参数传的是-1，等同于length-1。而且这里对数组是否可去尾用length做了判断，健壮性很强。

#### drop和tail
去头，和initial不同的是它有第二个参数n,其实用for循环组合不传第二个参数n也能起到同样的效果：
```
_.drop([1, 2, 3]);
// => [2, 3]
 
_.drop([1, 2, 3], 2);
// => [3]
```
来看实现：
```
function drop(array, n=1) {
  const length = array == null ? 0 : array.length
  return length
    ? slice(array, n < 0 ? 0 : n, length)
    : []
}
```
我们发现和initial的实现就差在n < 0 ? 0 : n这一句三元表达式上。

tail是不带第二个参数的纯去头版本：
```
_.tail([1, 2, 3]);
// => [2, 3]
```

#### pull

#### remove

#### without


### 改

#### fill