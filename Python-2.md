# Python 面试题答案（完整版）

---

## 4. 面向对象编程

### 简答题答案

**1. 解释面向对象的三大特性：封装、继承、多态。**

**答案：**

**封装（Encapsulation）：**
- 将数据（属性）和操作数据的方法（函数）捆绑在一起
- 隐藏内部实现细节，只暴露必要的接口
- 通过访问控制（公有、私有、保护）实现
- 目的：提高代码安全性，减少耦合

```python
class Person:
    def __init__(self, name):
        self._name = name  # 保护属性
    
    def get_name(self):     # 公开接口
        return self._name
```

**继承（Inheritance）：**
- 子类继承父类的属性和方法
- 实现代码复用，建立类层次关系
- 支持单继承和多重继承
- 子类可以重写（override）父类方法

```python
class Animal:
    def speak(self):
        pass

class Dog(Animal):  # 继承Animal
    def speak(self):  # 重写
        return "Woof!"
```

**多态（Polymorphism）：**
- 同一操作作用于不同对象会产生不同的结果
- 通过方法重写和鸭子类型实现
- 提高代码的灵活性和可扩展性

```python
def make_speak(animal):
    print(animal.speak())  # 不同的animal调用speak有不同结果

make_speak(Dog())    # Woof!
make_speak(Cat())    # Meow!
```

---

**2. Python中的`__init__`和`__new__`方法有什么区别？**

**答案：**

| 特性 | `__new__` | `__init__` |
|------|-----------|------------|
| 调用时机 | 对象创建之前 | 对象创建之后 |
| 作用 | 创建并返回实例对象 | 初始化实例对象的属性 |
| 第一个参数 | `cls`（类本身） | `self`（实例本身） |
| 返回值 | 必须返回一个实例 | 无返回值（或None） |
| 自定义 | 很少需要重写 | 几乎每个类都需要 |

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("Creating instance")
        instance = super().__new__(cls)  # 调用父类的__new__创建实例
        return instance
    
    def __init__(self, value):
        print("Initializing instance")
        self.value = value

obj = MyClass(10)
# 输出:
# Creating instance
# Initializing instance
```

**`__new__`的使用场景：**
- 实现单例模式
- 创建不可变类型的子类（如str, int）
- 元编程

---

**3. 什么是Python中的元类（metaclass）？它的作用是什么？**

**答案：**

**元类**是创建类的"类"。在Python中，类本身也是对象，元类就是创建这些类对象的类。

**Python类的创建过程：**
```
type(name, bases, dict) → 类对象 → 实例对象
```

**默认元类是`type`：**
```python
# 常规方式定义类
class MyClass:
    x = 1

# 等价于用type创建
MyClass = type('MyClass', (), {'x': 1})
```

**自定义元类：**
```python
class MyMeta(type):
    def __new__(cls, name, bases, attrs):
        # 在创建类时添加额外属性
        attrs['class_id'] = name.lower()
        return super().__new__(cls, name, bases, attrs)
    
    def __init__(cls, name, bases, attrs):
        # 类创建后的初始化
        print(f"Class {name} created")

# 使用元类
class MyClass(metaclass=MyMeta):
    pass

print(MyClass.class_id)  # myclass
```

**元类的作用：**
1. 类的自动注册（如ORM模型）
2. 自动添加方法或属性
3. 接口检查
4. 单例模式
5. 创建DSL（领域特定语言）

---

**4. 解释Python中的多重继承，以及MRO（方法解析顺序）。**

**答案：**

**多重继承**：一个类可以同时继承多个父类。

```python
class A: pass
class B: pass
class C(A, B): pass  # 同时继承A和B
```

**MRO（Method Resolution Order）**：决定在多重继承中，当调用一个方法时，按照什么顺序查找父类。

**Python的MRO算法：C3线性化**
- 子类优先于父类
- 如果有多个父类，按照在基类列表中的顺序检查
- 先检查第一个父类，然后递归检查它的父类，形成"深度优先"，但保持单调性

**查看MRO：**
```python
class A:
    def say(self):
        print("A")

class B(A):
    def say(self):
        print("B")

class C(A):
    def say(self):
        print("C")

class D(B, C):
    pass

print(D.__mro__)
# (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)

D().say()  # B （按照MRO顺序，找到第一个匹配的）
```

**菱形继承问题：**
MRO确保每个类只被访问一次，避免重复查找。

---

**5. 什么是抽象基类（ABC）？如何在Python中定义抽象类？**

**答案：**

**抽象基类（Abstract Base Class）** 是一种不能被实例化的类，用于定义接口规范，强制子类实现特定的方法。

**作用：**
- 定义接口契约
- 确保子类实现必要的方法
- 提供类型检查

**定义抽象基类：**
```python
from abc import ABC, abstractmethod

