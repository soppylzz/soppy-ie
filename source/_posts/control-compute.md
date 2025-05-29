---
title: Python笔记：控制与运算
date: 2024-10-14 18:45:50
tags:
  - python
  - tutorial
categories:
  - 笔记 
---

> 这里我只是简单介绍一下Python控制与运算的一些常用内容，详细内容请参考：[Python 3.13.0 文档](https://docs.python.org/zh-cn/3/) 。

## 运算

Python提供了多种运算符来支持丰富的表达式写法，这些算术运算中的**大部分**可以通过自定义类中的**魔术方法**来修改实现方式，Python运算符大致可以分为如下几类：

### 1. 算术运算符
Python支持如下算术运算：

| 常用算术运算符                  | 幂运算符                                   | 整除运算符                                |
| --------------------------- | ------------------------------------------ | ----------------------------------------- |
| `+` 、`-` 、`*` 、`/` 、`%` | <span style='color:cyan'>`**` </span> | <span style='color:cyan'>`//`</span> |

### 2. 赋值运算符
Python的赋值运算符有：`=` 、**`[operator]=`** 、<span style='color:cyan'>`:=`（海象运算符）</span> 。

**海象运算符** | `Walrus Operator` 是`Python 3.8`中引入的一种新的运算符，正式名称为 **赋值表达式** | `Assignment Expression` ，但它更广为人知的名称是海象运算符，因为它的符号`:=`看上去像是海象。这个运算符允许我们在表达式内部进行赋值。

```python
x = 0
if (x := 10) > 5:  
    print("x is greater than 5")
else:
    print("x is", x)
```

还需要注意的是，Python的`=`运算符的使用方法与`Golang`类似。Python允许我们用一个赋值运算符为多个变量赋值。

```python
def test_func():
    return 1,2

x=1
x, y=1, 2
x, y=test_func()
x, y=y, x
```
### 3. 身份运算符

Python的身份运算符有：`is`、`not is`。

```python
class Test:
    def __init__(self, value):
        self.value = value

        def __eq__(self, other):
            if other.value == self.value:
                return True
            else:
                return False

        def __hash__(self):
            return hash(self.value)

test1=Test(10)
test2=Test(10)

print(test1==test2)                         # True
print(test1.__hash__()==test2.__hash__())   # True

print(id(test1)==id(test2))                 # False
print(test1 is test2)                       # False
```

需要注意的是，`is`运算符**不对应**任何<u>魔术方法</u> ，它是直接比较内存地址的。

### 4. 解构运算符

`*`、`**`是Python中的解构运算符，其中`*`用于解构**数组**，`**`用于解构**字典**。

我们常见这样的函数头`def test_func(*args, **kwargs):`，这里的`*args`其实可以看做用解构后的数组`args`去接收<u>未指明形参的数据</u>，而`**kwargs`可以看作用解构后的字典`kwargs`去接受<u>指明了形参的数据</u>。

```python
args=[1,2,3]
kwargs={'attr1': 4, 'attr2': 5}

def test_func(*args, **kwargs):
    print(args)
    print(kwargs)

test_func(*args, **kwargs)
```

同`JavaScript`类似，Python的解构运算符不止用于接收函数参数，也可以用于数组与字典的扩展或融合</u>。需要注意的是，解构后的数据不再是对象，而是一串特殊的输出序列。解构数组的输出序列可看作`for i in arr`，解构字典的输出序列可以看作`for k, v in dic`。 

```python
arr = [1, 2, 3]
dic = {'attr1': 4, 'attr2': 5}

new_arr = [*arr,4,5,6]
new_dic = {**dic, 'add1':4, 'add2':5}

print(new_arr)
print(new_dic)

print(*new_arr)		# Equals: print(1,2,3,4,5,6)
print(**new_dic)	# Error
```

`*`除了可以解构数组外，还支持解构实现了`__iter__()`魔术方法的可遍历对象；而`**`则较为特殊，它并不支持解构实现了`__getitem__`、`__setitem__`、`__iter__`等魔术方法的类字典对象。

### 5. 其他运算符

| 类别           | 成员                                        |
| -------------- | ------------------------------------------- |
| **位运算符**   | `&`（与）、`\|`（或）、`^`（或）、`~`（非） |
| **比较运算符** | `==` 、`!==` 、`>` 、`>=` 、`<` 、`<=`      |
| **逻辑运算符** | `and` 、`or` 、`not`                        |
| **成员运算符** | `in` 、`not in`                             |

## 控制

### 1. 条件控制

Python的条件控制方式有两种：

* **`if-else` 语句**

    当只有两个分支时使用`if-else`：

    ```python
    n=10
    if n>10:
        pass
    else:
        pass
    ```

    当有多个分支时使用`if-elif-else`：

    ```python
    n=10
    if n>10:
        pass
    elif n<0:
        pass
    else:
        pass
    ```

    在其他语言中我们可以使用<span style='color:cyan'>**三目运算符**</span>来简化有关赋值的条件控制结构，而在Python中我们可以使用`if else`来实现相同的效果：

    ```python
    a=10
    b= a+5 if a>10 else a-5
    
    max=lambda a, b: a if a>b else b
    print(max(3,4))		# 4
    ```

* **`match-case` 语句**

    在`Python 3.10`及其以后版本，我们可以`match-case`简化 `if-elif-else` 语句段，类似于其他语言中的`switch-case`。

    `match-case`在多分支的实现上于其他方法有所不同：

    * `match-case`中一旦某个模式匹配成功，后续`case`将不再进行匹配，不需要像其他语言那样使用`break`推出匹配；
    * `match-case`中`_`用于匹配任何值，相当于其他语言的`default`；

    ```python
    n=10
    match n:
        case n<10:
            pass
        case n==10:
            pass		
        case n>=10:
            pass		# Wont enter this case
        case _:
            pass
    ```


### 2. 循环控制

Python的循环控制方式有两种：

*  **`while` 语句**

    `while`是循环控制的一种形式，需要注意的是Python中有`while-else`这种**独特**的语法。一般`while`循环如下：

    ```python
    acc=0
    while (acc:=acc+1) <= 5:
        print(acc, end=' ')
    
    # 1 2 3 4 5
    ```

    我们可以使用`break`跳出当前循环，使用`else`声明循环体正常结束后的任务。当我们使用`break`跳出循环体时，Python不会执行`else`之后的内容。这种`while-else`可以简化其他编程语言在使用`break`推出前更改`flag`值的行为。

    ```python
    acc = 0
    while (acc := acc + 1) <= 10:
        if acc == 5:
            break
            print(acc, end=' ')
        else:
            print('exit')
    
    # 1 2 3 4 
    ```


* **`for` 语句**

    `for`循环控制的另一种形式，需要注意的是Python中有`for-else`这种**独特**的语法。一般`for-else`循环如下：

    ```python
    for i in range(1, 5):
    	print(i, end=" ")
    else:
    	print('exit')
    
    # 1 2 3 4 exit
    ```

    如果要使得我们的类支持`for-in`遍历的话需要支持`__iter__`魔术方法。

    一般来说我们可以直接使用`for item in iterable:`循环遍历可遍历序列，我们也可以遍历`range(st, ed, foot)`生成的指定首尾和步长的可遍历序列。以下简单说明下`range`的使用：

    | 使用方式                                                     | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | **`range(ed)`**                                              | 当只有一个参数时，`range`默认`st=1`。需要注意的是Python并不支持带有默认值的参数写前面，故我们如果要自己手写一个`range`的话需要使用`*args`来接受参数 |
    | **`range(st, ed)`**                                          | 当只指定了起始于结束的参数时，`range`默认`foot=1`            |
    | **<span style='white-space: nowrap'>`range(st, ed, foot)`</span>** | `foot`可以为正数也可以为负数，但不能为0，注意`range`的退出条件 |

    ```python
    def print_range(range):
        print('[', end='')
        for i in range:
            print(i, end=', ')
        print(']')
    
    args=[2,10,2]
    for i in range(1,4):
        print_range(range(*args[:i]))
    ```

    `for`除了支持`break`，还支持使用`continue`来控制循环体。

    ```python
    for i in range(1, 10):
        if i==5:
            continue
        if i==8:
            break
        print(i, end=' ')
    
    # 1 2 3 4 6 7 
    ```

    当我们不需要使用每次循环的`i`时，我们可以使用`_`占位符来代替`i`。

    ```python
    for _ in range(10):
        print('Loop')
    ```


### 3. 异常控制

Python的异常都是从`BaseException`派生而来，`BaseException`的派生类可以参考：[内置异常 — Python 3.14.0a0 文档](https://docs.python.org/zh-cn/3.14/library/exceptions.html#BaseException)。 *<u>Python 的异常常随则版本更新而变化</u>* ，在Python中使用异常类时请参考对应Python版本的`BaseException`文档。以下列出来`BaseException`的第一层派生情况：

* <span style='color:cyan'>**`Exception`**</span>：所有内置的非系统退出类异常都派生自此类。 所有用户自定义异常也应当派生自此类；

* **`SystemExit`**：该异常由 [`sys.exit()`](https://docs.python.org/zh-cn/3.14/library/sys.html#sys.exit) 函数引发；
* **`KeyboardInterrupt`**：当用户按下 `Ctrl+C` 、`Del` 时被触发；
* **`BaseExceptionGroup`**： 用于成组处理异常（<span style='color:pink'>`python 3.11` 加入</span>）；

    下列异常是在有必要引发多个不相关联的异常时使用的。 它们是异常层级结构的一部分因此它们可以像所有其他异常一样通过[`except`](https://docs.python.org/zh-cn/3.14/reference/compound_stmts.html#except)来处理。 此外，它们还可被[`except*`](https://docs.python.org/zh-cn/3.14/reference/compound_stmts.html#except-star)所识别，此语法将基于所包含异常的类型来匹配其子分组。

    ```python
    try:
        raise ExceptionGroup("eg",[
            ValueError(1), 
            TypeError(2), 
            OSError(3), 
            OSError(4)
        ])
        except* TypeError as e:
            print(f'caught {type(e)} with nested {e.exceptions}')
    #   except* OSError as e:
    #    	print(f'caught {type(e)} with nested {e.exceptions}')
    ```

* **`GeneratorExit`**：用于生成器推出错误；

我们可以通过继承已有的异常类来创建我们的异常类。以下是编写异常类的最佳实践：
* **创建普通异常类**：继承`Exception`或者它的子类，重写父类方法，添加其他方法；
* **为模板创建异常类**：一个模块有可能抛出多种不同的异常时，一种通常的做法是为这个包建立一个基础异常类，然后基于这个基础类为不同的错误情况创建不同的子类；

Python的异常控制主要由两个板块构成：<u>处理异常</u>、<u>抛出异常</u>。

#### 🛠️ 异常处理

> `try-except`是Python异常处理的常用模板，此外`with`语句也与异常处理有关。

`try-except` 来捕获并处理异常，在 Python 在 `python 3.11` 之后引入了 `BaseExceptionGroup` ，为了处理异常集我们可以使用 `except*` 来捕获异常。由于支持 `except*` 与异常集的 Python 版本较新，在这里我只列出普通的 `try-except` 捕获异常的基本模板，如果需要在较新 Python 版本中使用异常集请查看相关文档：

```python
try:
    raise Exception('error')
except Exception as err:		# use err catch Exception ins
    pass				# run when error catched
else:
    pass				# run when error uncatched
finally:
    pass
```

`with` 语句用于包裹执行代码段，以便于管理资源，如文件操作、数据库连接、网络连接等。它确保了资源的正确使用和释放，即使在代码执行过程中发生了异常。我们使用 `with` 获取的类型实例实现了魔术方法 `__enter__` 与 `__exit__` 。`__enter__` 方法用于返回可供 `with` 代码段使用的实例对象，当我们在代码段中抛出了异常后， `__exit__` 实现了对该异常的处理代码，因此不需要我们在使用时进行处理。

```python
class ManagedResource:
    def __enter__(self):
        print("Enter the context")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Exit the context")
        if exc_type:
            print(f"Exception: {exc_type, exc_value}")

with ManagedResource() as resource:
    print("Inside 'with' statement")
    raise ValueError("Oops!")
```

#### ⚠️ 抛出异常

Python的`raise`语句类似于其他语言中的`throw`语句，用于抛出新建的异常。

```python
raise Exception('new error')
```

Python的`assert`语句用于断言某个条件是真的。如果条件为真，则程序继续执行；如果条件为假，则程序抛出`AssertionError`异常。

```python
try:
    assert 2+2==3, "AssertionError ins"
    # assert template
    # assert <bool expression>, <assert string>

except Exception as err:
    print(type(err))
    print(repr(err))

    # <class 'AssertionError'>
    # AssertionError('AssertionError ins')
```

需要注意的是，Python的断言常用于开发中进行调试。我们可以在使用Python进行解释执行的时候将全部`assert`语句进行移除，因此在代码中我们不应使用`assert`进行编写业务逻辑。

## 函数

### 1. 命名空间

命名空间是存储变量名称和值的映射表，而作用域是Python程序可以直接访问命名空间的正文区域。在Python文档中并没有 **作用域** | `scope` 相关的词条，作用域只是根据命名空间划分的正文片段。Python的命名空间有以下类型：

* **局部命名空间**：在<u>函数中定义的变量</u>将存储在对应函数的局部命名空间中，注意<span style='color:cyan'>局部命名空间可以嵌套</span>；
* **全局命名空间**：在<u>模块中定义的变量</u>将存储在对应模块的全局命名空间中；
    我们有`name1.py`文件如下：

    ```python
    name="test"
    ```

    我们在其它文件中调用`name1.py`中的变量或者函数时可以使用以下两种调用方式：

    ```python
    import name1
    from name1 import *
    
    print(name1.name)   # test
    print(name)         # test
    name='inside'
    print(name1.name)   # test
    print(name)			# inside
    ```

    需要注意的是第二条导入语句`from name1 import *`只是允许我们像使用本模块的全局命名空间中的变量那样使用`name1.py`中的变量，不是将`name`移动到本模块的全局命名空间中。这里我们使用`name='inside'`事实上是在本模块的全局命名空间中声明了该变量。
* **内置命名空间**：Python解释器提供的内置变量存储在内置命名空间中；
    想要了解Python的内置变量可以查询Python文档，或者使用 `dir()` 查询。

    ```python
    import builtins
    
    for key_name in dir(builtins):
     print(key_name, type(getattr(builtins, key_name)))
    ```


下面我们详细了解一下命名空间的变量使用规范：

* **引用变量**：当我们引用一个变量时，Python会按照由局部命名空间到内置命名空间的顺序搜索相应变量；

* **管理变量**：当我们在内部作用域想要<u>**修改、删除、声明**</u>外部作用域的变量时，需要使用`global`、`nonlocal`关键字声明对应变量；

    需要注意的是，在多层函数嵌套中`global`用于引用当前模块的<span style='color:cyan'>全局作用域</span>对应命名空间中的变量，`nonlocal`用于引用上一级<span style='color:cyan'>函数作用域</span>对应命名空间中的变量。

    当然，`global`与`nonlocal`关键字不只用于引用相应位置的变量，也可以用于声明相应位置的变量。 最后需要注意的是，`global`、`nonlocal`不能引用或者管理其他模块对应的全局命名空间中的变量。

    ```python
    x, y=1,1
    
    def test():
        global x
        global z
        global delete
        x+=1
        y=10
        z=100
        delete=1000
        del delete
    
        def closure():
            nonlocal y
            y+=1
            print(f"nonlocal y: {y}")
    
        return closure
    
    test()()
    print(f"global x: {x}")
    print(f"global y: {y}")
    print(f"global z: {z}")
    print(f"global delete: {delete}")
    
    # # Result:
    # # nonlocal y: 11
    # # global x: 2
    # # global y: 1
    # # global z: 100
    # # NameError: name 'delete' is not defined
    ```


### 2. 导入系统

导入系统是Python的核心部分，它允许我们将代码组织成模块和包，从而实现代码的重用和组织。关于导入系统我们可以参考Python文档：[导入系统 — Python 3.13.0 文档](https://docs.python.org/zh-cn/3/reference/import.html)。

在`JavaScript`中，我们可以使用`import`导入文件夹或者文件。在Python中我们可以使用`import`关键字导入模块、包或者模块、包中的变量，以下是几种导入语句：

* **`import module`**：导入对应模块、包，创建名为`module`的实例对象；

* **`from module import module`**：导入模块、包中的对应子模块、包；

* **`from module import var`**：导入模块、包中对应的变量；

    需要注意的是在这里说的变量是广义上的变量，即Python的类的实例类型。例如，函数便是`function`类的实例对象，类便是元类`type`的实例对象。

* **`from module import *`**：导入模块、包中所有可导出的变量；

    我们可以在模块或者包的`__init__.py`中声明`__all__`魔法变量，指定能被`from module import *` 导出的变量名。需要注意的是`__all__`只适用于该语句，`import module`和`from module import var`均可访问该变量。

    ```python
    # module_1.py
    
    __all__=['a', 'b']
    a,b,c=1,2,3
    ```

    ```python
    # test.py
    
    import module_1
    from module_1 import *
    
    print(module_1.a)	# 1
    print(a)		# 1
    
    print(module_1.c)	# 3
    print(c)		# NameError: name 'c' is not defined
    ```

在Python中我们使用`import`关键字导入模块或者包时，不需要考虑文件路径问题。Python解释器会在运行时在`sys.path`中查找相应的模块或者包。值得一提的是该查找过程也存在优先级关系，在`sys.path`中靠前的文件夹下的模块和包具有更高的优先级。

需要注意的是，当我们在`PyCharm`中使用图形化界面运行Python文件时，IDE会为我们添加**当前文件夹**和**项目文件夹**到`sys.path`中。

```python
import sys

for path in sys.path:
    print(path)
    
# # Result:
# # === IDE INSERT ===
# # D:\Code\Python\Science\learn\folder\folder
# # D:\Code\Python\Science\learn

# # === ORIGIN ===
# # D:\Software\Anaconda\envs\geo\python39.zip
# # D:\Software\Anaconda\envs\geo\DLLs
# # D:\Software\Anaconda\envs\geo\lib
# # D:\Software\Anaconda\envs\geo
# # D:\Software\Anaconda\envs\geo\lib\site-packages
```

我们在前面介绍了如何利用导入对象的属性来区别模块、常规包与命名空间包。我们在这里简单介绍一下模块对象的部分属性：

| 名称                                                      | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| **`__name__`**                                            | 该属性必须被设为模块的完整限定名称，其用来在导入系统中作为模块标识； |
| <span style='white-space: nowrap'>**`__loader__`**</span> | 该属性是导入系统在导入模块时使用的加载器对象（<span style='color:pink'>`Python 3.12` 后该属性仍可使用，但推荐使用 `__spec__.loader` 属性，`Python 3.14` 后该属性将被废弃</span>）； |
| **`__spec__`**                                            | 该属性是导入模块时要使用的模块规格说明；                     |
| **`__path__`**                                            | 如果模块是包，则`__path__`是可迭代的字符串序列。如果模块不是包，则`__path__`属性不存在； |

需要注意的是Python除了有`regular packages`还有`namespace packages`这种特殊的包导入模式。下面我们依次介绍这三种导入模式：

* **模块**：Python将单个Python文件称为模块；

* **常规包**：Python将包含`__init__.py`文件的文件夹或者压缩文件夹称为常规包；

    Python 中的`__init__.py`文件更像是`JavaScript`中的`index.js`或者`index.ts`文件，用于导出同级目录下的其他模块或者包。子包可以是常规包，也可以是命名空间包。

    我们可以使用`__path__`属性来确定导入的是模块还是包。

    ```python
    import module_1
    import package_1
    import namespace_package_1
    
    print(module_1.__path__)
    print(package_1.__path__)
    print(namespace_package_1.__path__)
    
    # # Result:
    # # AttributeError: module 'module_1' has no attribute '__path__'
    # # ['D:\\Code\\Python\\Science\\learn\\package_1']
    # # _NamespacePath([
    # # 	'D:\\Code\\Python\\Science\\learn\\namespace_package_1', 
    # # 	'D:\\Code\\Python\\Science\\learn\\namespace_package_1'
    # # ])
    ```

    当我们在编写同一个包的不同模块时，想要在一个模块中导入另一个模块，可以采用相对导入方法。

    ```
    mypackage/
    │
    ├──subpackage1/
    │   ├── __init__.py
    │   ├── module0.py
    │   └── module1.py
    │
    ├── subpackage2/
    │   ├── __init__.py
    │   └── module2.py
    │
    └── __init__.py
    ```

    ```python
    # mypackage/subpackage1/module0.py
    
    from . import module1
    from .. import subpackage2
    from ..subpackage2 import module2
    ```

* **命名空间包**：Python 将没有 `__init__.py` 的文件夹称为命名空间包（<span style='color:pink'>`Python 3.3` 后引入了命名空间包</span>）；

    Python的命名空间包允许我们将一个包跨多个目录拆分。当我们在不同文件夹下有相同名称的命名空间包，这些命名空间包共享一个命名空间。例如当第三方包中有名为`namespace_package_1`的命名空间包，我们在自己的项目文件夹下拓展了改包中的子包或这子模块，我们可以使用<span style='color:cyan'>相同的导入方式</span>导入拓展包与原始包，保证代码的可读性。

    ```python
    from namespace_package_1 import origin
    from namespace_packges_1 import new
    ```

    此外我们还可以在拓展包中相对导入原始包：

    ```python
    from . import origin
    ```

    关于命名空间包的使用实践可以参考这篇文章：[关于namespace packages - 知乎](https://zhuanlan.zhihu.com/p/379449893#:~:text=进入命名空间包的神奇)。

Python文档中推荐使用`importlib`动态导入模块，`importlib`模块是Python的标准库模块。使用`importlib`，我们可以动态地导入模块，这意味着可以在运行时根据需要导入模块，而不是在代码的开头静态地导入它们。关于`importlib`我们可以参考Python文档：[importlib — Python 3.13.0 文档](https://docs.python.org/zh-cn/3/library/importlib.html#)。我们在这里简单介绍一下`importlib`的部分用法：

* **`importlib.import_module(name, package=None)`**：  参数 *name* 指定了以绝对或相对导入方式导入什么模块。如果采用相对导入的方式，那么 *package* 必须设置为那个包名；

    ```python
    import importlib
    
    module_1=importlib.import_module('pkg.mod')
    module_2=importlib.import_module('..mod', 'pkg.subpkg')
    ```

* **`importlib.reload(module)`**：重新加载之前导入的 *module*。 那个参数必须是一个之前已经成功导入的模块对象。该方法常用于调试和开发中，因为它允许我们在修改模块代码后重新加载它们而不需要重启程序。这里不继续介绍该方法的使用细节，请使用时翻阅文档；


### 3. 函数与匿名函数

Python可以使用`def`关键字定义一个函数对象，以下是一个较为完善的函数模板：

```python
def test_func(a:int, b:str, *args, **kwargs):
    """
    Write function docs here

    :param a: Description of param 'a'
    :type a: int
    :param b: Description of param 'b'
    :type b: str
    :param args: Description of param 'args'
    :param kwargs: Description of param 'kwargs'
    :return: Description of return
    :rtype: None
    """
    print(a, b)
    print(args)
    print(kwargs)
    return

rt_data=test_func(1,'2',3,c=1,d=2,e=3)
print(rt_data)

# # Result:
# # 1 2
# # (3,)
# # {'c': 1, 'd': 2, 'e': 3}
# # None
```

我们从以下方面了解一下函数：

* **参数**

    函数在调用的时候可以接收数值作为参数值，也可以接收键值对作为参数值。我们可以用`*args`接收哪些未在函数头中指明形参的数值数组，其中接收到的数值序列是以元组的形式存储的；此外我们还可以使用`**kwargs`接收未在函数头中指明的键值对，其中接收到的键值对序列是以字典的形式存储的。**需要注意的有**：

    1. 在调用函数时，参数格式需要遵守<span style='color:cyan'>先数值后键值对的规范，否则程序会报错</span>。

        我们在函数头中指明了的参数名，在参考模板中为`a`、`b`实际上可以使用数值或者键值对这两种方式提供，但如果我们的函数头的参数定义类似上述模板，则只能使用数值的方式提供。因为，指明参数必须在未指明参数之前，数值参数必须在键值对参数之前。

    2. Python不支持<span style='color:cyan'>函数重载</span>，这意味着我们的函数如果需要有多个处理模式，可以设置`mode`参数进行区分，然后再根据对应模式处理每次调用的不定长参数序列`args`、`kwargs`。 我们也可以通过设置形参默认参数与`isinstance(ins, type)`来实现函数重载的类似效果。

        ```python
        def test_func1(mode, *args, **kwargs):
            pass
        
        def test_func2(a, b=None):
            if isinstance(a, int) and b is None:
                return a
            elif isinstance(a, int) and isinstance(b, int):
                return a + b
            else:
                raise TypeError("Unsupported types")
        ```

    3. 在`Python 3.8`中新增了一个函数形参语法，我们可以用`/`、`*`来分割函数形参的数值部分与键值对部分。在`/`左边的形参必须以数值方式提供，在`*`右边的形参必须以键值对方式提供，两者中的形参既可以使用数值也可以使用键值对方式提供。但需要注意的是，调用时参数仍需满足先数值后键值对的形参提供方式。

        ```python
        def test_func(a, b, /, c, d, *, e, f):
            pass
        
        test_func(1,2,3,d=4,e=5,f=6)
        ```

    在Python函数的学习中，我们还需要注意**参数传递**的细节。当我们传递的参数是<span style='color:cyan'>不可变数据类型</span>时，我们在函数体内修改参数的值不会影响到原对象，当我么传递的参数是<span style='color:cyan'>可变数据类型</span>时，我们在函数体内对参数进行修改会影响到原对象。关于可变数据类型与不可变数据类型请参考：

    ```python
    def test_func(a, b):
        a=10
        b.append(10)
    
    a, b=1, [1,2,3]
    test_func(a, b)
    print(a, b)		# 1 [1, 2, 3, 10]
    ```


* **返回值**

    我们可以在函数头后边使用`->`声明函数的返回值类型，这个函数返回值类型声明不是必须的。

    1. 当函数没有使用`return`返回返回值时，默认返回值是`None`。
    2. Python支持像`Golang`那样返回多个返回值，我们可以配合Python中`=`的多变量赋值特性，简化代码。

        ```python
        def test_func():
            return 1, 2
        
        x, y=test_func()
        ```

    Python的匿名函数的功能有限，我们可以使用`lambda`关键字来创建Python的匿名函数。以下是一个简单的匿名函数模板：

    ```python
    # # Template:
    # # lambda <arguments>: <return>
    func = lambda x: x+5
    print(func(10)) 	# 15
    
    max = lambda x, y: x if x>=y else y
    print(max(3, 4)) 	# 4
    ```


### 4. 装饰器

装饰器是Python中一种非常强大的工具，它们允许我们在不修改原有函数代码的情况下，给函数添加新的功能。装饰器有以下两种类型：

* **函数装饰器**

    以下是一个函数装饰器的模板。需要注意的是装饰器实际上是Python的 <u>*语法糖*</u> ，我们完全可以不用使用`@decorator`的方式实现相同的功能。

    ```python
    def my_decorator(func):
     def wrapper(x, y):
         print(f"Before {func.__name__}, x: {x}, y: {y}")
         result = func(x, y)
         print(f"After {func.__name__}, result: {result}")
         return result
     return wrapper
    
    @my_decorator
    def add(x, y):
     return x + y
    
    # # Equals to next line
    # # add=my_decorator(add)
    ```

    装饰器可以接收参数，这需要给装饰器外层再添加一层函数：

    ```python
    def repeat(num_times):
     def decorator_repeat(func):
         def wrapper(*args, **kwargs):
             for _ in range(num_times):
                 result = func(*args, **kwargs)
             return result
         return wrapper
     return decorator_repeat
    
    @repeat(num_times=3)
    def greet(name):
     print(f"Hello {name}")
    
    greet("World")
    
    # # Equals to next two lines
    # # my_decorator=repeat(num_times)
    # # greet=my_decorator(greet)
    ```


* **类装饰器**

    从`Python 3.4`开始，实现了`__call__`魔术方法的类也可以作为装饰器。

    ```python
    class MyDecorator:
     def __call__(self, func):
         def wrapper(x, y):
             print(f"Before {func.__name__}, x: {x}, y: {y}")
             result = func(x, y)
             print(f"After {func.__name__}, result: {result}")
             return result
         return wrapper
    
    @MyDecorator()
    def add(x, y):
     return x + y
    
    print(add(2, 3))
    ```


### 5. 迭代器与生成器

在Python中，**迭代器** | `Iterator` 和 **生成器** | `Generator` 是用于高效处理数据集合的两种机制。

* **迭代器**

    迭代器有两个基本的方法：

    * `iter()`：调用该方法返回一个迭代器对象，此方法对应的魔术方法为`__iter__`。需要注意的是，对对象调用`iter()`返回的对象，不一定需要是对象本身，只需是一个迭代器对象即可；

    * `next()`：调用该方法可以对迭代器对象进行迭代，迭代器对象需要实现`__next__`魔术方法。我们可以使用`StopIteration`异常来标记迭代完成，防止出现无限循环在`next()`方法中我们可以设置在完成指定循环次数后触发`StopIteration`异常来结束迭代；

        ```python
        class MyNumbers:
          def __iter__(self):
            self.a = 1
            return self
        
          def __next__(self):
            if self.a <= 20:
              x = self.a
              self.a += 1
              return x
            else:
              raise StopIteration
        
        myclass = MyNumbers()
        myiter = iter(myclass)
        
        for x in myiter:
          print(x)
        ```

    需要注意的是<span style='color:cyan'>对于一个迭代器来说，生命周期内只能进行一轮迭代</span>，之后再调用`next()`便会引起`StopIteration`的异常。当我们多次使用`for item in list`遍历列表等序列时，实际上是为每一轮迭代生成了一个迭代器。


* **生成器**

    在Python中，使用了`yield`的函数被称为 **生成器** | `generator` 。生成器是一种特殊的迭代器，它使用`yield`语句来产生一系列值。每次迭代到`yield`语句时，它会暂停执行并保存当前的运行状态，然后返回值给调用者。当下次迭代时，它会从上次暂停的地方继续执行。

    ```python
    def my_generator(data):
     for item in data:
         yield item
    
    my_gen_1=my_generator([1,2,3,4,5])
    for item in my_gen_1:
     print(item)
    
    my_gen_2 = my_generator([5,4,3,2,1])
    print(next(my_gen))
    print(next(my_gen))
    print(next(my_gen))
    print(next(my_gen))
    print(next(my_gen))
    ```


### 6. 内置方法

Python的标准库为我们提供了一些内置方法，想要了解Python的内置方法有哪些可以查询Python文档：[内置函数 — Python 3.13.0 文档](https://docs.python.org/zh-cn/3/library/functions.html)。在这里我们只列出一部分常用的Python内置方法。

#### 🔁 用于可迭代序列

* **`all(iterable)`**：如果 *iterable* 的所有元素均为真值（或可迭代对象为空）则返回`True`。 等价于：

    ```python
    def all(iterable):
        for element in iterable:
            if not element:
                return False
        return True
    ```

* **`any(iterable)`**：如果 *iterable* 的任一元素为真值则返回`True`。 如果可迭代对象为空，返回`False`。 等价于：

    ```python
    def any(iterable):
        for element in iterable:
            if element:
                return True
        return False
    ```

* **`zip(*iterables, strict=False)`**：该方法返回元组的迭代器，其中第 *i* 个元组包含的是每个参数迭代器的第 *i* 个元素。默认情况下，`zip()`会在最短的参数迭代器迭代完成后停止（<span style='color:pink'>`Python 3.10` 后引入 `strict` 。当 `strict` 为 `True` 时，`zip()` 会检查 *iterable* 是否长度相等。</span>）：

    ```python
    l1=[1,2,3]
    t1=(1,2,3,4,5)
    z1=zip(l1, t1)
    
    print(z1)
    for item in z1:
        print(item)
    
    # # Result:
    # # <zip object at 0x000001E0AA6EE900>
    # # (1, 1)
    # # (2, 2)
    # # (3, 3)
    ```

* **`enumerate(iter, start=0)`**：返回一个枚举对象。我们可以通过设置`start`参数来控制<u>计数器开始值</u>：

    ```python
    l1=['a', 'b', 'c', 'd']
    enum=enumerate(l1)
    
    print(enum)
    for item in enum:
        print(item)
    
    # # Result: 
    # # <enumerate object at 0x0000013B7D0CE7C0>
    # # (0, 'a')
    # # (1, 'b')
    # # (2, 'c')
    # # (3, 'd')
    ```

#### 🧱用于实例对象

* **`callable(obj)`**：判断 *obj* 是否为可调用类型实例（<span style='color:pink'>该方法在 `Python 3.0` 时被移除，然后在 `Python 3.2` 时被重新加入。</span>）；
* <span style='color:cyan'>**`dir(obj)`**</span>：如果没有实参，则返回当前本地作用域中的名称列表。如果有实参，它会尝试返回该对象的<span style='color:cyan'>有效属性列表</span>；

    ```python
    print(dir())
    
    # # Result: 
    # # [
    # #     '__annotations__', 
    # #    '__builtins__', 
    # #    '__cached__', 
    # #    '__doc__', 
    # #     '__file__', 
    # #     '__loader__', 
    # #     '__name__', 
    # #     '__package__', 
    # #     '__spec__'
    # # ]
    ```

* **`hasatter()`**：判断对象是否有对应属性。此方法是调用`getattr(object, name)`看是否有`AttributeError`异常来实现的；

* **`getattr(object, name)`**
    **`getattr(object, name, default)`**：获取对象的属性值。如果在调用时提供了`default`且没有该属性则返回`default`，如果在调用时没有提供`default`且没有该属性则会引起 `AttributeError`的异常；

* **`setattr(object, name, value)`**：设置对象对应属性值；

* **`delattr(obj, name)`**：删除对象的属性值。如果对象没有该属性则会引起`AttributeError`的异常；

#### 🧪用于执行与测试

我们在这里只简单介绍一下以下方法。关于 `breakpoint()` 、*globals* 、*locals* 等细节请参考Python文档。

| 函数                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`breakpoint(*args, **kwargs)`**                            | 我们可以在代码中使用`breakpoint()`暂停程序                   |
| **`eval(source, /, globals=None, locals=None)`**             | 该方法用于将字符串作为<u>Python 表达式</u>来执行，并返回表达式的值； |
| **`exec(source, /, globals=None, locals=None, *, closure=None)`** | 该方法用于执行存储在字符串或对象中的<u>Python 代码</u>；     |

#### 🆔 用于对象标识符

| 函数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| **`id(obj)`**   | 返回对象的“标识值”。该值是一个整数，在此对象的生命周期中保证是唯一且恒定的。两个生命期不重叠的对象可能具有相同的`id()`值 |
| **`hash(obj)`** | 返回该对象的哈希值。对于具有自定义`__hash__()`方法的对象，请注意`hash()`会根据宿主机的字长来截断返回值 |

#### 🔍 用于类型检查

| 函数                               | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| **`isinstance(obj, classinfo)`**   | 当前检查对象不是给定类型的，则函数总是返回`False`。当 *obj* 是 *classinfo* 的实例，则返回`True` |
| **`issubclass(class, classinfo)`** | 如果 *class* 是 *classinfo* 的子类（直接、间接或抽象基类的子类），则返回`True` |

关于上述两种方法的 *classinfo* 参数可以为一个类、多个类的元组。从`Python 3.10`开始，可以是一个`union`类型。

#### 💡 用于帮助

调用`help()`启动内置的帮助系统，此函数主要在<u>交互式</u>中使用。

1. 如果没有实参，解释器控制台里会启动交互式帮助系统。
2. 如果实参是一个字符串，则在模块、函数、类、方法、关键字或文档主题中搜索该字符串，并在控制台上打印帮助信息。
3. 如果实参是其他任意对象，则会生成该对象的帮助页。
