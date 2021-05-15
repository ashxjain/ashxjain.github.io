# Sequence Types

**Python Version:** 3.9.1 (Clang 12.0.0)

**iPython Version:** 7.19.0

#### Tuples

* Tuples are immutable and so are strings. A tuple is similar to strings, except that strings contains homogenous elements only:

  ```python
  In [1]: t1 = (1, 'a', True)
  
  In [2]: s1 = "tsai 4" # 4 is not an integer, it is part o string
    
  In [3]: t1[0] = 20
  ---------------------------------------------------------------------------
  TypeError                                 Traceback (most recent call last)
  <ipython-input-3-7956adee45a9> in <module>
  ----> 1 t1[0] = 20
  
  TypeError: 'tuple' object does not support item assignment
  ```

* As seen above tuple is immutable i.e. one cannot change the reference of the container, although one can change the reference of the within the element of container itself

* Tuples are lightweight data structures

* In tuples, we must remember what is element1, element2 etc:

  ```python
  In [4]: record = 'DJAI', 2018, 1, 19, 25_987, 26_072, 25_942, 26_072
  
  In [5]: symbol, year, month, day, open_, high, low, close = record
  ```

* If we just want to access few elements, we can use a more pythonic way using `*`:

  ```python
  In [6]: symbol, year, month, day, *_, close = record
  ```

* We can use `enumerate` function to get indices of each element:

  ```python
  In [9]: for index, c in enumerate("india"):
     ...:     print(index, c)
     ...:
  0 i
  1 n
  2 d
  3 i
  4 a
  ```

* A sample code using tuples in Monte Carlo Experiment (outcome of the result is Pi):

  ```python
  In [10]: from random import uniform
      ...: from math import sqrt
      ...:
      ...: def random_shot(radius):
      ...:     rand_x = uniform(-radius, radius)
      ...:     rand_y = uniform(-radius, radius)
      ...:
      ...:     if(sqrt(rand_x**2 + rand_y**2) <= radius):
      ...:         is_in_circle = True
      ...:     else:
      ...:         is_in_circle = False
      ...:
      ...:     return rand_x, rand_y, is_in_circle
      ...:
  
  In [11]: count_inside = 0
      ...: n = 1_000_000
      ...:
      ...: for i in range(n):
      ...:     *_, is_in_circle = random_shot(1)
      ...:     if is_in_circle:
      ...:         count_inside += 1
      ...:
      ...: print(f'The function gave value = {4 * count_inside / n}')
  
  The function gave value = 3.141032
  ```

#### Named Tuples

* NamedTuple shortens the amount of code as compared to writing a class and they are exactly classes. It is a class factory. A type of class factory is `type` itself:

  ```python
  In [13]: from collections import namedtuple
  
  In [14]: Pt = namedtuple('Abrakadabra', ('x', 'y'))
      ...: pt = Pt(10, 20)
      ...: pt
  Out[14]: Abrakadabra(x=10, y=20)
  
  In [15]: type(Pt)
  Out[15]: type
    
  In [17]: isinstance(pt, tuple)
  Out[17]: True
  ```

* `__eq__` function comes for free with namedtuples, whereas in class we have to define it:

  ```python
  In [19]: pt1 = Pt(10, 20)
  
  In [20]: pt2 = Pt(10, 20)
  
  In [21]: pt1 == pt2
  Out[21]: True
  
  In [23]: class Point3D:
        ...:     def __init__(self, x, y, z):
        ...:         self.x = x
        ...:         self.y = y
        ...:         self.z = z
        ...:
        ...:     def __repr__(self):
        ...:         return f"Point3D(x={self.x}, y={self.y}, z={self.z})"
        ...:
        ...:     def __eq__(self, other):
        ...:         if isinstance(other, Point3D):
        ...:             return self.x == other.x and self.y == other.y and self.z == other.z
        ...:         else:
        ...:             return False
  
  ```
  
* Perform actions like we perform with tuples:

  ```python
  In [22]: max(pt1)
  Out[22]: 20
  ```

* Tuples are used a lot in graphics, hence namedtuples are very helpful, rather than defining a class everytime

* We can specify all the elements in one go for the nametuple:

  ```python
  In [24]: City = namedtuple('City', 'name country population')
      ...: new_york = City('New York', 'USA', 8_500_000)
      ...: new_york
  Out[24]: City(name='New York', country='USA', population=8500000)
  ```

