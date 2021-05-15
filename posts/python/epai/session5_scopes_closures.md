# Scopes, Closures & Decorators

**Python Version:** 3.9.1 (Clang 12.0.0)

**iPython Version:** 7.19.0

#### Scopes

* The scope of a variable is based on where it is defined. So outside any function, if a variable is defined then it is of global scope. Hence it can be accessed from any inner scope

  ```python
  In [1]: a = 10
  
  In [2]: def my_func(n):
     ...:     print('global: ', a)
     ...:     c = a ** n
     ...:     return c
     ...:
  
  In [3]: my_func(2)
  global:  10
  Out[3]: 100
  ```

* Inside a function, if we want to change global variable then we must use keyword `global`, otherwise it will not change and is defined only within the function (local scope)

  ```python
  In [4]: a = 10
     ...: def my_func(n):
     ...:     a = 2
     ...:     c = a ** n
     ...:     return c
     ...:
     ...: print(my_func(2)), print(a)
  4
  10
  
  In [5]: a = 10
     ...: def my_func(n):
     ...:     global a
     ...:     a = 2
     ...:     c = a ** 2
     ...:     return c
     ...:
     ...: print(my_func(2)), print(a)
  4
  2
  ```

* Similarly global variable can also be defined inside a function by using a `global` keyword:

  ```python
  In [6]: def my_func(n):
     ...:     global var
     ...:     var = 'hello world'
     ...:     return n ** 2
     ...:
  
  In [7]: my_func(10)
  Out[7]: 100
  
  In [8]: print(var)
  hello world
  ```

* Python determines scope of the object at compile time. For example:

  ```python
  In [9]: a = 10
     ...: b = 100
     ...: def my_func():
     ...:     print(a)
     ...:     print(b)
     ...:     b = 1000 # becomes a local variable during compile time
     ...:
  
  In [10]: my_func()
  10
  UnboundLocalError: local variable 'b' referenced before assignment
  ```

* We can mask a global func/variable and unmask it as well. For example:

  ```python
  In [11]: print = lambda x: 'hello {0}!'.format(x)
      ...:
      ...: def my_func(name):
      ...:     return print(name)
      ...:
      ...: my_func('world')
  Out[11]: 'hello world!'
  
  In [12]: print("red")
  Out[12]: 'hello red!'
  
  In [13]: del print
  In [14]: print("red", end='SS')
  redSS
  ```

* Variables defined inside for block outside a function becomes of global scope. Hence this can leak memory:

  ```python
  In [15]: for i in range(10):
      ...:     x = 2 * i
      ...:
      ...: print(x)
  18
  ```

#### Non-Local Scope

* Python performs scope hopping to figure out where a variable is defined:

  ```python
  In [16]: def out_func():
      ...:     x = 'hello'
      ...:     def inner1():
      ...:         def inner2():
      ...:             print(x)
      ...:         inner2()
      ...:     inner1()
      ...:
      ...: out_func()
  hello
  ```

* We can define a new local variable inside the inner function:

  ```python
  In [17]: def out_func():
      ...:     x = 'hello'
      ...:     def inner_func():
      ...:         x = 'python' # x is redefined here as local variable
      ...:     inner_func()
      ...:     print(x)
      ...:
      ...: out_func()
  hello
  ```

* But how can one change variable of parent function? Using keyword `nonlocal` just like `global`

  ```python
  In [20]: def out_func():
      ...:     x = 'hello'
      ...:     def inner_func():
      ...:         nonlocal x
      ...:         x = 'python'
      ...:     inner_func()
      ...:     print(x)
      ...:
      ...: out_func()
  python
  ```

* But how far can it access nonlocal variable? Same as before, it performs scope hopping and uses a closest variable

  ```python
  In [21]: def outer():
      ...:     x = 'hello'
      ...:     def inner1():
      ...:         x = 'python'
      ...:         def inner2():
      ...:             nonlocal x
      ...:             x = 'monty'
      ...:         print('inner1 (before):', x)
      ...:         inner2()
      ...:         print('inner1 (after):', x)
      ...:     inner1()
      ...:     print('outer:', x)
      ...:
      ...: outer()
  inner1 (before): python
  inner1 (after): monty
  outer: hello
  ```

