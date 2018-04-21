# python学习记录

## 数据类型

python中的数据类型除了基本的数据类型之外还有list、tuple、dict和set
```python
list = [1, 2, 3, 4]
tuple = (1, 2, 3, 4)
dict = {'name':bob, 'score':85}
set = set([1, 2, 3, 4])
```

与其相关的特性：
- list
    - list是一个有序的集合
    - list中可以存储不同类型的数据
    - 可以使用-1来直接索引到倒数第一个元素
    - `list.append('element')`在末尾追加元素
    - `list.insert(index, 'element')`在指定位置插入元素
    - `list.pop()`删除末尾的元素
    - `list.pop(index)`删除指定位置的元素
    - `list[index] = 'element'`将指定位置的元素替换

- tuple
    - tuple是一个有序的集合
    - tuple一旦初始化就不能修改（tuple中的元素指向不会改变，指向一个list的话，list本身是可以变化的）
    - 可以使用 -1 来直接索引到倒数第一个元素
    - 通过`tuple = (element, )`来创建只有一个元素的tuple

- dict
    - dict是一个无序并且key值不重复的集合
    - 作为key的对象必须是不可变对象
    - 通过`'element' in dict`判断元素是否在dict里面
    - `dict.get(key, value)`获取key对应的值，如果不存在该值则返回value或者None（第二个参数value是可选的）
    - `dict.pop(key)`删除相应的key和对应的value

- set
    - set是一个无序且不重复的集合
    - `set.add('element')` 添加元素
    - `set.remove('element')`删除元素
    - 两个set可以做并和或操作，并即两个set中的所有元素且重复的元素只留一份，或即两个set都有的元素

数据类型转换格式： `int(Str)`



## 函数

函数的定义
```python
def functionName(parameter):
     functionBody
```

定义一个空函数
```python
#pass是一个占位符，在没想好怎么写的地方可以先放一个pass让代码先跑起来
def voidFunction(parameter):
    pass 
```

参数检查的方法
```python
if not isinstance(x, (int, float)):
    raise TypeError('bad parameter type')
```

返回多个值
```python
#函数返回多个值其实是返回一个tuple
def functionName(parameter):
    ...
    return x, y

a, b = functionName(parameter)
```

python函数的参数有：
- 位置参数
- 默认参数
- 可变参数
- 关键字参数
- 命名关键字参数
```python
def functionName(parameter1, parameter2) 位置参数
def functionName(parameter1, parameter2 = value) 默认参数
def functionName(*parameters) 可变参数
def functionName(**parameters) 关键字参数
def functionName(*, parameter1, parameter2) 命名关键字参数
def functionName(*parameters, parameter1, parameter2) 命名关键字参数（写在可变参数后面的）
```
位置参数没什么说的，就是普通的参数，下面说说默认参数、可变参数、关键字参数和命名关键字参数的一些特性:
- 默认参数
    - 默认参数的位置一定在位置参数的后面
    - 默认参数必须指向不变的对象
- 可变参数
    - 可变参数接收的是一个tuple
    - 传值的时候可以直接写多个参数，也可以传list和tuple但是前面要加*
- 关键字参数
    - 关键字参数接收的是一个dict
    - 传值的时候可以直接写多个键值对，也可以传dict但是前面要加**
- 命名关键字参数
    - 在关键字参数的基础上限制了关键字参数的参数名
    - 可以设定默认值，简化调用
    - 传值和关键字参数差不多，但是如果没有传相应的命名关键字参数且该参数没有默认值时会报错


## 高级特性

切片
```python
List[0:10] 取List中的第0位到第9位
List[10:] 取List中的第10位到结束
List[:10] 取List中的第0位到第9位
List[-10:] 取list中的倒数10位
List[:10:2] 前十位隔2个取1个
List[::5] 所有的元素隔5个取1个
List[:] 全部
```

迭代
迭代其实就是foreach循环。主要讲讲对于dict迭代的是key，可以用`for value in dict.values()`迭代value，还可以用`for key, value in dict.item()`迭代key和value
python内置的enumerate函数可以把一个list变成索引-元素对，例如`for i, value in enumerate(['a', 'b', 'c'])`

列表生成式
```python
[x * x for x in range(100) if x % 2 == 0]
[x + y for x in ['XYZ'] for y in 'ABC']
```

生成器
生成器中保存的是一种算法，生成所需数据的算法，是一种惰性序列。在生成器中可存放全体自然数等无线序列。获取生成器的值可通过next()函数和迭代。创建一个生成器和创建一个列表生成式差不多，只需把[]换为()即可
```python
(x * x for x in range(100) if x % 2 == 0)
(x + y for x in ['XYZ'] for y in 'ABC')
```
创建生成器还有另外一种方法，就是把普通的函数的return语句修改为yield语句即可，但需要注意的是，在执行这种生成的时候是遇到yield就停止，下次执行时从前一次退出的yield语句往下执行

迭代器
上文说道生成器可通过next()函数获取下一个值，那我们就把生成器叫做一个迭代器。可以通过next()函数不断返回下一个值的对象就叫做迭代器。list、tuple、str等数据可通过iter()函数。判断一个对象是否属于迭代器可使用isinstance()函数
*注：python的for循环其实也是通过next()函数实现的*



## 高阶函数

因为变量可以指向函数，所以呢，函数就可以接受另一个函数作为参数，那么这个函数就变成了高阶函数
python内建的高阶函数有map和reduce函数，我们一个个的介绍

map函数
用法：map(f, [1,2,3,4,5,6])
第一个参数是一个函数，map高阶函数的作用就是让f函数作用于后面的集合的每一个元素然后生成一个新的集合

reduce函数
用法：reduce(f, [1,2,3,4,5,6])
第一个参数和map一样是一个函数，但是不同的是这个函数必须要接受两个参数。reduce让这个函数计算前两个元素后得出一个结果又继续和第三个元素计算，以此类推一直到完成。

filter函数
用法：filter(f, [1,2,3,4,5,6])
和map一样，filter也是依次作用于序列的每一个元素。不同的是filter是根据f函数返回的结果来决定是保留还是丢弃这个该元素。