* Also, as seen above `__repr__` is already defined for us, for classes we have to define it

* NamedTupe doesn't allow field name to start with underscore:

  ```python
  In [26]: Person = namedtuple('Person', ['firstname', 'lastname', '_age', 'ssn'])
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  <ipython-input-26-b509f5673077> in <module>
  ----> 1 Person = namedtuple('Person', ['firstname', 'lastname', '_age', 'ssn'])
  
  /usr/local/Cellar/python@3.9/3.9.1_2/Frameworks/Python.framework/Versions/3.9/lib/python3.9/collections/__init__.py in namedtuple(typename, field_names, rename, defaults, module)
      397     for name in field_names:
      398         if name.startswith('_') and not rename:
  --> 399             raise ValueError('Field names cannot start with an underscore: '
      400                              f'{name!r}')
      401         if name in seen:
  
  ValueError: Field names cannot start with an underscore: '_age'
  ```

* But we can tell python to fix those errors in field names using `rename`:

  ```python
  In [29]: Person = namedtuple('Person', ['firstname', 'lastname', '_age', 'aadhaar'], rename =
      ...: True)
      ...:
      ...: eric = Person('Eric', 'Stanford', 42, 'AWFSASD1231231')
  
  In [30]: eric
  Out[30]: Person(firstname='Eric', lastname='Stanford', _2=42, aadhaar='AWFSASD1231231')
  ```

* As seen above, `_age` is no longer accessible to user, it is replaced by `_index`. This `_index` is not accessible

* We can get all the fields of the namedtuple using `_fields`:

  ```python
  In [32]: Person._fields
  Out[32]: ('firstname', 'lastname', '_2', 'aadhaar')
  ```

* We can modify a field of namedtuple:

  ```python
  In [34]: eric.firstname = "ERIC"
  ---------------------------------------------------------------------------
  AttributeError                            Traceback (most recent call last)
  <ipython-input-34-e311b77ab33b> in <module>
  ----> 1 eric.firstname = "ERIC"
  
  AttributeError: can't set attribute
  ```

* One must create a new namedtuple:

  ```python
  In [35]: Stock = namedtuple('Stock', 'symbol year month day open high low close')
  
  In [36]: djia = Stock('DJIA', 2018, 1, 25, 26_313, 26_458, 26_260, 26_393)
  
  In [37]: *values, _ = djia
  
  In [38]: values
  Out[38]: ['DJIA', 2018, 1, 25, 26313, 26458, 26260]
  
  In [39]: djia = Stock(*values, 26_393)
  
  In [40]: djia
      ...:
  Out[40]: Stock(symbol='DJIA', year=2018, month=1, day=25, open=26313, high=26458, low=26260, close=26393)
  ```

* But a way to do above is using `_replace`:

  ```python
  In [41]: djia5 = djia._replace(year=2019, day=26)
      ...: djia5
  Out[41]: Stock(symbol='DJIA', year=2019, month=1, day=26, open=26313, high=26458, low=26260, close=26393)
  ```

* We can set docs as well, can also set it for fields of the namedtuple:

  ```python
  In [42]: Stock.__doc__ = 'Representation of the stock price during the day'
  
  In [43]: Stock.close.__doc__ = 'The closing price of the stock'
  
  In [44]: help(Stock)
  ```

* Can access namedtuples like tuples:

  ```python
  In [45]: djia[:3] + (26,) + djia[4:]
  Out[45]: ('DJIA', 2018, 1, 26, 26313, 26458, 26260, 26393)
  ```

* We can also represent dictionary as named tuple:

  ```python
  In [46]: data_dict = dict(key1=100, key2=200, key3=300)
  
  In [47]: Data = namedtuple('Data', data_dict.keys())
  
  In [48]: Data._fields
  Out[48]: ('key1', 'key2', 'key3')
  
  In [49]: data_dict_2 = dict(key1=100, key3=300, key2=200)
  
  In [50]: d2 = Data(**data_dict_2)
  
  In [51]: d2
  Out[51]: Data(key1=100, key2=200, key3=300)
    
  In [63]: getattr(d2, "key1")
  Out[63]: 100
    
  In [65]: getattr(d2, "keyX", "Not exists")
  Out[65]: 'Not exists'
  ```

#### More On Sequence Types

* Sequence types are iterables which can be accessed as part of for loop

* Tuples, lists, string etc are of sequence types

