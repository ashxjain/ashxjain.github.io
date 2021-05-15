# Iterables and Iterators

**Python Version:** 3.9.1 (Clang 12.0.0)

**iPython Version:** 7.19.0

### Iterating Collections

* For custom sequence types, we need a way to figure out a way to get next element from the collection

* Iterables provide a method get next element from the collection

* Let's see how we can roll out our own `next` method for custom collection:

  ```python
  In [1]: class Squares:
     ...:     def __init__(self):
     ...:         self.i = 0
     ...:
     ...:     def next_(self):
     ...:         result = self.i ** 2
     ...:         self.i += 1
     ...:         return result
     ...:
  
  In [2]: sq = Squares()
  
  In [3]: sq.next_()
  Out[3]: 0
  
  In [4]: sq.next_()
  Out[4]: 1
  
  In [5]: sq.next_()
  Out[5]: 4
  ```

* Since `next` is a built-in method, we are using `next_` as function name

* We can also add a limit on how many times `next` function can be called by using `StopIteration` exception, this makes the collection `exhaustible`:

  ```python
  In [6]: class Squares:
     ...:     def __init__(self, length):
     ...:         self.length = length
     ...:         self.i = 0
     ...:
     ...:     def next_(self):
     ...:         if self.i >= self.length:
     ...:             raise StopIteration
     ...:         else:
     ...:             result = self.i ** 2
     ...:             self.i += 1
     ...:             return result
     ...:
     ...:     def __len__(self):
     ...:         return self.length
    
  In [7]: sq = Squares(5)
     ...: while True:
     ...:     try:
     ...:         print(sq.next_())
     ...:     except StopIteration:
     ...:         # reached end of iteration
     ...:         # stop looping
     ...:         break
  
  In [8]: len(sq)
  Out[8]: 5
  ```

* But we can't use this in a `for` loop as Python doesn't recognize `next_` method.

  ```python
  In [9]: for i in Squares(10):
     ...:     print(i)
  
  TypeError: 'Squares' object is not iterable
  ```

* We must define `__next__` method for this to work:

  ```python
  In [10]: class Squares:
      ...:     def __init__(self, length):
      ...:         self.length = length
      ...:         self.i = 0
      ...:
      ...:     def __next__(self):
      ...:         if self.i >= self.length:
      ...:             raise StopIteration
      ...:         else:
      ...:             result = self.i ** 2
      ...:             self.i += 1
      ...:             return result
      ...:
  ```

* But is it now iterable using `for` loop?

  ```python
  In [19]: sq = Squares(5)
  
  In [20]: for item in sq:
      ...:     print(item)
      ...:
  
  TypeError: 'Squares' object is not iterable
  ```

* Python still doesn't know that this object is iterable. To do that, we must define two methods: `__next__` and `__iter__`:

  ```python
  In [15]: class Squares:
      ...:     def __init__(self, length):
      ...:         self.length = length
      ...:         self.i = 0
      ...:
      ...:     def __iter__(self):
      ...:         return self
      ...:
      ...:     def __next__(self):
      ...:         if self.i >= self.length:
      ...:             raise StopIteration
      ...:         else:
      ...:             result = self.i ** 2
      ...:             self.i += 1
      ...:             return result
      ...:
  
  In [16]: sq = Squares(5)
  
  In [17]: for item in sq:
      ...:     print(item)
      ...:
  0
  1
  4
  9
  16
  ```

* `__iter__` just returns `self` object. And this makes the class an `iterable` object

* Just like Python's built-in function `next` calls `__next__` method. There is `iter` function which calls `__iter__` method

  ```python
  In [25]: sq = Squares(5)
  
  In [26]: id(sq)
  Out[26]: 4465025232
  
  In [27]: id(sq.__iter__())
  Out[27]: 4465025232
  
  In [28]: id(iter(sq))
  Out[28]: 4465025232
  ```

* So now, we can perform different operations like we do with a list object:

  ```python
  In [29]: sq = Squares(5)
      ...: sorted(sq, reverse=True)
  Out[29]: [16, 9, 4, 1, 0]
  ```

