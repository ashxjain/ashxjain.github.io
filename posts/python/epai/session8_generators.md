# Generators and Iteration tools

**Python Version:** 3.9.1 (Clang 12.0.0)

**iPython Version:** 7.19.0

### Generators

* `yield` is like `return` statement, only difference is that on execution of `yield` the code gets paused over there and on next call it continues from next statement onwards. Once we execute last `yield`, `StopIteration` exception is thrown

  ```python
  In [1]: def squares(sentinel):
     ...:     i = 0
     ...:     while True:
     ...:         if i < sentinel:
     ...:             yield i**2
     ...:             i += 1 # note how we can increment **after** the yield
     ...:         else:
     ...:             return 'all done!'
     ...:
  
  In [2]: for num in squares(5):
     ...:     print(num)
     ...:
  0
  1
  4
  9
  16
  ```

* Since generators are iterators, it will implement the interator protocol:

  ```python
  In [7]: '__next__' in dir(check)
  Out[7]: True
  
  In [8]: '__iter__' in dir(check)
  Out[8]: True
  
  In [9]: id(check) == id(iter(check))
  Out[9]: True
  ```

* So `__iter__` & `__next__` is all defined by `yield` function

* All generators are iterators, but not all iterators are generators

* Generators are exhaustive as well, this is good as it does lazy computation:

  ```python
  In [22]: import math
  In [23]: def factorials(n):
      ...:     for i in range(n):
      ...:         yield math.factorial(i)
      ...:
  
  In [24]: facts = factorials(5)
  
  In [25]: list(facts)
  Out[25]: [1, 1, 2, 6, 24]
  
  In [26]: list(facts)
  Out[26]: []
  
  In [27]: next(facts)
  StopIteration:
  ```

### Making an Iterable from a Generator

* As we now know, generators are iterators.

* This means that they become exhausted - so sometimes we want to create an iterable instead. There's no magic here, we simply have to implement a class that implements the iterable protocol:

  ```python
  In [28]: def squares_gen(n):
      ...:     for i in range(n):
      ...:         yield i ** 2
      ...:
  
  In [29]: class Squares:
      ...:     def __init__(self, n):
      ...:         self.n = n
      ...:
      ...:     def __iter__(self):
      ...:         return squares_gen(self.n)
      ...:
  
  In [30]: sq = Squares(5)
  
  In [31]: [num for num in sq]
  Out[31]: [0, 1, 4, 9, 16]
  
  In [32]: [num for num in sq] # No longer exhaustive
  Out[32]: [0, 1, 4, 9, 16]
  ```

### Generator Expressions

* Just like list comprehension, if we replace `[` with `(`, we get generator:

  ```python
  In [33]: l = [i ** 2 for i in range(5)]
  
  In [34]: type(l)
  Out[34]: list
  
  In [35]: g = (i ** 2 for i in range(5))
  
  In [36]: type(g)
  Out[36]: generator
  
  In [37]: for item in g:
      ...:     print(item)
  0
  1
  4
  9
  16
  
  In [38]: for item in g:
      ...:     print(item)
  ```

* To understand what is happening internal, we can use `dis` library:

  ```python
  In [42]: exp = compile('(i ** 2 for i in range(5))', filename='<string>', mode='eval')
  
  In [43]: dis.dis(exp)
    1           0 LOAD_CONST               0 (<code object <genexpr> at 0x108928240, file "<string>", line 1>)
                2 LOAD_CONST               1 ('<genexpr>')
                4 MAKE_FUNCTION            0
                6 LOAD_NAME                0 (range)
                8 LOAD_CONST               2 (5)
               10 CALL_FUNCTION            1
               12 GET_ITER
               14 CALL_FUNCTION            1
               16 RETURN_VALUE
  
  Disassembly of <code object <genexpr> at 0x108928240, file "<string>", line 1>:
    1           0 LOAD_FAST                0 (.0)
          >>    2 FOR_ITER                14 (to 18)
                4 STORE_FAST               1 (i)
                6 LOAD_FAST                1 (i)
                8 LOAD_CONST               0 (2)
               10 BINARY_POWER
               12 YIELD_VALUE
               14 POP_TOP
               16 JUMP_ABSOLUTE            2
          >>   18 LOAD_CONST               1 (None)
               20 RETURN_VALUE
  ```