* Immutable objects are hashed in memory and hence are better performing than mutable objects:

  ```python
  In [1]: s = '123'
     ...: hash(s)
  Out[1]: -3955157100701503827
  
  In [2]: r = range(10)
     ...: hash(r)
  Out[2]: -7546101314042312252
  
  In [4]: l = [1,2,3]
  In [5]: hash(l)
  ---------------------------------------------------------------------------
  TypeError                                 Traceback (most recent call last)
  <ipython-input-5-7a4e953a7f49> in <module>
  ----> 1 hash(l)
  
  TypeError: unhashable type: 'list'
      
  In [9]: s = {1,2,3}
  In [10]: hash(s)
  ---------------------------------------------------------------------------
  TypeError                                 Traceback (most recent call last)
  <ipython-input-10-9333020f3184> in <module>
  ----> 1 hash(s)
  TypeError: unhashable type: 'set'
      
  In [11]: s = frozenset(s)
  In [12]: hash(s)
  Out[12]: -272375401224217160
  ```

* We should be careful with mutable object:

  ```python
  In [13]: x = [2000]
  
  In [14]: l = x + x
  
  In [15]: l
  Out[15]: [2000, 2000]
  
  In [16]: id(l[0]), id(l[1])
  Out[16]: (4456574032, 4456574032)
  
  In [17]: l[0] is l[1]
  Out[17]: True
  ```

* We can use `clear` object to clear all the references in the memory:

  ```python
  In [18]: suits = ['Spades', 'Hearts', 'Diamonds', 'Clubs']
      ...: alias = suits
      ...: suits = []
      ...: print(suits, alias)
  [] ['Spades', 'Hearts', 'Diamonds', 'Clubs']
  
  In [19]: suits = ['Spades', 'Hearts', 'Diamonds', 'Clubs']
      ...: alias = suits
      ...: suits.clear()
      ...: print(suits, alias)
  [] []
  ```

* Lets see how the references changes on updating a mutable object:

  ```python
  In [20]: l = [1, 2, 3]
      ...: print(id(l))
      ...: l.append(4)
      ...: print(l, id(l))
  4455246272
  [1, 2, 3, 4] 4455246272
  
  In [21]: l = [1, 2, 3]
      ...: print(id(l))
      ...: l = l + [4]
      ...: print(id(l), l)
  4455193728
  4455246272 [1, 2, 3, 4]
  
  In [22]: l = [1, 2, 3, 4, 5]
      ...: print(id(l))
      ...: l.extend({'a', 'b', 'c'})
      ...: print(id(l), l)
  4457837184
  4457837184 [1, 2, 3, 4, 5, 'b', 'c', 'a']
  
  In [23]: l = [1, 2, 3, 4]
      ...: print(id(l))
      ...: popped = l.pop(1)
      ...: print(id(l), popped, l)
  4457802048
  4457802048 2 [1, 3, 4]
  ```

* We can create a new reference of the same object using `copy`:

  ```python
  In [24]: l = [1, 2, 3, 4]
      ...: print(id(l))
      ...: l2 = l.copy()
      ...: print(id(l2), l2)
  4455246272
  4458091136 [1, 2, 3, 4]
  ```

* What if the sequence object we are copying is immutable and has mutable objects as its elements? We must use `deepcopy`

#### Constant Folding

* During compile time, python tries to recognize constant expressions, evaluate them, and replaces the expression with this newly evaluated value, making the runtime leaner

* Let's see how tuple & list is handled:

  ```python
  In [25]: from dis import dis
  
  In [26]: dis(compile('(1, 2, 3, "a")', 'string', 'eval'))
    1           0 LOAD_CONST               0 ((1, 2, 3, 'a'))
                2 RETURN_VALUE
  
  In [27]: dis(compile('[1, 2, 3, "a"]', 'string', 'eval'))
    1           0 BUILD_LIST               0
                2 LOAD_CONST               0 ((1, 2, 3, 'a'))
                4 LIST_EXTEND              1
                6 RETURN_VALUE
  ```

* As seen above, tuple as stored as a single constant, where as for list it handles differently

* We can also see how long it takes for python to store tuple vs list:

  ```python
  In [28]: from timeit import timeit
  
  In [29]: timeit("(1,2,3,4,5,6,7,8,9)", number=10_000_000)
  Out[29]: 0.1642877110000427
  
  In [30]: timeit("[1,2,3,4,5,6,7,8,9]", number=10_000_000)
  Out[30]: 0.9399306859999115
  ```