* Let's see how Python calls `__iter__` and `__next__` methods:

  ```python
  In [30]: class Squares:
      ...:     def __init__(self, length):
      ...:         self.length = length
      ...:         self.i = 0
      ...:
      ...:     def __iter__(self):
      ...:         print('calling __iter__')
      ...:         return self
      ...:
      ...:     def __next__(self):
      ...:         print('calling __next__')
      ...:         if self.i >= self.length:
      ...:             raise StopIteration
      ...:         else:
      ...:             result = self.i ** 2
      ...:             self.i += 1
      ...:             return result
  
  In [31]: sq = Squares(5)
      ...: [item for item in sq if item%2==0]
  calling __iter__
  calling __next__
  calling __next__
  calling __next__
  calling __next__
  calling __next__
  calling __next__
  Out[31]: [0, 4, 16]
  ```

* So as seen above, Python first calls `__iter__` method to figure it out if it is iterable and then calls `__next__` function until it receives `StopIteration` exception. Let's try it out manually what Python does internally:

  ```python
  In [33]: sq = Squares(5)
      ...: sq_iterator = iter(sq)
      ...: print(id(sq), id(sq_iterator))
      ...: while True:
      ...:     try:
      ...:         item = next(sq_iterator)
      ...:         print(item)
      ...:     except StopIteration:
      ...:         break
      ...:
  calling __iter__
  4465277248 4465277248
  calling __next__
  0
  calling __next__
  1
  calling __next__
  4
  calling __next__
  9
  calling __next__
  16
  calling __next__
  ```

### Iterators and Iterables

* One issue we faced above was that after iterating the collection, it was exhausted. And another issue we faced above was that we had to return the `self` object using `__iter__` function. Let's see how we can fix these two issues

* Let's start with following example:

  ```python
  In [34]: class Cities:
      ...:     def __init__(self):
      ...:         self._cities = ['Paris', 'Berlin', 'Rome', 'Madrid', 'Lon
      ...: don']
      ...:         self._index = 0
      ...:
      ...:     def __iter__(self):
      ...:         return self
      ...:
      ...:     def __next__(self):
      ...:         if self._index >= len(self._cities):
      ...:             raise StopIteration
      ...:         else:
      ...:             item = self._cities[self._index]
      ...:             self._index += 1
      ...:             return item
      ...:
  
  In [35]: cities = Cities()
      ...: list(enumerate(cities))
  Out[35]: [(0, 'Paris'), (1, 'Berlin'), (2, 'Rome'), (3, 'Madrid'), (4, 'London')]
  
  In [36]: list(enumerate(cities))
  Out[36]: []
  
  In [37]: cities=Cities()
      ...: [item.upper() for item in cities]
  Out[37]: ['PARIS', 'BERLIN', 'ROME', 'MADRID', 'LONDON']
  ```

* As seen above, we `cities` is exhaustive and internally cities is reference to last index. So to perform any new operation, we have to create a new object. This is waste of memory. We need to figure out how to reuse same memory, some way to reset it.

* One way to achieve this is by separating the iterator code from dataset code:

  ```python
  In [38]: class Cities:
      ...:     def __init__(self):
      ...:         self._cities = ['New York', 'Newark', 'New Delhi', 'Newca
      ...: stle']
      ...:
      ...:     def __len__(self):
      ...:         return len(self._cities)
  
  In [39]: class CityIterator:
      ...:     def __init__(self, city_obj):
      ...:         # cities is an instance of Cities
      ...:         self._city_obj = city_obj
      ...:         self._index = 0
      ...:
      ...:     def __iter__(self):
      ...:         return self
      ...:
      ...:     def __next__(self):
      ...:         if self._index >= len(self._city_obj):
      ...:             raise StopIteration
      ...:         else:
      ...:             item = self._city_obj._cities[self._index]
      ...:             self._index += 1
      ...:             return item
      ...:
  
  In [40]: cities = Cities()
  
  In [41]: iter_1 = CityIterator(cities)
      ...: for city in iter_1:
      ...:     print(city)
      ...:
  New York
  Newark
  New Delhi
  Newcastle
  
  In [43]: iter_2 = CityIterator(cities)
      ...: [city.upper() for city in iter_2]
  Out[43]: ['NEW YORK', 'NEWARK', 'NEW DELHI', 'NEWCASTLE']
  ```

