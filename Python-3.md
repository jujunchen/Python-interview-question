# Python 面试题答案（第7-15章）

---

## 7. 装饰器与闭包

### 简答题答案

**1. 什么是闭包（closure）？闭包的三个条件是什么？**

**答案：**

**闭包**是指一个函数对象能够记住并访问它的词法作用域中的变量，即使这个函数在它的词法作用域之外被调用。

**闭包的三个条件：**
1. **必须有一个嵌套函数**（内部函数）
2. **内部函数必须引用外部函数中的变量**
3. **外部函数必须返回内部函数**

```python
def outer(x):
    # 外部函数
    def inner(y):
        # 内部函数（嵌套函数）
        return x + y  # 引用外部函数的变量x
    return inner  # 返回内部函数

add5 = outer(5)  # add5就是一个闭包
print(add5(3))   # 8
```

**闭包的特性：**
- 记住外层函数的变量状态
- 即使外层函数执行完毕，内部函数仍然能访问那些变量
- 每个闭包有自己独立的环境

```python
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

c1 = make_counter()
c2 = make_counter()
print(c1())  # 1
print(c1())  # 2
print(c2())  # 1 （独立的计数器）
```

---

**2. 什么是装饰器？装饰器的应用场景有哪些？**

**答案：**

**装饰器**是一种特殊的闭包，用于在不修改原函数代码的情况下，增强或修改函数的行为。

**装饰器本质：** 接收一个函数作为参数，返回一个新的函数

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        # 调用前的操作
        print("Before function call")
        result = func(*args, **kwargs)
        # 调用后的操作
        print("After function call")
        return result
    return wrapper

# 使用装饰器
@decorator
def hello():
    print("Hello!")

hello()
```

**应用场景：**

1. **日志记录**：记录函数调用信息
2. **性能监控**：统计函数执行时间
3. **权限验证**：检查用户是否有权限执行
4. **缓存**：缓存函数结果（如`lru_cache`）
5. **输入验证**：检查函数参数是否合法
6. **重试机制**：失败时自动重试
7. **事务处理**：确保操作原子性
8. **类型检查**：运行时参数类型检查

```python
# 示例：缓存装饰器
def cache(func):
    cached = {}
    def wrapper(*args):
        if args not in cached:
            cached[args] = func(*args)
        return cached[args]
    return wrapper

@cache
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)
```

---

**3. 解释装饰器的执行时机（定义时 vs 调用时）。**

**答案：**

**装饰器在函数定义时就会执行一次！**

```python
def my_decorator(func):
    print(f"Decorating {func.__name__}")  # 定义时执行
    def wrapper():
        print(f"Calling {func.__name__}")  # 调用时执行
        func()
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

# 此时已经输出: Decorating say_hello
print("Function defined")

# 调用时
say_hello()
# 输出:
# Calling say_hello
# Hello!
```

**执行时机总结：**

| 时机 | 执行内容 |
|------|---------|
| **定义时**（导入模块时） | 装饰器函数本身执行一次，wrapper被创建 |
| **调用时**（每次调用） | wrapper函数执行 |

**重要：** 装饰器只在定义时执行一次，不是每次调用都执行装饰器本身。

---

**4. 如何给装饰器传递参数？**

**答案：**

带参数的装饰器需要**多一层嵌套**：最外层接收参数，中间层接收函数，最内层是wrapper。

```python
def repeat(times):  # 第一层：接收装饰器参数
    def decorator(func):  # 第二层：接收函数
        def wrapper(*args, **kwargs):  # 第三层：包装函数
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

# 使用带参数的装饰器
@repeat(3)
def greet():
    print("Hello!")

greet()
# 输出:
# Hello!
# Hello!
# Hello!
```

**对比：**
```python
# 无参数装饰器
@decorator
def func():
    pass
# 等价于: func = decorator(func)

# 带参数装饰器
@decorator(arg)
def func():
    pass
