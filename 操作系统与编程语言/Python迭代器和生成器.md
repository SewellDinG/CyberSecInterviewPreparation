Ref：[https://www.runoob.com/python3/python3-iterator-generator.html](https://www.runoob.com/python3/python3-iterator-generator.html)

## Python迭代器

可迭代对象（Iterable）：简单的来理解就是可以使用 for 来循环遍历的对象，比如常见的 list、set、tuple、dict和string。

迭代器（Iterator）：内部实现了`iter()`方法和`next()`方法的对象就是迭代器，调用`next()`方法会返回下一个值（直到没有数据时抛出`StopIteration`错误）。

## Python生成器

生成器（Generator）：就是`带有 yield 的函数`。跟普通函数不同的是，生成器是一个返回迭代器的函数，只能用于迭代操作，更简单点理解生成器就是一个迭代器。

在调用生成器运行的过程中，每次遇到 yield 时函数会暂停并保存当前所有的运行信息，返回 yield 的值，并在下一次执行 next() 方法时从当前位置继续运行。

调用一个生成器函数，返回的是一个迭代器对象。

