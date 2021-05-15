# All about Booleans

**Python Version:** 3.9.1 (Clang 12.0.0)

#### Booleans: Class

* `True` and `False` are **singleton instances** of `bool` class i.e Can only have one instance in whole code and hence will always point to same memory address

  ```python
  >>> type(True), id(True), int(True)
  (<class 'bool'>, 4441390736, 1)
  >>> type(False), id(False), int(False)
  (<class 'bool'>, 4441390768, 0)
  >>> id(True), id(1<2)
  (4441390736, 4441390736)
  ```

* Boolean are subclassed from  `int` class. Hence we can perform same operations like integers on boolean:

  ```python
  >>> issubclass(bool, int)
  True
  >>> 4 + 5, True + True
  (9, 2)
  >>> (True*5)%2
  1
  ```

* Only 0 is `False`, every other number is `True`: `bool(x) = True for any value except 0`

  ```python
  >>> bool(-40), bool(34), bool(0)
  (True, True, False)
  ```

* `int` value of `False` is `0` & `int` value of `True` is `1`

* But `id(1) is not id(True)`

  ```python
  >>> id(1), id(True)
  (4442229040, 4441390736)
  ```

* Every `int` class object will have `__bool__` function. Most of the objects will implement `__bool__() or __len__()`. Even if they don't, the bool value of that class will be `True`

  ```python
  >>> from fractions import Fraction
  >>> from decimal import Decimal
  >>> bool(10), bool(1.5), bool(Fraction(3, 4)), bool(Decimal('-1.2'))
  (True, True, True, True)
  >>> bool(0), bool(-0), bool(Fraction(0, 4)), bool(Decimal('0'))
  (False, False, False, False)
  ```

* Empty list/set/dict/tuple/string & None object will give `False`:

  ```python
  >>> bool([1, 2, 3]), bool(''), bool([]), bool(set()), bool({}), bool(None)
  (True, False, False, False, False, False)
  ```

* Objects when used in conditional expressions will use associated value if it is not of bool type:

  ```python
  >>> a = [1, 2, 3]
  >>> if a: print("yes")
  ...
  yes
  ```

#### Booleans: Precedence and Short-Circuiting