* Here's another example of `nonlocal` scope, here using `nonlocal` we have ended up changing the first definition of variable `x`:

  ```python
  In [22]: def outer():
      ...:     x = 'hello'
      ...:     def inner1():
      ...:         nonlocal x
      ...:         x = 'python'
      ...:         def inner2():
      ...:             nonlocal x
      ...:             x = 'monty'
      ...:         print('inner1 (before):', x)
      ...:         inner2()
      ...:         print('inner1 (after):', x)
      ...:     inner1()
      ...:     print('outer:', x)
      ...:
      ...: outer()
  inner1 (before): python
  inner1 (after): monty
  outer: monty
  ```

* Here's another example using `global` and `nonlocal` keywords:

  ```python
  In [23]: x = 100
      ...: def outer():
      ...:     x = 'python'  # masks global x
      ...:     def inner1():
      ...:         nonlocal x  # refers to x in outer
      ...:         x = 'monty' # changed x in outer scope
      ...:         def inner2():
      ...:             global x  # refers to x in global scope
      ...:             x = 'hello'
      ...:             print('inner1 (before):', x)
      ...:         inner2()
      ...:         print('inner1 (after):', x)
      ...:     inner1()
      ...:     print('outer', x)
      ...:
      ...: outer()
      ...: print(x)
  inner1 (before): hello
  inner1 (after): monty
  outer monty
  hello
  ```

#### Closures

* We can attach some data to the code using a technique called closure

  ```python
  In [24]: def outer():
      ...:     x = 'python'
      ...:     def inner():
      ...:         print(x)
      ...:     return inner
      ...:
      ...: fn = outer()
  
  In [25]: fn.__code__.co_freevars
  Out[25]: ('x',)
  
  In [26]: fn.__closure__
  Out[26]: (<cell at 0x10b0df700: str object at 0x109765330>,)
  ```

* Let's verify that memory address of variable inside closure is matching:

  ```python
  In [27]: def outer():
      ...:     x = [1, 2, 3]
      ...:     print('outer:', hex(id(x)))
      ...:     def inner():
      ...:         print('inner:', hex(id(x)))
      ...:         print(x)
      ...:     return inner
      ...:
      ...: fn = outer()
  outer: 0x10b013680
  
  In [28]: fn()
  inner: 0x10b013680
  [1, 2, 3]
  
  In [29]: fn.__closure__
  Out[29]: (<cell at 0x10b021a00: list object at 0x10b013680>,)
  ```

* Simple usage of closure showing **shared extended scope**:

  ```python
  In [30]: def counter():
      ...:     count = 0 # local variable
      ...:     def inc():
      ...:         nonlocal count # this is the count variable
      ...:         count += 1
      ...:         return count
      ...:     return inc
      ...:
      ...: c = counter()
  
  In [31]: for k in range(5):
      ...:     print(c())
  1
  2
  3
  4
  5
  ```

* Everytime we create a closure it will define a new free memory:

  ```python
  In [32]: def pow(n):
      ...:     # n is local to pow
      ...:
      ...:     def inner(x):
      ...:         # x is local to inner
      ...:         return x ** n
      ...:     return inner
      ...:
  
  In [33]: square = pow(2)
  
  In [34]: square(5)
  Out[34]: 25
  
  In [35]: cube = pow(3)
  
  In [36]: cube(5)
  Out[36]: 125
  
  In [37]: square.__closure__
  Out[37]: (<cell at 0x10b15adf0: int object at 0x109006950>,)
  
  In [38]: cube.__closure__
  Out[38]: (<cell at 0x10b1316d0: int object at 0x109006970>,)
  ```

