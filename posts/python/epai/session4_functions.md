# Functions as First Class Citizen
**Python Version:** 3.9.1 (Clang 12.0.0)

**iPython Version:** 7.19.0

#### Function documentation

* doc string in a function is shown as part of help of the function output:

  ```python
  >>> def my_func(a, b):
  ...     '''Returns the product of a and b'''
  ...			# comments are not shown as part of help output
  ...     return a * b
  >>>
  >>> help(my_func)
  Help on function my_func in module __main__:
  
  my_func(a, b)
      Returns the product of a and b
  >>> # Dunder doc returns documentations of the function
  >>> my_func.__doc__
  'Returns the product of a and b'
  ```

#### Annotations

* One can add annotations for variables:

  ```python
  >>> def my_func(a: 'please keep this as an integer',
  ...             b: 'please keep this also as an integer'):
  ...             '''this function just
  ...             gives a^b as the output'''
  ...             return a ** b
  ...
  >>> help(my_func)
  Help on function my_func in module __main__:
  
  my_func(a: 'please keep this as an integer', b: 'please keep this also as an integer')
      this function just
      gives a^b as the output
  ...
  >>> my_func.__doc__
  'this function just\n\t\tgives a^b as the output'
  ```

* Annotation can be any object and not just string:

  ```python
  >>> def my_func(a: str):
  ...     return a * max(x,y)
  ...
  >>> help(my_func)
  Help on function my_func in module __main__:
  
  my_func(a: str)
  ```

* We can also add annotation for return value (as seen below annotation can be an expresion as well and we can also set the default value for a argument along with annotation):

  ```python
  >>> x, y = 3, 5
  >>> def my_func(a: str='a') -> '"a" repeated ' + str(max(3, 5)) + ' times':
  ...     return a * max(x,y)
  ...
  >>> help(my_func)
  Help on function my_func in module __main__:
  
  my_func(a: str = 'a') -> '"a" repeated 5 times'
  ```

* How can one access annotations of the function?

  ```python
  >>> my_func.__annotations__
  {'a': <class 'str'>, 'return': '"a" repeated 5 times'}
  ```

#### Lambda Functions

* Anonymous function as it doesn't have name to it:

  ```python
  >>> def my_func(x):
  ...     return x ** 2
  ...
  >>> my_func(4)
  16
  >>>
  >>> f = lambda x: x ** 2
  >>> f(4)
  16
  ```

* One can also pass default values to args in lambda function:

  ```python
  >>> func_1 = lambda x, y = 10 : (x, y)
  >>> func_1(1, 2), func_1(10)
  ((1, 2), (10, 10))
  ```

* `*args`, `**kwargs` can also be used with lambda functions:

  ```python
  >>> func_2 = lambda x, *args, y, **kwargs: (x, *args, y, {**kwargs})
  >>> func_2(1, 'a', 'b', y=100, a = 10, b = 24, c = 'python')
  (1, 'a', 'b', 100, {'a': 10, 'b': 24, 'c': 'python'})
  >>>
  >>> def apply_fn(fn, *args, **kwargs):
  ...      return fn(*args, **kwargs)
  ...
  >>> apply_fn(lambda x, y: x + y, 1, 2)
  3
  >>> apply_fn(lambda *args: sum(args), 1, 2, 3, 4, 5, 6, 7, 8)
  36
  >>> apply_fn(sum, (1, 2, 3, 4, 5, 6, 7, 8))
  36
  ```

* Can be used as part of sort function:

  ```python
  >>> l = ['a', 'B', 'c', 'D']
  >>> sorted(l)
  ['B', 'D', 'a', 'c']
  >>> sorted(l, key = str.upper)
  ['a', 'B', 'c', 'D']
  >>> sorted(l, key = lambda s: s.upper())
  ['a', 'B', 'c', 'D']
  ```

  Sorting dictionaries:

  ```python
  >>> d = {'def': 300, 'abc': 200, 'ghi': 100, 'jlk': 50}
  >>> sorted(d)
  ['abc', 'def', 'ghi', 'jlk']
  >>> sorted(d, key = lambda k: d[k])
  ['jlk', 'ghi', 'abc', 'def']
  ```

  Sorting complex numbers:

  ```python
  >>> l = [3+3j, 1+1j, 0+0j]
  >>> sorted(l)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: '<' not supported between instances of 'complex' and 'complex'
  >>> sorted(l, key = lambda x: (x.real)**2 + (x.imag)**2)
  [0j, (1+1j), (3+3j)]
  ```