* Another example of generator expressions:

  ```python
  In [44]: start = 1
      ...: stop = 10
      ...:
      ...: mult_list = ( [i * j
      ...:                for j in range(start, stop+1)]
      ...:              for i in range(start, stop+1))
  
  In [45]: for item in mult_list:
      ...:     print(item)
      ...:
  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
  [3, 6, 9, 12, 15, 18, 21, 24, 27, 30]
  [4, 8, 12, 16, 20, 24, 28, 32, 36, 40]
  [5, 10, 15, 20, 25, 30, 35, 40, 45, 50]
  [6, 12, 18, 24, 30, 36, 42, 48, 54, 60]
  [7, 14, 21, 28, 35, 42, 49, 56, 63, 70]
  [8, 16, 24, 32, 40, 48, 56, 64, 72, 80]
  [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
  [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
  ```

* Let's look at the memory usage when we use a list comprehension and a generator comprehension:

  ```python
  In [46]: import tracemalloc
  
  In [47]: def mult_list(size):
      ...:     l = [ [i * j for j in range(start, stop+1)] for i in range(start, s
      ...: top+1)]
      ...:     for row in l:
      ...:         for item in row:
      ...:             pass
      ...:     stats = tracemalloc.take_snapshot().statistics('lineno')
      ...:     print(stats[0].size, 'bytes')
  
  In [48]: def mult_gen(size):
      ...:     l = ( (i * j for j in range(start, stop+1)) for i in range(start, s
      ...: top+1))
      ...:     for row in l:
      ...:         for item in row:
      ...:             pass
      ...:     stats = tracemalloc.take_snapshot().statistics('lineno')
      ...:     print(stats[0].size, 'bytes')
  
  In [49]: tracemalloc.stop()
  In [50]: tracemalloc.clear_traces()
  In [51]: tracemalloc.start()
  In [52]: mult_list(1000000)
  15456 bytes
  
  In [53]: tracemalloc.stop()
  In [54]: tracemalloc.clear_traces()
  In [55]: tracemalloc.start()
  In [56]: mult_gen(1000000)
  8704 bytes
  ```

* Generator consumes less memory as it does lazy iteration

### Yield From

* We can read from a nested generator expression using the following code:

  ```python
  In [59]: def matrix(n):
      ...:     gen = ( (i * j for j in range(1, n+1))
      ...:             for i in range(1, n+1)
      ...:           )
      ...:     return gen
      ...:
  
  In [60]: def matrix_iterator(n):
      ...:     for row in matrix(n):
      ...:         for item in row:
      ...:             yield item
      ...:
  
  In [61]: for i in matrix_iterator(3):
      ...:     print(i)
      ...:
  1
  2
  3
  2
  4
  6
  3
  6
  9
  ```

* One other way of writing this is by using `yield from <iterator>` :

  ```python
  In [62]: def matrix_iterator(n):
      ...:     for row in matrix(n):
      ...:         yield from row
      ...:
  
  In [63]: for i in matrix_iterator(3):
      ...:     print(i)
      ...:
  1
  2
  3
  2
  4
  6
  3
  6
  9
  ```

### Aggregators

* Takes generators and aggregates them:

  ```python
  In [64]: def squares(n):
      ...:     for i in range(n):
      ...:         yield i**2
      ...:
  
  In [65]: list(squares(5))
  Out[65]: [0, 1, 4, 9, 16]
  
  In [66]: min(squares(5))
  Out[66]: 0
  ```