* The captured variable is a reference which is established when the closurre is created, but the value is looked up only when closure is called:

  ```python
  In [39]: def outer():
      ...:     x = 0
      ...:     def inner():
      ...:         y = 10
      ...:         return y / x
      ...:     return inner
      ...:
      ...: fn = outer()
  
  In [40]: fn.__closure__
  Out[40]: (<cell at 0x10b15aee0: int object at 0x109006910>,)
  
  In [41]: fn() # Error is only returned after closure is called
  ZeroDivisionError: division by zero
  ```

* Another example, as seen below, although the closure is defined, but the value for captured variable (`n`) is only looked up when it is called. And when it is called value of `n` is 4, hence all adders are returning same value:

  ```python
  In [42]: def create_adders():
      ...:     adders = []
      ...:     for n in range(1, 5):
      ...:         adders.append(lambda x: x + n)
      ...:     return adders
      ...: adders = create_adders()
      ...: adders
  Out[42]:
  [<function __main__.create_adders.<locals>.<lambda>(x)>,
   <function __main__.create_adders.<locals>.<lambda>(x)>,
   <function __main__.create_adders.<locals>.<lambda>(x)>,
   <function __main__.create_adders.<locals>.<lambda>(x)>]
  
  In [43]: adders[0](10), adders[1](10), adders[2](10),adders[3](10)
  Out[43]: (14, 14, 14, 14)
  ```

* The captured variable inside a closure can be any object. It can be a function as well. This opens lots of use cases for closures. One can define say an auth function which has to be called before calling the inner function.

* Closure example: Write a function to perform moving averages over continous data. First let's write it without using closure:

  ```python
  In [51]: class Average:
      ...:     def __init__(self):
      ...:         self._count = 0
      ...:         self._total = 0
      ...:
      ...:     def add(self, value):
      ...:         self._total += value
      ...:         self._count += 1
      ...:         return self._total/self._count
      ...:
      ...: a = Averager()
      ...: a.add(10), a.add(20), a.add(30)
  Out[51]: (10.0, 15.0, 20.0)
  ```

* Class comes with lot of overhead. Now let's use closure:

  ```python
  In [52]: def averager():
      ...:     count = 0
      ...:     total = 0
      ...:
      ...:     def add(value):
      ...:         nonlocal total, count
      ...:         total += value
      ...:         count += 1
      ...:         return 0 if count == 0 else total/count
      ...:     return add
      ...:
      ...: a = averager()
      ...: a(10), a(20), a(30)
  Out[52]: (10.0, 15.0, 20.0)
  ```

#### Decorators

* Closures are functions wrapped around private variables and objects, which the closure function can access

  ```python
  In [53]: def counter(fn):
      ...:     cnt = 0
      ...:
      ...:     def inner(*args, **kwargs):
      ...:         nonlocal cnt
      ...:         cnt = cnt + 1
      ...:         print(f'{fn.__name__} has been called {cnt} times')
      ...:         return fn(*args, **kwargs)
      ...:
      ...:     return inner
      ...:
  
  In [54]: def mult(a: float, b: float=1, c: float=1) -> float:
      ...:     """
      ...:     returns the product of a, b, and c
      ...:     """
      ...:     return a * b * c
      ...:
  
  In [55]: mult = counter(mult)
      ...: mult(1,2, 3)
  mult has been called 1 times
  Out[55]: 6
  ```

* Closure in a way is decorating the inner function, the short hand to represent that is called Decorator. So above can be rewritten as:

  ```python
  In [56]: @counter # ====> mult = counter(mult)
      ...: def mult(a: float, b: float=1, c: float=1) -> float:
      ...:     """
      ...:     returns the product of a, b, and c
      ...:     """
      ...:     return a * b * c
      ...:
  
  In [57]: mult(1,2, 3)
  mult has been called 1 times
  Out[57]: 6
  ```

* Using decorator, we end up losing lots of metadata:

  ```python
  In [58]: mult.__name__
  Out[58]: 'inner'
  In [59]: help(mult)
  Help on function inner in module __main__:
  
  inner(*args, **kwargs)
  In [60]: inspect.signature(mult)
  Out[60]: <Signature (*args, **kwargs)>
  ```

