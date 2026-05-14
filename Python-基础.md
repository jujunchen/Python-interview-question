## 1. 基础语法

### 简答题答案

**1. Python中的PEP 8是什么？请列举至少5条PEP 8的编码规范。**

**答案：**
PEP 8是Python的官方代码风格指南，全称Python Enhancement Proposal 8。

主要规范：

- 缩进使用4个空格，不使用制表符
- 每行最大长度79字符
- 函数和类定义之间空两行，方法定义之间空一行
- 导入顺序：标准库 → 第三方库 → 本地库，各组之间空行
- 命名规范：
  - 类名使用大驼峰命名法（CamelCase）
  - 函数名和变量名使用小写加下划线（snake_case）
  - 常量使用全大写加下划线
- 表达式和语句中适当使用空格

---

**2. Python 2和Python 3的主要区别是什么？（至少列举5点）**

**答案：**

1. **print语句 vs print函数**：
   
   - Python 2: `print "hello"`
   - Python 3: `print("hello")`

2. **整数除法**：
   
   - Python 2: `3 / 2 = 1`（整数除法），`3 // 2 = 1`
   - Python 3: `3 / 2 = 1.5`（返回浮点数），`3 // 2 = 1`

3. **字符串编码**：
   
   - Python 2: ASCII `str` 类型，单独的 `unicode` 类型
   - Python 3: Unicode `str` 类型，字节用 `bytes` 类型

4. **xrange函数**：
   
   - Python 2: `range` 返回列表，`xrange` 返回迭代器
   - Python 3: `range` 像Python 2的`xrange`，没有`xrange`

5. **异常处理语法**：
   
   - Python 2: `except ValueError, e:`
   - Python 3: `except ValueError as e:`

6. **类型提示**：Python 3.5+支持类型注解

7. **f-string**：Python 3.6+支持格式化字符串字面值

---

**3. 解释Python中的变量作用域规则（LEGB规则）。**

**答案：**
LEGB是Python变量查找的顺序规则：

- **L（Local）**：局部作用域，函数内部定义的变量
- **E（Enclosing）**：闭包函数外的函数中，嵌套函数的外层函数作用域
- **G（Global）**：全局作用域，模块级别定义的变量
- **B（Built-in）**：内置作用域，Python内置函数和异常

查找顺序：L → E → G → B

```python
x = "global"  # G

def outer():
    x = "enclosing"  # E

    def inner():
        x = "local"  # L
        print(x)

    inner()

outer()  # 输出 "local"
```

---

**4. 什么是Python的GIL？它对Python程序有什么影响？**

**答案：**
GIL（Global Interpreter Lock，全局解释器锁）是CPython解释器中的一个互斥锁，用于保护对Python对象的访问，防止多线程同时执行Python字节码。

**影响：**

- **CPU密集型任务**：多线程无法真正并行执行，因为任何时候只有一个线程能执行Python代码，多线程甚至可能比单线程更慢（线程切换开销）
- **IO密集型任务**：影响较小，因为线程在等待IO时会释放GIL，其他线程可以继续执行

**绕过GIL的方法：**

- 使用多进程（`multiprocessing`）
- 使用C扩展
- 使用PyPy解释器

---

**5. 解释Python中的深拷贝和浅拷贝的区别，以及它们的使用场景。**

**答案：**

**浅拷贝**：创建一个新对象，但只复制对象的引用，不复制嵌套对象。

- 方式：`list.copy()`、切片`[:]`、`dict.copy()`、`copy.copy()`

**深拷贝**：创建一个新对象，并递归复制所有嵌套对象。

- 方式：`copy.deepcopy()`

**区别示例：**

```python
import copy

a = [1, [2, 3]]
b = copy.copy(a)      # 浅拷贝
c = copy.deepcopy(a)  # 深拷贝

a[1].append(4)
print(a)  # [1, [2, 3, 4]]
print(b)  # [1, [2, 3, 4]]  # 嵌套对象共享
print(c)  # [1, [2, 3]]      # 完全独立
```

**使用场景：**