* **Any**:

  * The `any` function is a predicate (a function that returns `True` or `False`) that takes an iterable and returns `True` if any one element of that iterable is True (or have an associated True truth-value, i.e. **truthy**).

  * Note: By default custom objects are always truthy:

    ```python
    In [67]: class Person:
        ...:     pass
    
    In [68]: p = Person()
    
    In [69]: bool(p)
    Out[69]: True
    ```

  * For sequences, `len` is called internally:

    ```python
    In [70]: class MySeq:
        ...:     def __init__(self, n):
        ...:         self.n = n
        ...:         print("init")
        ...:
        ...:     def __len__(self):
        ...:         print("len")
        ...:         return self.n
        ...:
        ...:     def __getitem__(self, s):
        ...:         print("getitem")
        ...:         pass
        ...:
    
    In [71]: my_seq = MySeq(0)
    init
    
    In [72]: bool(my_seq)
    len
    Out[72]: False
    
    In [73]: my_seq = MySeq(10)
    init
    
    In [74]: bool(my_seq)
    len
    Out[74]: True
    
    In [75]: any([0, '', None, 'hello'])
    Out[75]: True
    ```

* **All**: Here all elements must be true:

  ```python
  In [76]: all([1, 'abc', [1, 2], range(5)])
  Out[76]: True
  ```

* Simple example is a function to figure out if all the elements in a list is a Number or not:

  ```python
  In [79]: from numbers import Number
  
  In [77]: def is_number(x):
      ...:     return is_instance(x, Number)
  
  In [80]: l = [10, 20, 30, 40]
      ...: all(isinstance(x, Number) for x in l)
  Out[80]: True
  
  In [81]: l = [10, 20, 30, 40, 'hello']
      ...: all(isinstance(x, Number) for x in l)
  Out[81]: False
  ```

### Slicing Iterables

* We can slice directly using this format: `list[slice_start:slice_end]` or we can define `slice` object and pass it to list:

  ```python
  In [83]: l = [1, 2, 3, 4, 5]
  
  In [84]: l[0:2]
  Out[84]: [1, 2]
  
  In [85]: s = slice(0, 2)
  
  In [86]: l[s]
  Out[86]: [1, 2]
  ```

* This doesnot work with iterables that are not of sequence types:

  ```python
  In [87]: import math
      ...:
      ...: def factorials(n):
      ...:     for i in range(n):
      ...:         yield math.factorial(i)
      ...:
  
  In [88]: facts = factorials(100)
  
  In [89]: facts[0:2]
  TypeError: 'generator' object is not subscriptable
  ```

* But we write a function to implement this:

  ```python
  In [90]: def slice_(iterable, start, stop):
      ...:     for _ in range(0, start):
      ...:         next(iterable)
      ...:
      ...:     for _ in range(start, stop):
      ...:         yield(next(iterable))
      ...:
  
  In [91]: list(slice_(factorials(100), 1, 5))
  Out[91]: [1, 2, 6, 24]
  ```

* There is already a inbuilt function to do this:

  ```python
  In [92]: from itertools import islice
  
  In [93]: islice(factorials(10), 0, 3)
  Out[93]: <itertools.islice at 0x1089d5220>
  
  In [94]: list(islice(factorials(10), 0, 3))
  Out[94]: [1, 1, 2]
  ```

* This becomes very useful for infinite iterators:

  ```python
  In [95]: def factorials():
      ...:     index = 0
      ...:     while True:
      ...:         print(f'yielding factorial({index})...')
      ...:         yield math.factorial(index)
      ...:         index += 1
      ...:
  
  In [96]: list(islice(factorials(), 3))
  yielding factorial(0)...
  yielding factorial(1)...
  yielding factorial(2)...
  Out[96]: [1, 1, 2]
  ```

* Note: `islice` is using `yield` to get next value. So it does consume the generator and hence is exhaustive:

  ```python
  In [98]: f = factorials()
  
  In [99]: list(islice(f, 3))
  yielding factorial(0)...
  yielding factorial(1)...
  yielding factorial(2)...
  Out[99]: [1, 1, 2]
  
  In [100]: list(islice(f, 3))
  yielding factorial(3)...
  yielding factorial(4)...
  yielding factorial(5)...
  Out[100]: [6, 24, 120]
  ```