* To overcome above issue, we will use `wraps` function from `functools` module. It will copy-paste all the metadata:

  ```python
  In [64]: from functools import wraps
      ...:
      ...: def counter(fn):
      ...:     count = 0
      ...:
      ...:     @wraps(fn)
      ...:     def inner(*args, **kwargs):
      ...:         nonlocal count
      ...:         count += 1
      ...:         print("{0} was called {1} times".format(fn.__name__, count))
      ...:     return inner
      ...:
  
  In [65]: @counter
      ...: def add(a: int, b: int=10) -> int:
      ...:     """
      ...:     returns sum of two integers
      ...:     """
      ...:     return a + b
      ...:
  
  In [66]: help(add)
  Help on function add in module __main__:
  
  add(a: int, b: int = 10) -> int
      returns sum of two integers
  
  In [68]: inspect.signature(add)
  Out[68]: <Signature (a: int, b: int = 10) -> int>
  ```

* Now let's write a decorator to time a function:

  ```python
  In [76]: def timed(fn):
      ...:     from time import perf_counter
      ...:     from functools import wraps
      ...:
      ...:     @wraps(fn)
      ...:     def inner(*args, **kwargs):
      ...:         start = perf_counter()
      ...:         result = fn(*args, **kwargs)
      ...:         end = perf_counter()
      ...:         elapsed = end - start
      ...:
      ...:         args_ = [str(a) for a in args]
      ...:         kwargs_ = ['{0}={1}'.format(k, v) for (k, v) in kwargs.items()]
      ...:
      ...:         all_args = args_ + kwargs_
      ...:         args_str = ','.join(all_args)
      ...:         print('{0}({1}) took {2:.6f}s to run.'.format(fn.__name__,
      ...:                                                          args_str,
      ...:                                                          elapsed))
      ...:         return result
      ...:
      ...:     return inner
      ...:
  
  In [77]: @timed
      ...: def fib_loop(n):
      ...:     fib_1 = 1
      ...:     fib_2 = 1
      ...:     for i in range(3, n+1):
      ...:         fib_1, fib_2 = fib_2, fib_1 + fib_2
      ...:     return fib_2
      ...:
  
  In [78]: fib_loop(35)
  fib_loop(35) took 0.000006s to run.
  Out[78]: 9227465
  ```

* Similarly we can write another utility decorator to perform logging:

  ```python
  In [79]: def logged(fn):
      ...:     from functools import wraps
      ...:     from datetime import datetime, timezone
      ...:
      ...:     @wraps(fn)
      ...:     def inner(*args, **kwargs):
      ...:         run_dt = datetime.now(timezone.utc)
      ...:         result = fn(*args, **kwargs)
      ...:         print('{0}: called {1}'.format(fn.__name__, run_dt))
      ...:         return result
      ...:
      ...:     return inner
      ...:
  
  In [80]: @logged
      ...: def func_1():
      ...:     pass
      ...:
      ...: @logged
      ...: def func_2():
      ...:     pass
      ...:
  
  In [81]: func_1()
  func_1: called 2021-02-20 03:23:52.021363+00:00
  
  In [82]: func_2()
  func_2: called 2021-02-20 03:23:56.142286+00:00
  ```

* We can use multiple decorators as well:

  ```python
  In [83]: @timed
      ...: @logged
      ...: def factorial(n):
      ...:     from operator import mul
      ...:     from functools import reduce
      ...:     return reduce(mul, range(1, n+1))
      ...:
      
  In [84]: factorial(10)
  factorial: called 2021-02-20 03:25:06.991704+00:00
  factorial(10) took 0.000046s to run.
  Out[84]: 3628800
  ```

#### Memoization