* But we have to remember calling iterator on object everytime we want to iterate through the collection. So a better way to do this is the following:

  ```python
  In [44]: class CityIterator:
      ...:     def __init__(self, city_obj):
      ...:         # cities is an instance of Cities
      ...:         print('Calling CityIterator __init__')
      ...:         self._city_obj = city_obj
      ...:         self._index = 0
      ...:
      ...:     def __iter__(self):
      ...:         print('Calling CitiyIterator instance __iter__')
      ...:         return self
      ...:
      ...:     def __next__(self):
      ...:         print('Calling __next__')
      ...:         if self._index >= len(self._city_obj):
      ...:             raise StopIteration
      ...:         else:
      ...:             item = self._city_obj._cities[self._index]
      ...:             self._index += 1
      ...:             return item
      ...:
  
  In [45]: class Cities:
      ...:     def __init__(self):
      ...:         self._cities = ['New York', 'Newark', 'New Delhi', 'Newca
      ...: stle']
      ...:
      ...:     def __len__(self):
      ...:         return len(self._cities)
      ...:
      ...:     def __iter__(self):
      ...:         print('Calling Cities instance __iter__')
      ...:         return CityIterator(self)
      ...:
  ```

* So by defining the `__iter__` object on the dataset object, we can make it return the `Iterator` object. This way when we iterate dataset object, a new iterator is initialised as `__iter__` is called once.

  ```python
  In [50]: cities = Cities()
  
  In [51]: [ii for ii in cities]
  Calling Cities instance __iter__
  Calling CityIterator __init__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Out[51]: ['New York', 'Newark', 'New Delhi', 'Newcastle']
  
  In [52]: [ii for ii in cities]
  Calling Cities instance __iter__
  Calling CityIterator __init__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Out[52]: ['New York', 'Newark', 'New Delhi', 'Newcastle']
  ```

* Just to make sure Iterator object is not in global scope, we can define iterator as class inside the dataset class:

  ```python
  In [53]: class Cities:
      ...:     def __init__(self):
      ...:         self._cities = ['New York', 'Newark', 'New Delhi', 'Newcastle']
      ...:
      ...:     def __len__(self):
      ...:         return len(self._cities)
      ...:
      ...:     def __iter__(self):
      ...:         print('Calling Cities instance __iter__')
      ...:         return self.CityIterator(self)
      ...:
      ...:     class CityIterator:
      ...:         def __init__(self, city_obj):
      ...:             # cities is an instance of Cities
      ...:             print('Calling CityIterator __init__')
      ...:             self._city_obj = city_obj
      ...:             self._index = 0
      ...:
      ...:         def __iter__(self):
      ...:             print('Calling CitiyIterator instance __iter__')
      ...:             return self
      ...:
      ...:         def __next__(self):
      ...:             print('Calling __next__')
      ...:             if self._index >= len(self._city_obj):
      ...:                 raise StopIteration
      ...:             else:
      ...:                 item = self._city_obj._cities[self._index]
      ...:                 self._index += 1
      ...:                 return item
  
  In [54]: cities = Cities()
  
  In [55]: list(enumerate(cities))
  Calling Cities instance __iter__
  Calling CityIterator __init__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Out[55]: [(0, 'New York'), (1, 'Newark'), (2, 'New Delhi'), (3, 'Newcastle')]
  
  In [56]: list(enumerate(cities))
  Calling Cities instance __iter__
  Calling CityIterator __init__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Calling __next__
  Out[56]: [(0, 'New York'), (1, 'Newark'), (2, 'New Delhi'), (3, 'Newcastle')]
  ```

### Mixing Iterables and Sequences

* We now saw how to create iterable objects, but they are not sequences:

  ```python
  In [57]: cities = Cities()
  
  In [58]: len(cities)
  Out[58]: 4
  
  In [59]: cities[1]
  TypeError: 'Cities' object is not subscriptable
  ```