#### Function Introspection

* Since functions are objects, we can add attributes to it:

  ```python
  >>> def my_func(a,b):
  ...     return a ** b
  ...
  >>> my_func.short_desc = "multiple function"
  >>> my_func.short_desc
  'multiple function'
  ```

* One can use `dir()` function to get list of attributes of a fuction:

  ```python
  >>> dir(my_func)
  ['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'short_desc']
  ```

* One can use name attribute to get name of the function:

  ```python
  >>> def my_func(a, b = 2, c = 3, *, kw1, kw2= 2, **kwargs):
  ...     pass
  ...
  >>> f = my_func
  >>> my_func.__name__
  'my_func'
  >>> f.__name__
  'my_func'
  ```

* Can also look at default values:

  ```python
  >>> my_func.__defaults__
  (2, 3)
  ```

* One can analyze the function code as well using `__code__` object:

  ```python
  >>> my_func.__code__
  <code object my_func at 0x109c76500, file "<stdin>", line 1>
  >>> dir(my_func.__code__)
  ['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_kwonlyargcount', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_posonlyargcount', 'co_stacksize', 'co_varnames', 'replace']
  >>>
  >>> my_func.__code__.co_varnames
  ('a', 'b', 'c', 'kw1', 'kw2', 'kwargs')
  >>> my_func.__code__.co_argcount # doesn't count args/kwargs, only local & global variables
  3
  ```

* **Inspect Module:** Can use this module to do much better inspection:

  ```python
  >>> import inspect
  >>> inspect.isfunction(my_func)
  True
  >>> inspect.ismethod(my_func) # false, since not bounded to a class
  False
  
  In [5]: class MyClass:
     ...:     def f_instance(self):
     ...:         'Instance methods are bound to the instance/object of the class'
     ...:         pass
     ...:
     ...:     @classmethod
     ...:     def f_class(cls):
     ...:         'Class method are bound to the CLASS, not instances'
     ...:         pass
     ...:
     ...:     @staticmethod
     ...:     def f_static():
     ...:         'Statis methods are not bounded to either class or its objects/instances'
     ...:         pass
  
  In [8]: inspect.isfunction(MyClass.f_instance), inspect.ismethod(MyClass.f_instance)
  Out[8]: (True, False)
  
  In [9]: inspect.isfunction(MyClass.f_class), inspect.ismethod(MyClass.f_class)
  Out[9]: (False, True)
  
  In [10]: inspect.isfunction(MyClass.f_static), inspect.ismethod(MyClass.f_static)
  Out[10]: (True, False)
  ```

* When a function is bounded to a class, it is method otherwise it is a function:

  ```python
  In [12]: inspect.isfunction(my_obj.f_instance), inspect.ismethod(my_obj.f_instance)
  Out[12]: (False, True)
  
  In [13]: inspect.isfunction(MyClass.f_instance), inspect.ismethod(MyClass.f_instance)
  Out[13]: (True, False)
  
  In [14]: inspect.isfunction(my_obj.f_class), inspect.ismethod(my_obj.f_class),
  Out[14]: (False, True)
  
  In [15]: inspect.isfunction(my_obj.f_static), inspect.ismethod(my_obj.f_static)
  Out[15]: (True, False)
  ```

* One can use routine to check if it is function or method:

  ```python
  In [17]: inspect.isroutine(MyClass.f_instance)
  Out[17]: True
  ```
  
* Source code of a function:

  ```python
  In [18]: print(inspect.getsource(MyClass.f_instance))
      def f_instance(self):
          'Instance methods are bound to the instance/object of the class'
          pass
  ```

* Get module details:

  ```python
  In [21]: import math
  
  In [22]: inspect.getmodule(math.sin)
  Out[22]: <module 'math' from '/usr/local/Cellar/python@3.9/3.9.1_2/Frameworks/Python.framework/Versions/3.9/lib/python3.9/lib-dynload/math.cpython-39-darwin.so'>
  ```

* Comments of a code (but cannot get comments inside a func):

  ```python
  In [19]: # setting up variable
      ...: i = 10
      ...: # comment line 1
      ...: # comment line 2
      ...: def my_func(a, b = 1):
      ...:     # comment inside my_func
      ...:     pass
      ...:
  
  In [20]: inspect.getcomments(my_func)
      ...:
  Out[20]: '# comment line 1\n# comment line 2\n'
  ```

