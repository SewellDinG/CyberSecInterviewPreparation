## 浅拷贝？深拷贝？

- **直接赋值：**其实就是对象的引用（别名）。
- **浅拷贝(copy)：**拷贝父对象，不会拷贝对象的内部的子对象。
- **深拷贝(deepcopy)：** copy 模块的 deepcopy 方法，完全拷贝了父对象及其子对象。

1、**b = a:** 赋值引用，a 和 b 都指向同一个对象。

![img](https://www.runoob.com/wp-content/uploads/2017/03/1489720931-7116-4AQC6.png)

**2、b = a.copy():** 浅拷贝, a 和 b 是一个独立的对象，但他们的子对象还是指向统一对象（是引用）。

![img](https://www.runoob.com/wp-content/uploads/2017/03/1489720930-6827-Vtk4m.png)

**3、b = copy.deepcopy(a):** 深度拷贝, a 和 b 完全拷贝了父对象及其子对象，两者是完全独立的。

![img](https://www.runoob.com/wp-content/uploads/2017/03/1489720930-5882-BO4qO.png)