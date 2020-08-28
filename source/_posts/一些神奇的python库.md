---
title: 一些神奇的python库
date: 2020-08-10 20:28:37
categories:
- 花里胡哨的东西
tags:
- 科研
- python
- sympy
- line_profiler
- memory_profiler
- pdb
- pipreqs
- pigar
- dis
---

>主要是记录一些自己觉得科研或者项目可能用到的，但是又似乎不是那么大众的库，会随时更新。这里每一个库都会举一个小例子。因为记性不好，记不住api，所以也算是之后要用的话，一个简单的小笔记吧。

<!--more-->

## sympy 符号计算，可以输出为latex公式

> sympy 可以做一些符号计算，如求解线性方程，求导，求积分等。更方便的是他提供公式的简化，以及公式的latex输出，而且也可以用作简单的制图。如果论文中有繁琐的公式求导的部分，可以直接用这玩意来写...然后扔进latex里。当然，如果你是一个深度学习新手，面对复杂的公式，也可以通过这种方式来验证自己的高数水平

- toy example

  ```python
  >>>from sympy import *
  >>>
  >>>x, y, z = symbols("x y z")
  >>>expr = x**2 + y**2 + z**2
  >>>result = diff(expr, x).subs(x,2)
  
  >>>result
  >>> 2*x
  ```


- API
  - `print_latex`:输出为latex公式
  - `simplify`:化简公式

## line_profiler 

>line_profiler 可以逐行查看运行时间。当你的代码出现性能瓶颈的时候，你就可以清晰的定位到，具体是哪个函数的哪一行慢，以便更好地做优化。当然它有一些小缺点，就是需要引入函数装饰器，但是不需要import这个库，所以其无法通过直接运行.py文件来看(会报错)，必须通过指定的命令来运行。另外在一些异步的程序中，可能会有一些时间错位的问题，这个似乎是无法避免的，除非把程序改为同步。最典型的例子就是pytorch的GPU与CPU的切换，这是一个异步过程。如果需要准确测速，需要将其设置为阻塞式的同步过程。

- toy example

```python
# 不用import line_profiler

@profile
def test_time():
    pass
```

​	之后

```shell
kernprof -l -v main.py  # 分析
```

​	分析后会展示出结果，同时会生成 `main.py.lprof`的文件供之后随时查看，如果要运行这个文件:

```shell
python -m line_profiler main.py.lprof
```

## memory_profiler

>与line_profiler 功能类似，可以逐行查看内存的占用，优点是不同于line_profiler，其虽然也要加函数装饰器，但是可以通过import 这个库，然后直接运行.py文件来查看

- toy example

```python
import memory_profiler

@profile(precision=4,stream=open('memory_profiler.log','w+'))
def test_memory():
    pass
```

## pdb

>一行set_trace()即可实现断点添加的功能，适合偶尔使用，虽然不如很多IDE提供的debug功能丰富，但是日常快速使用足以。进入断点后就类似ipython了，是一个可交互的shell，随便打印什么看什么都行，还有一些命令可以帮助我们更好的debug。

- toy example

```python
import pdb

...
result = foo()
pdb.set_trace()
...
```

- 常用的命令：
  - `n`: 运行断点下一条语句
  - `c`: 运行至下一个断点或结束
  - `s`: 运行断点下一条语句，如果是函数，则进入函数
  - `q`: 退出debug

##pipreqs，pigar

>用来做项目打包生成requirements.txt的时候要用到的库，没啥特殊的地方。pipreqs生成的比较简洁，但是要注意编码问题，可能会报gbk的错误，所以一般要显式的使用utf-8，而pigar可以具体到哪一行使用了这个库。看情况选择使用吧。

- `pipreqs`的toy example

```shell
cd target_path  # 进入需要打包的路径
pipreqs ./ --encoding=utf-8
```

- `pigar`的toy example

```shell
pigir  # 简单明了，生成当前所在目录的依赖，然后到requirements.txt里

# pigar -p requirements_path -P target_path
pigar -p ../dev-requirements.txt -P ../
```

# dis

>非常的神奇的库，可以直接将代码转化为汇编。之所以列出这个库，仅仅是因为我觉得它神奇，目前我还不知道要怎么用它...

- toy example

```python
>>>import dis
>>>
>>> def add(a, b = 0):
...     return a + b
... 
>>> dis.dis(add)
  2           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 RETURN_VALUE
>>> 
```