* As we know, as part of recursion for factorial, function is called for every iteration. This can be avoided using memoization:

  ```python
  In [86]: class Fib:
      ...:     def __init__(self):
      ...:         self.cache = {1: 1, 2: 1}
      ...:
      ...:     def fib(self, n):
      ...:         if n not in self.cache:
      ...:             print('Calculating fib({0})'.format(n))
      ...:             self.cache[n] = self.fib(n-1) + self.fib(n-2)
      ...:         return self.cache[n]
      ...:
  
  In [87]: f = Fib()
      ...: f.fib(6)
  Calculating fib(6)
  Calculating fib(5)
  Calculating fib(4)
  Calculating fib(3)
  Out[87]: 8
  
  In [88]: f.fib(7)
  Calculating fib(7)
  Out[88]: 13
  ```

* Performing the same using decorators:

  ```python
  In [95]: def memoize(fn):
      ...:     cache = dict()
      ...:
      ...:     @wraps(fn)
      ...:     def inner(*args):
      ...:         if args not in cache:
      ...:             cache[args] = fn(*args)
      ...:         return cache[args]
      ...:
      ...:     return inner
      ...:
  
  In [96]: @memoize
      ...: def fib(n):
      ...:     print ('Calculating fib({0})'.format(n))
      ...:     return 1 if n < 3 else fib(n-1) + fib(n-2)
      ...:
  
  In [97]: fib(3)
  Calculating fib(3)
  Calculating fib(2)
  Calculating fib(1)
  Out[97]: 2
  
  In [98]: fib(3)
  Out[98]: 2
  
  In [99]: fib(4)
  Calculating fib(4)
  Out[99]: 3
    
  In [100]: @memoize
       ...: def fact(n):
       ...:     print('Calculating {0}!'.format(n))
       ...:     return 1 if n < 2 else n * fact(n-1)
       ...:
  
  In [101]: fact(3)
  Calculating 3!
  Calculating 2!
  Calculating 1!
  Out[101]: 6
  
  In [102]: fact(3)
  Out[102]: 6
  
  In [103]: fact(4)
  Calculating 4!
  Out[103]: 24
  ```

* Memoization technique is a common technique and hence Python provides a built-in way to perform it using  `lru_cache`:

  ```python
  In [104]: from functools import lru_cache
  
  In [105]: @lru_cache
       ...: def fact(n):
       ...:     print('Calculating {0}!'.format(n))
       ...:     return 1 if n < 2 else n * fact(n-1)
       ...:
  
  In [106]: fact(3)
  Calculating 3!
  Calculating 2!
  Calculating 1!
  Out[106]: 6
  
  In [107]: fact(3)
  Out[107]: 6
  
  In [108]: fact(4)
  Calculating 4!
  Out[108]: 24
  ```

* We can pass maxsize of the cache to the decorator:

  ```python
  In [161]: @lru_cache(maxsize=4)
       ...: def fact(n):
     ...:     print('Calculating {0}!'.format(n))
       ...:     return 1 if n < 2 else n * fact(n-1)
       ...:
  
  In [162]: fact(10)
  Calculating 10!
  Calculating 9!
  Calculating 8!
  Calculating 7!
  Calculating 6!
  Calculating 5!
  Calculating 4!
  Calculating 3!
  Calculating 2!
  Calculating 1!
  Out[162]: 3628800
  
  In [163]: fact(10)
  Out[163]: 3628800
  
  In [164]: fact(9)
  Out[164]: 362880
  
  In [165]: fact(8)
  Out[165]: 40320
  
  In [166]: fact(7)
  Out[166]: 5040
  
  In [167]: fact(6)
  Calculating 6!
  Calculating 5!
  Calculating 4!
  Calculating 3!
  Calculating 2!
  Calculating 1!
  Out[167]: 720
  ```
#### Decorator parameters
* A decorator only takes 1 argument:

  ```python
  In [118]: def dec(fn):
       ...:     print ("running dec")
       ...:
       ...:     def inner(*args, **kwargs):
       ...:         print("running inner")
       ...:         return fn(*args, **kwargs)
       ...:
       ...:     return inner
       ...:
       ...: @dec
       ...: def my_func():
       ...:     print('running my_func')
       ...:
  running dec
  
  In [119]: my_func()
  running inner
  running my_func
  ```