* Memory is reused for tuple:

  ```python
  In [31]: l1 = [1, 2, 3, 4, 5, 6, 7, 8, 9]
      ...: t1 = (1, 2, 3, 4, 5, 6, 7, 8, 9)
  
  In [32]: l2 = list(l1)
      ...: t2 = tuple(t1)
  
  In [33]: l1 is l2, t1 is t2
  Out[33]: (False, True)
  ```

* Lets look at the overhead for creating a tuple & a list:

  ```python
  In [36]: prev = 0
      ...: for i in range(10):
      ...:     c = tuple(range(i+1))
      ...:     size_c = sys.getsizeof(c)
      ...:     delta, prev = size_c - prev, size_c
      ...:     print(f'{i+1} items: {size_c}, delta={delta}')
      ...:
  1 items: 48, delta=48
  2 items: 56, delta=8
  3 items: 64, delta=8
  4 items: 72, delta=8
  5 items: 80, delta=8
  6 items: 88, delta=8
  7 items: 96, delta=8
  8 items: 104, delta=8
  9 items: 112, delta=8
  10 items: 120, delta=8
  
  In [43]: c = []
      ...: prev = sys.getsizeof(c)
      ...: print(f'0 items: {sys.getsizeof(c)}')
      ...: for i in range(255):
      ...:     c.append(i)
      ...:     size_c = sys.getsizeof(c)
      ...:     delta, prev = size_c - prev, size_c
      ...:     print(f'{i+1} items: {size_c}, delta={delta}')
      ...:
  0 items: 56
  1 items: 88, delta=32
  2 items: 88, delta=0
  3 items: 88, delta=0
  4 items: 88, delta=0
  5 items: 120, delta=32
  6 items: 120, delta=0
  7 items: 120, delta=0
  8 items: 120, delta=0
  9 items: 184, delta=64
  10 items: 184, delta=0
  11 items: 184, delta=0
  12 items: 184, delta=0
  13 items: 184, delta=0
  14 items: 184, delta=0
  15 items: 184, delta=0
  16 items: 184, delta=0
  17 items: 248, delta=64
  18 items: 248, delta=0
  ```

  * As seen above, for list, python does over allocation. Whenever it sees an increase in elements, it over-allocates the memory. Hence we see spikes in delta

#### Copying Sequences

* One simple way of copying is by defining a new list:

  ```python
  In [44]: l1 = [1, 2, 3]
      ...:
      ...: l1_copy = []
      ...: for item in l1:
      ...:     l1_copy.append(item)
      ...:
      ...: print(l1_copy)
  [1, 2, 3]
  
  In [45]: l1 is l1_copy
  Out[45]: False
  ```

* Or using list comprehension:

  ```python
  In [46]: l1 = [1, 2, 3]
      ...: l1_copy = [item for item in l1]
      ...: print(l1_copy)
  [1, 2, 3]
  
  In [47]: l1 is l1_copy
  Out[47]: False
  ```

* And using `copy`:

  ```python
  In [48]: l1 = [1, 2, 3]
      ...: l1_copy = l1.copy()
      ...: print(l1_copy)
  [1, 2, 3]
  
  In [49]: l1 is l1_copy
  Out[49]: False
  ```

* Using `copy` library:

  ```python
  In [50]: import copy
  
  In [51]: l1 = [1, 2, 3]
      ...: l1_copy = copy.copy(l1)
      ...: print(l1_copy)
      ...: print(l1 is l1_copy)
  [1, 2, 3]
  False
  
  In [52]: t1 = (1, 2, 3)
      ...: t1_copy = copy.copy(t1)
      ...: print(t1_copy)
      ...: print(t1 is t1_copy)
  (1, 2, 3)
  True
  ```

* What we have been doing now is shallow copy. It works well for immutable objects, but for mutable objects it doesn't:

  ```python
  In [53]: v1 = [0, 0]
      ...: v2 = [0, 0]
      ...:
      ...: line1 = [v1, v2]
  
  In [54]: print(line1)
      ...: print(id(line1[0]), id(line1[1]))
  [[0, 0], [0, 0]]
  4458124480 4458161728
  
  In [55]: line2 = line1.copy()
  
  In [56]: line1 is line2
  Out[56]: False
  
  In [57]: print(id(line1[0]), id(line1[1]))
      ...: print(id(line2[0]), id(line2[1]))
  4458124480 4458161728
  4458124480 4458161728
  
  In [58]: line2[0][0] = 100
  
  In [59]: line2, line1
  Out[59]: ([[100, 0], [0, 0]], [[100, 0], [0, 0]])
  ```