* To make it a sequence, we must define `__getitem__` method:

  ```python
  In [60]: class Cities:
      ...:     def __init__(self):
      ...:         self._cities = ['New York', 'Newark', 'New Delhi', 'Newcastle']
      ...:
      ...:     def __len__(self):
      ...:         return len(self._cities)
      ...:
      ...:     def __getitem__(self, s):
      ...:         print('getting item...')
      ...:         return self._cities[s]
      ...:
      ...:     def __iter__(self):
      ...:         print('Calling Cities instance __iter__')
      ...:         return self.CityIterator(self)
      ...:
      ...:     class CityIterator:
      ...:         def __init__(self, city_obj):
      ...:             # cities is an instance of Cities
      ...:             print('Calling CityIterator __init__')
      ...:             self._city_obj = city_obj
      ...:             self._index = 0
      ...:
      ...:         def __iter__(self):
      ...:             print('Calling CitiyIterator instance __iter__')
      ...:             return self
      ...:
      ...:         def __next__(self):
      ...:             print('Calling __next__')
      ...:             if self._index >= len(self._city_obj):
      ...:                 raise StopIteration
      ...:             else:
      ...:                 item = self._city_obj._cities[self._index]
      ...:                 self._index += 1
      ...:                 return item
      ...:
  
  In [61]: cities = Cities()
  
  In [62]: cities[0]
  getting item...
  Out[62]: 'New York'
  
  In [63]: next(iter(cities))
  Calling Cities instance __iter__
  Calling CityIterator __init__
  Calling __next__
  Out[63]: 'New York'
  ```

### Python Built-In Iterables and Iterators

* List, set, dict, tuple all have iterators built-in and are all iterables. Just like we saw above, by using `__iter__` object we can get their built-in iterators:

  ```python
  In [64]: l = [1, 2, 3]
  
  In [65]: iter_l = iter(l) #or could use iter_1 = l.__iter__()
  
  In [66]: type(iter_l)
  Out[66]: list_iterator
  
  In [67]: next(iter_l)
  Out[67]: 1
  
  In [68]: next(iter_l)
  Out[68]: 2
  
  In [69]: id(iter_l), id(iter(iter_l))
  Out[69]: (4463354688, 4463354688)
  
  In [70]: '__next__' in dir(iter_l)
  Out[70]: True
  
  In [71]: '__iter__' in dir(iter_l)
  Out[71]: True
  
  In [72]: '__getitem__' in dir(set)
  Out[72]: False
  
  In [73]: s = {1, 2, 3}
      ...: '__next__' in dir(iter(s))
  Out[73]: True
  ```

* Note above datastructures (list, set, dict, tuple, all sequence types) have `__iter__` method, but no `__next__` method, just like we saw earlier:

  ```python
  In [74]: type(l)
  Out[74]: list
  
  In [75]: '__iter__' in dir(l)
  Out[75]: True
  
  In [76]: '__next__' in dir(l)
  Out[76]: False
  ```

* Same can be done with string as well:

  ```python
  In [77]: s = 'I sleep all night, and I work all day'
  
  In [78]: iter_s = iter(s)
  
  In [79]: print(next(iter_s))
      ...: print(next(iter_s))
      ...: print(next(iter_s))
      ...: print(next(iter_s))
      ...: print(next(iter_s))
      ...: print(next(iter_s))
      ...: print(next(iter_s))
  I
  
  s
  l
  e
  e
  p
  ```

* Other built-in iterables and iterators:

  ```python
  In [98]: r_10 = range(10)
  
  In [99]: type(r_10)
  Out[99]: range
  
  In [100]: '__iter__' in dir(r_10)
  Out[100]: True
  
  In [101]: '__next__' in dir(r_10)
  Out[101]: False
  
  In [102]: r_10
  Out[102]: range(0, 10) # Lazy evaluation, hence doesn't show all data
  ```

* Sample code where iterators is used to read data from a CSV file with first row as column-names, second row as column-types and rest of the rows is the data:

  ```python
  In [81]: def cast_row(data_types, data_row):
      ...:     return [cast(data_type, value)
      ...:             for data_type, value in zip(data_types, data_row)]
  
  In [82]: from collections import namedtuple
      ...:
      ...: with open('cars.csv') as file:
      ...:     file_iter = iter(file)
      ...:     headers = next(file_iter).strip('\n').split(';')
      ...:     data_types = next(file_iter).strip('\n').split(';')
      ...:     cars_data = [cast_row(data_types,
      ...:                           line.strip('\n').split(';'))
      ...:                    for line in file_iter]
      ...:     cars = [Car(*item) for item in cars_data]
  ```