* Get signature of a function:

  ```python
  In [23]: # TODO: Provide implementation
      ...: def my_func(a: 'a string',
      ...:             b: int = 1,
      ...:             *args: 'additional argument args',
      ...:             kw1: 'first keyword only arg',
      ...:             kw2: 'second keyword only arg',
      ...:             **kwargs: 'additional keyword only args') -> str:
      ...:             """does something
      ...:             or ther other"""
      ...:             pass
      ...:
  
  In [24]: inspect.signature(my_func)
  Out[24]: <Signature (a: 'a string', b: int = 1, *args: 'additional argument args', kw1: 'first keyword only arg', kw2: 'second keyword only arg', **kwargs: 'additional keyword only args') -> str>
    
  In [26]: sig = inspect.signature(my_func)
      ...:
  
  In [27]: for param_name, param in sig.parameters.items():
      ...:     print(param_name, param)
      ...:
  a a: 'a string'
  b b: int = 1
  args *args: 'additional argument args'
  kw1 kw1: 'first keyword only arg'
  kw2 kw2: 'second keyword only arg'
  kwargs **kwargs: 'additional keyword only args'
  ```

* One can write their own PyCharm like feature to get details of a function:

  ```python
  In [28]: def print_into(f: "callable") -> None:
      ...:     print(f.__name__)
      ...:     print('=' * len(f.__name__), end = '\n\n')
      ...:     print(f'{inspect.getcomments(f)}\n{inspect.cleandoc(f.__doc__)}')
      ...:     print(f'Inputs\n{"-"*len("Inputs")}')
      ...:
      ...:     sig = inspect.signature(f)
      ...:     for param in sig.parameters.values():
      ...:         print("Name:", param.name)
      ...:         print("default:", param.default)
      ...:         print("annotation:", param.annotation)
      ...:         print("kind:", param.kind)
      ...:         print('----------------------------\n')
      ...:
      ...:     print(f'\n\nOutput {"-"*len("Output")}')
      ...:     print(sig.return_annotation)
      ...:
      ...: print_into(my_func)
  my_func
  =======
  
  # TODO: Provide implementation
  
  does something
  or ther other
  Inputs
  ------
  Name: a
  default: <class 'inspect._empty'>
  annotation: a string
  kind: POSITIONAL_OR_KEYWORD
  ----------------------------
  
  Name: b
  default: 1
  annotation: <class 'int'>
  kind: POSITIONAL_OR_KEYWORD
  ----------------------------
  
  Name: args
  default: <class 'inspect._empty'>
  annotation: additional argument args
  kind: VAR_POSITIONAL
  ----------------------------
  
  Name: kw1
  default: <class 'inspect._empty'>
  annotation: first keyword only arg
  kind: KEYWORD_ONLY
  ----------------------------
  
  Name: kw2
  default: <class 'inspect._empty'>
  annotation: second keyword only arg
  kind: KEYWORD_ONLY
  ----------------------------
  
  Name: kwargs
  default: <class 'inspect._empty'>
  annotation: additional keyword only args
  kind: VAR_KEYWORD
  ----------------------------
  
  
  Output ------
  <class 'str'>
  ```

* For **some of the built-in** modules, python doesn't allow keyword arguments for positional arguments:

  ```python
  In [29]: help(divmod)
  Help on built-in function divmod in module builtins:
  
  divmod(x, y, /)
      Return the tuple (x//y, x%y).  Invariant: div*y + mod == x.
  
  In [31]: divmod(10, 3)
  Out[31]: (3, 1)
  
  In [32]: divmod(x = 10, y = 3)
  ---------------------------------------------------------------------------
  TypeError                                 Traceback (most recent call last)
  <ipython-input-32-aeb448ab4ec4> in <module>
  ----> 1 divmod(x = 10, y = 3)
  
  TypeError: divmod() takes no keyword arguments
  ```

* All functions & classes are callable:

  ```python
  In [33]: callable(print)
  Out[33]: True
  
  In [34]: from decimal import Decimal
      ...:
      ...: callable(Decimal)
  Out[34]: True
  ```

