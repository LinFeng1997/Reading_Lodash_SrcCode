# lodash代码注释

lodash的注释非常清晰明了，以至于代码中一大半其实都是注释，这份占了这么多源码空间的注释对于我们阅读源码是非常有帮助的，可以这样说，靠着注释我们可以写出我们自己的lodash库。

## 函数外注释
这种注释一般是说明性的。

让我们先来看一段简单的函数外注释：
```
/**
   * The base implementation of `_.property` without support for deep paths.
   *
   * @private
   * @param {string} key The key of the property to get.
   * @returns {Function} Returns the new accessor function.
   */
```
这段代码由四个部分组成
	1. 编写目的，即为了解决什么问题：它是不支持深度路径的_.property的实现基础函数
	2. 访问权限：是私有的，即内部不暴露的函数
	3. 传入参数：是一个sting类型的key，它的语义是需要得到值的键
	4. 返回结果：是一个Function类型的函数，它的语义是一个新的访问器函数

相信大家通过这个注释不看源码都能实现一个差不多的函数了吧？

好，我们再来看个复杂的
```
/**
* Adds all own enumerable string keyed function properties of a source
* object to the destination object. If `object` is a function, then methods
* are added to its prototype as well.
*
* **Note:** Use `_.runInContext` to create a pristine `lodash` function to
* avoid conflicts caused by modifying the original.
*
* @static
* @since 0.1.0
* @memberOf _
* @category Util
* @param {Function|Object} [object=lodash] The destination object.
* @param {Object} source The object of functions to add.
* @param {Object} [options={}] The options object.
* @param {boolean} [options.chain=true] Specify whether mixins are chainable.
* @returns {Function|Object} Returns `object`.
* @example
*
* function vowels(string) {
*   return _.filter(string, function(v) {
*     return /[aeiou]/i.test(v);
*   });
* }
*
* _.mixin({ 'vowels': vowels });
* _.vowels('fred');
* // => ['e']
*
* _('fred').vowels().value();
* // => ['e']
*
* _.mixin({ 'vowels': vowels }, { 'chain': false });
* _('fred').vowels();
* // => ['e']
*/
```
除了之前四个部分外，还新增了：提示、版本号、所属对象、所属类别、测试用例，我们自己写代码的时候也要尽量参考lodash的注释风格，最好是编写具体的代码之前就写好一份类似的伪代码，然后编写完具体的代码后根据需要删除部分伪代码形成最终的注释。

## 函数内注释 
这种注释一般是解释性的，因为读者可能会困惑这里为什么这样写。

比如baseFlatten中的一段：
```
// Recursively flatten arrays (susceptible to call stack limits).
baseFlatten(value, depth - 1, predicate, isStrict, result);
```
特意指出了这里会递归扁平化数组，并且可能会调用栈时受限。

还有createCtor里的一段：
```
// Use a `switch` statement to work with class constructors. See
// http://ecma-international.org/ecma-262/7.0/#sec-ecmascript-function-objects-call-thisargument-argumentslist
// for more details.
```
这里特意指出了相关代码是和某个es规范有关的。