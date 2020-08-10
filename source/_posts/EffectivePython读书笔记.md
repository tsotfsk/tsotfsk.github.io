---
title: EffectivePython读书笔记
date: 2020-03-07 15:43:54
tags:
- python
- effective
---

>对python的高级编程还了解不多，所以来取取经。这本书看起来挺快的，大概花了我一天的时间吧。这里整理下笔记，以便以后能用到。这里略过了一些比如线程和协程之类的使用。

<!--more-->

### 2.遵循PEP8风格

这里我只列几个我不曾注意到的。参考[PEP 8 中文](https://www.cnblogs.com/bymo/p/9567140.html#_label2_0)

- import 不要用相对位置。如果导入系统没有正确的配置（比如包里的一个目录在sys.path里的路径后），使用绝对路径会更加可读并且性能更好（至少能提供更好的错误信息）:

- 二元运算符换行放行首

- 算术表达式问题。总是在二元运算符两边加一个空格：赋值（=），增量赋值（+=，-=），比较（==,<,>,!=,<>,<=,>=,in,not,in,is,is not），布尔（and, or, not）。

  ```python
  i = i + 1
  submitted += 1
  x = x*2 - 1
  hypot2 = x*x + y*y
  c = (a+b) * (a-b)
  ```

- 文档字符单行的话三引号不换行，两行及以上第二个应当换行

  ```python
  """Return a foobang
  
  Optional plotz says to frobnicate the bizbaz first.
  """
  ```

- from M import * 导入的模块应该使用all机制去防止内部的接口对外暴露，或者使用在全局变量前加下划线的方式

- 从Exception继承异常，而不是BaseException。

- 使用 ”.startswith() 和 ”.endswith() 代替通过字符串切割的方法去检查前缀和后缀。

  ```python
  推荐: if foo.startswith('bar'):
  糟糕: if foo[:3] == 'bar':
  ```

- 对象类型的比较应该用isinstance()而不是直接比较type。

  ```python
  正确: if isinstance(obj, int):
  糟糕: if type(obj) is type(1):
  ```

### 3.了解bytes、str和unicode的区别

![](EffectivePython读书笔记\编码.png)

程序核心部分应使用Unicode，编码与解码应放在外围部分。

### 9.用生成器表达式来改写数据量较大的列表推导

优点就是生成器不用一次载入，格式是(...)

### 13.try/except/else/finally

try代码不应过多，只做主要判断，应写在else中。

### 14.尽量用异常来表示特殊情况，而不要返回None

因为None可以通过if条件语句。

### 17.在参数上迭代时，要小心

因为迭代器只能迭代一次。

### 20.用None和文档字符串来描述具有动态默认值的参数

也就是参数默认值不要给一个具体的，比如函数，或者空字典，因为默认参数在载入的时候只初始化一遍。如果return的恰好也是默认参数会出问题。

### 21.用只能以关键字形式指定的参数来确保代码明晰

尽量不要采用位置赋值，而要采用关键字赋值，易读性强也更容易维护。

为了避免关键字参数被按位置参数赋值，可以使用**kwargs来提取参数，但是可读性差一点，要求增加函数文档，python3的话可以通过在位置参数和关键字参数中间增加一个` * `

```python
def foo(number,dvisor, *, ignore=False)
```

### 25.用super初始化父类

python 3支持不带参数的super，所以没必要加参数。

```python
super().__init__(...)
```

### 26.只在私用mix-in组件的时候进行多重继承

这个了解一下吧，mix-in提供的就是一系列通用功能。

### 27.多用public属性，少用private属性

这个挺重要的，python的private属性实际上是假的，比如类Server含有一个`self.__post`的私有属性，虽然我无法通过函数去返回这个，但是我可以直接调用` _Server__post `来获得这个属性，那何必自欺欺人呢，大家都是成年人了。真需要的话不如加一个_，就当是提醒别人注意吧。

### 29. 用纯属性来取代get和set方法

这不是python风格的...我也挺想吐槽了，根本没必要。

### 32. 用`__getarr__`、`__getattribute__` 和`__setattr__`实现按需生成的属性

这些就是在获取类中不存在的值的时候定义的操作，比如data中没有foo这个变量，那么当我调用data.foo的时候会去调用`__getarr__` 然后我在里面使用 setattr(self, name, value)就可以将foo这个变量添加到类的`__dict__` 中。

### 41.考虑使用concurrent.futures 来实现真正的平行计算

这里大概讲的多进程的东西，还提到了不要使用multiprocessing，说是有一些复杂特性，要避开？但python的多进程本来就挺麻烦的...

### 42. 用functools.wraps定义函数修饰器

之所以要用这个，是因为修饰器会更改函数的名字，这可能影响调试器等的效果。

```python
def trace(func):
	@wraps(func)
	def wrapper(*args, **kwargs):
		# ...
	return wrapper
	
@trace
def fibonacci(n)
	# ...
```

### 43.考虑以contextlib和with语句来改写可复用的try/finally代码

这个可以定义一宗情景管理器，比如以下代码

```python
@contextmanager
def debug_logging(level):
	logger = logging.getLogger()
	old_level = logger.getEffectiveLevel()
	logger . setLevel(level)
	try:
		yield
	finally:
		logger.setLevel(old_level)
with debug_logging(logging.DEBUG):
    # ...
```

这段代码定义了一个可以暂时提升logging的level的方法，在执行到yield的时候会进入with下面的语句执行，执行完毕后会返回，然后执行finally中的语句。

### 44.用copyreg实现可靠的pickle操作

pickle是不可靠的序列化(json是可靠的序列化)，所以其只能建立在相互信任的程序之间使用。它可以保存一些类啊什么的，使用copyreg是在类的定义或者什么改变的条件下，依然可以load的解决方法。

### 46.使用内置算法与数据结构

这个用到再说吧，主要是有一些队列啊，有序字典(python 3.6+后字典都是有序的)，还有堆栈啊什么的，以后做leetcode可以用一用。

### 51.为自编的模块定义根异常，以便将调用者与API相隔离

这个没啥，注意到基类比如叫Error继承自Exception就可以了。

### 53.用适当的方式打破循环依赖关系

我觉得还是不要写这种代码好了，如果真写了，再回来看吧

### 56.用unittest来测试全部代码

保留意见

### 57.用pdb来实现交互调试

这个要学一下，一个pdb.set_trace()就可以开始调试了，很方便啊。