- 浅拷贝：对象结构简单，不需要独立的嵌套对象，节省内存
- 深拷贝：需要完全独立的对象副本，修改不影响原对象

---

### 代码题答案

**1. 写出下面代码的输出结果并解释原因：**

```python
a = [1, 2, 3]
b = a
a.append(4)
print(b)
```

**答案：**
输出：`[1, 2, 3, 4]`

**解释：**
Python中列表是可变对象，赋值操作`b = a`只是让b指向同一个列表对象，而不是创建新列表。所以当a修改时，b也会看到同样的变化。

---

**2. 写出下面代码的输出结果并解释原因：**

```python
def func(x, lst=[]):
    lst.append(x)
    return lst

print(func(1))
print(func(2))
print(func(3, []))
print(func(4))
```

**答案：**
输出：

```
[1]
[1, 2]
[3]
[1, 2, 4]
```

**解释：**
Python的默认参数在**函数定义时**只计算一次，而不是每次调用时。所以`lst=[]`这个空列表在函数定义时创建，后续调用如果不传入新列表，会一直使用同一个列表对象。

- `func(1)`: 列表变为`[1]`
- `func(2)`: 在同一列表追加，变为`[1, 2]`
- `func(3, [])`: 传入新列表，所以是`[3]`
- `func(4)`: 又使用那个默认列表，变为`[1, 2, 4]`

**最佳实践：** 默认参数使用不可变类型，或用`None`作为哨兵值。

---

**3. 解释下面代码中`*args`和`**kwargs`的作用：**

```python
def example_func(*args, **kwargs):
    print(args)
    print(kwargs)
```

**答案：**

- `*args`：收集所有**位置参数**到一个**元组**中
- `**kwargs`：收集所有**关键字参数**到一个**字典**中

**示例调用：**

```python
example_func(1, 2, 3, name="Alice", age=25)
# 输出:
# (1, 2, 3)
# {'name': 'Alice', 'age': 25}
```

---

### 编程题答案

**1. 编写一个函数，判断一个字符串是否是回文字符串。**

**答案：**

```python
def is_palindrome(s: str) -> bool:
    # 方法1：切片反转
    s = ''.join(c.lower() for c in s if c.isalnum())
    return s == s[::-1]

def is_palindrome2(s: str) -> bool:
    # 方法2：双指针
    import re
    s = re.sub(r'[^a-zA-Z0-9]', '', s).lower()
    left, right = 0, len(s) - 1
    while left < right:
        if s[left] != s[right]:
            return False
        left += 1
        right -= 1
    return True
```

---

**2. 编写一个函数，计算斐波那契数列的第n项（要求至少两种实现方式）。**

**答案：**

```python
# 方式1：递归（简单但效率低，n大时栈溢出）
def fib_recursive(n):
    if n <= 1:
        return n
    return fib_recursive(n-1) + fib_recursive(n-2)

# 方式2：迭代（推荐，O(n)时间，O(1)空间）
def fib_iterative(n):
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(n - 1):
        a, b = b, a + b
    return b

# 方式3：动态规划 + 缓存
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_memo(n):
    if n <= 1:
        return n
    return fib_memo(n-1) + fib_memo(n-2)

# 方式4：矩阵快速幂（O(log n)时间）
def fib_matrix(n):
    def matrix_mult(a, b):
        return [
            [a[0][0]*b[0][0] + a[0][1]*b[1][0], a[0][0]*b[0][1] + a[0][1]*b[1][1]],
            [a[1][0]*b[0][0] + a[1][1]*b[1][0], a[1][0]*b[0][1] + a[1][1]*b[1][1]]
        ]

    def matrix_pow(mat, power):
        result = [[1, 0], [0, 1]]  # 单位矩阵
        while power > 0:
            if power % 2 == 1:
                result = matrix_mult(result, mat)
            mat = matrix_mult(mat, mat)
            power //= 2
        return result

    if n <= 1:
        return n
    mat = [[1, 1], [1, 0]]
    result = matrix_pow(mat, n - 1)
    return result[0][0]
```

---

## 2. 数据类型与数据结构

### 简答题答案

**1. Python中有哪些基本数据类型？它们分别是可变的还是不可变的？**