class Shape(ABC):  # 继承ABC
    
    @abstractmethod
    def area(self):
        """计算面积"""
        pass
    
    @abstractmethod
    def perimeter(self):
        """计算周长"""
        pass

# 抽象类不能实例化
# s = Shape()  # TypeError: Can't instantiate abstract class

# 子类必须实现所有抽象方法
class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return 3.14 * self.radius ** 2
    
    def perimeter(self):
        return 2 * 3.14 * self.radius

c = Circle(5)
print(c.area())  # 78.5
```

**抽象属性：**
```python
from abc import abstractproperty

class MyABC(ABC):
    @abstractproperty
    def value(self):
        pass
    
    # Python 3.3+ 可以这样写
    @property
    @abstractmethod
    def value(self):
        pass
```

---

### 代码题答案

**1. 解释下面代码中`self`的作用：**
```python
class MyClass:
    def __init__(self, value):
        self.value = value
    
    def get_value(self):
        return self.value
```

**答案：**

**`self`** 代表类的实例对象本身，用于访问实例的属性和方法。

**关键点：**
1. `self`是约定俗成的名字，可以改成其他名（但强烈不建议）
2. 它是实例方法的第一个参数
3. 调用方法时Python自动传入实例对象，不需要手动传
4. 通过`self`可以访问实例的属性和其他方法

```python
obj = MyClass(10)
# 等价于
MyClass.__init__(obj, 10)  # self就是obj

obj.get_value()
# 等价于
MyClass.get_value(obj)  # self就是obj
```

---

**2. 写出下面代码的输出结果：**
```python
class A:
    def say(self):
        print("A")

class B(A):
    def say(self):
        print("B")

class C(A):
    def say(self):
        print("C")

class D(B, C):
    pass

d = D()
d.say()
print(D.__mro__)
```

**答案：**

输出：
```
B
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
```

**解释：**
- D继承B和C，MRO顺序是 D → B → C → A → object
- 调用`d.say()`时，按照MRO顺序查找，首先在B中找到say方法，所以输出"B"

---

**3. 解释下面这些魔术方法的作用：**
   - `__str__` 和 `__repr__`
   - `__len__`
   - `__getitem__`、`__setitem__`、`__delitem__`
   - `__call__`

**答案：**

**`__str__(self)`**：返回对象的"用户友好"字符串表示
- `str(obj)`、`print(obj)` 时调用
- 目标是可读性

**`__repr__(self)`**：返回对象的"官方"字符串表示
- `repr(obj)`、交互式环境输入obj时调用
- 目标是明确性，通常应该能通过`eval(repr(obj))`重建对象

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):
        return f"{self.name}, {self.age} years old"
    
    def __repr__(self):
        return f"Person('{self.name}', {self.age})"

p = Person("Alice", 25)
print(str(p))   # Alice, 25 years old
print(repr(p))  # Person('Alice', 25)
```

**`__len__(self)`**：让对象支持`len()`函数
```python
class MyList:
    def __init__(self, data):
        self.data = data
    
    def __len__(self):
        return len(self.data)
```

**`__getitem__(self, key)`**：让对象支持下标访问 `obj[key]`
**`__setitem__(self, key, value)`**：让对象支持赋值 `obj[key] = value`
**`__delitem__(self, key)`**：让对象支持删除 `del obj[key]`

```python
class MyDict:
    def __init__(self):
        self.data = {}
    
    def __getitem__(self, key):
        return self.data[key]
    
    def __setitem__(self, key, value):
        self.data[key] = value
    
    def __delitem__(self, key):
        del self.data[key]
```

**`__call__(self, *args, **kwargs)`**：让对象可以像函数一样被调用
```python
class Adder:
    def __init__(self, n):
        self.n = n
    
    def __call__(self, x):
        return x + self.n

add5 = Adder(5)
print(add5(3))  # 8
```

---

### 编程题答案

**1. 实现一个单例模式的类（至少两种实现方式）。**

**答案：**

**方式1：使用`__new__`方法**
```python
class Singleton:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super().__new__(cls)
        return cls._instance

# 测试
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

**方式2：使用装饰器**
```python
def singleton(cls):
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class MyClass:
    pass