* So to pass an arg to decorator, we must add a function which returns a decorator. This is called decorator factory:

  ```python
  In [121]: def dec_factory():
       ...:     print("running dec_factory")
       ...:     def dec(fn):
       ...:         print("running a dec")
       ...:         def inner(*args, **kwargs):
       ...:             print("running inner")
       ...:             return fn(*args, **kwargs)
       ...:         return inner
       ...:     return dec
       ...:
  
  In [122]: @dec_factory()
       ...: def my_func(a, b):
       ...:     print(a, b)
       ...: my_func(10, 20)
  running dec_factory
  running a dec
  running inner
  10 20
  ```

* Using above we can now pass params to decorator:

  ```python
  In [123]: def dec_factory(a, b):
       ...:     def dec(fn):
       ...:         def inner(*args, **kwargs):
       ...:             print('running decorator inner')
       ...:             print('free vars: ', a, b)  # a and b are free variables!
       ...:             return fn(*args, **kwargs)
       ...:         return inner
       ...:     return dec
       ...:
       ...: @dec_factory(10, 20)
       ...: def my_func():
       ...:     print('python rocks')
       ...:
       ...: my_func()
  running decorator inner
  free vars:  10 20
  python rocks
  ```

#### Decorator Class

* We can make a class callable using `__call__` method. And by making the `__call__` function a decorator we make the class as decorator:

  ```python
  In [124]: class MyClass:
       ...:     def __init__(self, a, b):
       ...:         self.a = a
       ...:         self.b = b
       ...:
       ...:     def __call__(self, fn):
       ...:         def inner(*args, **kwargs):
       ...:             print('MyClass instance called: a={0}, b={1}'.format(self.
       ...: a, self.b))
       ...:             return fn(*args, **kwargs)
       ...:         return inner
       ...:
  
  In [125]: @MyClass(10, 20)
       ...: def my_func(s):
       ...:     print('Hello {0}!'.format(s))
       ...:
  
  In [126]: my_func("Ashish")
  MyClass instance called: a=10, b=20
  Hello Ashish!
  ```


#### Decorating Classes

* We can always add new function to an existing class:

  ```python
  In [127]: from fractions import Fraction
  
  In [128]: Fraction.speak = lambda self: 'This is a late parrot.'
  
  In [129]: Fraction.is_integral = lambda self: self.denominator == 1
  
  In [130]: f = Fraction(2, 3)
  
  In [131]: f.speak()
  Out[131]: 'This is a late parrot.'
  
  In [132]: f.is_integral()
  Out[132]: False
  ```

* We can add a decorator to do same as above as well:

  ```python
  In [133]: def dec_speak(cls):
       ...:     cls.speak = lambda self: 'This is a very late parrot.'
       ...:     return cls
       ...:
  
  In [134]: @dec_speak
       ...: class Parrot:
       ...:     def __init__(self):
       ...:         self.state = 'late'
       ...:
  
  In [135]: polly = Parrot()
       ...: polly.speak()
  Out[135]: 'This is a very late parrot.'
  ```