### Selecting and Filtering Iterators

* `filter` function is also a lazy function:

  ```python
  In [101]: def is_odd(x):
       ...:     return x % 2 == 1
       ...:
  
  In [102]: def gen_cubes(n):
       ...:     for i in range(n):
       ...:         print(f'yielding {i}')
       ...:         yield i**3
       ...:
  
  In [103]: list(filter(is_odd, gen_cubes(5)))
  yielding 0
  yielding 1
  yielding 2
  yielding 3
  yielding 4
  Out[103]: [1, 27]
  ```

* There is another method called `filterfalse` which does the opposite of the predicate function:

  ```python
  In [104]: from itertools import filterfalse
  
  In [105]: evens = filterfalse(is_odd, gen_cubes(5))
  
  In [106]: list(evens)
  yielding 0
  yielding 1
  yielding 2
  yielding 3
  yielding 4
  Out[106]: [0, 8, 64]
  ```

* **takewhile**: Like filter, only difference is that it stops taking value once the predicate fails:

  ```python
  In [107]: from math import sin, pi
       ...:
       ...: def sine_wave(n):
       ...:     start = 0
       ...:     max_ = 2 * pi
       ...:     step = (max_ - start) / (n-1)
       ...:     for _ in range(n):
       ...:         yield round(sin(start), 2)
       ...:         start += step
       ...:
  
  In [108]: list(filter(lambda x: 0 <= x <= 0.9, sine_wave(15)))
  Out[108]: [0.0, 0.43, 0.78, 0.78, 0.43, 0.0, -0.0]
  
  In [109]: from itertools import takewhile
  In [110]: list(takewhile(lambda x: 0 <= x <= 0.9, sine_wave(15)))
  Out[110]: [0.0, 0.43, 0.78]
  ```

* **dropwhile**: Starts the iteration once the predicate becomes false

  ```python
  In [111]: from itertools import dropwhile
  
  In [112]: l = [1, 3, 5, 2, 1]
  
  In [113]: list(dropwhile(lambda x: x < 5, l))
  Out[113]: [5, 2, 1]
  ```

* **compress**: A filter that takes two iterables, one is data and other is selector (of different length) and outputs values that matches the index of truth values of selector:

  ```python
  In [114]: data = ['a', 'b', 'c', 'd', 'e']
       ...: selectors = [True, False, 1, 0]
  
  In [115]: list(zip(data, selectors))
  Out[115]: [('a', True), ('b', False), ('c', 1), ('d', 0)]
  
  In [116]: [item for item, truth_value in zip(data, selectors) if truth_value]
  Out[116]: ['a', 'c']
  
  # Compress method does the same thing, but does it lazily
  In [117]: from itertools import compress
  In [118]: list(compress(data, selectors))
  Out[118]: ['a', 'c']
  ```

### Infinite Iterators