* As seen above updating `line2`, ended up changing `line1`. One way to fix this is using list comprehension:

  ```python
  In [60]: line2 = [item[:] for item in line1]
  
  In [61]: print(id(line1[0]), id(line1[1]))
      ...: print(id(line2[0]), id(line2[1]))
  4458124480 4458161728
  4458371520 4457934016
  
  In [62]: line1[0][0] = 100
      ...: print(line1)
      ...: print(line2)
  [[100, 0], [0, 0]]
  [[100, 0], [0, 0]]
  ```

* But above will work only for two level, what if there are multiple mutable objects, nested. We must use `deepcopy` for this:

  ```python
  In [63]: line2 = copy.deepcopy(line1)
      ...: print(id(line1[0]), id(line1[1]))
      ...: print(id(line2[0]), id(line2[1]))
  4458124480 4458161728
  4458157888 4457728640
  ```


#### Custom Sequences

* `__getitem__` is used to retreive individual element from a sequence

  ```python
  In [1]: my_list = [0, 1, 2, 3, 4, 5]
  
  In [2]: my_list.__getitem__(0)
  Out[2]: 0
  
  In [3]: my_list.__getitem__(slice(0,6,2))
  Out[3]: [0, 2, 4]
  ```

* We can mimick Python's for loop using `__getitem__`:

  ```python
  In [5]: for item in my_list:
     ...:     print(item ** 2, end=',')
     ...:
  0,1,4,9,16,25,
  In [6]: index = 0
     ...:
     ...: while True:
     ...:     try:
     ...:         item = my_list.__getitem__(index)
     ...:     except IndexError:
     ...:         # reached the end of the sequence
     ...:         break
     ...:     # do something with the item...
     ...:     print(item ** 2, end=',')
     ...:     index += 1
     ...:
  0,1,4,9,16,25,
  ```

* We can use `__getitem__` on other classes as well:

  ```python
  In [7]: class MySequence:
     ...:     def __getitem__(self, index):
     ...:         print(type(index), index)
     ...:
  
  In [8]: my_seq = MySequence()
  
  In [9]: my_seq[0:2]
  <class 'slice'> slice(0, 2, None)
  
  In [10]: my_seq[100]
  <class 'int'> 100
  ```

* A note on slice object as we will be using in future examples:

  ```python
  In [11]: l = 'python'
      ...: s = slice(0, 6, 2)
      ...: l[s]
  Out[11]: 'pto'
  
  In [12]: s.start, s.stop, s.step
  Out[12]: (0, 6, 2)
    
  In [13]: s.indices(6)
  Out[13]: (0, 6, 2)
  
  In [14]: s.indices(5)
  Out[14]: (0, 5, 2)
  ```

* Let's write a sample code to get Fibonacci number by creating a new sequence type:

  ```python
  In [16]: from functools import lru_cache
  
  In [17]: class Fib:
      ...:     def __init__(self, n):
      ...:         self._n = n
      ...:
      ...:     def __len__(self):
      ...:         return self._n
      ...:
      ...:     def __getitem__(self, s):
      ...:         if isinstance(s, int):
      ...:             # single item requested
      ...:             if s < 0:
      ...:                 s = self._n + s
      ...:             if s < 0 or s > self._n - 1:
      ...:                 raise IndexError
      ...:             return self._fib(s)
      ...:         else:
      ...:             # slice being requested
      ...:             idx = s.indices(self._n)
      ...:             rng = range(idx[0], idx[1], idx[2])
      ...:             return [self._fib(n) for n in rng]
      ...:
      ...:     @lru_cache(2**32)
      ...:     def _fib(self, n):
      ...:         if n < 2:
      ...:             return 1
      ...:         else:
      ...:             return self._fib(n-1) + self._fib(n-2)
      ...: f = Fib(10)
  ```

  * So by defining `__getitem__` & `__len__` method, we are able to make a custom sequence

    ```python
    In [62]: len(f)
    Out[62]: 10
    
    In [63]: f[1]
    Out[63]: 1
    
    In [64]: f[2]
    Out[64]: 2
    
    In [65]: f[6]
    Out[65]: 13
    
    In [66]: f[9]
    Out[66]: 55
    
    In [67]: f[1:6]
    Out[67]: [1, 2, 3, 5, 8]
      
    In [68]: f[10]
    ---------------------------------------------------------------------------
    IndexError                                Traceback (most recent call last)
    <ipython-input-68-2887197187eb> in <module>
    ----> 1 f[10]
    ```

