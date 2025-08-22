---
title: Python笔记：数据结构
date: 2024-10-20 19:31:14
tags:
  - python
  - tutorial
categories:
  - 笔记
cover: https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/python-3.png
banner: https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/python-3.png
poster:
  headline: Python笔记：数据结构
---

> 这里我只是简单介绍一下Python数据结构的一些常用内容，详细内容请参考：[Python 3.13.0 文档](https://docs.python.org/zh-cn/3/) 。

## 数据类型

Python为我们提供了`type()`、`isinstance()`等方法来帮助我们查看变量的数据类型。执行如下代码我们得到了Python中的基本数据类型。这些基本类型都是由Python的标准库模块`builtins`提供。

Python中的基本数据类型可以大致分为：**可变数据类型** | `Mutable Type`、**不可变数据类型** | `Immutable Type` 。可变数据在定义后可以进行修改；而不可变数据在定义后不可以进行修改，当我们对值为<span style='color:cyan'>不可变数据类型</span>的变量进行修改时，<u>实际上是删除原有数据后重新赋值而已</u>。

* **不可变数据类型**：`int`、`float`、`bool`、`complex`、`str`、<span style='color:cyan'>**`tuple`**</span>、<span style='color:cyan'>**`bytes`**</span> 。
* **可变数据类型**：`list`、`dict`、`set`、`bytearray` 。

```python
t1=1
t2=3.14
t3=3+4j
t4=True
t5='123'
t6=[1, 2]
t7=(1, 2)
t8={1, 2}
t9={'a':1}
t10=b'00000001'
t11=bytearray(t10)

print(type(t1))     # <class 'int'>
print(type(t2))     # <class 'float'>
print(type(t3))     # <class 'complex'>
print(type(t4))     # <class 'bool'>
print(type(t5))     # <class 'str'>
print(type(t6))     # <class 'list'>
print(type(t7))     # <class 'tuple'>
print(type(t8))     # <class 'set'>
print(type(t9))     # <class 'dict'>
print(type(t10))    # <class 'bytes'>
print(type(t11))    # <class 'bytearray'>
```

我们详细介绍一下上面提到的数据类型：

### 1. 数值类型

Python的数值类型有：`int`、`bool`、`float`、`complex`：

* **`int` & `bool`**：

    **`Python 3`的整型是没有限制大小的**，可以当作`Long`类型使用，所以`Python 3`没有`Python 2`的`Long`类型。`Python 3`的布尔类型是整型的子类，需要注意的是布尔值首字母需要大写`True`。由于的布尔类型是整型的子类，在使用`==`比较时有以下登上成立：

    ```python
    print(1==True)	# True
    print(0==False)	# True
    ```

* **`float`**：

    浮点型由整数部分与小数部分组成，浮点型也可以使用科学计数法表示，**`2.5e5`即`2.5*10^5`**。

    ```python
    f1=250000.0
    f2=2.5e5
    print(f1==f2)	# True
    ```

* **`complex`**：

    即复数类型，由实部与虚部组成，可以使用以下两种方式生成。

    ```python
    c1=3+4j
    c2=complex(3, 4)
    
    print(c1==c2)	# True
    ```


### 2. 序列类型

Python 的序列类型有：`list`、`tuple`、`set`、`dict`。这四者的异同主要体现在三方面：元素的<span style='color:cyan'>可变性</span>、<span style='color:cyan'>唯一性</span>、<span style='color:cyan'>有序性</span>。

|          | 可变性 | 唯一性            | 有序性            |
| -------- | ------ | ----------------- | ----------------- |
| **列表** | ✅      | ❌                 | ✅                 |
| **元组** | ❌      | ❌                 | ✅                 |
| **集合** | ✅      | ✅                 | ❌                 |
| **字典** | ✅      | ✔️ \| `Unique Key` | ✔️ \| `python 3.7` |

* **可变性**：元素在定义后是否可以进行修改。

* **唯一性**：元素在同一序列中是否可以重复。

    这些可以有元素唯一性特性的数据类型在判断内部元素是否具有唯一性时，是调用元素的 `__eq__()`、`__hash__()` 方法来判断的，这里需要注意的是：对于`dict`而言，是`dict`的唯一性是对于**键**而言的。

    我们将这类可以判断唯一性的数据类型称为 **<span style='color:cyan'>Hashable</span>** 类型。以下是常用类型的唯一性判断标准：

    1. `Immutable Type` 大多数都是 <span style='color:cyan'>可哈希</span> 的；

    2. `Mutable Container` 都 <span style='color:cyan'>不可哈希</span> 的，例如 `list`、`dict` ；

    3. `Immutable Container` 仅当它们的元素均为可哈希时才是 <span style='color:cyan'>可哈希</span> 的，例如 `tuple`、`frozenset` 。

    **用户定义类的实例对象默认是可哈希的**，它们在比较时一定不相同（除非是与自己比较），它们的哈希值是使用`id()`获取的。`id(object)` 返回对象的 <span style='color:cyan'>标识值</span> 。该值是一个整数，在此对象的生命周期中保证是唯一且恒定的。两个生命期不重叠的对象可能具有相同的`id()`值。

    如果我们想为自己的类实例定制化`hash(ins)`的输出结果，可以重写 `__hash__()`、`__eq__()` 这两个魔术方法，以下是示例：

    ```python
    class HashableClass:
        def __init__(self, id):
            self._id = id
    
        def __hash__(self):
            # Get hash, make sure using same id with __eq__()
            return hash(self._id)
    
        def __eq__(self, other):
            # Check self equals to other
            if isinstance(other, HashableClass):
                return self._id == other._id
            return False
    ```

* **有序性**：元素的存储顺序是否有序。

    列表与元组的有序性显而易见，这里我们着重讨论一下字典的有序性。

    字典在`Python 3.7`之前是随机存储键值对的，我们使用`dict.popitem()`弹出的是一个随机的键值对。在`Python 3.7`之后，字典采用 **后进先出**\| ``LIFO``的策略管理键值对，故我们使用`dict.popitem()`弹出的始终是最近进入字典的键值对。

    ```python
    d1={1: 'None',2: 'None',3: 'None', 4: "None"}
    d1.popitem()    
    print(d1)       # {1: 'None', 2: 'None', 3: 'None'}
    d1[4]=123
    d1[2]=123
    d1.popitem()
    print(d1)       # {1: 'None', 2: 123, 3: 'None'}
    ```

    需要注意的是，只有<span style='color:cyan'>新增的键</span>才被视为<span style='color:cyan'>进</span>，如果使用`dict.update()`、`dict[key]=value`来更新字典内容不会使该键值对重新进入字典。

    ```python
    d1={1: 'None',2: 'None',3: 'None', 4: "None"}
    d1.popitem()
    print(d1)       # {1: 'None', 2: 'None', 3: 'None'}
    d1[4]=123
    d1.update({1:1})
    d1.popitem()
    print(d1)       # {1: 'None', 2: 123, 3: 'None'}
    ```

#### 🔧 公共方法

以下是这四种类型的公共方法：

* **`len()`、`max()`、`min()`** ：

    `builtins`提供了列表、元组和集合的通用函数：`len()`、`max()`、`min()`。

    ```python
    targets = {
     'list': [1, 2, 2],
     'tuple': (1, 2, 2),
     'set': {1, 2},
    }
    functions = [len, max, min]
    
    for key in targets:
     print(key)
     for f in functions:
         print(f"\t{f.__name__}({key}) => {f(targets[key])}")
     print()
    ```

    <span style='color:cyan'>字典也可以使用上述通用函数</span>。对于`len()`所有字典都可以使用，对于`max()`、`min()`则只有当作为键的类型实现了关于比较的方法后才能使用`max()`、`min()`。需要实现的魔术方法有：`__eq__()`、`__lt__()`、`__le__()`、`__gt__()`、`__ge__()`等。

    ```python
    dict1={True: 1, 2: 2, 3.14:3}
    dict2={True: 1, 2: 2, "3.14":3}
    dicts={
     'dict1': dict1,
     'dict2': dict2,
    }
    functions = [len, max, min]
    
    for dict_name in dicts:
     for f in functions:
         try:
             print(f"{f.__name__}({dict_name}) = {f(dicts[dict_name])}")
         except TypeError as err:
             print(f"{f.__name__}({dict_name}) => {err}")
    
    # # Result: 
    # # len(dict1) = 3
    # # max(dict1) = 3.14
    # # min(dict1) = True
    # # len(dict2) = 3
    # # max(dict2) => '>' not supported between instances of 'str' and 'int'
    # # min(dict2) => '<' not supported between instances of 'str' and 'bool'
    ```

* **切片与删除**：

    列表、元组等**有序序列**都支持切片，需要注意的是虽然 ***字典*** 也可以看作有序序列 ，但其却并不支持切片操作。

    切片遵循<span style='color:cyan'>**左闭右开**</span>的裁切方式。左`index`大于右`index`，左右`index`大于`len(list)`都不会报错，请在裁切时做好<u>防御性编程</u>。以下是切片实验代码，运行他们以便你更好的了解切片：

    ```python
    l1=[1,2,3,4,5]
    l2=l1[:]
    l3=l1[0:]
    l4=l1[0:-1] 
    l5=l1[4:3]  # Wont report error
    l6=l1[7:8]  # Wont report error
    
    print(l1)   # [1, 2, 3, 4, 5]
    print(l2)   # [1, 2, 3, 4, 5]
    print(l3)   # [1, 2, 3, 4, 5]
    print(l4)   # [1, 2, 3, 4]
    print(l5)   # []
    print(l6)   # []
    ```

    列表、元组、字典等**有序序列**都支持使用`del`关键字进行删除操作，***集合*** 仅支持使用`del`删除集合本身：

    ```python
    l1=[1,2,3]		# list
    d1={1: 1, '1': 1}	# dict
    t1=(1,2,3)		# tuple
    
    del l1[1]
    del d1['1']
    
    print(l1)       	# [1, 3]
    print(d1)       	# {1: 1}
    
    del l1
    del d1
    del t1
    
    s1={1,2,3}
    del s1
    ```

#### 🧰 实例方法

上述四种类型的使用以及实例方法如下：

* **`list`常用实例方法**：

    | 方法                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | `list.count(obj)`                                            | 统计某个元素在列表中出现的次数                               |
    | `list.clear()`                                               | 清空列表                                                     |
    | `list.append(obj)`                                           | 在列表末尾添加新的对象                                       |
    | `list.remove(obj)`                                           | 移除列表中某个值的第一个匹配项                               |
    | `list.insert(index, obj)`                                    | 从列表中找出某个值第一个匹配项的索引位置                     |
    | `list.extend(seq)`                                           | 在列表末尾一次性追加另一个序列中的多个值                     |
    | <span style='color: cyan;white-space: nowrap'>`list.sort(key=None, reverse=False)`</span> | 对原列表进行排序                                             |
    | `list.reverse()`                                             | 反转列表中元素                                               |
    | **`list.copy()=>list`**                                      | 复制列表                                                     |
    | **`list.pop(index=-1)=>obj`**                                | 移除列表中的一个元素（默认最后一个元素），并且返回该元素的值 |

* **`set`常用实例方法** ：

    | 方法                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | `set.add(hashable_el)`                                       | 向集合添加一个 ***Hashable*** 元素                           |
    | `set.clear()`                                                | 移除集合中的所有元素                                         |
    | `set.copy() => set`                                          | 返回集合的浅拷贝                                             |
    | <span style='color:silver'>\# *删除操作* ：</span>           |                                                              |
    | `set.pop()`                                                  | 从集合中移除并返回任意一个元素。 如果集合为空则会引发`KeyError` |
    | `set.remove(hashable_el)`                                    | 从集合中移除元素 *hashable_el*。 如果 *hashable_el* 不存在于集合中则会引发`KeyError` |
    | `set.discard(hashable_el)`                                   | 如果元素 *hashable_el* 存在于集合中则将其移除                |
    | <span style='color:silver'>\# *原集合不变返回新集合* ：</span> |                                                              |
    | `set.isdisjoint(other)=>bool`                                | 判断 *set* 是否与 *other* **无交集**                         |
    | `set.issubset(other)=>bool`                                  | 判断 *set* 是否为 *other* 的子集                             |
    | `set.issupperset(other)=>bool`                               | 判断 *set* 是否为 *other* 的超集                             |
    | <span style='color:silver'>\# *原集合不变返回新集合* ：</span> |                                                              |
    | `set.union(other)=>set`                                      | 返回两个集合的合集                                           |
    | `set.difference(other)=>set`                                 | 返回两个集合的差集                                           |
    | `set.intersection(other)=>set`                               | 返回两个集合的交集                                           |
    | `set.symmetric_difference(other)=>set`                       | 返回两个集合中不重复的元素集合                               |
    | <span style='color:silver'>\# *在原集合上更新* ：</span>     |                                                              |
    | `set.update(other)`                                          | 为集合添加来自 *others* 中的所有元素                         |
    | `set.difference_update(other)`                               |                                                              |
    | `set.intersection_update(other)`                             |                                                              |
    | `set.symmetric_difference_update(other)`                     |                                                              |

    注意！`set`元素必须为 ***Hashable*** 元素，如果不是则会报错`KeyError`。

* **`dict`**：

    我们可以使用多种类型实例作为字典的键，但需要注意的是，该类型实例必须为 ***Hashable*** 元素。即实现了`__hash__()`、`__eq__()`方法。

    当我们使用`str`类型作为字典的键时，不可以省略`‘’`，这一点不同于`JavaScript`。

    ```python
    dict1={
    	'a': 100,
    	'b': True
    }
    # Realize iterable
    for i in dict1:             	# a 100
    	print(i, dict1[i])      # b True
    ```

    我们可以通过字典键来管理字典内容：

    ```python
    dict1={}
    dict1['key1']=1
    dict1[2]=2
    print(dict1)    # {'key1': 1, 2: 2}
    
    del dict1[2]
    print(dict1)    # {'key1': 1}
    ```

    字典的常用实例方法如下：

    | 方法                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | <span style='white-space: nowrap'>`fromkeys(iterable, value=None)=>dict`</span> | 使用来自 *iterable* 的键创建一个新字典，并将键值设为 *value*。`fromkeys()`是`dict`类型的静态方法，使用过程如下： |
    
    ```python
    iter=[1,2,3]
    dict1=dict.fromkeys(iter, 'hello')
    print(dict1)	# {1: 'hello', 2: 'hello', 3: 'hello'}
    ```
    
    | 方法                                                       | 说明                                                         |
    | ---------------------------------------------------------- | ------------------------------------------------------------ |
    | `dict.clear()`                                             | 移除字典中的所有元素                                         |
    | `dict.copy()`                                              | 返回原字典的浅拷贝                                           |
    | <span style='color:silver'>\# *字典值获取方法* ：</span>   |                                                              |
    | `dict.get(key, default=None)`                              | 如果 *key* 存在于字典中则返回 *key* 的值，否则返回 *default* 。注意  *default* 初始值为`None`因此不会为报`KeyError` |
    | `dict.pop(key[,default])=>value`                           | 如果 *key* 存在于字典中则将其移除并返回其值，否则返回 *default*。 如果 *default* 未给出且 *key* 不存在于字典中，则会引发`KeyError` |
    | <span style='color:silver'>\# *字典键值获取方法* ：</span> |                                                              |
    | `dict.popitem()=>(key, value)`                             | 从字典中移除并返回一个`(key, value)`对，键值对会按 LIFO 的顺序被返回；<br/><span style='color:pink'>`Python 3.7` 采用 LIFO 策略即`stack`，管理字典的键值对。在 `Python 3.7` 之前此方法会移除并返回一个随机键值对。</span> |
    | <span style='color:silver'>\# *字典键值更新方法* ：</span> |                                                              |
    | `dict.update(other)`                                       | 使用 *other* 中的键值对更新 *dict* ，也可以使用`dict.update(key1=value1, ...)`的方式更新。注意使用`dict.update(key1=value1)`时，字典会自动将 *key1* 转化为`‘key1’`字符串作为键 |
    
     ```python
     d1={1: 'None'}
     d1.update(key='None')
     print(d1)				# {1: 'None', 'key': 'None'}
     ```
    
    | 方法                                                         | 说明                                                         |
    | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | <span style='white-space: nowrap'>`dict.setdefault(key, default=None)`</span> | 如果字典存在键 *key* ，返回它的值。如果不存在，插入值为 *default* 的键 *key* ，并返回 *default* |
    | <span style='color:silver'>\# *字典输出方法* ：</span>       |                                                              |
    | `dict.items()`                                               | 返回由字典项`(key, value)`组成的一个新视图，可以视作一个 *<span style='color:cyan'>iterable_tuples</span>* |
    | `dict.keys()`                                                | 返回由**字典键**组成的一个新视图                             |
    | `dict.values()`                                              | 返回由**字典值**组成的一个新视图                             |

* **`tuple`** ：

    元组中只包含一个元素时，需要在元素后面添加逗号`,`，否则括号会被当作运算符使用：
    
    ```python
    t1=(50)
    t2=(50, )
    
    print(type(t1))     # <class 'int'>
    print(type(t2))     # <class 'tuple'>
    ```
    
    元组之间可以使用运算符进行运算。这就意味着他们可以组合和复制，运算后会生成一个新的元组。
    
    ```python
    t1=(50, 10, 5)
    t2=(50, 20, 30)
    t3=t1+t2
    t4=t1*2
    
    print(t3)       # (50, 10, 5, 50, 20, 30)
    print(t4)       # (50, 10, 5, 50, 10, 5)
    ```

### 3. 字符串

在Python中我们可以使用`'`、`"` 、`"""`创建字符串，其中单引号与双引号用于创建普通字符串，三引号可以用于创建多行字符串与多行注释。以下是字符串的简单使用样例：

```python
str1='123'
str2="123"
str3 = """This is a
               multi-line string."""

print(str1)
print(str2)
print(str3)

# # Result:
# # 123
# # 123
# # This is a
# #            multi-line string.

def funcDocString(attr1, attr2):
    """Details in <Python|Best Exercise>"""
    return f'{attr1} {attr2}'
```

下面我们从以下方面来学习字符串：

#### ✏️ 常用转义字符

常用模板字符串有：**换行符** | `\n`、 **回车符** | `\r`、 **制表符** | `\t`、 **单引号** | `\'`、 **双引号** | `\"`、 **续行符** | `\`、 **反斜杠** | `\\` 。

```python
print("a\nb")		# a
			# b
print("a\rb")		# b
print("a\tb")		# a    b
print("a\'b")		# a'b
print("a\"b")		# a"b
print("a\			
b")			# ab
print("a\\b")		# a\b
```

#### ➕ 字符串运算符

常用的字符串运算符有：

| 运算符                                                  | 说明                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| `+`                                                     | 用于字符串拼接，当使用字符串拼接类型实例时，会调用实例对象的`__str__`方法生成对应字符串 |
| `*`                                                     | 用于字符串重复输出                                           |
| <span style='white-space: nowrap'>`in`、`not in`</span> | 用于判断字符串中是否包含指定字符串                           |
| `r''`                                                   | 原始字符串，不能输出转义字符                                 |
| `f''`                                                   | 模板字符串，等同于`‘’.format(*args)`                         |
| `''%()`                                                 | 格式化字符串，用于格式化输出                                 |

`+`用于字符串拼接，当使用字符串拼接类型实例时，会调用实例对象的`__str__`方法生成对应字符串；`*`用于字符串重复输出。

```python
s="123"
print(s+"456")	# 123456
print(s*2)		# 123123
```

`in`与`not in`这两个成员运算符作用于字符串时不同于列表、元组与字典。当这两个成员运算符作用于字符串时更像是执行了`bmp`算法，而当它们作用于列表等容器时<span style='color:cyan'>是将比较对象看作子元素而不是子集</span>。

```python
s1="123"
s2="123456"

l1=[1,2,3]
l2=[1,2,3,4,5,6]

print('1' in s2)    # True
print(s1 in s2)     # True

print(1 in l2)      # True
print(l1 in l2)     # False
```


当我们不想在字符串中使用转义字符时，可以使用原始字符串`r''`。

```python
print(r'\n\r\t\'\"\\')  # \n\r\t\'\"\\
```

#### 🧩 模板字符串

当我们需要拼接多个字符串与对象时，可以使用模板字符串进行拼接。Python的模板字符串有以下两种使用方式：

* `“{0}, {1}”.format(*args)`：我们使用`{index}`来为`format(*args)`中对应的对象占位，当输出字符串时会自动调用对象的`__str__()`方法，将生成的字符串插入到对应的`{index}`中；
* `f“{a1}, {a2}”`：我们也可以使用类似`JavaScript`中生成模板字符串的方式，给我们的模板字符串插入对应对象调用`__str__()`方法返回的字符串；

#### 🧵 字符串格式化

Python支持格式化字符串的输出。需要注意的是Python的字符串格式化不止服务于`print`方法，这是属于字符串的格式化方法。以下是Python字符串格式化符号：

| 符号 | 描述                                 |
| :--- | :----------------------------------- |
| `%c` | 格式化字符及其`ASCII`码              |
| `%s` | 格式化字符串                         |
| `%d` | 格式化整数                           |
| `%u` | 格式化无符号整型                     |
| `%o` | 格式化无符号八进制数                 |
| `%x` | 格式化无符号十六进制数               |
| `%X` | 格式化无符号十六进制数（大写）       |
| `%f` | 格式化浮点数字，可指定小数点后的精度 |
| `%e` | 用科学计数法格式化浮点数             |
| `%E` | 作用同`%e`，用科学计数法格式化浮点数 |
| `%g` | `%f`和`%e`的简写                     |
| `%G` | `%f`和`%E`的简写                     |
| `%p` | 用十六进制数格式化变量的地址         |

以下是字符串格式化的辅助符号：

| 符号      | 说明                                                  |
| --------- | ----------------------------------------------------- |
| `*`       | 用于动态指定宽度或精度（如%*.*f`）                    |
| `-`       | 左对齐输出内容                                        |
| `+`       | 在正数前显示加号（如`+3.2f`）                         |
| `<sp>`    | 正数前显示空格（负数仍显示负号）                      |
| `#`       | 启用进制前缀（如`0`、`0x`、`0X`）用于八进制或十六进制 |
| `0`       | 使用零填充输出的空白部分（如`08d`表示宽度为8，前补0） |
| `%%`      | 输出一个`%`字符                                       |
| `(*args)` | 使用元组或列表映射多个格式变量                        |
| `m.n`     | `m`表示最小总宽度，`n`表示小数点后的位数（如`%6.2f`） |

```python
s="%d\n%6.1f\n%  d\n%%"%(1, 6.111111, 1)
print(s)

# # Result:
# # 1
# #    6.1
# #  1
# # %
```

### 4. 字节数组类型

Python的字节数组类型有：`bytes`、`bytearray`。其中，`bytes`是二进制形式的不可变序列，`bytearray`是二进制形式的可变序列。由于`Python 3`的自带字符默认使用`utf-8`格式编码和显示，故这两种字节数组类型的默认编码格式是`utf-8`。

关于`bytes`与`bytearray`需要注意的地方有：

#### 🧱 字节数组与hashable

`bytes`的实现了`__hash__()`、`__eq__()`这两个魔术方法，故其实例对象可以作为字典键与集合元素，而`bytearray`则不行。

```python
s={1, 2}

b=b"123"
ba=bytearray(b)
print(b.__hash__())		# Run succeed
s.add(b)				
print(s)			# {b'123', 1, 2}

s.add(ba)			# TypeError: unhashable type: 'bytearray'
```

#### 🔣 字节数组与`int`

`bytes`与`bytearray`的底层是`int`类型，*<u>取值</u>* 与 *<u>切片</u>* 的结果如下：

```python
b=b"123"
ba=bytearray(b)

slice_b=b[:]
slice_ba=ba[:]

print(b[0], type(b[0]))         # 49 <class 'int'>
print(ba[0], type(ba[0]))       # 49 <class 'int'>

print(slice_b, type(slice_b))   # b'123' <class 'bytes'>
print(slice_ba, type(slice_ba)) # bytearray(b'123') <class 'bytearray'>
```

#### 🛠️ 创建字节数组

我们执行`help(bytearray)`可以看到`bytearray`的构造函数用法如下。

```python
bytearray(iterable_of_ints) 		# -> bytearray
bytearray(string, encoding[, errors]) 	# -> bytearray
bytearray(bytes_or_buffer) 		# -> mutable copy of bytes_or_buffer
bytearray(int) 		# -> bytes array of size given by 
			# the parameter initialized with null bytes
bytearray() 		# -> empty bytes array
```

我们执行`help(bytes)`则可以看到`bytes`的构造函数可以用如下方法使用：

```python
bytes(iterable_of_ints) 				# -> bytes
bytes(string, encoding[, errors]) 			# -> bytes
bytes(bytes_or_buffer) 					# -> immutable copy of 
							# 	bytes_or_buffer
bytes(int) 			# -> bytes object of size given by the 
				# parameter initialized with null bytes
bytes() 			# -> empty bytes object
```

我们发现这里`help`告诉我们的整型序列不是用的`list`、`tuple`这些关键词，而是用 *<span style='color:cyan'>iterable</span>* 描述的整型序列。如果想要我们的类支持通过`bytes`或者`bytearray`实现二进制序列化，则需要实现`__iter__()`这一魔术方法。关于此方法的说明和使用请参考下一节[面向对象编程](#面向对象编程)。

```python
class MyCollection:
def __init__(self, data):
self.data = data

def __iter__(self):
return iter(self.data)
```

字节数组构造函数的调用规律如下：

1. 可以使用可遍历的`int`序列创建，例如：`(1,2,3)`、`{1,2,3}`等。

    ```python
    b1=bytes((1,2,3))
    b2=bytes([1,2,3])
    b3=bytes({1,2,3})
    b4=bytes({1: '123'})	# Int-keys of dict is iterable
    
    print(b1)               # b'\x01\x02\x03'
    print(b2)               # b'\x01\x02\x03'
    print(b3)               # b'\x01\x02\x03'
    print(b4)               # b'\x01'
    ```

2. 可以使用`string`值创建，在创建时需要指定编码方式，默认是`utf-8`。

3. 可以使用`bytes`或者`buffer`创建。

4. 可以使用`int`值创建，根据`int`值生成对应长度的字节数组。

    ```python
    b1=bytes(1)
    
    for i in b1:
        print(i)		# 0 
    ```

5. 可以创建空的`bytes`或者`bytearray`。 

Python为我们提供了类型转换工具，这有助于我们更好的处理数据。我们常用`type_name(other_type_ins)`的方式进行类型转换，这实际上就是调用了对于类型的构造函数进行转换。Python的基本数据类型的类型转换规则如下：

1. **数值类型间可以相互转换**：

    ```python
    raw_list=[1, 1.0, 3+4j, True]
    trans_functions=[int, float, complex, bool]
    
    for raw in raw_list:
    for trans in trans_functions:
    try:
       print(f"{trans.__name__}({raw}) => {trans(raw)}")
    except TypeError as err:
       print(f"{trans.__name__}({raw}) | {err}")
    
    # # Result:
    # # int(1) => 1
    # # float(1) => 1.0
    # # complex(1) => (1+0j)
    # # bool(1) => True
    # # int(1.0) => 1
    # # float(1.0) => 1.0
    # # complex(1.0) => (1+0j)
    # # bool(1.0) => True
    # # int((3+4j)) | can't convert complex to int
    # # float((3+4j)) | can't convert complex to float
    # # complex((3+4j)) => (3+4j)
    # # bool((3+4j)) => True
    # # int(True) => 1
    # # float(True) => 1.0
    # # complex(True) => (1+0j)
    # # bool(True) => True
    ```

    执行上述代码，我们可以发现只有`complex`的转换成其他类型时有些限制，其他类型之间都可以相互转换。

2. **`list`、`set`、`tuple`与字节数组类型间可以相互转换**：

    稍微更改上一节的代码并执行可以发现，`list`、`set`、`tuple`之间可以相互转换。当这三者的元素数据类型只有`int`或者`bool`时，可以通过调用`bytes()`、`bytearray()`转换为字节数组。

    ```python
    list1=[1,2,True]
    tuple1=(1,2,3.1)
    set1={1,2,'3'}
    
    raw_list=[list1, tuple1, set1]
    trans_functions=[list, tuple, set, bytes, bytearray]
    
    for raw in raw_list:
    for trans in trans_functions:
    try:
       print(f"{trans.__name__}({raw}) => {trans(raw)}")
    except TypeError as err:
       print(f"{trans.__name__}({raw}) | {err}") 
    ```

3. `dict`与`list`、`set`、`tuple`、`bytes`、`bytearray`的**转换规则**：

    字典在使用`list()`、`set()`、`tuple()`转换为相应类型时，是将字典值数组作为可遍历序列进行转换的。使用`bytes()`、`bytearray()`进行转换时，值类型也要满足上小节提到的要求。

    其他可遍历类型若想转化为`dict`，可以自己通过遍历实现或者采用使用`dict`的静态方法 `fromkeys(iterable, value=None)` 实现。  

4. `bool()`还支持转换其他数据类型为`bool`。

5. `str`类型可以被看作字符数组与`list`、`tuple`、`set`进行类型转换。

6. 如果需要我们的类支持向这些基本数据类型进行转换，需要支持编写相应的魔术方法。目前支持的魔术方法有：`__bool__()`、`__int__()`、`__float__()`、`__complex__()`等。魔术方法详见下一节[面向对象编程](#面向对象编程)。

## 面向对象编程

Python是一门面向对象的语言，所有基本数据类型对应的类都可以在`builtins`中找到。在本节中我们将从以下方面了解Python的面向对象编程：

### 1.`class`基础

我们使用`class`关键字定义一个类，以下是定义类时需注意的事项：

#### 🧬 属性与方法

* 关于类的属性与方法有如下实现细节：

    * 私有字段：我们可以使用`__`作为属性名或者方法名的前缀，这样定义的方法就不能通过类的实例对象访问。

    * **静态属性**：当我们用`ClassName.static_attr`调用属性时，得到的就是该类型的静态属性。静态属性不能通过实例对象进行修改，只能直接修改。

    * **静态方法**：我们可以使用 `@staticmethod` 装饰器来装饰类的 <u>实例方法</u> ，装饰后的方法应当没有入参`self`，我们可以用`ClassName.static_func()`来调用静态方法。

    * <span style='color:cyan'>**抽象类方法**</span>：我们可以使用 `@abstractmethod` 装饰器来装饰类的 <u>抽象方法</u> ，在定义类的抽象方法时，我们只需要给出方法的入参即可，方法的函数体与返回值使用`pass`代替。当有子类继承了该类，需要实现该抽象方法。

        ```python
        class Test:
            @abstractmethod
            def __abstract_func(self):
                pass
        
        class SubTest:
            def __abstract_func(self):
                print("Writing implement here")
        ```

    * <span style='color:cyan'>**类方法**</span>：我们可以使用 `@classmethod` 装饰器来装饰类的 <u>类方法</u>。

        与实例方法不同，类的类方法用于处理类本身即`cls`对象，我们可以通过`cls`对象访问类中的静态方法或者属性，需要注意的是类的静态属性与静态方法实际上就是类的元类实例即`cls`对象的实例方法与属性。

        ```python
        class Test:
            @classmethod
            def class_func(cls, *args, **kwargs):
                pass
        ```

    * <span style='color:cyan'>**属性代理方法**</span>：我们可以使用 `@property`、`@xxx.getter`、`@xxx.setter`、`@xxx.deleter` 来代理类实例的属性或者私有属性。

        需要注意的是 `@property`、`@xxx.getter` 实现的效果都是代理属性的获取功能，但是为了代码的可读性考虑，当我们需要对代理属性进行 **删、改、查** 操作时优先考虑后三者，当我们只需要 **查** 代理属性时优先考虑 `@property` 。

        对于 `@property` 而言，`def property():` 是它实际的函数形式。我们除了在类定义时使用装饰器语法，还可以在元类中为类的方法添加 `@property` 。

        ```python
        class ImmutableMeta(type):
            def __new__(cls, name, bases, dct):
                for key, value in dct.items():
                    print(key, value)
                    if not key.startswith('__'):
                        dct[key] = property(value)
        
                return super().__new__(cls, name, bases, dct)
        
        class MyClass(metaclass=ImmutableMeta):
            def __init__(self, value):
                self._value = value
        
            def get_value(self):
                return self._value
        
        obj = MyClass(10)
        print(obj.get_value) 
        ```

#### <span style='color:cyan'>👨‍👩‍👦 类的继承</span>

Python实现了 **多继承** ，我们在编写类时支持通过`class ClassName(FatherClass1, FatherClass2)`的方式继承父类的属性与方法。Python的继承的**注意事项**有：

* 子类实例对象访问父类的属性或者方法时，Python将 **<span style='color:cyan'>*从左往右*</span>** 依次查找父类中是否有对应方法。
* 子类可以对父类的方法进行重写，`super()`是Python提供的用于子类调用父类属性或方法的一个方法。
* 我们使用`super()`在子类中访问父类属性或方法时，不能访问父类的私有属性或方法。可以被看做在使用父类的一个实例对象。

### 2.`class`进阶

Python的所有类型都是由`builtins`的`object`类型继承而来，这意味着我们可以在所有类型中使用`object`的方法（`object`中没有属性）。我们执行`help(object)`得到`object`的内置方法如下：

```shell
Help on class object in module builtins:

class object
 |  ...
 |
 |  Methods defined here:
 |  
 |  __delattr__(self, name, /)		Implement delattr(self, name).
 |  __dir__(self, /)			Default dir() implementation.
 |  __eq__(self, value, /)		Return self==value.
 |  __format__(self, format_spec, /)	Default object formatter.
 |  __ge__(self, value, /)		Return self>=value.
 |  __getattribute__(self, name, /)	Return getattr(self, name).
 |  __gt__(self, value, /)		Return self>value.
 |  __hash__(self, /)			Return hash(self).
 |  __init__(self, /, *args, **kwargs)	Initialize self.  See help(type(self)).
 |  __le__(self, value, /)		Return self<=value.
 |  __lt__(self, value, /)		Return self<value.
 |  __ne__(self, value, /)		Return self!=value.
 |  __reduce__(self, /)			Helper for pickle.
 |  __reduce_ex__(self, protocol, /)	Helper for pickle.
 |  __repr__(self, /)			Return repr(self).
 |  __setattr__(self, name, value, /)	Implement setattr(self, name, value).
 |  __sizeof__(self, /)			Size of object in memory, in bytes.
 |  __str__(self, /)			Return str(self).
 |
 |  ----------------------------------------------------------------------
 |  Class methods defined here:
 |  
 |  __init_subclass__(...) from builtins.type
 |      This method is called when a class is subclassed.
 |      
 |      The default implementation does nothing. It may be
 |      overridden to extend subclasses.
 |  
 |  __subclasshook__(...) from builtins.type
 |      Abstract classes can override this to customize issubclass().
 |      
 |      This is invoked early on by abc.ABCMeta.__subclasscheck__().
 |      It should return True, False or NotImplemented.  If it returns
 |      NotImplemented, the normal algorithm is used.  Otherwise, it
 |      overrides the normal algorithm (and the outcome is cached).
 |  
 |  ----------------------------------------------------------------------
 |  Static methods defined here:
 |  
 |  __new__(*args, **kwargs) from builtins.type
 |      Create and return a new object.  See help(type) for accurate signature.
 |  
 |  ----------------------------------------------------------------------
 |  Data and other attributes defined here:
 |  
 |  __class__ = <class 'type'>
 |      type(object) -> the object's type
 |      type(name, bases, dict, **kwds) -> a new type
```

其中形如`__xxx__()`、`__xxx__`的属性或者方法我们称之为 <span style='color:cyan'>*魔术属性*</span> 或者 **<span style='color:cyan'>*魔术方法*</span>** 。在编程语言中，我们常用 **魔术数字** | `magic number` 指代那些在直接硬编码的数字，它们通常没有一个明确的名称或含义。Python中的魔术字段也是硬编码在Python基础库或者解释器里面，它们各自服务于Python类型实例生命周期的不同片段。需要说明的一点是，

<u>Python会因不同的`PEP(Python Enhancement Proposal)`会不断添加删除魔术方法</u>，故下面分类列出的魔术方法具有**时效性**，请在使用时查看相应Python文档或者使用`help()`工具。魔术方法按功能可分为如下几类：

#### 🛠️ 创建实例

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `__init__(self, *args, **kwargs)`                            | 用于初始化对象，在使用`ClassName()`时自动调用。这里的`self`对象是由`__new__()`创建的 |
| <span style='color:cyan'>`__new__(cls, *args, **kwargs)`</span> | 在`__init__()`之前调用，用于创建`self`对象。`cls` 代表类本身 |
| <span style='color:silver'>`__del__(self)`</span>            | 在对象被垃圾回收时执行特定的操作，如关闭`socket`，文件描述符等。因为当Python解释器退出时，对象有可能依然未被释放，因此`__del__()`中定义册操作不一定能被执行，故在实际中应避免使用`__del__()` |

#### 🎮 实例属性

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <span style='color:cyan'>`__getattr__(self, item)`</span> | 拦截访问实例不存在的属性，即在访问不存在实例的`__dict__`中的`key`时，进行调用拦截 |
| `__getattribute__(self, item)`                               | 拦截所有的实例属性访问操作，应当在函数体内避免使用`self.key`这会造成死循环 |
| `__setattr__(self, key, value)`                              | 拦截所有的实例属性赋值操作，应当在函数体内避免使用`self.key=value`这会造成死循环 |
| `__delattr__(self, item)`                                    | 拦截所有的实例属性清除操作，应当在函数体内避免使用`del self.key`这会造成死循环 |

#### 🧭 描述实例

* `__str__(self)`：Python 的`toString()`，使类型实例支持`print()`。

* <span style='color:cyan'>`__repr__(self)`</span>：使类型实例支持`repr()`，`repr()` 方法用于获取对象的详细信息包括存储地址等。`__str__(self)` 中的信息一般是`__repr__()`的一部分

* <span style='color:cyan'>`__format__(self, format_spec)`</span>：使类型实例支持 `“{}”.format(obj, key)` 或者 `f"{obj}"`、`f"{obj:key}"` 。

    ```python
    class EnableFormat:
        value=None
        default='default'
        def __init__(self, value):
            self.value = value
    
        def __format__(self, format_spec):
            raw_out=''
            if format_spec == 'value':
                raw_out=self.value
            elif format_spec == 'default':
                raw_out=self.default
            elif format_spec == '':
                raw_out=f"EnableFormat(value={self.value}, \
                	default={self.default})"
            else:
                raw_out=f"EnableFormat(value={self.value}, \
                	default={self.default})"
    
            return raw_out
    
    test=EnableFormat('enter')
    print(f"{test}")
    print("{}".format(test))
    print(f"{test:value}")
    print("{:default}".format(test))
    
    # # Result:
    # # EnableFormat(value=enter, default=default)
    # # enter
    # # EnableFormat(value=enter, default=default)
    # # default
    ```

* `__hash__(self)`：用于 *Hashable* 类型定义，这前面已经介绍过了。

* `__dir__(self)`：调用`dir()`方法时对象的返回值，一般不需要重写该方法，但在定义`__getattr__`时有可能需要重写，以打印动态添加的属性。

* <span style='color:silver'>`__sizeof__(self)`</span>：定义`sys.getsizeof()`方法调用时的返回，指类的实例的大小，单位是字节，在使用 C 扩展编写`Python`类时比较有用。

#### 🧪 实例间计算

在这里主要介绍关于 ***比较计数*** 的魔术方法：

| 方法                | 说明               |
| ------------------- | ------------------ |
| `__eq__(self, obj)` | 使类型实例支持`==` |
| `__ne__(self, obj)` | 使类型实例支持`!=` |
| `__le__(self, obj)` | 使类型实例支持`<=` |
| `__ge__(self, obj)` | 使类型实例支持`>=` |
| `__lt__(self, obj)` | 使类型实例支持`<`  |
| `__gt__(self, obj)` | 使类型实例支持`>`  |

此外还有许多计算相关的魔术方法，在这里就不一一列举了（因为用不到）。这里的`l`、`g`、`eq`分别是`less`、`greater`、`equal`的缩写，`le`、`lt`则是`less equal`、`less than`的缩写。

#### 🧷 序列化实例

定义一个不可变容器，只需要实现`__len__`和`__getitem__`，可变容器在此基础上还需实现`__setitem__`和`__delitem__`，如果希望容器是可迭代的，还需实现`__iter__`方法。

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <span style='color:silver'># *不可变容器魔术方法* ：</span>  |                                                              |
| `__len__(self)`                                              | 返回序列的长度，使类型实例支持`len(instance)`                |
| <span style='color:silver'># *可变容器魔术方法* ：</span>    |                                                              |
| `__getitem__(self, key)`                                     | 使类型实例支持`self[key]`                                    |
| `__setitem__(self, key, value)`                              | 使类型实例支持`self[key]=value`                              |
| `__delitem__(self, key)`                                     | 使类型实例支持`del self[key]`                                |
| <span style='color:silver'># *可迭代容器魔术方法* ：</span>  |                                                              |
| `__iter__(self)`                                             | 使类型实例支持`for item in self`。通常执行`__iter__()`返回的实例对象需要实现`__next__()`方法`__next__()`方法用于获取迭代器的下一个元素 |
| `__reversed__(self)`                                         | 使类型实例支持`reversed(instance)`                           |
| `__contains__(self, val)`                                    | 使类型实例支持`val in instance`                              |
| <span style='color:silver'># *字典子类魔术方法* ：</span>    |                                                              |
| <span style='color:cyan'>`__missing__(self)`</span>：。 | 当定制序列是 **<u>`dict`的子类</u>** 时，使用`dict[key]`读取元素而`key`没有定义时调用 |

```python
class MyDict(dict):
    def __missing__(self, key):
        return f"Key '{key}' not found."

my_dict = MyDict(a=1)
print(my_dict['a'])  # 1
print(my_dict['b'])  # Key 'b' not found.
```

#### 🧼 可调用化实例

`__call__(self, *args, **kwargs)`：使类型 <u>实例像函数一样调用</u> 。需要注意的是当我们在元类中书写该方法时有些许不同，关于元类的详见下一小节 <u>`type` 与元类</u> 。

```python
class FunctionGenerator:
    def __call__(self, s):
        return s**2;

func=FunctionGenerator()
print(func(2))
```

#### ♻️ 复制实例

`copy`模块是Python的基础库之一。其中，`copy.copy()`是浅复制，常用于 <u>*不可变数据*</u> 的复制，当对 <u>*可变数据类型*</u> 进行浅复制时，只是复制了对象本身与对象包含的引用，不是复制了指向的对象。`copy.deepcopy()`是深复制，它会递归复制需要复制的对象。

* `__copy__(self)`：执行`copy.copy()`时调用。

* `__deepcopy__(self, memodict={})`：执行`copy.deepcopy()`时调用。`memodict`用于缓存目前已经复制的对象，以下是参考代码：

    ```python
    class Copy:
        def __init__(self, value):
        	self.value=value
    
        def __deepcopy(self, memodict={}):
            # Copy part code, base on your class data structure
            new=Copy(self.value)
            memodict[id(new)]=new
    
            return new
    ```

#### 🪜 上下文管理

Python 中使用`with`语句执行上下文管理，处理内存块被创建时和内存块执行结束时上下文应该执行的操作，上下文即程序执行操作的环境，包括异常处理，文件开闭。有两个魔术方法负责处理上下文管理。

* <span style='color:cyan'>`__enter__(self)`</span>：上下文创建时执行的操作，该方法返回`as`后面的对象。

* <span style='color:cyan'>`__exit__(self, exc_type, exc_val, exc_tb)`</span>：上下文结束时执行的操作。

    ```python
    class Closer:
        def __init__(self, obj):
            self.obj = obj
    
        def __enter__(self):
            print('__enter__ called')
            return self.obj # bound to target
    
        def __exit__(self, exception_type, exception_val, trace):
            print('__exit__ called')
            try:
               self.obj.close()
            except AttributeError:
               print('Not closable. here')
               return True
    
    with Closer(open("1.txt", "w")) as f:
        f.write("1") 
    ```


#### 🧱 描述器

描述器更像是`JavaScript`中的`Proxy`，我们直接可以直接对描述器实例对象就行取值、赋值、删除操作，而这些操作均会被`__get__()`、`__set__()`、`__delete__()`拦截并代理。与`JavaScript`不同的是，描述器类型的实例只有在被**赋值给属性**后，才能进行拦截。

* <span style='color:cyan'>`__get__(self, instance, owner)`</span>：查找描述器属性时调用。

* <span style='color:cyan'>`__set__(self, instance, value)`</span>：给描述器属性赋值时调用。

* <span style='color:cyan'>`__delete__(self, instance)`</span>：移除描述器属性时调用。

    ```python
    class AttrProxy:
        value=None
        def __get__(self, instance, owner):
            print("in __get__()")
            return self.value
    
        def __set__(self, instance, value):
            print("in __set__()")
            self.value = value
    
    class Attach:
        data=AttrProxy()
        def __init__(self, value):
            self.data = value
    
        def print(self):
            print(self.data)
    
    test=Attach(10)
    # in __set__()
    test.print()
    # in __get__()
    # 10
    print(test.data)
    # in __get__()
    # 10
    ```

#### 🧨 `super()`

Python 提供的内置函数中有两个是与面向对象编程密切相关的：一个是`staticmethod()`装饰器，它 <u>用于声明类的静态方法</u> ；另一个是`super()`，它用于在子类中 <u>调用父类的方法与属性</u> 。除了我们常用的`super().func()`的调用方法，`super()`还有其他调用方式。我们使用`help(super)`来查看Python给我们提供的解释：

   ```
class super(object)
 |  super() -> same as super(__class__, <first argument>)
 |  super(type) -> unbound super object
 |  super(type, obj) -> bound super object; requires isinstance(obj, type)
 |  super(type, type2) -> bound super object; requires issubclass(type2, type)
 |  ...
   ```

对于`super`我们的关注点主要是在`super()`构造函数。观察上述运行结果，我们发现，`super()`总是接受 `super(type, other)` 的参数形式，这里的`type`实际上就是指`type`元类的实例对象即类。

   * `super()`：该方式调用的同等效果参考上述描述。在一般情况下第一个参数有两种`cls`、`self`。

   * `super(type)`：使用该方式调用会返回一个未绑定的`type`实例。

     注意，由于返回的实例对象未绑定到`self`或者`cls`，对这返回对象的处理不会影响到`self`。
     
     ```python
     class Base:
     def hello(self):
       print("Hello from Base")
     
     class Derived(Base):
     def hello(self):
       super(Base).hello(self)  # Hello from Base
     
     derived = Derived()
     derived.hello()
     ```

   * `super(type, obj)`：使用该方法调用会将返回的`super`实例对象绑定到`obj`上，这是我们一般调用方式的实际调用方式。

   * <span style='color:silver'>`super(type1, type2)`</span>：这个涉及到了Python的 **方法解析顺序**\| ``MRO`` ，后续会开专题解析。

### 3. `type`与元类

Python是一门面向对象的语言，几乎所有的一切都是基于类实现的，连类本身也不例外。当我们执行以下代码便可以得知用`class`声明的类本身也是`type`类的实例。

```python
class MyClass:
    pass
ins=MyClass()

print(type(ins))        # <class '__main__.MyClass'>
print(type(MyClass))    # <class 'type'>
```

我们将用于创建一般类的类称为 **元类**\| ``metaclass``：

* **静态字段**：普通类的静态字段实际上就是使用元类生成的元类实例的实例方法与属性。
* **默认元类 `type`** ：在定义类时使用`class MyClass(*args, metaclass=MyMetaClass)`为指定元类。如果没有指定元类，Python则会指定默认元类`type`作为该类的元类。Python的内置类的元类都是`type`。

为了更好的了解元类，我们可以详细学习下`type()`方法。我们执行`help(type)`有如下结果：

```shell
class type(object)
 |  type(object) -> the object's type
 |  type(name, bases, dict, **kwds) -> a new type
 |  
 |  Methods defined here:
 |
 |  __call__(self, /, *args, **kwargs)		Call self as a function
 |  __delattr__(self, name, /)			Implement delattr(self, name)
 |  __dir__(self, /)				Specialized __dir__ 
 | 							implementation for types
 |  __getattribute__(self, name, /)		Return getattr(self, name)
 |  __init__(self, /, *args, **kwargs)		Initialize self.  See 
 |							help(type(self)) for 
 | 							accurate signature
 |  __instancecheck__(self, instance, /)	Check if an object is 
 | 							an instance
 |  __repr__(self, /)Return repr(self)
 |  __setattr__(self, name, value, /)		Implement setattr(self, 
 | 							name, value)
 |  __sizeof__(self, /)				Return memory consumption
 | 							of the type object
 |  __subclasscheck__(self, subclass, /)	Check if a class is a subclass
 |  __subclasses__(self, /)			Return a list of immediate 
 | 							subclasses
 |  mro(self, /)				Return a type's method 
 | 							resolution order
 |  ----------------------------------------------------------------------
 |  Class methods defined here:
 |  
 |  __prepare__(...)
 |      __prepare__() -> dict
 |      used to create the namespace for the class statement
 |  
 |  ----------------------------------------------------------------------
 |  Static methods defined here:
 |  
 |  __new__(*args, **kwargs)
 |      Create and return a new object.  See help(type) for accurate signature.
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __abstractmethods__
 |  __dict__
 |  __text_signature__
 |  
 |  ----------------------------------------------------------------------
 |  Data and other attributes defined here:
 |  
 |  __base__ = <class 'object'>
 |      The base class of the class hierarchy.
 |      
 |      When called, it accepts no arguments and returns a new featureless
 |      instance that has no instance attributes and cannot be given any.
 |  
 |  __bases__ = (<class 'object'>,)
 |  __basicsize__ = 880
 |  __dictoffset__ = 264
 |  __flags__ = 2148293632
 |  __itemsize__ = 40
 |  __mro__ = (<class 'type'>, <class 'object'>)
 |  __weakrefoffset__ = 368
```

我们主要关注`type`的构造函数与`type`类型实例的实例方法，即类的自带静态方法。

* **`type()`**：根据上述描述可知，该方法的的用法除了用于判断一个实例的类，还可以用`type(name, bases, dct)`创建一个类。
* **实例方法**：类的自带静态方法实际上就是类的**魔术方法**，我们可以通过重写类的魔术方法为更好的处理类实例的生命周期以及使用。

我们也可以定制自己的元类。我们在创建类时继承`type`便创建了一个元类。元类中的某些**魔术方法**实现方式也与普通类不一样。

* **`__new__(cls, name, bases, dict)`**：用于创建元类的实例对象即元类为该类的类。需要注意的是，这里的`cls`是实际上的类对象。这个`cls`无论是普通类还是元类都是这样的。

    ```python
    class MetaClass(type):
    def __new__(cls, name, bases, dct):
    print(cls)				# <class '__main__.MetaClass'>
    return super().__new__(cls, name, bases, dict)
    
    class MyClass(metaclass=MetaClass):
    pass
    ```
    
    下面我们解释一下`name`、`bases`、`dct`分别指什么：

    * `name`：使用该元类创建类时的类名。
    * `bases`：使用该元类创建类时指定的父类。
    * `dct`：使用字典表示用该元类创建的类的主体。
    
    ```python
    class MetaClass(type):
        def __new__(cls, *args):
            name, bases, dct=args
            print(f"Create class: {name}")
            print(f"Extends class: {bases}")
            print(dct)
            return super().__new__(cls, name, bases, dct)
    
    class MyClass(metaclass=MetaClass):
    	def __init__(self):
    		pass
    ```
    
* **`__init__(cls, name, bases, dict)`**：用于初始化元类实例对象。

* **`__call__(cls, *args, **kwargs)`**：`__call__`在普通类中是用于类实例的可调用化，即用使用函数的方式使用类实例。在这里是用于元类实例的可调用化，即处理类的构造函数。

    ```python
    class MetaClass(type):
    def __call__(self, *args, **kwargs):
      print('In meta __call__')
    
    class MyClass(metaclass=MetaClass):
    def __init__(self):
      pass
    
    test=MyClass()      # In meta __call__
    ```
    
    