### Cyclic Iterators

* We can cyclically iterate a finite list, there by performing infinite iteration on the object:

  ```python
  In [83]: class CyclicIterator:
      ...:     def __init__(self, lst):
      ...:         self.lst = lst
      ...:         self.i = 0
      ...:
      ...:     def __iter__(self):
      ...:         return self
      ...:
      ...:     def __next__(self):
      ...:         result = self.lst[self.i % len(self.lst)]
      ...:         self.i += 1
      ...:         return result
  
  In [84]: iter_cycl = CyclicIterator('NSWE')
  
  In [85]: for i in range(10):
      ...:     print(next(iter_cycl))
      ...:
  N
  S
  W
  E
  N
  S
  W
  E
  N
  S
  ```

* So now we have inexhaustible list. Now we can zip it with an exhaustible list and can only only as much data as exhaustible list needs:

  ```python
  In [86]: n = 10
      ...: list(zip(range(1, n+1), 'NSWE' * (n//4 + 1)))
  Out[86]:
  [(1, 'N'),
   (2, 'S'),
   (3, 'W'),
   (4, 'E'),
   (5, 'N'),
   (6, 'S'),
   (7, 'W'),
   (8, 'E'),
   (9, 'N'),
   (10, 'S')]
  ```

* Obviously Python supports from`itertools` package:

  ```python
  In [87]: import itertools
  
  In [88]: n = 10
      ...: iter_cycl = itertools.cycle('NSWE')
      ...: [f'{i}{next(iter_cycl)}' for i in range(1, n+1)]
  Out[88]: ['1N', '2S', '3W', '4E', '5N', '6S', '7W', '8E', '9N', '10S']
  ```

* So in summary, an iterable is an object that can return an iterator, inturn an iterator is an object that can return itself and it can return the next value when asked using the `next` method.

### Lazy Iterables

* What if we have to iterate through a data that will be generated in future? Say we want to through 1 billion objects, should we load all of it in memory?

* We use lazy evaluation to tackle above issue. Lazy evaluation is when evaluating a value is defered until it is actually requested. This is generic Python feature which will be used here for iterables

* Following is a simple way of perfoming Lazy evaluation:

  ```python
  In [90]: import math
      ...:
      ...: class Circle:
      ...:     def __init__(self, r):
      ...:         self.radius = r
      ...:
      ...:     @property
      ...:     def radius(self):
      ...:         return self._radius
      ...:
      ...:     @radius.setter
      ...:     def radius(self, r):
      ...:         self._radius = r
      ...:         self.area = math.pi * r**2
  
  In [91]: c = Circle(1)
  In [92]: c.area
  Out[92]: 3.141592653589793
  ```

  * As seen above, area is calcuated right when we set radius. We can delay area calculation by defining it as different method, this is lazy evaluation:

    ```python
    In [95]: class Circle:
        ...:     def __init__(self, r):
        ...:         self.radius = r
        ...:
        ...:     @property
        ...:     def radius(self):
        ...:         return self._radius
        ...:
        ...:     @radius.setter
        ...:     def radius(self, r):
        ...:         self._radius = r
        ...:         self._area = None
        ...:
        ...:     @property
        ...:     def area(self):
        ...:         if self._area is None:
        ...:             print('Calculating area...')
        ...:             self._area = math.pi * self.radius ** 2
        ...:         return self._area
    ```

* Another example to lazy evaluation, here we are making it a lazy iterable:

  ```python
  In [96]: class Factorials:
      ...:     def __iter__(self):
      ...:         return self.FactIter()
      ...:
      ...:     class FactIter:
      ...:         def __init__(self):
      ...:             self.i = 0
      ...:
      ...:         def __iter__(self):
      ...:             return self
      ...:
      ...:         def __next__(self):
      ...:             result = math.factorial(self.i)
      ...:             self.i += 1
      ...:             return result
      ...:
  
  In [97]: factorials = Factorials()
      ...: fact_iter = iter(factorials)
      ...:
      ...: for _ in range(10):
      ...:     print(next(fact_iter))
      ...:
  1
  1
  2
  6
  24
  120
  720
  5040
  40320
  362880
  ```