* There are three functions in `itertools` module that generate infinite iterators:

  ```
  from itertools import (
      count,
      cycle,
      repeat, 
      islice)
  ```

  * `count` : just like range method, but can also take `float` as argument

    ```python
    In [2]: g = count(10.5, 0.5)
    
    In [3]: list(islice(g, 5))
    Out[3]: [10.5, 11.0, 11.5, 12.0, 12.5]
    
    In [4]: g = count(1+1j, 1+2j)
    
    In [5]: list(islice(g, 5))
    Out[5]: [(1+1j), (2+3j), (3+5j), (4+7j), (5+9j)]
    ```

  * `cycle`: it will cycle through the iterable infinitely:

    ```python
    In [7]: g = cycle(('red', 'green', 'blue'))
    
    In [8]: list(islice(g, 8))
    Out[8]: ['red', 'green', 'blue', 'red', 'green', 'blue', 'red', 'green']
      
    In [9]: def colors():
       ...:     yield 'red'
       ...:     yield 'green'
       ...:     yield 'blue'
       ...:
    
    In [10]: cols = colors()
        ...: g = cycle(cols)
    
    In [11]: list(islice(g, 10))
    Out[11]: ['red', 'green', 'blue', 'red', 'green', 'blue', 'red', 'green', 'blue', 'red']
    ```

  * `repeat`: repeats the objects infinitely by default, we can set a limit as well:

    ```python
    In [13]: g = repeat('Python')
        ...: for _ in range(5):
        ...:     print(next(g))
        ...:
    Python
    Python
    Python
    Python
    Python
    
    In [14]: g = repeat('Python', 4)
    
    In [15]: list(g)
    Out[15]: ['Python', 'Python', 'Python', 'Python']
    ```

    * `repeat` repeats the same object, so if we modify one, it'll change others as well:

      ```python
      In [16]: g = repeat([1,2,3], 2)
      
      In [17]: o = list(g)
      
      In [18]: o
      Out[18]: [[1, 2, 3], [1, 2, 3]]
      
      In [19]: o[0][0] = 8
      
      In [20]: o
      Out[20]: [[8, 2, 3], [8, 2, 3]]
      ```

### Chaining and Teeing Iterators

* Often we would like to chain iterators/generators:

  ```python
  In [21]: l1 = (i**2 for i in range(4))
      ...: l2 = (i**2 for i in range(4, 8))
      ...: l3 = (i**2 for i in range(8, 12))
  
  In [22]: for gen in (l1, l2, l3):
      ...:     for item in gen:
      ...:         print(item)
      ...:
  0
  1
  4
  9
  16
  25
  36
  49
  64
  81
  100
  121
  ```

* Here's another way to do this:

  ```python
  In [23]: def chain_iterables(*iterables):
      ...:     for iterable in iterables:
      ...:         yield from iterable
  
  In [25]: l1 = (i**2 for i in range(4))
      ...: l2 = (i**2 for i in range(4, 8))
      ...: l3 = (i**2 for i in range(8, 12))
      ...:
      ...: for item in chain_iterables(l1, l2, l3):
      ...:     print(item)
      ...:
  0
  1
  4
  9
  16
  25
  36
  49
  64
  81
  100
  121
  ```

* There's also an in-built function to do this:

  ```python
  In [26]: from itertools import chain
  
  In [27]: l1 = (i**2 for i in range(4))
      ...: l2 = (i**2 for i in range(4, 8))
      ...: l3 = (i**2 for i in range(8, 12))
      ...:
      ...: for item in chain(l1, l2, l3):
      ...:     print(item)
  ```

* `chain` expects an iterator:

  ```python
  In [28]: def squares():
      ...:     yield (i**2 for i in range(4))
      ...:     yield (i**2 for i in range(4, 8))
      ...:     yield (i**2 for i in range(8, 12))
      ...:
  
  In [29]: for item in chain(*squares()):
      ...:     print(item)
      ...:
  0
  1
  4
  9
  16
  25
  36
  49
  64
  81
  100
  121
  ```

* `*squares()` unpacking is not lazy operation. We can make it lazy operation by doing the following:

  ```python
  In [30]: def squares():
      ...:     print('yielding 1st item')
      ...:     yield (i**2 for i in range(4))
      ...:     print('yielding 2nd item')
      ...:     yield (i**2 for i in range(4, 8))
      ...:     print('yielding 3rd item')
      ...:     yield (i**2 for i in range(8, 12))
      ...:
  
  In [31]: c = chain.from_iterable(squares())
  
  In [32]: for item in c:
      ...:     print(item)
      ...:
  yielding 1st item
  0
  1
  4
  9
  yielding 2nd item
  16
  25
  36
  49
  yielding 3rd item
  64
  81
  100
  121
  ```