# 等价于: func = decorator(arg)(func)
```

**示例：带条件的日志装饰器**
```python
def log(level="INFO"):
    def decorator(func):
        def wrapper(*args, **kwargs):
            print(f"[{level}] Calling {func.__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@log(level="DEBUG")
def process_data():
    print("Processing...")
```

---

**5. 解释`functools.wraps`的作用，为什么需要它？**

**答案：**

`functools.wraps`用于保留原函数的元信息（函数名、文档字符串、参数列表等）。

**不使用wraps的问题：**
```python
def decorator(func):
    def wrapper(*args, **kwargs):
        """This is wrapper"""
        return func(*args, **kwargs)
    return wrapper

@decorator
def hello():
    """Say hello"""
    print("Hello")

print(hello.__name__)    # wrapper （不是hello！）
print(hello.__doc__)     # This is wrapper （不是原函数的文档！）
```

**使用wraps后：**
```python
from functools import wraps

def decorator(func):
    @wraps(func)  # 保留原函数元信息
    def wrapper(*args, **kwargs):
        """This is wrapper"""
        return func(*args, **kwargs)
    return wrapper

@decorator
def hello():
    """Say hello"""
    print("Hello")

print(hello.__name__)    # hello （正确！）
print(hello.__doc__)     # Say hello （正确！）
```

**`@wraps`保留的信息：**
- `__name__`：函数名
- `__doc__`：文档字符串
- `__module__`：模块名
- `__annotations__`：类型注解
- `__qualname__`：限定名

**为什么需要它？**
- 调试时能看到正确的函数名
- 文档工具（如help()、pydoc）能正确显示
- 序列化、反射等场景依赖正确的元信息
- 保持代码的可维护性

---

### 代码题答案

**1. 写出下面代码的输出结果：**
```python
def outer(x):
    def inner(y):
        return x + y
    return inner

add5 = outer(5)
print(add5(3))
```

**答案：**

输出：`8`

**解释：**
- `outer(5)`执行，x=5，返回inner函数，此时inner记住了x=5（闭包）
- `add5`就是这个记住了x=5的inner函数
- 调用`add5(3)`时，y=3，所以 x + y = 5 + 3 = 8

---

**2. 解释下面装饰器的执行顺序：**
```python
def decorator1(func):
    print("decorator1")
    def wrapper():
        print("wrapper1")
        func()
    return wrapper

def decorator2(func):
    print("decorator2")
    def wrapper():
        print("wrapper2")
        func()
    return wrapper

@decorator1
@decorator2
def hello():
    print("hello")

hello()
```

**答案：**

输出：
```
decorator2
decorator1
wrapper1
wrapper2
hello
```

**解释：**

**定义时（从上到下装饰，从下到上执行）：**
```python
# @decorator1
# @decorator2
# def hello():
# 等价于:
hello = decorator1(decorator2(hello))
```
1. 先执行`decorator2(hello)`，输出"decorator2"
2. 再执行`decorator1(...)`，输出"decorator1"

**调用时（从上到下执行wrapper）：**
```
hello() → wrapper1() → wrapper2() → 原hello()
```
1. 调用最外层的wrapper1，输出"wrapper1"
2. wrapper1调用func（即decorator2返回的wrapper2），输出"wrapper2"
3. wrapper2调用func（即原hello函数），输出"hello"

---

### 编程题答案

**1. 实现一个带参数的装饰器，控制函数被调用的次数。**

**答案：**
```python
from functools import wraps

def call_limit(max_calls):
    """限制函数调用次数的装饰器"""
    def decorator(func):
        call_count = 0  # 记录调用次数（闭包变量）
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal call_count
            if call_count >= max_calls:
                raise RuntimeError(
                    f"Function {func.__name__} can only be called {max_calls} times"
                )
            call_count += 1
            print(f"Call {call_count}/{max_calls}: {func.__name__}")
            return func(*args, **kwargs)
        
        return wrapper
    return decorator


# 使用示例
@call_limit(3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")  # Call 1/3: greet
greet("Bob")    # Call 2/3: greet
greet("Charlie")# Call 3/3: greet
greet("Dave")   # RuntimeError!


# 进阶：允许重置计数
def call_limit_with_reset(max_calls):
    def decorator(func):
        call_count = 0
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal call_count
            if call_count >= max_calls:
                raise RuntimeError(f"Call limit exceeded")
            call_count += 1
            return func(*args, **kwargs)
        
        def reset():
            nonlocal call_count
            call_count = 0
            print("Call count reset")
        
        # 给wrapper添加方法
        wrapper.reset = reset
        return wrapper
    return decorator


@call_limit_with_reset(2)
def func():
    print("func called")

func()
func()
func.reset()  # 重置计数
func()        # 可以再次调用
```

---

**2. 实现一个类装饰器。**

**答案：**
```python
from functools import wraps

class LogCalls:
    """类装饰器：记录函数调用"""
    def __init__(self, log_file=None):
        self.log_file = log_file
        self.call_count = 0
    
    def __call__(self, func):
        """当装饰器应用到函数时调用"""
        @wraps(func)
        def wrapper(*args, **kwargs):
            self.call_count += 1
            log_msg = f"Call #{self.call_count}: {func.__name__} with args={args}, kwargs={kwargs}"
            
            if self.log_file:
                with open(self.log_file, 'a') as f:
                    f.write(log_msg + '\n')
            else:
                print(log_msg)
            
            result = func(*args, **kwargs)
            return result
        
        return wrapper


# 使用类装饰器
@LogCalls()
def add(a, b):
    return a + b

@LogCalls(log_file='calls.log')
def multiply(a, b):
    return a * b

add(2, 3)
multiply(4, 5)


# 方式2：使用类装饰整个类
def singleton(cls):
    """单例装饰器（装饰类）"""
    instances = {}
    
    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class Database:
    def __init__(self):
        print("Database connection created")

db1 = Database()
db2 = Database()
print(db1 is db2)  # True


# 方式3：用类作为装饰器（实现__call__）
class Timer:
    def __init__(self, func):
        self.func = func
        wraps(func)(self)  # 保留原函数信息
    
    def __call__(self, *args, **kwargs):
        import time
        start = time.time()
        result = self.func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{self.func.__name__} took {elapsed:.4f}s")
        return result

@Timer
def slow_function():
    import time
    time.sleep(1)

slow_function()
```

---

**3. 实现一个缓存装饰器（类似lru_cache）。**

**答案：**
```python
from functools import wraps
from collections import OrderedDict

def simple_cache(func):
    """简单缓存装饰器"""
    cache = {}
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        # 创建缓存键：参数的哈希
        key = (args, tuple(sorted(kwargs.items())))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
            print(f"[CACHE MISS] {func.__name__}{args}")
        else:
            print(f"[CACHE HIT] {func.__name__}{args}")
        return cache[key]
    
    return wrapper


def lru_cache(maxsize=128):
    """LRU缓存装饰器（最近最少使用淘汰）"""
    def decorator(func):
        cache = OrderedDict()
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            
            if key in cache:
                # 命中：移到末尾表示最近使用
                cache.move_to_end(key)
                print(f"[LRU HIT]")
            else:
                # 未命中：计算并存入
                if len(cache) >= maxsize:
                    # 淘汰最早的
                    cache.popitem(last=False)
                    print(f"[LRU EVICT]")
                cache[key] = func(*args, **kwargs)
                print(f"[LRU MISS]")
            
            return cache[key]
        
        # 添加缓存管理方法
        def cache_info():
            return f"CacheInfo(hits={...}, misses={...}, maxsize={maxsize})"
        
        def cache_clear():
            cache.clear()
        
        wrapper.cache_info = cache_info
        wrapper.cache_clear = cache_clear
        return wrapper
    return decorator


# 测试
@lru_cache(maxsize=3)
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)

print(fib(5))
print(fib(5))  # HIT
print(fib(6))


# 带过期时间的缓存
import time

def ttl_cache(ttl_seconds=60):
    """带过期时间的缓存"""
    def decorator(func):
        cache = {}  # key: (value, timestamp)
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            now = time.time()
            
            if key in cache:
                value, timestamp = cache[key]
                if now - timestamp < ttl_seconds:
                    print(f"[TTL HIT]")
                    return value
            
            # 缓存未命中或已过期
            result = func(*args, **kwargs)
            cache[key] = (result, now)
            print(f"[TTL MISS]")
            return result
        
        return wrapper
    return decorator
```

---

## 8. 迭代器与生成器

### 简答题答案

**1. 什么是可迭代对象（Iterable）和迭代器（Iterator）？它们的区别是什么？**

**答案：**

**可迭代对象（Iterable）：**
- 实现了`__iter__()`方法的对象
- 可以被`for`循环遍历
- 例子：list, tuple, dict, str, set

**迭代器（Iterator）：**
- 实现了`__iter__()`和`__next__()`方法的对象
- 是一个有状态的对象，记住遍历的位置
- 调用`__next__()`返回下一个元素
- 没有元素时抛出`StopIteration`异常

**关系：**
```python
# 可迭代对象调用iter()得到迭代器
my_list = [1, 2, 3]          # Iterable
it = iter(my_list)            # Iterator

print(next(it))  # 1
print(next(it))  # 2
print(next(it))  # 3
print(next(it))  # StopIteration
```

**区别：**

| 特性 | Iterable | Iterator |
|------|----------|----------|
| 实现方法 | `__iter__()` | `__iter__()` + `__next__()` |
| 状态 | 无状态 | 有状态（记住位置） |
| 重复遍历 | 可以多次遍历 | 只能遍历一次 |
| 示例 | list, tuple | iter(list), generator |

```python
# Iterable可以多次遍历
nums = [1, 2, 3]
for n in nums:
    print(n)  # 1, 2, 3
for n in nums:
    print(n)  # 1, 2, 3 （又从头开始）

# Iterator只能遍历一次
it = iter(nums)
for n in it:
    print(n)  # 1, 2, 3
for n in it:
    print(n)  # 什么都不输出（已经遍历完了）
```

---

**2. 什么是生成器（Generator）？生成器的优点是什么？**

**答案：**

**生成器**是一种特殊的迭代器，使用函数定义（包含`yield`语句），可以惰性地产生值。

**创建生成器的两种方式：**
```python
# 方式1：生成器函数
def count(n):
    while n > 0:
        yield n
        n -= 1

gen = count(3)
print(next(gen))  # 3
print(next(gen))  # 2
print(next(gen))  # 1

# 方式2：生成器表达式
gen = (x**2 for x in range(10))
```

**生成器的优点：**

1. **内存高效（惰性求值）**：
   - 只在需要时计算下一个值
   - 处理大数据时不占用大量内存
   ```python
   # 列表：占用大量内存
   big_list = [x**2 for x in range(1000000)]  # 内存中存储所有值

   # 生成器：几乎不占内存
   big_gen = (x**2 for x in range(1000000))   # 只保存计算逻辑
   ```

2. **表示无限序列**：
   ```python
   def infinite_counter():
       n = 0
       while True:
           yield n
           n += 1
   
   counter = infinite_counter()
   print(next(counter))  # 0
   print(next(counter))  # 1
   # ... 可以无限下去
   ```

3. **代码更简洁**：无需手动实现`__iter__`和`__next__`

4. **管道处理**：多个生成器可以链式组合，形成数据处理管道

---

**3. 解释`yield`关键字的工作原理。**

**答案：**

**`yield`**是生成器的核心，它的作用是：
1. 暂停函数执行，返回一个值给调用者
2. 保存函数的状态（局部变量、指针位置等）
3. 下次调用`next()`时，从暂停的地方继续执行

```python
def generator():
    print("Start")
    yield 1
    print("After first yield")
    yield 2
    print("After second yield")
    yield 3
    print("End")

gen = generator()  # 创建生成器对象，函数还没执行！

result = next(gen)  # 执行到第一个yield
# 输出: Start
print(result)  # 1

result = next(gen)  # 从暂停处继续执行
# 输出: After first yield
print(result)  # 2

result = next(gen)
# 输出: After second yield
print(result)  # 3

next(gen)  # 继续执行
# 输出: End
# 抛出 StopIteration
```

**`yield` vs `return`：**
| `yield` | `return` |
|---------|----------|
| 暂停函数，下次继续 | 终止函数 |
| 可以有多个 | 通常只有一个 |
| 返回生成器迭代器 | 返回值 |

---

**4. 什么是生成器表达式？它与列表推导式的区别是什么？**

**答案：**

**生成器表达式**是创建生成器的简洁语法，用圆括号包裹。

**语法：**
```python
# 列表推导式（方括号）
list_comp = [x**2 for x in range(10)]

# 生成器表达式（圆括号）
gen_exp = (x**2 for x in range(10))
```

**区别：**

| 特性 | 列表推导式 | 生成器表达式 |
|------|-----------|-------------|
| 内存 | 立即计算所有值，占用内存 | 惰性计算，几乎不占内存 |
| 执行时机 | 定义时立即计算 | 迭代时才计算 |
| 遍历次数 | 可多次遍历 | 只能遍历一次 |
| 表示无限序列 | 不可能 | 可以 |
| 语法 | `[...]` | `(...)` |

```python
# 内存对比
import sys

list_comp = [x**2 for x in range(1000000)]
gen_exp = (x**2 for x in range(1000000))

print(sys.getsizeof(list_comp))  # 约8MB（取决于实现）
print(sys.getsizeof(gen_exp))    # 约128字节！

# 执行时机对比
def func(x):
    print(f"Calculating {x}")
    return x**2

print("Creating list...")
list_comp = [func(x) for x in range(3)]  # 立即输出3次
print("List created")

print("Creating generator...")
gen_exp = (func(x) for x in range(3))   # 没有输出
print("Generator created")

print("Iterating generator...")
for x in gen_exp:  # 现在才开始计算
    pass
```

**使用场景：**
- 列表推导式：数据量小，需要多次访问，需要索引访问
- 生成器表达式：数据量大，只需遍历一次，内存受限

---

**5. 解释协程（coroutine）的概念，以及`yield from`的作用。**

**答案：**

**协程**是一种可以暂停和恢复执行的函数，用于协作式多任务。

**协程的状态：**
- `GEN_CREATED`: 等待开始执行
- `GEN_RUNNING`: 正在执行
- `GEN_SUSPENDED`: 在yield处暂停
- `GEN_CLOSED`: 执行结束

```python
def coroutine():
    print("Starting")
    while True:
        value = yield  # 暂停，等待发送值
        print(f"Received: {value}")

# 创建协程
c = coroutine()

# 必须先启动（预激）
next(c)  # 输出: Starting，在yield处暂停

# 发送数据
c.send(1)  # 输出: Received: 1
c.send(2)  # 输出: Received: 2

# 关闭协程
c.close()
```

**`yield from`的作用：**

1. **委托给子生成器**：简化生成器嵌套调用
   ```python
   def sub_generator():
       yield 1
       yield 2

   def main_generator():
       # 不使用yield from
       for x in sub_generator():
           yield x
       
       # 使用yield from
       yield from sub_generator()  # 等价于上面两行
   ```

2. **透明传递值和异常**：自动传递`send()`、`throw()`、`close()`

```python
def sub():
    while True:
        x = yield
        print(f"Sub got: {x}")

def main():
    yield from sub()  # 透明传递所有操作

m = main()
next(m)
m.send(42)  # Sub got: 42
```

3. **获取子生成器返回值**：
   ```python
   def calc_sum():
       total = 0
       while True:
           x = yield
           if x is None:
               break
           total += x
       return total  # 子生成器返回值

   def main():
       result = yield from calc_sum()
       print(f"Sum = {result}")

   m = main()
   next(m)
   m.send(1)
   m.send(2)
   m.send(None)  # Sum = 3
   ```

---

### 代码题答案

**1. 写出下面代码的输出结果：**
```python
def generator():
    yield 1
    yield 2
    yield 3

g = generator()
print(next(g))
print(next(g))
print(next(g))
```

**答案：**

输出：
```
1
2
3
```

**解释：**
- `generator()`创建生成器对象g
- 第一次`next(g)`：执行到第一个yield，返回1
- 第二次`next(g)`：继续执行到第二个yield，返回2
- 第三次`next(g)`：继续执行到第三个yield，返回3
- 再调用next(g)会抛出StopIteration

---

**2. 解释下面代码的区别：**
```python
# 列表推导式
list_comp = [x**2 for x in range(1000000)]

# 生成器表达式
gen_exp = (x**2 for x in range(1000000))
```

**答案：**

| 区别 | list_comp | gen_exp |
|------|-----------|---------|
| 类型 | list | generator |
| 内存占用 | 约8MB（100万个整数） | 约128字节 |
| 计算时机 | 定义时立即计算所有值 | 迭代时逐个计算 |
| 是否可索引 | 是（`list_comp[100]`） | 否 |
| 遍历次数 | 可多次 | 只能一次 |
| 是否可修改 | 是 | 否 |

**实际影响：**
```python
import sys

print(type(list_comp))  # <class 'list'>
print(type(gen_exp))    # <class 'generator'>

print(sys.getsizeof(list_comp))  # 8448728 bytes (~8MB)
print(sys.getsizeof(gen_exp))    # 128 bytes

# 列表可以多次遍历
for x in list_comp:
    pass
for x in list_comp:
    pass  # 正常

# 生成器只能遍历一次
for x in gen_exp:
    pass
for x in gen_exp:
    pass  # 什么都不做！已经耗尽了
```

---

### 编程题答案

**1. 实现一个自定义迭代器，产生斐波那契数列。**

**答案：**
```python
# 方式1：实现迭代器协议
class FibonacciIterator:
    def __init__(self, max_n=None):
        """
        斐波那契数列迭代器
        max_n: 最多生成多少项，None表示无限
        """
        self.max_n = max_n
        self.a = 0
        self.b = 1
        self.count = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.max_n is not None and self.count >= self.max_n:
            raise StopIteration
        
        result = self.a
        self.a, self.b = self.b, self.a + self.b
        self.count += 1
        return result


# 方式2：生成器函数（更简洁）
def fibonacci(max_n=None):
    a, b = 0, 1
    count = 0
    while max_n is None or count < max_n:
        yield a
        a, b = b, a + b
        count += 1


# 方式3：直到某个最大值
def fibonacci_until(max_value):
    a, b = 0, 1
    while a <= max_value:
        yield a
        a, b = b, a + b


# 使用示例
print("前10项:")
for num in FibonacciIterator(10):
    print(num, end=' ')
print()

print("生成器版本:")
for num in fibonacci(8):
    print(num, end=' ')
print()

print("不超过100:")
for num in fibonacci_until(100):
    print(num, end=' ')
print()


# 进阶：无限斐波那契
def infinite_fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 找到第一个大于10000的斐波那契数
for num in infinite_fibonacci():
    if num > 10000:
        print(f"第一个大于10000的斐波那契数: {num}")
        break
```

---

**2. 使用生成器实现一个大文件的逐行读取处理。**

**答案：**
```python
def read_large_file(filepath, encoding='utf-8'):
    """逐行读取大文件，内存高效"""
    with open(filepath, 'r', encoding=encoding) as f:
        for line in f:
            yield line.strip()


def filter_lines(lines, keyword):
    """过滤包含特定关键词的行"""
    for line in lines:
        if keyword in line:
            yield line


def process_lines(lines):
    """处理每一行数据"""
    for line in lines:
        # 模拟复杂处理
        data = line.split(',')
        yield {
            'raw': line,
            'fields': len(data),
            'length': len(line)
        }


# 生成器管道：数据流式处理
def process_file_pipeline(filepath, keyword):
    """
    生成器组合形成处理管道
    数据流式通过：read -> filter -> process
    """
    lines = read_large_file(filepath)
    filtered = filter_lines(lines, keyword)
    processed = process_lines(filtered)
    
    for result in processed:
        yield result


# 分块读取二进制大文件
def read_chunks(filepath, chunk_size=1024*1024):
    """按块读取二进制文件，默认1MB一块"""
    with open(filepath, 'rb') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk


# CSV特定处理：处理带换行的字段
def read_csv_rows(filepath):
    """正确读取CSV行（处理引号内的换行）"""
    import csv
    with open(filepath, 'r', newline='', encoding='utf-8') as f:
        reader = csv.reader(f)
        for row in reader:
            yield row


# 使用示例
if __name__ == '__main__':
    print("处理大文件示例:")
    
    # 统计日志文件中的错误行
    error_count = 0
    for line in read_large_file('access.log'):
        if 'ERROR' in line:
            error_count += 1
            if error_count <= 5:  # 只打印前5条错误
                print(f"错误#{error_count}: {line}")
    
    print(f"总共找到 {error_count} 条错误记录")
    
    # 使用管道处理
    print("\n使用处理管道:")
    results = process_file_pipeline('data.csv', 'important')
    for i, result in enumerate(results):
        if i >= 3:
            break
        print(result)
```

---

**3. 实现一个简单的协程，展示生产者-消费者模式。**

**答案：**
```python
import time
import random

# 消费者协程
def consumer(name):
    print(f"[{name}] 消费者准备就绪")
    total = 0
    while True:
        item = yield  # 暂停等待生产者发送
        if item is None:  # 结束信号
            print(f"[{name}] 收到结束信号，共处理 {total} 个任务")
            break
        total += 1
        print(f"[{name}] 处理任务: {item}")
        time.sleep(random.uniform(0.1, 0.5))  # 模拟处理时间


# 生产者
def producer(consumers, items):
    # 先启动所有消费者
    for c in consumers:
        next(c)  # 预激协程
    
    # 分发任务
    for i, item in enumerate(items):
        consumer = consumers[i % len(consumers)]  # 轮询分发
        consumer.send(item)
    
    # 发送结束信号
    for c in consumers:
        c.send(None)


# 带返回值的协程
def processor(name):
    print(f"处理器 {name} 启动")
    processed = []
    while True:
        task = yield  # 接收任务
        if task is None:
            break
        result = f"{task}-processed-by-{name}"
        processed.append(result)
    return processed  # 返回处理结果


# 使用yield from收集结果
def coordinator(tasks):
    p1 = processor("A")
    p2 = processor("B")
    
    next(p1)
    next(p2)
    
    for i, task in enumerate(tasks):
        if i % 2 == 0:
            p1.send(task)
        else:
            p2.send(task)
    
    # 获取返回值
    try:
        p1.send(None)
    except StopIteration as e:
        results1 = e.value
    
    try:
        p2.send(None)
    except StopIteration as e:
        results2 = e.value
    
    return results1 + results2


# 主函数
def main():
    print("=== 生产者-消费者模式 ===")
    
    # 创建多个消费者
    consumers = [
        consumer("Worker1"),
        consumer("Worker2"),
        consumer("Worker3")
    ]
    
    # 生产任务
    tasks = [f"Task{i}" for i in range(10)]
    producer(consumers, tasks)
    
    print("\n=== 协程收集返回值 ===")
    results = coordinator(["Data1", "Data2", "Data3", "Data4"])
    print("所有结果:", results)


if __name__ == '__main__':
    main()


# 进阶：async/await版本（Python 3.5+）
import asyncio

async def async_consumer(name, queue):
    print(f"[{name}] 异步消费者启动")
    while True:
        item = await queue.get()
        if item is None:
            print(f"[{name}] 结束")
            break
        print(f"[{name}] 处理: {item}")
        await asyncio.sleep(random.uniform(0.1, 0.3))
        queue.task_done()

async def async_producer(queue, items):
    for item in items:
        await asyncio.sleep(0.1)
        await queue.put(item)
        print(f"生产: {item}")
    # 发送结束信号
    for _ in range(3):
        await queue.put(None)

async def async_main():
    queue = asyncio.Queue()
    
    # 创建消费者任务
    consumers = [
        asyncio.create_task(async_consumer(f"Worker{i}", queue))
        for i in range(3)
    ]
    
    # 创建生产者
    producer = asyncio.create_task(
        async_producer(queue, [f"Job{i}" for i in range(8)])
    )
    
    await producer
    await asyncio.gather(*consumers)

# asyncio.run(async_main())
```

---

(文档持续更新中，以上为第7-8章完整答案)