* `and` operation has higher precedence than `or operation

  ```python
  >>> True or True and False
  True
  >>> True or (True and False)
  True
  >>> (True or True) and False
  False
  ```

* Python performs short-circuting when it can figure out output of complete expression by just partial evaluation:

  ````python
  >>> a = 10
  >>> b = 0
  >>> if a/b: print("yes")
  ...
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  ZeroDivisionError: division by zero
  >>> if b and a/b: print("yes")
  ...
  >>>
  ````

* So it evaluated `b` and saw it is False and hence didn't continue with `a/b`

* Another example:

  ```python
  >>> import string
  >>> name = ''
  >>> if name[0] in string.digits: print("yes")
  ...
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  IndexError: string index out of range
  >>> if name and name[0] in string.digits: print("yes")
  ...
  >>>
  ```

* Hence always ensure we check existince of the object first and then do the check

#### Booleans: Operators

* **or** operator: `X or Y` -> If `X` is false, return `Y`, otherwise evaluates and returns` X`. So only `X` is evaluated and never `Y`

  ```python
  >>> '' or 'abc'
  'abc'
  >>> 0 or 100
  100
  >>> [] or [1, 2, 3]
  [1, 2, 3]
  >>> [1, 2] or [1, 2, 3]
  [1, 2]
  >>> '' or []
  []
  ```

  * Hence we can also define our own or function:

  ```python
  >>> def _or(x, y):
      if x:
          return x
      else:
          return y
  ...
  >>> _or(0, 100)
  100
  >>> _or(1, 1/0) # BUT!! Here both X & Y are evaluated
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  ZeroDivisionError: division by zero
  ```

* **and** operator: `X and Y` -> If `X` if falsy, return 'X', otherwise evaluates and returns `Y`

  ```python
  >>> s1 = None
  >>> s2 = ''
  >>> s3 = 'abc'
  >>> s1 and s1[0]
  >>> s2 and s2[0]
  ''
  >>> s3 and s3[0]
  'a'
  ```

  * So python does short-ciruiting in above case as well. For example: `s1 and s1[0]` only evaluated `s1` and not `s1[0]` if it did it would've returned an error

* **not** operator:

  ```python
  >>> not 'abc'
  False
  >>> not ''
  True
  >>> not []
  True
  >>> not None
  True
  ```



# Comparison Operators

- Comparison of memory addresses of the objects is done using `is` and `is not`. [But gives warning from Python 3.8 onwards]:

  ```python
  >>> 0.1 is (3 + 4j)
  <stdin>:1: SyntaxWarning: "is" with a literal. Did you mean "=="?
  False
  >>> 1 is 1
  <stdin>:1: SyntaxWarning: "is" with a literal. Did you mean "=="?
  True
  ```

- Existence check is performed using `in`:

  ```python
  >>> 1 in [1, 2, 3]
  True
  >>> 'key1' in {'key1': 'rohan', 'key2': 'mohan'}
  True
  ```

- Value comparison is done using `==` and `!=`:

  ```python
  >>> from decimal import Decimal
  >>> from fractions import Fraction
  >>> Decimal('0.1') == Fraction(1, 10)
  True
  >>> 1 == 1 + 0j
  True
  >>> True == Fraction(2, 2)
  True
  >>> False == 0j
  True
  ```

- `<` `>`comparison:

  ```python
  >>> 3<4
  True
  >>> 1 + 1j < 2 + 2j
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: '<' not supported between instances of 'complex' and 'complex'
  >>> 1 < 'a'
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: '<' not supported between instances of 'int' and 'str'
  >>> Decimal('0.01') < Fraction(1, 10)
  True
  ```

- Chain comparison `a < b < c  equivalent to a < b and b < c`:

  ```python
  >>> 1 < 2 < 3
  True
  
  >>> 1 < 2 > -5 < 50 > 4
  True
  # If expression evaluates to True, then larger number is carried forward for the check
  # So above is:
  # 1 < 2: True
  # 2 > -5: True
  # 2 < 50: True
  # 50 > 4: True
  
  >>> import string
  >>> 'A' < 'a' < 'z' > 'Z' in string.ascii_letters
  True
  >>> 'A' < 'a' and 'a' < 'z' and 'z'> 'Z' and 'Z' in string.ascii_letters
  True
  ```



# Functions

#### Default & Non-Default Arguments

- Function supports default and non-default arguments:

  ```python
  >>> def my_func(a, b = 2, c =3):
  ...     print(f'a={a}, b={b}, c={c}')
  ...
  >>> my_func(1, 20, 30)
  a=1, b=20, c=30
  >>> my_func(1)
  a=1, b=2, c=3
  ```

- But non-default argument cannot follow default argument:

  ```python
  >>> def my_func(a, b = 2, c):
    File "<stdin>", line 1
      def my_func(a, b = 2, c):
                             ^
  SyntaxError: non-default argument follows default argument
  ```

#### Keyword Arguments (named arguments)

- Can specify names to arguments while passing it to functions:

  ```python
  >>> my_func(10, c = 20, b = 40)
  a=10, b=40, c=20
  >>> my_func(10, c = 20)
  a=10, b=2, c=20
  >>> my_func(10, b = 30, c = 40)
  a=10, b=30, c=40
  ```

- One must specify positional arguments:

  ```python
  >>> my_func(b = 30, c = 40)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: my_func() missing 1 required positional argument: 'a'
  ```

# Unpacking Iterables

- Right Hand Side is always evaluated first and then its values are unpacked and stored in variables on Left Hand Side:

  ```python
  >>> l = [1, 2, 3, 4]
  >>> a, b, c, d = l
  >>> print(a, b, c, d)
  1 2 3 4
  >>>
  >>> l = 'abcd'
  >>> a, b, c, d = l
  >>> print(a, b, c, d)
  a b c d
  >>>
  >>> a, b = 10, 20 # Just made a tuple and unpacked it
  >>> b, a = a, b
  >>> print(a, b)
  20 10
  ```

- One **comma** can make an argument **tuple**!:

  ```python
  >>> dict1 = {'a':1},
  >>> type(dict1)
  <class 'tuple'>
  ```

- Dicts are now ordered in Python (> 3.7):

  ```python
  >>> dict1 = {'p': 1, 'y': 2, 't': 3, 'h': 4, 'o': 5, 'n': 6}
  >>> for c in dict1: print(c)
  ...
  p
  y
  t
  h
  o
  n
  >>> a, b, c, d, e, f = dict1
  >>> print(a, c, e)
  p t o
  ```

- Order is not guaranteed in sets:

  ```python
  >>> s = {'p', 'y', 't', 'h', 'o', 'n'}
  >>> type(s), print(s)
  {'y', 't', 'o', 'n', 'p', 'h'}
  (<class 'set'>, None)
  >>> a, b, c, d, e, f = s
  >>> print(a, b, c, d, e, f)
  y t o n p h
  ```

- List unpacking:

  ```python
  >>> l = [1, 2, 3, 4, 5, 6]
  >>> l[:]
  [1, 2, 3, 4, 5, 6]
  >>> l[-1:] # From last index to end
  [6]
  >>> l[:-1] # From start index to last-1 index
  [1, 2, 3, 4, 5]
  >>> l[:-2] # From start index to last-2 index
  [1, 2, 3, 4]
  >>> l[None:None] # From none (defaults to start index) to none (defaults to end index)
  [1, 2, 3, 4, 5, 6]
  ```

- Unpack multiple values to one variable:

  ```python
  >>> l
  [1, 2, 3, 4, 5, 6]
  >>> a, b = l[0], l[1:]
  >>> a, b
  (1, [2, 3, 4, 5, 6])
  >>>
  >>> a, b = l
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  ValueError: too many values to unpack (expected 2)
  >>>
  >>> a, *b = l
  >>> a, b
  (1, [2, 3, 4, 5, 6])
  ```

- Same can be done with tuple:

  ```python
  >>> t = -10, 34, 12, 45
  >>> type(t)
  <class 'tuple'>
  >>> a, *b = t
  >>> a, b
  (-10, [34, 12, 45])
  >>> type(a), type(b)
  (<class 'int'>, <class 'list'>)
  ```

- Same can be done with string:

  ```python
  >>> a, *b = 'rohan'
  >>> a, b
  ('r', ['o', 'h', 'a', 'n'])
  ```

- Python is smart in unpacking other way round as well:

  ```python
  >>> *a, b = 'rohan'
  >>> a, b
  (['r', 'o', 'h', 'a'], 'n')
  >>>
  >>> a, b, c
  (['r', 'o', 'h'], 'a', 'n')
  >>>
  >>> s = 'aeroplane'
  >>> a, b, *c, d = s
  >>> a, b, c, d
  ('a', 'e', ['r', 'o', 'p', 'l', 'a', 'n'], 'e')
  >>>
  >>> a, b, (c, d, *e, f) = [1, 2, 'python']
  >>> c, d, f
  ('p', 'y', 'n')
  ```

- Above is also called slicing, a naive way of slicing is:

  ```python
  >>> a, b, c, d = s[0], s[1], s[2:-1], s[-1]
  >>> a, b, c, d
  ('a', 'e', 'roplan', 'e')
  ```

- But obvious failures should be avoided:

  ```python
  >>> *a, *b = rohan
    File "<stdin>", line 1
  SyntaxError: multiple starred expressions in assignment
  ```

- Unpacking can be done in other direction as well:

  ```python
  >>> l1 = [1, 2, 3]
  >>> l2 = [4, 5, 6]
  >>> l = [*l1, *l2]
  >>> l
  [1, 2, 3, 4, 5, 6]
  >>>
  >>> s = 'ABC'
  >>> l = [*l1, *s]
  >>> l
  [1, 2, 3, 'A', 'B', 'C']
  ```

- Unpacking sets:

  ```python
  >>> s1 = {1, 2, 3}
  >>> s2 = {4, 5, 6}
  >>> s1.union(s2)
  {1, 2, 3, 4, 5, 6}
  >>> print({*s1, *s2})
  {1, 2, 3, 4, 5, 6}
  ```

- Unpacking dicts:

  ```python
  >>> d1 = {'key1': 1, 'key2': 2}
  >>> d2 = {'key2': 3, 'key3': 3}
  >>> [*d1, *d2] # ONLY keys
  ['key1', 'key2', 'key2', 'key3']
  >>>
  >>> {**d1, **d2}
  {'key1': 1, 'key2': 3, 'key3': 3}
  >>>
  >>> {**d1, **d2} # Unpacked dicts into one with common keys OVERRIDDEN!
  {'key1': 1, 'key2': 3, 'key3': 3}
  >>>
  >>> d1 = {'key1': 1, 'key2': 2}
  >>> d2 = {'key2': 3, 'key3': 3}
  >>> {'a': 1, 'b': 3, **d1, **d2, 'c': 3}
  {'a': 1, 'b': 3, 'key1': 1, 'key2': 3, 'key3': 3, 'c': 3}
  ```

  

# *args && *kwargs

#### *args

* args/kwargs are just naming conventions, they can be of any name

* `*args` can take any number of arguments

  ```python
  >>> def func1(a, b, *args):
  ...     print(a)
  ...     print(b)
  ...     print(args)
  ...
  >>> func1(1, 2, 3, 4, 5 , 7, 8)
  1
  2
  (3, 4, 5, 7, 8) # Is a TUPLE!
  ```

* keyword only arguments requires named argument:

  ```python
  >>> def func1(a, b, *args, d):
  ...     print(d)
  ...
  >>> func1(1, 2, 3, 4 ,5 , 7, 8)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: func1() missing 1 required keyword-only argument: 'd'
  >>>
  >>> func1(1, 2, 3, 4, 5, 7, d=8)
  8
  >>>
  >>> def func1(a, b, c, *d):
  ...     print(a, b, c, d)
  ...
  >>> func1(10, c=30, b = 20, 'a', 'b')
    File "<stdin>", line 1
      func1(10, c=30, b = 20, 'a', 'b')
                                      ^
  SyntaxError: positional argument follows keyword argument
  >>>
  >>> def func1(a, b, *c, d):
  ...     print(a, b, c, d)
  ...
  >>> func1(1, 1, 2, d=4)
  1 1 (2,) 4
  ```

* Unpacking arguments:

  ```python
  >>> def func1(a, b, c):
  ...     print(a, b, c)
  ...
  >>> l = [10, 20, 30]
  >>> func1(*l)
  10 20 30
  ```

* Hence the rule here is: Before the `*` operator, no named arguments. After the `*` operator, only named arguments

  ```python
  >>> def func1(a, b, *c, d):
  ...     print(a, b, c, d)
  ...
  >>> func1(1, 2, 3, d = 4)
  1 2 (3,) 4
  >>> func1(1, 2, d = 4)
  1 2 () 4
  ```

* In a way named arguments are helping code to be more readable. What if we want to mandatorily make user give all arguments as named arguments? We can do something like this:

  ```python
  >>> def func(*args, a, b, c):
  ...     print(a, b, c)
  ...
  >>> func(1, 2, 3)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: func() missing 3 required keyword-only arguments: 'a', 'b', and 'c'
  >>> func(a=1, b=2, c=3)
  1 2 3
  ```

  But a user might send other args:

  ```python
  >>> func(1, a=1, b=2, c=3)
  1 2 3
  ```

  Not what we wanted, we must **exhaust** all positional arguments!?

  ```python
  >>> def func(*, a, b, c):
  ...     print(a, b, c)
  ...
  >>> func(a=1, b=2, c=3)
  1 2 3
  >>>
  >>> func(1, a=1, b=2, c=3)
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  TypeError: func() takes 0 positional arguments but 1 positional argument (and 3 keyword-only arguments) were given
  ```

  Hence we can use just `*` as an argument to **exhaust** all positional arguments and only allow keyword arguments. and `*args` to **absorb** all positional arguments

* Other examples:

  ```python
  >>> def func(a, b = 10, *, c, d = 12):
  ...     print(a, b, c, d)
  ...
  >>> func(1, c = 12)
  1 10 12 12
  >>>
  >>> def func(a, b, *args, c, d = 12):
  ...     print(a, b, args, c, d)
  ...
  >>> func(1, 2, 3, 4, 5, 6, 7, 8, 90, c = 10)
  1 2 (3, 4, 5, 6, 7, 8, 90) 10 12
  ```

  

#### **kwargs

* Makes arguments available as dictionary:

  ```python
  >>> def func(**kwargs):
  ...     print(kwargs)
  ...
  >>> func(x=100, y=200)
  {'x': 100, 'y': 200}
  >>>
  >>> # Can mix it up with *args as well
  >>> def func(*args, **kwargs):
  ...     print(args)
  ...     print(kwargs)
  ...
  >>> func(1, 2, a = 100, b = 200)
  (1, 2)
  {'a': 100, 'b': 200}
  ```

* `**kwargs` can only take values ONLY after all positional arguments are **exhausted**

  ```python
  >>> def func(a, *, d, **kwargs):
  ...     print(d, kwargs)
  ...
  >>> func(3, d = 4)
  4 {}
  ```

* NOTE: before `*args` only positional arguments are allowed:

  ```python
  >>> def func(a, b, *args):
  ...     print(a, b, args)
  >>>
  >>> func(a = 1, b = 2, 'a', 'f', 45)
    File "<stdin>", line 1
      func(a = 1, b = 2, 'a', 'f', 45)
                                     ^
  SyntaxError: positional argument follows keyword argument
  ```

* But`**kwargs` can do the above:

  ```python
  >>> def func(a, b, *args, **kwargs):
  ...     print(a, b, args, kwargs)
  ...
  >>> func(a = 1, b = 2, c = 10)
  1 2 () {'c': 10}
  ```

* Other examples:

  ```python
  >>> def func(a, b, *, c, d=4, **kwargs):
  ...     print(a, b, c, d, kwargs)
  ...
  >>> func(1, 2, c= 13, e = 'e')
  1 2 13 4 {'e': 'e'}
  >>>
  >>> def func(a, b, *args, c, d = 14, **kwargs):
  ...     print(a, b, c, args, d, kwargs)
  ...
  >>> func('a', 'b', 1, 2, 3, 4, c = 10, d = 'arlington', e = '34')
  a b 10 (1, 2, 3, 4) arlington {'e': '34'}
  ```

* `def func(*args, **kwargs):` is used heavily as it can take any type of inputs:

  ```python
  >>> def func(*args, **kwargs):
  ...     print(args, kwargs)
  ...
  >>> func(1, 2, 3, 4, 5, 6, d = 10, a = 10, g = 10, gg = '12312')
  (1, 2, 3, 4, 5, 6) {'d': 10, 'a': 10, 'g': 10, 'gg': '12312'}
  ```

* Sample code which shows the concepts we have learned:

  ```python
  >>> def calc_hi_lo_avg(*args, log_to_console = False):
  ...     hi = int(bool(args)) and max(args)
  ...     lo = int(bool(args)) and min(args)
  ...     avg = (hi + lo)/2
  ...     if log_to_console:
  ...             print(f'high = {hi}, low = {lo}, avg = {avg}')
  ...     return avg
  ...
  >>> print(calc_hi_lo_avg(1, 2, 3, 4, 5, 6, 7, 8, log_to_console = True))
  high = 8, low = 1, avg = 4.5
  4.5
  ```

  **Note:** `int(bool(args)) and max(args)` This line makes sure only integer values reaches `max()` function