* Built-in lazy iterables: `range`, `zip`, `enumerate` etc

  ```python
  In [102]: r_10 = range(10)
  In [102]: r_10
  Out[102]: range(0, 10)
  
  In [103]: z = zip([1, 2, 3], 'abc')
  In [104]: z
  Out[104]: <zip at 0x10a1a1400>
  
  In [105]: with open('cars.csv') as f:
       ...:     print(type(f))
       ...:     print('__iter__' in dir(f))
       ...:     print('__next__' in dir(f))
       ...:
  <class '_io.TextIOWrapper'>
  True
  True
  
  In [106]: e = enumerate('Python rocks!')
  In [107]: print('__iter__' in dir(e))
       ...: print('__next__' in dir(e))
  True
  True
  In [108]: e, iter(e)
  Out[108]: (<enumerate at 0x10a1ae740>, <enumerate at 0x10a1ae740>)
  
  In [110]: d = {'a': 1, 'b': 2}
  In [111]: keys = d.keys()
  In [112]: '__iter__' in dir(keys), '__next__' in dir(keys) # keys is iterable
  Out[112]: (True, False)
  ```

* Note: Iterables will not have `__next__` defined, iterator will have `__next__` defined.

### The `iter()` Function

* `iter(iterable) = iterator` and `iter(iterator) = iterator`

  ```python
  In [113]: l = [1, 2, 3, 4]
  In [114]: l_iter = iter(l)
  
  In [115]: type(l_iter)
  Out[115]: list_iterator
  
  In [116]: next(l_iter)
  Out[116]: 1
  ```

* We can make iterators out of sequence types without defining `__iter__` and `__next__` methods:


  ```python
  In [117]: class Squares:
       ...:     def __init__(self, n):
       ...:         self._n = n
       ...:
       ...:     def __len__(self):
       ...:         return self._n
       ...:
       ...:     def __getitem__(self, i):
       ...:         if i >= self._n:
       ...:             raise IndexError
       ...:         else:
       ...:             return i ** 2
       ...:
  
  In [118]: sq = Squares(5)
  
  In [119]: sq_iter = iter(sq)
  
  In [120]: type(sq_iter)
  Out[120]: iterator
  
  In [121]: for i in 10:
       ...:     print(i)
  TypeError: 'int' object is not iterable
  ```

* Python uses `__getitem__` method to get index-0 element to simulate `__iter__` method

* `iter()` also supports another argument:

  ```python
  Help on built-in function iter in module builtins:
  
  iter(...)
      iter(iterable) -> iterator
      iter(callable, sentinel) -> iterator
  
      Get an iterator from an object.  In the first form, the argument must
      supply its own iterator, or be a sequence.
      In the second form, the callable is called until it returns the sentinel.
  ```

* Example of sentinel argument:

  ```python
  In [126]: def fact():
       ...:     i = 0
       ...:     def inner():
       ...:         nonlocal i
       ...:         result = math.factorial(i)
       ...:         i += 1
       ...:         return result
       ...:     return inner
       ...:
  
  In [127]: fact_iter = iter(fact(), math.factorial(5))
  
  In [128]: for num in fact_iter:
       ...:     print(num)
       ...:
  1
  1
  2
  6
  24
  ```

### How to test if an object is Iterable?

* We can use `iter` method to check this:

  ```python
  In [122]: def is_iterable(obj):
       ...:     try:
       ...:         iter(obj)
       ...:         return True
       ...:     except TypeError:
       ...:         return False
       ...:
  
  In [123]: is_iterable(Squares(5))
  Out[123]: True
  ```

### Yielding and Generators

* Anywhere we find `yield` is a `generator` object.

  ```python
  In [129]: def my_func():
       ...:     print('line 1')
       ...:     yield 'Flying'
       ...:     print('line 2')
       ...:     yield 'Circus'
       ...:
  
  In [130]: my_func()
  Out[130]: <generator object my_func at 0x109db8820>
  
  In [131]: gen_my_func = my_func()
  
  In [132]: next(gen_my_func)
  line 1
  Out[132]: 'Flying'
  
  In [133]: next(gen_my_func)
  line 2
  Out[133]: 'Circus'
  
  In [134]: next(gen_my_func)
  StopIteration:
  ```

