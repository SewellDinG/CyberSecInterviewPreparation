## Python3 与 Python2 的区别？

1、print

2、`/，//`除

```
py3
In [1]: 10/3
Out[1]: 3.3333333333333335
In [2]: 10//3
Out[2]: 3
py2
>>> 10/3
3
>>> 10//3
3
```

3、编码。Python2中默认的编码格式是 ASCII 格式，程序文件中如果包含中文字符（包括注释部分）需要在文件开头加上 `# -*- coding: UTF-8 -*- `或者 #coding=utf-8 就行了；Python3默认支持Unicode。

4、格式化字符串。format和`f""`。

5、xrange -> range