* Here's an example on how powerful this is:

  ```python
  In [136]: from datetime import datetime, timezone
  
  In [137]: def debug_info(cls):
     ...:     def info(self):
     ...:         results = []
     ...:         results.append('time: {0}'.format(datetime.now(timezone.utc)))
     ...:         results.append('class: {0}'.format(self.__class__.__name__))
     ...:         results.append('id: {0}'.format(hex(id(self))))
     ...:
     ...:         if vars(self):
     ...:             for k, v in vars(self).items():
     ...:                 results.append('{0}: {1}'.format(k, v))
     ...:
     ...:         # we have not covered lists, the extend method and generators,
     ...:         # but note that a more Pythonic way to do this would be:
     ...:         #if vars(self):
     ...:         #    results.extend('{0}: {1}'.format(k, v)
     ...:         #                   for k, v in vars(self).items())
     ...:
     ...:         return results
     ...:
     ...:     cls.debug = info
     ...:
     ...:     return cls
     ...:
  
  In [138]: @debug_info
       ...: class Person:
       ...:     def __init__(self, name, birth_year):
       ...:         self.name = name
       ...:         self.birth_year = birth_year
       ...:
       ...:     def say_hi():
       ...:         return 'Hello there!'
       ...:
  
  In [139]: p1 = Person('John', 1939)
  
  In [140]: p1.debug()
  Out[140]:
  ['time: 2021-02-20 04:06:20.901160+00:00',
   'class: Person',
   'id: 0x10b0612e0',
   'name: John',
   'birth_year: 1939']
  ```

#### Single Dispatch Generic Function

* Below is simple html parser, with `htmlize` as a dispatch function

  ```python
  In [144]: from html import escape
       ...:
       ...: def html_escape(arg):
       ...:     return escape(str(arg))
       ...:
       ...: def html_int(a):
       ...:     return '{0}(<i>{1}</i)'.format(a, str(hex(a)))
       ...:
       ...: def html_real(a):
       ...:     return '{0:.2f}'.format(round(a, 2))
       ...:
       ...: def html_str(s):
       ...:     return html_escape(s).replace('\n', '<br/>\n')
       ...:
       ...: def html_list(l):
       ...:     items = ('<li>{0}</li>'.format(html_escape(item))
       ...:              for item in l)
       ...:     return '<ul>\n' + '\n'.join(items) + '\n</ul>'
       ...:
       ...: def html_dict(d):
       ...:     items = ('<li>{0}={1}</li>'.format(html_escape(k), html_escape(v))
       ...:
       ...:              for k, v in d.items())
       ...:     return '<ul>\n' + '\n'.join(items) + '\n</ul>'
       
  In [145]: print(html_str("""this is
       ...: a multi line string
       ...: with special characters: 10 < 100"""))
  this is <br/>
  a multi line string<br/>
  with special characters: 10 &lt; 100
  
  In [146]: from decimal import Decimal
       ...:
       ...: def htmlize(arg):
       ...:     if isinstance(arg, int):
       ...:         return html_int(arg)
       ...:     elif isinstance(arg, float) or isinstance(arg, Decimal):
       ...:         return html_real(arg)
       ...:     elif isinstance(arg, str):
       ...:         return html_str(arg)
       ...:     elif isinstance(arg, list) or isinstance(arg, tuple):
       ...:         return html_list(arg)
       ...:     elif isinstance(arg, dict):
       ...:         return html_dict(arg)
       ...:     else:
       ...:         # default behavior - just html escape string representation
       ...:         return html_escape(str(arg))
       ...:
       
  In [147]: print(htmlize(["""first element is
       ...: a multi-line string""", (1, 2, 3)]))
  <ul>
  <li>first element is
  a multi-line string</li>
  <li>(1, 2, 3)</li>
  </ul>
  ```

* Above dispatch function (`htmlize`) has lots of if/elif cases, let's handle this in a subtle way:

  ```python
  In [151]: def singledispatch(fn):
       ...:     registry = dict()
       ...:
       ...:     registry[object] = fn
       ...:     registry[int] = lambda arg: '{0}(<i>{1}</i)'.format(arg, str(hex(a
       ...: rg)))
       ...:     registry[float] = lambda arg: '{0:.2f}'.format(round(arg, 2))
       ...:
       ...:     def inner(arg):
       ...:         fn = registry.get(type(arg), registry[object])
       ...:         return fn(arg)
       ...:     return inner
       ...:
  
  In [152]: @singledispatch
       ...: def htmlize(a):
       ...:     return escape(str(a))
       ...:
  
  In [153]: htmlize(10)
  Out[153]: '10(<i>0xa</i)'
  
  In [154]: htmlize(3.34554)
  Out[154]: '3.35'
  ```