* As seen above, Python `generators` are iterators. Hence, it has both `__iter__` and `__next__` :

  ```python
  In [135]: '__iter__' in dir(gen_my_func)
  Out[135]: True
  
  In [136]: '__next__' in dir(gen_my_func)
  Out[136]: True
  ```

* Since both object and the `iter` of that object is same, it is an `iterator` object:

  ```python
  In [137]: gen_my_func
  Out[137]: <generator object my_func at 0x10a2e35f0>
  
  In [138]: iter(gen_my_func)
  Out[138]: <generator object my_func at 0x10a2e35f0>
  ```

* Simple examples:

  ```python
  In [139]: def squares(sentinel):
       ...:     i = 0
       ...:     while True:
       ...:         if i < sentinel:
       ...:             yield i**2
       ...:             i += 1 # note how we can incremenet **after** the yield
       ...:         else:
       ...:             return 'all done!'
       ...:
  
  In [140]: sq = squares(3)
  
  In [141]: next(sq)
  Out[141]: 0
  
  In [142]: next(sq)
  Out[142]: 1
  
  In [143]: next(sq)
  Out[143]: 4
  ```

* Different ways to write Fibonacci sequence:

  * Recursion:
    ```python
    In [150]: from functools import lru_cache
    
    In [151]: @lru_cache()
         ...: def fib_recursive(n):
         ...:     if n <= 1:
         ...:         return 1
         ...:     else:
         ...:         return fib_recursive(n-1) + fib_recursive(n-2)
    
    In [152]: fib_recursive(5)
    Out[152]: 8
    ```
  
  * Iteration:
  
    ```python
    In [153]: def fib(n):
         ...:     fib_0 = 1
         ...:     fib_1 = 1
         ...:     for i in range(n-1):
         ...:         fib_0, fib_1 = fib_1, fib_0 + fib_1
         ...:     return fib_1
    
    In [154]: [fib(i) for i in range(7)]
    Out[154]: [1, 1, 2, 3, 5, 8, 13]
    ```
  
  * Iterator:
  
    ```python
    In [155]: class Fib:
         ...:     def __init__(self, n):
         ...:         self.n = n
         ...:
         ...:     def __iter__(self):
         ...:         return self.FibIter(self.n)
         ...:
         ...:     class FibIter:
         ...:         def __init__(self, n):
         ...:             self.n = n
         ...:             self.i = 0
         ...:
         ...:         def __iter__(self):
         ...:             return self
         ...:
         ...:         def __next__(self):
         ...:             if self.i >= self.n:
         ...:                 raise StopIteration
         ...:             else:
         ...:                 result = fib(self.i)
         ...:                 self.i += 1
         ...:                 return result
    
    In [155]: fib_iterable = Fib(7)
    
    In [156]: for num in fib_iterable:
         ...:     print(num)
         ...:
    1
    1
    2
    3
    5
    8
    13
    ```
  
  * Closure:
  
    ```python
    In [157]: def fib_closure():
         ...:     i = 0
         ...:     def inner():
         ...:         nonlocal i
         ...:         result = fib(i)
         ...:         i += 1
         ...:         return result
         ...:     return inner
    
    In [158]: fib_numbers = fib_closure()
         ...: fib_iter = iter(fib_numbers, fib(7))
         ...: for num in fib_iter:
         ...:     print(num)
         ...:
    1
    1
    2
    3
    5
    8
    13
    ```
  
  * Generator:
  
    ```python
    In [159]: def fib_gen(n):
         ...:     fib_0 = 1
         ...:     yield fib_0
         ...:     fib_1 = 1
         ...:     yield fib_1
         ...:     for i in range(n-2):
         ...:         fib_0, fib_1 = fib_1, fib_0 + fib_1
         ...:         yield fib_1
    
    In [160]: [num for num in fib_gen(7)]
    Out[160]: [1, 1, 2, 3, 5, 8, 13]
    ```