**答案：**

| 类型          | 名称   | 可变/不可变 | 示例                   |
| ----------- | ---- | ------ | -------------------- |
| `int`       | 整数   | 不可变    | `42`, `-10`          |
| `float`     | 浮点数  | 不可变    | `3.14`, `1.0`        |
| `bool`      | 布尔值  | 不可变    | `True`, `False`      |
| `str`       | 字符串  | 不可变    | `"hello"`, `'world'` |
| `tuple`     | 元组   | 不可变    | `(1, 2, 3)`          |
| `list`      | 列表   | 可变     | `[1, 2, 3]`          |
| `dict`      | 字典   | 可变     | `{'a': 1}`           |
| `set`       | 集合   | 可变     | `{1, 2, 3}`          |
| `frozenset` | 冻结集合 | 不可变    | `frozenset([1, 2])`  |
| `NoneType`  | 空值   | 不可变    | `None`               |

**注意：**

- 不可变：对象创建后内容不能改变，修改会创建新对象
- 可变：对象创建后内容可以修改，id保持不变

---

**2. 解释列表（list）、元组（tuple）、集合（set）和字典（dict）的区别和适用场景。**

**答案：**

| 特性   | list     | tuple    | set  | dict           |
| ---- | -------- | -------- | ---- | -------------- |
| 有序   | ✓        | ✓        | ✗    | ✓ (3.7+)       |
| 可重复  | ✓        | ✓        | ✗    | 键唯一，值可重复       |
| 可索引  | ✓ (数字索引) | ✓ (数字索引) | ✗    | ✓ (键索引)        |
| 可变性  | 可变       | 不可变      | 可变   | 可变             |
| 元素类型 | 任意       | 任意       | 可哈希  | 键可哈希，值任意       |
| 表示   | `[]`     | `()`     | `{}` | `{key: value}` |

**适用场景：**

- **list**：需要动态修改的数据，有序序列，如用户列表、购物车
- **tuple**：不需要修改的数据，用作字典的键，函数多返回值，如坐标点、配置项
- **set**：去重、成员测试、集合运算（交、并、差），如标签系统
- **dict**：键值对映射，快速查找，如用户信息、配置字典

---

**3. 什么是哈希表？Python中的字典是如何实现的？**

**答案：**

**哈希表（Hash Table）** 是一种通过哈希函数将键映射到数组索引的数据结构，支持平均O(1)时间复杂度的查找、插入和删除。

**Python字典的实现：**

1. **哈希函数**：对键调用`hash()`得到哈希值
2. **计算索引**：`index = hash_value & mask`（mask = 数组长度 - 1）
3. **开放寻址法**：如果发生哈希冲突，使用探测序列找下一个空位
4. **动态扩容**：当负载因子（已用槽数/总槽数）超过2/3时，数组大小翻倍

**哈希冲突处理：**
Python使用开放寻址法中的伪随机探测，而不是链表法。

**注意事项：**

- 字典的键必须是可哈希的（不可变类型）
- 哈希值相等的两个对象不一定相等，但相等的对象哈希值一定相等
- Python 3.7+ 字典保持插入顺序

---

**4. 解释Python中的列表推导式、集合推导式和字典推导式。**

**答案：**

**列表推导式**：快速创建列表

```python
# 语法：[表达式 for 变量 in 可迭代对象 if 条件]
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]
```

**集合推导式**：快速创建集合

```python
# 语法：{表达式 for 变量 in 可迭代对象 if 条件}
unique_lengths = {len(word) for word in ['a', 'bb', 'ccc', 'a']}
# {1, 2, 3}
```

**字典推导式**：快速创建字典

```python
# 语法：{键表达式: 值表达式 for 变量 in 可迭代对象 if 条件}
word_lengths = {word: len(word) for word in ['apple', 'banana', 'cherry']}
# {'apple': 5, 'banana': 6, 'cherry': 6}

# 交换键值
original = {'a': 1, 'b': 2}
swapped = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b'}
```

**嵌套推导式**：