#### Sorting

* Any iterable can be sortable, but it must be finite and we should be able to do pairwise comparison

  ```python
  In [33]: d = {3: 100, 2: 200, 1: 10}
      ...: sorted(d)
  Out[33]: [1, 2, 3]
  
  In [34]:
  
  In [34]: d = {'a': 100, 'b': 50, 'c': 10}
      ...: sorted(d, key=lambda k: d[k])
  Out[34]: ['c', 'b', 'a']
  
  In [34]: d = {'a': 100, 'b': 50, 'c': 10}
      ...: sorted(d, key=lambda k: d[k])
  Out[34]: ['c', 'b', 'a']
  
  In [35]: t = 'this', 'parrot', 'is', 'a', 'late', 'bird', 'crow'
      ...: sorted(t)
  Out[35]: ['a', 'bird', 'crow', 'is', 'late', 'parrot', 'this']
  
  In [36]: sorted(t, key = lambda x: len(x)) # stable sorting
  Out[36]: ['a', 'is', 'this', 'late', 'bird', 'crow', 'parrot']
  ```

* As seen above python performs stable sorting

#### List Comprehension

* Let's how python handles list comprehension internally:

  ```python
  In [46]: import dis
  
  In [47]: compiled_code = compile('[i**2 for i in (1, 2, 3)]',
      ...:                         filename='', mode='eval')
  
  In [48]: dis.dis(compiled_code)
    1           0 LOAD_CONST               0 (<code object <listcomp> at 0x10ea982f0, file "", line 1>)
                2 LOAD_CONST               1 ('<listcomp>')
                4 MAKE_FUNCTION            0
                6 LOAD_CONST               2 ((1, 2, 3))
                8 GET_ITER
               10 CALL_FUNCTION            1
               12 RETURN_VALUE
  
  Disassembly of <code object <listcomp> at 0x10ea982f0, file "", line 1>:
    1           0 BUILD_LIST               0
                2 LOAD_FAST                0 (.0)
          >>    4 FOR_ITER                12 (to 18)
                6 STORE_FAST               1 (i)
                8 LOAD_FAST                1 (i)
               10 LOAD_CONST               0 (2)
               12 BINARY_POWER
               14 LIST_APPEND              2
               16 JUMP_ABSOLUTE            4
          >>   18 RETURN_VALUE
  ```

* As seen above python internally creates a function `MAKE_FUNCTION`

* Hence list comprehension can access variables of local scope, global scope etc:

  ```python
  In [49]: [ [i * j for j in range(1, 11)] for i in range(1, 11)]
  Out[49]:
  [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
   [2, 4, 6, 8, 10, 12, 14, 16, 18, 20],
   [3, 6, 9, 12, 15, 18, 21, 24, 27, 30],
   [4, 8, 12, 16, 20, 24, 28, 32, 36, 40],
   [5, 10, 15, 20, 25, 30, 35, 40, 45, 50],
   [6, 12, 18, 24, 30, 36, 42, 48, 54, 60],
   [7, 14, 21, 28, 35, 42, 49, 56, 63, 70],
   [8, 16, 24, 32, 40, 48, 56, 64, 72, 80],
   [9, 18, 27, 36, 45, 54, 63, 72, 81, 90],
   [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]]
  ```

* Another example of pascal triange:

  ```python
  In [60]: from math import factorial
      ...:
      ...: def combo(n, k):
      ...:     return factorial(n) // (factorial(k) * factorial(n-k))
      ...:
      ...: size = 10  # global variable
      ...: pascal = [ [combo(n, k) for k in range(n+1)] for n in range(size+1) ]
      ...:
      ...: pascal
  Out[60]:
  [[1],
   [1, 1],
   [1, 2, 1],
   [1, 3, 3, 1],
   [1, 4, 6, 4, 1],
   [1, 5, 10, 10, 5, 1],
   [1, 6, 15, 20, 15, 6, 1],
   [1, 7, 21, 35, 35, 21, 7, 1],
   [1, 8, 28, 56, 70, 56, 28, 8, 1],
   [1, 9, 36, 84, 126, 126, 84, 36, 9, 1],
   [1, 10, 45, 120, 210, 252, 210, 120, 45, 10, 1]]
  ```

  