* Class instances **maybe** callable, one must define `__call__` to make it callable:

  ```python
  In [35]: class MyClass:
      ...:     def __init__(self):
      ...:         print('initialzing ... ')
      ...:         self.counter = 0
      ...:
      ...:     def __call__(self, x = 1):
      ...:         self.counter += x
      ...:         print(self.counter)
      ...:
  
  In [36]: my_obj = MyClass()
  initialzing ...
  
  In [37]: callable(my_obj.__call__)
  Out[37]: True
  
  In [39]: callable(my_obj)
  Out[39]: True
  
  In [40]: class MyClass:
      ...:     def __init__(self):
      ...:         print('initialzing ... ')
      ...:         self.counter = 0
      ...:
      ...: my_obj = MyClass()
  initialzing ...
  
  In [41]: callable(my_obj)
  Out[41]: False
  ```

### Higher-Order Functions: Map and Filter

* A function which takes a function and returns a function is called higher-order functions

  ```python
  In [42]: def fact(n):
      ...:     return 1 if n < 2 else n * fact(n-1)
      ...:
  
  In [43]: map(fact, [1, 2, 3, 4, 5]) # -> No computation is done here yet, because it is an iterator
  Out[43]: <map at 0x111a1c070>
  
  In [44]: list(map(fact, [1, 2, 3, 4, 5])) # -> Computation is done here
  Out[44]: [1, 2, 6, 24, 120]
  
  In [44]: list(map(fact, [1, 2, 3, 4, 5]))
  Out[44]: [1, 2, 6, 24, 120]
  
  In [45]: l = [0, 1, 1, 0, 1, False, True]
      ...: list(filter(None, l))
  Out[45]: [1, 1, 1, True]
  
  In [46]: l = [0, 1, 1, 0, 1, False, True]
      ...: list(filter(lambda x: x*5, l))
  Out[46]: [1, 1, 1, True]
  
  In [47]: l = [0, 1, 1, 0, 1, False, True] # [-1, 0, 0, -1, 0, -1, 0]
      ...: list(filter(lambda x: x - 1, l))
  Out[47]: [0, 0, False]
  
  In [48]: filter(None, l)
  Out[48]: <filter at 0x111a507c0> # -> Output is an iterator
  ```

* Avoid using map & filter as one can achieve the same using comprehensions! So where ever possible use comprehensions

  ```python
  In [59]: list(filter(lambda y: y< 25, map(lambda x: x**2, range(10))))
  Out[59]: [0, 1, 4, 9, 16]
  
  In [60]: [x**2 for x in range(10) if x**2 < 25]
  Out[60]: [0, 1, 4, 9, 16]
  ```

* Iterators are exhaustible:

  ```python
  In [49]: whatisthis = (fact(i) for i in l)
      ...: for e in whatisthis:
      ...:     print(e, end='')
      ...:
  1111111
  In [50]: for e in whatisthis:
      ...:     print(e)
  ```

* But iterables are not exhaustible:

  ```python
  In [51]: r = range(10)
      ...:
      ...: for k in r:
      ...:     print(k, end='')
      ...:
  0123456789
  In [52]: for k in r:
      ...:     print(k, end='')
      ...:
  0123456789
  ```

* One can combine multiple lists using `zip`:

  ```python
  In [54]: l1 = 1, 2, 3
      ...: l2 = ['a', 'b', 'c', 'd']
      ...: l3 = (1, 2, 3, 4, 5, 6, 7, 8)
      ...: list(zip(l1, l2, l3))
  Out[54]: [(1, 'a', 1), (2, 'b', 2), (3, 'c', 3)]
  
  In [55]: l1 = range(100000000000)
      ...: l2 = 'python'
      ...:
      ...: list(zip(l1, l2))
  Out[55]: [(0, 'p'), (1, 'y'), (2, 't'), (3, 'h'), (4, 'o'), (5, 'n')]
  
  In [58]: l1 = [1, 2, 3, 4, 5]
      ...: l2 = [10, 20, 30, 40, 50]
      ...:
      ...: [z1+ z2 for z1, z2 in zip(l1, l2)]
  Out[58]: [11, 22, 33, 44, 55]
  ```

#### Reducing Functions

* Can use `reduce` function to perform cumulative calculation on iterables:

  ```python
  In [64]: l = [1, 2, 3, 4, 5]
  In [66]: from functools import reduce
  
  In [67]: reduce(lambda a, b: a if a > b else b, l)
  Out[67]: 5
  ```

* `any` can be used to check if any of the value is true:

  ```python
  In [68]: l = [0, 0.01, False]
      ...: any(l)
  Out[68]: True
  
  In [69]: reduce(lambda a, b: bool(a or b), l) # our implementation of any
  Out[69]: True
  ```