```python
# 扁平化二维列表
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

**5. 如何在Python中实现一个栈和队列？它们的时间复杂度是多少？**

**答案：**

**栈（Stack）**：后进先出（LIFO）

```python
# 使用list实现栈
stack = []

# 入栈：O(1) 均摊
stack.append(1)
stack.append(2)

# 出栈：O(1) 均摊
item = stack.pop()  # 返回2

# 查看栈顶：O(1)
top = stack[-1]

# 判空：O(1)
is_empty = len(stack) == 0
```

**队列（Queue）**：先进先出（FIFO）

```python
# 方式1：使用deque（推荐，两端操作都是O(1)）
from collections import deque

queue = deque()

# 入队：O(1)
queue.append(1)
queue.append(2)

# 出队：O(1)
item = queue.popleft()  # 返回1

# 方式2：使用list（不推荐，pop(0)是O(n)）
queue = []
queue.append(1)
item = queue.pop(0)  # O(n)，需要移动所有元素

# 方式3：双端队列其他操作
queue.appendleft(0)  # 左端添加：O(1)
queue.pop()          # 右端弹出：O(1)
```

**时间复杂度总结：**
| 操作 | list栈 | deque队列 | list队列（不推荐） |
|------|---------|-----------|-------------------|
| 尾部追加 | O(1)均摊 | O(1) | O(1)均摊 |
| 尾部弹出 | O(1)均摊 | O(1) | O(1)均摊 |
| 头部弹出 | - | O(1) | O(n) |
| 头部追加 | - | O(1) | O(n) |

---

### 代码题答案

**1. 写出下面代码的输出结果：**

```python
s = "hello"
print(s[1:4])
print(s[-1])
print(s[::2])
```

**答案：**
输出：

```
ell
o
hlo
```

**解释：**
Python切片语法：`s[start:end:step]`

- `s[1:4]`：从索引1到4（不包含4），字符是 e, l, l → "ell"
- `s[-1]`：倒数第一个字符 → "o"
- `s[::2]`：从头到尾，步长为2 → h (0), l (2), o (4) → "hlo"

---

**2. 写出下面代码的输出结果并解释原因：**

```python
d = {'a': 1, 'b': 2}
d2 = d
d2['c'] = 3
print(d)
```

**答案：**
输出：`{'a': 1, 'b': 2, 'c': 3}`

**解释：**
字典是可变对象，赋值操作`d2 = d`只是让d2引用同一个字典对象，而不是创建新字典。所以通过d2修改字典时，d也指向同一个被修改后的字典。

---

**3. 解释下面代码的区别：**

```python
list1 = [1, 2, 3]
list2 = list1      # 引用赋值
list3 = list1.copy()  # 浅拷贝
list4 = list1[:]   # 切片浅拷贝
```

**答案：**

```
list1 ──→ [1, 2, 3]
           ↑
list2 ────┘  # 同一个对象，list1 is list2 → True

list3 ──→ [1, 2, 3]  # 新对象，list1 is list3 → False
list4 ──→ [1, 2, 3]  # 新对象
```

**验证：**

```python
list1.append(4)
print(list1)  # [1, 2, 3, 4]
print(list2)  # [1, 2, 3, 4]  # 跟着变
print(list3)  # [1, 2, 3]     # 不变
print(list4)  # [1, 2, 3]     # 不变
```

**注意：** 对于嵌套的可变对象，浅拷贝仍然是共享的：

```python
a = [[1, 2], [3, 4]]
b = a.copy()
a[0].append(5)
print(b)  # [[1, 2, 5], [3, 4]]  # 嵌套列表还是共享的
```

---

### 编程题答案

**1. 给定两个列表，找出它们的交集和并集。**

**答案：**

```python
def intersection_and_union(list1, list2):
    # 方式1：使用集合（自动去重）
    set1 = set(list1)
    set2 = set(list2)

    intersection = list(set1 & set2)  # 或 set1.intersection(set2)
    union = list(set1 | set2)         # 或 set1.union(set2)

    return intersection, union

def intersection_preserve_duplicates(list1, list2):
    # 方式2：保留重复元素的交集
    from collections import Counter
    count1 = Counter(list1)
    count2 = Counter(list2)

    intersection = []
    for item in count1.keys() & count2.keys():
        intersection.extend([item] * min(count1[item], count2[item]))

    return intersection