# 测试
c1 = MyClass()
c2 = MyClass()
print(c1 is c2)  # True
```

**方式3：使用元类**
```python
class SingletonMeta(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class MyClass(metaclass=SingletonMeta):
    pass
```

**方式4：使用模块（Pythonic方式）**
```python
# singleton.py
class Singleton:
    pass

instance = Singleton()  # 模块只被导入一次

# 使用
from singleton import instance
```

---

**2. 实现一个自定义的上下文管理器。**

**答案：**

**方式1：使用类实现`__enter__`和`__exit__`**
```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        """进入with语句时调用，返回值被as后的变量接收"""
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出with语句时调用
        exc_type: 异常类型，如果没有异常为None
        exc_val: 异常值
        exc_tb: 异常追踪栈
        返回True则异常被吞噬，返回False则异常继续传播
        """
        self.file.close()
        if exc_type is not None:
            print(f"An exception occurred: {exc_val}")
        # return True  # 取消注释可以吞噬异常

# 使用
with FileManager('test.txt', 'w') as f:
    f.write('Hello World!')
```

**方式2：使用contextmanager装饰器**
```python
from contextlib import contextmanager

@contextmanager
def file_manager(filename, mode):
    # __enter__ 部分
    f = open(filename, mode)
    try:
        yield f  # yield之前是__enter__，之后是__exit__
    finally:
        # __exit__ 部分
        f.close()

# 使用
with file_manager('test.txt', 'w') as f:
    f.write('Hello World!')
```

**其他示例：数据库连接上下文管理器**
```python
import sqlite3

class DBConnection:
    def __init__(self, db_name):
        self.db_name = db_name
        self.conn = None
        self.cursor = None
    
    def __enter__(self):
        self.conn = sqlite3.connect(self.db_name)
        self.cursor = self.conn.cursor()
        return self.cursor
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            self.conn.commit()
        else:
            self.conn.rollback()
        self.conn.close()

# 使用
with DBConnection('test.db') as cursor:
    cursor.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
```

---

**3. 设计一个简单的类继承体系，展示多态的使用。**

**答案：**
```python
from abc import ABC, abstractmethod
import math

# 抽象基类
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass
    
    def info(self):
        print(f"This is a {self.__class__.__name__}")
        print(f"Area: {self.area():.2f}")
        print(f"Perimeter: {self.perimeter():.2f}")

# 圆形
class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return math.pi * self.radius ** 2
    
    def perimeter(self):
        return 2 * math.pi * self.radius

# 矩形
class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

# 正方形
class Square(Rectangle):
    def __init__(self, side):
        super().__init__(side, side)

# 三角形
class Triangle(Shape):
    def __init__(self, a, b, c):
        self.a = a
        self.b = b
        self.c = c
    
    def area(self):
        # 海伦公式
        s = (self.a + self.b + self.c) / 2
        return math.sqrt(s * (s - self.a) * (s - self.b) * (s - self.c))
    
    def perimeter(self):
        return self.a + self.b + self.c

# 展示多态
def print_shape_info(shape):
    """统一的接口，接受任何Shape的子类"""
    shape.info()
    print("-" * 30)

# 使用多态
shapes = [
    Circle(5),
    Rectangle(4, 6),
    Square(5),
    Triangle(3, 4, 5)
]

for shape in shapes:
    print_shape_info(shape)  # 同一函数调用，不同结果
```

---

## 5. 异常处理

### 简答题答案

**1. 解释Python中的异常处理机制，以及try-except-else-finally的执行流程。**

**答案：**

**Python异常处理结构：**
```python
try:
    # 可能抛出异常的代码
    pass
except ValueError as e:
    # 捕获特定类型异常
    pass
except (TypeError, IndexError) as e:
    # 捕获多种异常
    pass
except Exception as e:
    # 捕获所有其他异常（不推荐放在最前面）
    pass
else:
    # 没有异常时执行
    pass
finally:
    # 无论是否有异常都会执行
    pass
```

**执行流程：**
1. 执行`try`块中的代码
2. 如果没有异常：
   - 跳过所有`except`块
   - 执行`else`块（如果有）
3. 如果发生异常：
   - 按顺序匹配`except`块，找到第一个匹配的
   - 执行匹配的`except`块
   - 不执行`else`块
4. 最后执行`finally`块（无论是否有异常）

**常见场景：**
- `try`: 业务逻辑
- `except`: 错误处理
- `else`: 成功时的逻辑
- `finally`: 资源清理（关闭文件、数据库连接等）

---

**2. Python中常见的内置异常类型有哪些？（至少列举5个）**

**答案：**

| 异常类型 | 说明 |
|----------|------|
| `SyntaxError` | 语法错误 |
| `IndentationError` | 缩进错误 |
| `NameError` | 变量未定义 |
| `TypeError` | 类型错误，如不支持的操作类型 |
| `ValueError` | 值错误，类型正确但值不合适 |
| `KeyError` | 字典中不存在的键 |
| `IndexError` | 序列下标越界 |
| `AttributeError` | 对象没有该属性 |
| `ZeroDivisionError` | 除以零 |
| `IOError` / `OSError` | IO操作错误（Python 3.3+合并为OSError） |
| `FileNotFoundError` | 文件不存在 |
| `ImportError` | 导入模块失败 |
| `RuntimeError` | 运行时错误 |
| `NotImplementedError` | 方法未实现 |
| `AssertionError` | assert语句失败 |

**异常继承层次（部分）：**
```
BaseException
├── Exception
│   ├── ArithmeticError
│   │   └── ZeroDivisionError
│   ├── LookupError
│   │   ├── IndexError
│   │   └── KeyError
│   ├── NameError
│   ├── TypeError
│   ├── ValueError
│   └── OSError
│       └── FileNotFoundError
├── SystemExit
├── KeyboardInterrupt
└── GeneratorExit
```

---

**3. 如何自定义异常类？什么时候需要自定义异常？**

**答案：**

**自定义异常类：**
```python
# 最简单的自定义异常
class MyError(Exception):
    pass

# 带自定义信息的异常
class ValidationError(Exception):
    def __init__(self, message, field=None, value=None):
        super().__init__(message)
        self.field = field
        self.value = value
    
    def __str__(self):
        base_msg = super().__str__()
        if self.field:
            return f"Field '{self.field}': {base_msg} (value: {self.value})"
        return base_msg

# 使用
raise ValidationError("Invalid email format", field="email", value="invalid")
```

**需要自定义异常的场景：**

1. **特定业务逻辑错误**：
```python
class InsufficientFundsError(Exception):
    """账户余额不足"""
    pass

def withdraw(account, amount):
    if account.balance < amount:
        raise InsufficientFundsError(f"Need {amount}, but only {account.balance} available")
```

2. **区分不同的错误类型**：便于调用方精确捕获处理

3. **携带额外的错误信息**：错误码、HTTP状态码等

4. **创建异常层次结构**：
```python
class APIError(Exception):
    """基础API异常"""
    pass

class AuthenticationError(APIError):
    """认证失败"""
    status_code = 401

class PermissionDeniedError(APIError):
    """权限不足"""
    status_code = 403
```

---

**4. 解释异常捕获的顺序原则，为什么要先捕获更具体的异常？**

**答案：**

**捕获顺序原则：** 先捕获更具体（子类）的异常，再捕获更通用（父类）的异常。

**错误示例（会导致第二个except永远不会执行）：**
```python
try:
    x = 1 / 0
except Exception:  # 父类异常放在前面
    print("Exception caught")
except ZeroDivisionError:  # 永远不会执行！
    print("ZeroDivisionError caught")
```

**正确示例：**
```python
try:
    x = int("invalid")
except ValueError:
    print("ValueError: cannot convert to int")
except TypeError:
    print("TypeError: wrong type")
except Exception:
    print("Some other error occurred")
```

**原因：**
- Python按`except`出现的顺序依次匹配异常类型
- 如果父类异常在前，它会匹配所有子类异常，导致后面的子类`except`永远不会执行
- 从最具体到最通用的顺序可以确保精确的错误处理

**最佳实践：**
1. 只捕获你能处理的异常
2. 尽量捕获具体的异常类型，避免裸`except:`
3. 最后可以用`except Exception`作为兜底（但也要谨慎）
4. 不要捕获`BaseException`（会捕获SystemExit、KeyboardInterrupt等）

---

**5. 什么是异常链？如何在Python中使用`raise ... from ...`？**

**答案：**

**异常链**：当一个异常在处理另一个异常的过程中被抛出时，Python会记录这两个异常之间的关系。

**隐式异常链（`__context__`）：**
```python
try:
    1 / 0
except ZeroDivisionError:
    raise ValueError("Division failed")  # 隐式关联原始异常

# 输出会显示:
# ZeroDivisionError: division by zero
# During handling of the above exception, another exception occurred:
# ValueError: Division failed
```

**显式异常链（`__cause__`），使用`raise ... from ...`：**
```python
try:
    1 / 0
except ZeroDivisionError as e:
    raise ValueError("Division failed") from e  # 显式设置原因

# 输出会显示:
# ZeroDivisionError: division by zero
# The above exception was the direct cause of the following exception:
# ValueError: Division failed
```

**抑制异常链（`from None`）：**
```python
try:
    1 / 0
except ZeroDivisionError:
    raise ValueError("Division failed") from None  # 不显示原始异常

# 只输出 ValueError，不显示 ZeroDivisionError
```

**访问异常链：**
```python
try:
    try:
        1 / 0
    except ZeroDivisionError as e:
        raise ValueError("Failed") from e
except ValueError as e:
    print(e)           # Failed
    print(e.__cause__) # division by zero
```

---

### 代码题答案

**1. 找出下面代码中的问题并修正：**
```python
try:
    x = 1 / 0
except Exception:
    print("Exception")
except ZeroDivisionError:
    print("ZeroDivisionError")
```

**答案：**

**问题：** `except Exception` 放在了更具体的 `except ZeroDivisionError` 前面。因为 `Exception` 是 `ZeroDivisionError` 的父类，它会先匹配到，导致第二个except永远不会执行。

**修正：**
```python
try:
    x = 1 / 0
except ZeroDivisionError:
    print("ZeroDivisionError")  # 先捕获具体异常
except Exception:
    print("Exception")           # 再捕获通用异常
```

---

**2. 解释下面代码的输出：**
```python
def func():
    try:
        return 1
    finally:
        return 2

print(func())
```

**答案：**

输出：`2`

**解释：**
`finally`块中的代码**总是会执行**，即使`try`块中有`return`语句。
- 当执行到`try`块中的`return 1`时，函数准备返回1
- 但在实际返回前，必须执行`finally`块
- `finally`块中的`return 2`会覆盖之前的返回值
- 最终函数返回2

**注意：** 尽量避免在`finally`中使用`return`，这会导致意外的行为。`finally`应该只用于资源清理。

---

### 编程题答案

**1. 编写一个函数，读取一个文件内容，使用适当的异常处理。**

**答案：**
```python
def read_file(filename, encoding='utf-8'):
    """
    安全读取文件内容
    
    Args:
        filename: 文件路径
        encoding: 文件编码
    
    Returns:
        文件内容，如果读取失败返回None
    """
    try:
        with open(filename, 'r', encoding=encoding) as f:
            return f.read()
    except FileNotFoundError:
        print(f"错误: 文件 '{filename}' 不存在")
    except PermissionError:
        print(f"错误: 没有权限读取文件 '{filename}'")
    except UnicodeDecodeError:
        print(f"错误: 文件 '{filename}' 编码不正确，尝试使用其他编码")
    except IOError as e:
        print(f"IO错误: {e}")
    except Exception as e:
        print(f"发生未知错误: {e}")
    return None

# 进阶：处理大文件，逐行读取
def process_large_file(filename, process_func):
    """处理大文件，逐行处理避免内存溢出"""
    try:
        with open(filename, 'r', encoding='utf-8') as f:
            for line_num, line in enumerate(f, 1):
                try:
                    process_func(line.strip())
                except ValueError as e:
                    print(f"第{line_num}行数据格式错误: {e}")
                    # 跳过错误行，继续处理
                    continue
    except FileNotFoundError:
        print(f"文件不存在: {filename}")
        raise  # 向上传播，让调用者决定如何处理
```

---

**2. 自定义一个验证异常类，并在输入验证失败时抛出。**

**答案：**
```python
class ValidationError(Exception):
    """验证异常基类"""
    def __init__(self, message, field=None, value=None):
        super().__init__(message)
        self.field = field
        self.value = value
    
    def __str__(self):
        msg = super().__str__()
        if self.field:
            return f"[{self.field}] {msg}"
        return msg

class EmailValidationError(ValidationError):
    """邮箱验证错误"""
    pass

class PasswordValidationError(ValidationError):
    """密码验证错误"""
    pass

class AgeValidationError(ValidationError):
    """年龄验证错误"""
    pass


def validate_email(email):
    import re
    if not email:
        raise EmailValidationError("邮箱不能为空", field="email", value=email)
    
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(pattern, email):
        raise EmailValidationError("邮箱格式不正确", field="email", value=email)
    
    return True

def validate_password(password):
    if len(password) < 8:
        raise PasswordValidationError("密码长度不能少于8位", field="password", value="*" * len(password))
    
    has_upper = any(c.isupper() for c in password)
    has_lower = any(c.islower() for c in password)
    has_digit = any(c.isdigit() for c in password)
    
    if not (has_upper and has_lower and has_digit):
        raise PasswordValidationError(
            "密码必须包含大小写字母和数字", 
            field="password", 
            value="*" * len(password)
        )
    
    return True

def validate_age(age):
    try:
        age = int(age)
    except (ValueError, TypeError):
        raise AgeValidationError("年龄必须是整数", field="age", value=age)
    
    if age < 0 or age > 150:
        raise AgeValidationError("年龄必须在0-150之间", field="age", value=age)
    
    return True


# 使用示例
def register_user(email, password, age):
    try:
        validate_email(email)
        validate_password(password)
        validate_age(age)
        print("用户注册成功!")
        return True
    except ValidationError as e:
        print(f"验证失败: {e}")
        return False

# 测试
register_user("invalid-email", "123456", "twenty")
register_user("user@example.com", "Pass1234", "25")
```

---

## 6. 文件操作

### 简答题答案

**1. 解释Python中打开文件的模式（r, w, a, r+, w+, a+, b）的区别。**

**答案：**

| 模式 | 说明 | 文件不存在 | 现有文件内容 |
|------|------|-----------|-------------|
| `r` | 只读（默认） | 报错 | 保留 |
| `w` | 只写 | 创建 | 清空（覆盖） |
| `a` | 追加 | 创建 | 保留，写入到末尾 |
| `r+` | 读写 | 报错 | 保留，可在任意位置写入 |
| `w+` | 读写 | 创建 | 清空 |
| `a+` | 读写追加 | 创建 | 保留，只能在末尾写入 |
| `b` | 二进制模式（与其他模式组合） | - | - |

**二进制模式组合：**
- `rb`, `wb`, `ab`, `r+b`, `w+b`, `a+b`

**详细说明：**
```python
# r: 只读，文件必须存在
f = open('file.txt', 'r')
content = f.read()
f.write('x')  # ❌ 错误，不支持写入

# w: 只写，文件不存在则创建，存在则清空
f = open('file.txt', 'w')
f.write('hello')
f.read()  # ❌ 错误，不支持读取

# a: 追加，文件不存在则创建，存在则在末尾写入
f = open('file.txt', 'a')
f.write('world')  # 写在末尾

# r+: 读写，文件必须存在，可在任意位置读写
f = open('file.txt', 'r+')
content = f.read()  # 读全部
f.seek(0)           # 回到开头
f.write('overwrite')  # 从开头覆盖写入

# b: 二进制模式，用于非文本文件（图片、视频、exe等）
with open('image.jpg', 'rb') as f:
    image_data = f.read()
```

---

**2. 为什么使用`with`语句处理文件是更好的实践？**

**答案：**

**`with`语句的优点：**

1. **自动关闭文件**：无论是否发生异常，文件都会被正确关闭
   ```python
   # 不使用with（容易忘记关闭）
   f = open('file.txt', 'r')
   try:
       content = f.read()
   finally:
       f.close()  # 必须手动写

   # 使用with（自动关闭）
   with open('file.txt', 'r') as f:
       content = f.read()
   # 离开with块后自动关闭
   ```

2. **代码更简洁清晰**：减少样板代码

3. **异常安全**：即使处理过程中抛出异常，文件也能正确关闭，避免资源泄漏

4. **支持多个上下文管理器**：
   ```python
   with open('input.txt', 'r') as infile, open('output.txt', 'w') as outfile:
       outfile.write(infile.read())
   ```

**`with`语句的原理：**
上下文管理器实现了`__enter__`和`__exit__`两个方法：
- 进入`with`块时调用`__enter__`
- 离开`with`块时调用`__exit__`（即使发生异常）

---

**3. 什么是文件描述符？Python中如何管理文件描述符？**

**答案：**

**文件描述符（File Descriptor）** 是操作系统为每个打开的文件/套接字/管道等分配的一个小整数，用于标识该资源。

- 标准输入：0
- 标准输出：1
- 标准错误：2
- 用户打开的文件：从3开始

**Python中的管理：**

1. **文件对象持有文件描述符**：
   ```python
   f = open('file.txt')
   print(f.fileno())  # 获取文件描述符
   ```

2. **自动关闭**：`with`语句确保文件描述符被释放

3. **文件描述符泄漏问题**：
   ```python
   # 不好的做法：如果忘记关闭，文件描述符泄漏
   for i in range(10000):
       f = open(f'file{i}.txt')  # 打开太多文件会报错

   # OSError: [Errno 24] Too many open files
   ```

4. **手动管理（不推荐）**：
   ```python
   import os
   fd = os.open('file.txt', os.O_RDONLY)
   data = os.read(fd, 100)
   os.close(fd)  # 必须手动关闭
   ```

---

**4. 解释文本模式和二进制模式的区别。**

**答案：**

| 特性 | 文本模式（t） | 二进制模式（b） |
|------|--------------|----------------|
| 换行符处理 | 自动转换（\n ↔ 系统换行符） | 不转换，原样读写 |
| 编码 | 需要指定编码（默认utf-8） | 不涉及编码，直接读写字节 |
| 读写单位 | 字符串（str） | 字节（bytes） |
| 适用文件 | 文本文件（.txt, .py, .md） | 二进制文件（图片、视频、exe） |

**详细说明：**

```python
# 文本模式
with open('file.txt', 'rt', encoding='utf-8') as f:
    content = f.read()  # str类型
    print(type(content))  # <class 'str'>

# 二进制模式
with open('file.txt', 'rb') as f:
    content = f.read()  # bytes类型
    print(type(content))  # <class 'bytes'>
```

**换行符转换（Windows）：**
- 写入时：`\n` → `\r\n`
- 读取时：`\r\n` → `\n`
- 二进制模式不进行转换

**编码问题：**
```python
# 文本模式可以指定编码
with open('file.txt', 'w', encoding='gbk') as f:
    f.write('中文')

# 二进制模式需要手动编码解码
with open('file.txt', 'wb') as f:
    f.write('中文'.encode('gbk'))

with open('file.txt', 'rb') as f:
    content = f.read().decode('gbk')
```

---

**5. 如何处理大文件（GB级别）以避免内存溢出？**

**答案：**

**处理大文件的原则：不要一次性读取整个文件到内存。**

**方法1：逐行迭代（最常用）**
```python
with open('huge_file.txt', 'r', encoding='utf-8') as f:
    for line in f:  # 文件对象是迭代器，逐行读取
        process_line(line.strip())
```

**方法2：分块读取**
```python
def read_in_chunks(file_obj, chunk_size=8192):
    """按块读取文件"""
    while True:
        data = file_obj.read(chunk_size)
        if not data:  # 读到文件末尾
            break
        yield data

with open('huge_file.bin', 'rb') as f:
    for chunk in read_in_chunks(f, chunk_size=1024*1024):  # 1MB
        process_chunk(chunk)
```

**方法3：使用`linecache`（随机访问特定行）**
```python
import linecache

# 获取第1000行（从1开始计数）
line = linecache.getline('huge_file.txt', 1000)

# 用完后清理缓存
linecache.clearcache()
```

**方法4：使用第三方库（Pandas处理大CSV）**
```python
import pandas as pd

# 分块读取CSV
chunk_iter = pd.read_csv('huge_data.csv', chunksize=10000)
for chunk in chunk_iter:
    process_chunk(chunk)
```

**注意事项：**
- 避免使用`read()`或`readlines()`读取大文件（一次性加载全部内容）
- 使用生成器/迭代器处理数据
- 及时释放不再需要的对象引用

---

### 代码题答案

**1. 解释下面代码的区别：**
```python
# 方式1
f = open('file.txt', 'r')
content = f.read()
f.close()

# 方式2
with open('file.txt', 'r') as f:
    content = f.read()
```

**答案：**

**方式1的问题：**
- 如果`read()`抛出异常，`f.close()`不会执行，导致文件句柄泄漏
- 容易忘记写`close()`
- 需要更多样板代码（try/finally）

**方式2的优点：**
- `with`语句保证文件一定会被关闭，即使读取过程中发生异常
- 代码更简洁
- 更安全，避免资源泄漏

**方式1的安全版本（等价于方式2）：**
```python
f = open('file.txt', 'r')
try:
    content = f.read()
finally:
    f.close()  # 无论是否异常都会执行
```

---

**2. 写出读取文件的几种方式及其适用场景：**
   - `read()`
   - `readline()`
   - `readlines()`
   - 迭代文件对象

**答案：**

```python
# 1. read([size]): 读取全部或指定大小的内容
with open('file.txt', 'r') as f:
    content = f.read()  # 读取全部到一个字符串
    # 适用：小文件，需要一次性处理全部内容
```

```python
# 2. readline(): 读取一行
with open('file.txt', 'r') as f:
    while True:
        line = f.readline()
        if not line:  # 读到末尾返回空字符串
            break
        process(line)
    # 适用：逐行处理，需要控制读取过程
```

```python
# 3. readlines(): 读取所有行到列表
with open('file.txt', 'r') as f:
    lines = f.readlines()  # 返回列表，每个元素是一行
    # 适用：小文件，需要随机访问行
    # 注意：大文件不要用，会占用大量内存
```

```python
# 4. 迭代文件对象（推荐）
with open('file.txt', 'r') as f:
    for line in f:  # 文件对象是可迭代的
        process(line.strip())
    # 适用：大文件，内存高效，逐行处理
    # 最常用，最Pythonic
```

**总结：**
| 方法 | 返回类型 | 内存使用 | 适用场景 |
|------|---------|---------|---------|
| `read()` | str | 高 | 小文件 |
| `readline()` | str | 低 | 需要精细控制 |
| `readlines()` | list | 高 | 小文件，需要随机访问 |
| `for line in f` | str（每次一行） | 低 | 大文件，推荐使用 |

---

### 编程题答案

**1. 编写一个函数，统计一个文本文件的行数、单词数和字符数。**

**答案：**
```python
def count_file_stats(filename, encoding='utf-8'):
    """
    统计文件的行数、单词数和字符数
    
    返回: (行数, 单词数, 字符数)
    """
    lines_count = 0
    words_count = 0
    chars_count = 0
    
    try:
        with open(filename, 'r', encoding=encoding) as f:
            for line in f:
                lines_count += 1
                chars_count += len(line)
                words_count += len(line.split())
        
        return lines_count, words_count, chars_count
    
    except FileNotFoundError:
        print(f"错误: 文件 '{filename}' 不存在")
        return None
    except Exception as e:
        print(f"发生错误: {e}")
        return None


# 类似wc命令的完整版
def wc(filename, mode='lwc'):
    """
    模拟Unix wc命令
    mode: l=行数, w=单词数, c=字符数
    """
    stats = count_file_stats(filename)
    if stats is None:
        return
    
    lines, words, chars = stats
    result = []
    
    if 'l' in mode:
        result.append(str(lines))
    if 'w' in mode:
        result.append(str(words))
    if 'c' in mode:
        result.append(str(chars))
    
    result.append(filename)
    print(' '.join(result))


# 使用示例
stats = count_file_stats('example.txt')
if stats:
    lines, words, chars = stats
    print(f"行数: {lines}")
    print(f"单词数: {words}")
    print(f"字符数: {chars}")

wc('example.txt')  # 像Unix wc一样输出
```

---

**2. 实现一个函数，将一个大文件分割成多个小文件。**

**答案：**
```python
import os

def split_file_by_lines(input_file, lines_per_file=1000, output_prefix='part_', encoding='utf-8'):
    """
    按行数分割大文件
    
    Args:
        input_file: 输入文件路径
        lines_per_file: 每个文件的行数
        output_prefix: 输出文件前缀
        encoding: 文件编码
    """
    file_num = 1
    current_lines = []
    
    with open(input_file, 'r', encoding=encoding) as f:
        for line_num, line in enumerate(f, 1):
            current_lines.append(line)
            
            # 达到指定行数，写入文件
            if len(current_lines) >= lines_per_file:
                output_file = f"{output_prefix}{file_num:04d}.txt"
                with open(output_file, 'w', encoding=encoding) as out_f:
                    out_f.writelines(current_lines)
                print(f"Created: {output_file}")
                
                file_num += 1
                current_lines = []
        
        # 处理剩余的行
        if current_lines:
            output_file = f"{output_prefix}{file_num:04d}.txt"
            with open(output_file, 'w', encoding=encoding) as out_f:
                out_f.writelines(current_lines)
            print(f"Created: {output_file}")


def split_file_by_size(input_file, size_per_file=10*1024*1024, output_prefix='part_'):
    """
    按大小分割文件（二进制模式，适用于任何文件）
    
    Args:
        input_file: 输入文件路径
        size_per_file: 每个文件的大小（字节），默认10MB
        output_prefix: 输出文件前缀
    """
    file_num = 1
    
    with open(input_file, 'rb') as f:
        while True:
            chunk = f.read(size_per_file)
            if not chunk:
                break
            
            output_file = f"{output_prefix}{file_num:04d}"
            with open(output_file, 'wb') as out_f:
                out_f.write(chunk)
            print(f"Created: {output_file}")
            file_num += 1


def merge_files(output_file, input_files, encoding='utf-8'):
    """合并多个文件"""
    with open(output_file, 'w', encoding=encoding) as out_f:
        for input_file in sorted(input_files):
            with open(input_file, 'r', encoding=encoding) as in_f:
                out_f.write(in_f.read())
            print(f"Merged: {input_file}")


# 使用示例
if __name__ == '__main__':
    # 按行数分割文本文件
    split_file_by_lines('big_log.txt', lines_per_file=5000)
    
    # 按大小分割二进制文件
    split_file_by_size('big_data.bin', size_per_file=5*1024*1024)
```

---

(文档持续更新中，以上为前6章完整答案)
