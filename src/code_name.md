# lodash代码命名

## 命名风格
命名这一块看似不起眼，但是命名的好坏往往决定代码的质量。一个好的命名是可读的、易记的和恰如其分的，可以给阅读代码的人提供大量信息。

整体上lodash是使用驼峰式命名风格，比如argsTag。常量命名使用全大写+下划线的格式，比如FUNC_ERROR_TEXT。构造函数首字母大写，比如LodashWrapper。这些都是和JS主流的命名风格一致的。

另外，为了区分代码块功能，命名的时候要加一些限定词，比如内部函数基本是base开头的，在功能类似的子程序和变量常量之间也会使用对仗词来方便阅读，比如min和max。

## 常量命名
新版的lodash是使用const来声明常量的，可是build后的版本需要兼容老浏览器只能用var声明，不过还好，可以通过命令来标识常量。

lodash中的常量不多，比如FLAG(COMPARE_PARTIAL_FLAG、COMPARE_UNORDERED_FLAG)系列辅助子程序选择性执行逻辑，FUNC_ERROR_TEXT辅助抛出错误，VERSION辅助记录库版本，
INFINITY和MAX_SAFE_INTEGER来防止处理数字的时候出错。

## 变量命名
基本上是驼峰式命名，主要由形容词、名词、动词组成。

* 循环下标：lodash中涉及的循环不用for用while，其中循环下标只用index命名，含义其实并不是很清晰，不过边界都是length系列。
* 状态变量：具有代表性的有Tag系列(arrayTag、funcTag等)，它把实际意义找出来作为名称。另外还有bitmask等
* 临时变量：没有见到temp这类通用且意义不明的变量，一般都是以这个变量真正的含义来命名的，比如assignValue函数中的objValue
* 布尔变量：一眼就能看出来是布尔类型，只有真和假，比如fromRight

## 函数命名相关
lodash的主体就是各种函数，从它的函数命名我们就看出统一的代码布局

* 普通函数命名：基本结构为前缀+语义命名，比如baseMap
* 构造函数命名:首字母大写，core版本只有LodashWrapper，full版本还有LazyWrapper等

## 缩写
一些常用的名字可以采用缩写，比如function缩写成func，arguments缩写成args。