# 示例
list1 = [1, 2, 2, 3, 4]
list2 = [2, 2, 3, 5]
print(intersection_and_union(list1, list2))
# 交集: [2, 3], 并集: [1, 2, 3, 4, 5]
```

---

**2. 编写一个函数，统计一个字符串中每个字符出现的次数。**

**答案：**

```python
from collections import Counter

def count_chars(s):
    # 方式1：使用Counter（推荐）
    return dict(Counter(s))

def count_chars_manual(s):
    # 方式2：手动实现
    count = {}
    for char in s:
        count[char] = count.get(char, 0) + 1
    return count

def count_chars_defaultdict(s):
    # 方式3：使用defaultdict
    from collections import defaultdict
    count = defaultdict(int)
    for char in s:
        count[char] += 1
    return dict(count)

print(count_chars("hello world"))
# {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}
```

---

**3. 实现一个LRU缓存类。**

**答案：**

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        # 移到末尾表示最近使用
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache[key] = value
            self.cache.move_to_end(key)
        else:
            if len(self.cache) >= self.capacity:
                # 移除最久未使用的（头部）
                self.cache.popitem(last=False)
            self.cache[key] = value

# 使用字典 + 双向链表实现（面试常考）
class DLinkedNode:
    def __init__(self, key=0, value=0):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache2:
    def __init__(self, capacity: int):
        self.cache = {}
        self.capacity = capacity
        self.size = 0
        # 伪头结点和伪尾结点
        self.head = DLinkedNode()
        self.tail = DLinkedNode()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _add_to_head(self, node):
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node

    def _remove_node(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _move_to_head(self, node):
        self._remove_node(node)
        self._add_to_head(node)

    def _remove_tail(self):
        node = self.tail.prev
        self._remove_node(node)
        return node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._move_to_head(node)
        return node.value

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            node.value = value
            self._move_to_head(node)
        else:
            new_node = DLinkedNode(key, value)
            self.cache[key] = new_node
            self._add_to_head(new_node)
            self.size += 1
            if self.size > self.capacity:
                removed = self._remove_tail()
                del self.cache[removed.key]
                self.size -= 1
```

---

## 3. 函数与模块

### 简答题答案

**1. 解释Python中函数的参数传递机制（传值还是传引用？）。**

**答案：**
Python的参数传递是**"传对象引用"**（pass by object reference），既不是纯粹的传值，也不是纯粹的传引用。

**关键点：**

- 函数参数接收的是对象的引用，不是对象的副本
- 如果对象是**可变的**（list, dict, set），函数内部修改会影响外部
- 如果对象是**不可变的**（int, str, tuple），函数内部"修改"实际上是创建新对象，外部不受影响

```python
def modify_list(lst):
    lst.append(4)  # 修改原列表

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)  # [1, 2, 3, 4]  # 外部改变了

def modify_num(n):
    n = n + 1     # 创建新整数

my_num = 10
modify_num(my_num)
print(my_num)    # 10  # 外部不变
```

**重新绑定 vs 修改内容：**

```python
def reassign(lst):
    lst = [4, 5, 6]  # 重新绑定局部变量，不影响外部

my_list = [1, 2, 3]
reassign(my_list)
print(my_list)  # [1, 2, 3]  # 不变！
```

---

**2. 什么是匿名函数（lambda）？它的使用场景是什么？**

**答案：**
**lambda函数**是一种小型匿名函数，可以接受任意数量的参数，但只能有一个表达式。

**语法：**

```python
lambda 参数1, 参数2, ...: 表达式
```

**与普通函数的区别：**

```python
# 普通函数
def add(x, y):
    return x + y

# lambda函数
add = lambda x, y: x + y
```

**使用场景：**

1. **作为高阶函数的参数**：
   
   ```python
   # sorted按第二个元素排序
   points = [(1, 2), (3, 1), (5, 0)]
   sorted_points = sorted(points, key=lambda x: x[1])
   # [(5, 0), (3, 1), (1, 2)]
   
   ```

