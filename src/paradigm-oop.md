# lodash中的面向对象编程思想

## 面向对象简介
面向对象是开发领域里一个很经典的编程思想，Java/C++/Typescript等语言都是支持面向对象编程的。但是由于Javascript本身没有类，不过有对象，一些习惯于OOP的程序员自然把这种风格带到了前端开发中，lodash也吸收了部分有利的思想。

## 抽象
抽象是以简化的形式来看待复杂操作的能力。

简单来说，lodash就是一个对数组/对象/集合等进行各种操作的一个抽象实体。

lodash提供一组明显相关的子程序，每个子程序都在朝着一致的目标而工作，这种抽象使得lodash具有强大的内聚性。比如Array系列的子程序都是为了解决数组中某个具体的问题而编写的，Collection/Function/Object等系列同样如此。

## 封装
封装是可以强制阻止使用者看到细节。

lodash中的所有子程序都封装的很好。这对使用api者非常爽，因为他不需要关注lodash的内部实现细节，当需要解决一个问题的时候，比如你需要求和，只需要做两件事情：
1. 引入lodash库
2. 使用sum函数
```
_.sum([4, 2, 8, 6]);
// => 20
```

如果有兴趣去看看这些子程序的细节，我们会发现里面存在着复杂的关系和结构（就像我们这个系列的解读一样）。就拿sum为例 **来自于full版本**
```
function sum(array) {
  return (array && array.length) ? baseSum(array, identity) : 0;
}

function baseSum(array, iteratee) {
	var result,
	  index = -1,
	  length = array.length;

	while (++index < length) {
	  var current = iteratee(array[index]);
	  if (current !== undefined) {
	    result = result === undefined ? current : (result + current);
	  }
	}
	return result;
}

function iteratee(){
	//省略
}
```

另外lodash也尽可能的限制了子程序中成员的可访问性，没有随便公开暴露成员数据，一个独立的子程序不会跑到另外一个子程序的内部使用它的内部属性或者调用它的方法。

这对阅读代码也是很有帮助的，可以把精力聚焦于一处，等一个地方弄清楚了再去下一个地方，比如sum->baseSum->iteratee，而不是随意在各个子程序间跳来跳去。

## 继承
继承实现了类的特化，非常适用于“是一个”的情况。

JS里不支持类，所以使用原型链的委托来模拟类之间的继承，关于原型编程请看2.3节。

这里面所有通过LodashWrapper包装的对象都是一个lodash对象，拥有lodash的方法。

继承可以让我们更好地管理复杂度，在面对几千乃至上万行源码时，可以让我们的脑海里快速构建出结构。