* `all` can be used to check if all of the values are true:

  ```python
  In [70]: l = [True, 0.01, -1231231, None]
      ...: all(l)
  Out[70]: False
  
  In [71]: reduce(lambda a, b: bool(a and b), l) # our implementation of all
  Out[71]: False
  ```

* One can sett initializer for reduce function:

  ```python
  In [74]: l = []
      ...:
      ...: reduce(lambda x, y: x*y, l)
  ---------------------------------------------------------------------------
  TypeError                                 Traceback (most recent call last)
  <ipython-input-74-fa376e086deb> in <module>
        1 l = []
        2
  ----> 3 reduce(lambda x, y: x*y, l)
  
  TypeError: reduce() of empty sequence with no initial value
  
  In [75]: reduce(lambda x, y: x*y, l, 1)
  Out[75]: 1
  ```

#### Partial Function

* A function can be initalized with partial values:

  ```python
  In [76]: from functools import partial
  
  In [77]: def my_func(a, b, c):
      ...:     print(a, b*b, c)
      ...:
      ...: f = partial(my_func, 10)
      ...:
      ...: f(4, 20)
  10 16 20
  ```

* Same can be achieved with lambda functions:

  ```python
  In [79]: fn = lambda b, c: my_func(10, b, c)
      ...:
      ...: fn(4, 20)
  10 16 20
  ```

* Partial functions allows us to simplify our function, give new variants to it:

  ```python
  In [80]: def my_func(a, b, *args, k1, k2, **kwargs):
      ...:     print(a, b, args, k1, k2, kwargs)
      ...: f = partial(my_func, 10, k1='a')
      ...: f(20, 30, 40, 50, k2 = 'rohan', l=123, o=12312)
  10 20 (30, 40, 50) a rohan {'l': 123, 'o': 12312}
  
  In [81]: def power(base, exponent):
      ...:     return base ** exponent
      ...: square = partial(power, exponent=2)
      ...: cube = partial(power, exponent=3)
  In [82]: square(4)
  Out[82]: 16
  In [83]: cube(4)
  Out[83]: 64
  In [84]: cube(base=4)
  Out[84]: 64
  In [85]: cube(base=4,exponent=2) # NOTE: one can still change arg values
  Out[85]: 16
  ```

* Another example to sort the coordinates based on dist from origin:

  ```python
  In [86]: origin = (0, 0)
      ...: l = [(1, 1), (0, 2), (-3, 2), (5, 4), (0, 0)]
  
  In [87]: dist2 = lambda x, y: (x[0] - y[0])**2 + (x[1] - y[1])**2
      ...:
  
  In [88]: dist_from_origin = partial(dist2, (0, 0))
  
  In [89]: sorted(l, key=dist_from_origin)
  Out[89]: [(0, 0), (1, 1), (0, 2), (-3, 2), (5, 4)]
  ```

#### Operator Module

* Module to perform operations:

  ```python
  In [92]: import operator
  
  In [93]: operator.add(1,2)
  Out[93]: 3
  
  In [94]: operator.floordiv(13, 2)
  Out[94]: 6
  
  In [95]: operator.truediv(13, 2)
  Out[95]: 6.5
  
  In [97]: operator.truth([1, 2])
  Out[97]: True
  
  In [98]: operator.truth([])
  Out[98]: False
  
  In [99]: operator.le(10, 100)
  Out[99]: True
  ```

* Becomes handy with reduce function

  ```python
  In [96]: reduce(operator.mul, [1, 2, 3, 4,5 ])
  Out[96]: 120
  ```

* Operator does simple operators as well, but they handle lots of cases and also generic inputs:

  ```
  In [100]: my_list = [1, 2, 3, 4,5 ]
  
  In [101]: my_list[4]
  Out[101]: 5
  
  In [102]: operator.getitem(my_list, 4)
  Out[102]: 5
  ```

* We can use it with Class as well:

  ```python
  In [104]: class MyClass:
       ...:     def __init__(self):
       ...:         self.a = 10
       ...:         self.b = 20
       ...:         self.c = 30
       ...:
       ...:     def test(self):
       ...:         print('test method running...')
       ...:
  
  In [105]: obj = MyClass()
       ...:
       ...: obj.a, obj.b, obj.c
  Out[105]: (10, 20, 30)
  
  In [106]: f = operator.attrgetter('a')
       ...:
       ...: f(obj)
  Out[106]: 10
  
  In [107]: obj2 = MyClass()
       ...: obj2.a = 100
       ...:
       ...: f(obj), f(obj2)
  Out[107]: (10, 100)
  ```