# map/filter

nums = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, nums))
evens = list(filter(lambda x: x % 2 == 0, nums))

```

2. **简单的回调函数**：
```python
# Tkinter按钮点击事件
button = tk.Button(text="Click", command=lambda: print("Clicked"))
```

**lambda的限制：**

- 只能有一个表达式，不能包含多条语句
- 不能包含return语句（表达式结果自动返回）
- 不能有类型注解（Python 3.6+实际上可以，但不推荐）

---

**3. 解释Python中的模块和包的概念，以及`__init__.py`文件的作用。**

**答案：**

**模块（Module）：**

- 一个`.py`文件就是一个模块
- 包含函数、类、变量等定义
- 使用`import`语句导入

```
my_module.py
    def hello():
        print("Hello")
    PI = 3.14159
```

**包（Package）：**

- 包含多个模块的目录
- 用于组织相关的模块，避免命名冲突
- 通常包含`__init__.py`文件

```
my_package/
    __init__.py
    module1.py
    module2.py
    subpackage/
        __init__.py
        module3.py
```

**`__init__.py`的作用：**

1. **标识包目录**：告诉Python该目录是一个Python包
2. **初始化包**：在包被导入时执行，可进行初始化操作
3. **定义`__all__`**：控制`from package import *`导入哪些模块
   
   ```python
   # __init__.py
   __all__ = ['module1', 'module2']  # import *时只导入这两个
   ```
4. **导出公共API**：在`__init__.py`中导入子模块的函数、类，简化导入路径
   
   ```python
   # __init__.py
   from .module1 import ClassA, func_a
   from .module2 import ClassB
   
   ```

# 用户可以直接这样导入，而不需要知道内部结构

from my_package import ClassA, func_a

```

**注意：** Python 3.3+ 支持隐式命名空间包，不需要`__init__.py`也能识别包，但推荐仍然使用它。

---

**4. 什么是命名空间？Python中有哪些命名空间？**

**答案：**

**命名空间（Namespace）** 是一个存储变量名到对象映射的字典。不同命名空间中的名称互不干扰。

**Python中的命名空间：**

1. **内置命名空间（Built-in）**：
   - 包含Python内置函数和异常
   - 如：`print()`, `len()`, `ValueError`
   - 生命周期：Python解释器启动时创建，退出时销毁

2. **全局命名空间（Global）**：
   - 每个模块（.py文件）有自己的全局命名空间
   - 包含模块级别的变量、函数、类
   - 生命周期：模块导入时创建，解释器退出时销毁

3. **局部命名空间（Local）**：
   - 每次函数调用创建新的局部命名空间
   - 包含函数参数和内部定义的变量
   - 生命周期：函数调用时创建，返回时销毁

4. **非局部命名空间（Enclosing）**：
   - 嵌套函数的外层函数的局部命名空间
   - 用于闭包

```python
x = "global"  # 全局命名空间

def outer():
    y = "enclosing"  # outer的局部命名空间（也是inner的非局部命名空间）

    def inner():
        z = "local"  # inner的局部命名空间
        print(x, y, z)

    inner()
```

**命名空间查找顺序：** LEGB规则
Local → Enclosing → Global → Built-in

---

**5. 解释`__name__ == '__main__'`的作用和原理。**

**答案：**

**作用：** 判断Python文件是**被直接运行**还是**被导入为模块**。

**原理：**

- 每个Python文件都有一个内置变量`__name__`
- 如果文件**被直接运行**（`python script.py`），`__name__`的值是`'__main__'`
- 如果文件**被导入**（`import script`），`__name__`的值是模块名（`'script'`）

**示例：**

```python
# script.py
def hello():
    print("Hello")

if __name__ == '__main__':
    # 只有直接运行时才执行这里
    print("Running directly")
    hello()
```

```bash
# 直接运行
python script.py
# 输出:
# Running directly
# Hello

# 在Python交互环境导入
import script  # 没有输出，hello()函数可用
script.hello()  # 输出: Hello
```

**常见用途：**

1. 模块的测试代码：只在直接运行时执行测试
2. 脚本的主程序入口
3. 避免模块被导入时执行不必要的代码