* Same as above can be achieved by doing following as well:

  ```python
  In [33]: def chain_(*args):
      ...:     for item in args:
      ...:         yield from item
      ...:
  
  In [34]: def chain_iter(iterable):
      ...:     for item in iterable:
      ...:         yield from item
      ...:
  ```

### Mapping and Reducing

* We have seen `map` already now:

  ```python
  In [35]: def add(t):
      ...:     return t[0] + t[1]
      ...:
  
  In [36]: list(map(add, [(0,0), [1,1], range(2,4)]))
  Out[36]: [0, 2, 5]
  ```

* But what if we want to do the following:

  ```python
  In [37]: def add(x, y):
      ...:     return x + y
      ...:
  
  In [38]: t = (2, 3)
      ...: add(*t)
  Out[38]: 5
  
  In [39]: list(map(add, [(0,0), (1,1), (2,2)]))
  TypeError: add() missing 1 required positional argument: 'y'
  ```

* To achieve this, we must use `starmap`:

  ```python
  In [40]: from itertools import starmap
  
  In [41]: list(starmap(add, [(0,0), (1,1), (2,2)]))
  Out[41]: [0, 2, 4]
  ```

* We have seen `reduce` as well:

  ```python
  In [43]: from functools import reduce
  
  In [44]: reduce(lambda x, y: x*y, [1, 2, 3, 4], 10) # 10 is initial value
  Out[44]: 240
  ```

* But above only shows final value, what if we want to see intermediate values?

* To do this we can write the following function:

  ```python
  In [45]: def running_reduce(fn, iterable, start=None):
      ...:     it = iter(iterable)
      ...:     if start is None:
      ...:         accumulator = next(it)
      ...:     else:
      ...:         accumulator = start
      ...:     yield accumulator
      ...:
      ...:     for item in it:
      ...:         accumulator = fn(accumulator, item)
      ...:         yield accumulator
      ...:
  
  In [46]: import operator
  
  In [47]: list(running_reduce(operator.add, [10, 20, 30]))
  Out[47]: [10, 30, 60]
  ```

* There is already an in-built function for this:

  ```python
  In [48]: from itertools import accumulate
  
  In [49]: list(accumulate([10, 20, 30])) # default to sum
  Out[49]: [10, 30, 60]
  
  In [50]: list(accumulate([1, 2, 3, 4], operator.mul))
  Out[50]: [1, 2, 6, 24]
  ```

### Zipping

* Already seen `zip`:

  ```python
  In [51]: l1 = [1, 2, 3, 4, 5]
      ...: l2 = [1, 2, 3, 4]
      ...: l3 = [1, 2, 3]
  
  In [52]: list(zip(l1, l2, l3))
  Out[52]: [(1, 1, 1), (2, 2, 2), (3, 3, 3)]
  ```

* Works with iterables as well:

  ```python
  In [53]: def integers(n):
      ...:     for i in range(n):
      ...:         yield i
      ...:
      ...: def squares(n):
      ...:     for i in range(n):
      ...:         yield i**2
      ...:
      ...: def cubes(n):
      ...:     for i in range(n):
      ...:         yield i**3
      ...:
  
  In [54]: iter1 = integers(6)
      ...: iter2 = squares(5)
      ...: iter3 = cubes(4)
  
  In [55]: list(zip(iter1, iter2, iter3))
  Out[55]: [(0, 0, 0), (1, 1, 1), (2, 4, 8), (3, 9, 27)]
  ```

* For zipping objects on different size, we can use `zip_longest` to zip till the longest iteratable:

  ```python
  In [56]: from itertools import zip_longest
  
  In [57]: l1 = [1, 2, 3, 4, 5]
      ...: l2 = [1, 2, 3, 4]
      ...: l3 = [1, 2, 3]
  
  In [58]: list(zip_longest(l1, l2, l3, fillvalue='N/A'))
  Out[58]: [(1, 1, 1), (2, 2, 2), (3, 3, 3), (4, 4, 'N/A'), (5, 'N/A', 'N/A')]
  ```

  