---

### 代码题答案

**1. 写出下面代码的输出结果：**

```python
def outer():
    x = 10
    def inner():
        nonlocal x
        x = 20
    inner()
    print(x)

outer()
```

**答案：**
输出：`20`

**解释：**

- `nonlocal`关键字声明`x`不是局部变量，而是外层函数（outer）的变量
- 如果没有`nonlocal`，`x = 20`会在inner函数中创建一个新的局部变量x，而不会修改outer的x
- 加上`nonlocal`后，inner函数中修改的就是outer函数中的x，所以输出20

对比：

```python
def outer():
    x = 10
    def inner():
        x = 20  # 这是inner的局部变量，不影响outer
    inner()
    print(x)

outer()  # 输出: 10
```

---

**2. 解释下面代码中`map`、`filter`、`reduce`的作用：**

```python
from functools import reduce

nums = [1, 2, 3, 4, 5]
print(list(map(lambda x: x*2, nums)))
print(list(filter(lambda x: x%2 == 0, nums)))
print(reduce(lambda x, y: x+y, nums))
```

**答案：**

**`map(function, iterable)`**：将函数应用于可迭代对象的每个元素，返回结果迭代器

```python
# map(lambda x: x*2, nums) 对每个元素乘2
# 结果: [2, 4, 6, 8, 10]
```

**`filter(function, iterable)`**：过滤可迭代对象中使函数返回True的元素

```python
# filter(lambda x: x%2 == 0, nums) 保留偶数
# 结果: [2, 4]
```

**`reduce(function, iterable[, initializer])`**：对可迭代对象累积应用函数，返回单个值

```python
# reduce(lambda x, y: x+y, nums) 计算累加
# 过程: ((1+2)+3)+4)+5 = 15
# 结果: 15
```

**输出：**

```
[2, 4, 6, 8, 10]
[2, 4]
15
```

**注意：**

- Python 3中`map`和`filter`返回迭代器，需要用`list()`转换才能看到全部元素
- `reduce`在Python 3中被移到了`functools`模块

---

### 编程题答案

**1. 编写一个函数，接受任意数量的参数，返回它们的和。**

**答案：**

```python
def sum_args(*args):
    """接受任意数量的位置参数，返回它们的和"""
    return sum(args)

# 或者手动实现
def sum_args_manual(*args):
    total = 0
    for num in args:
        total += num
    return total

# 还可以支持关键字参数求和
def sum_kwargs(**kwargs):
    return sum(kwargs.values())

# 综合版
def sum_all(*args, **kwargs):
    return sum(args) + sum(kwargs.values())

# 测试
print(sum_args(1, 2, 3, 4))          # 10
print(sum_args(1, 2, 3, 4, 5, 6))    # 21
print(sum_all(1, 2, a=3, b=4))       # 10
```

---

**2. 编写一个装饰器，计算函数的执行时间。**

**答案：**

```python
import time
from functools import wraps

def timer(func):
    @wraps(func)  # 保留原函数的元信息
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"函数 {func.__name__} 执行时间: {end_time - start_time:.4f} 秒")
        return result
    return wrapper

# 使用示例
@timer
def slow_function(seconds):
    time.sleep(seconds)
    print("Done")

slow_function(1)
# 输出:
# Done
# 函数 slow_function 执行时间: 1.0013 秒

# 进阶版：支持输出单位选择
def timer(unit='s'):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            result = func(*args, **kwargs)
            elapsed = time.time() - start

            if unit == 'ms':
                elapsed *= 1000
                print(f"{func.__name__}: {elapsed:.2f} ms")
            elif unit == 'us':
                elapsed *= 1000000
                print(f"{func.__name__}: {elapsed:.2f} us")
            else:
                print(f"{func.__name__}: {elapsed:.4f} s")

            return result
        return wrapper
    return decorator

# 使用带参数的装饰器
@timer(unit='ms')
def fast_function():
    return sum(range(1000000))
```

---

(注：由于篇幅限制，文档将分章节补充。以上为前3章完整答案，后续章节将继续补充...)
