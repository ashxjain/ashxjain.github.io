#Understanding Python Memory Usage & Optimizations

### Debugging Cyclic References

Following code shows how cyclic references are made and with no proper cleanup can lead to bloating of memory:

```python
class Something(object):
  def __init__(self):
		# References 'SomethingNew' object

class SomethingNew(object):
  def __init__(self):
		# References 'Something' object

def add_something(collection: List[Something], i: int):
  # Initializes 'Something' object
  # Adds 'Something' object to 'collection' list
  
def clear_memory(collection: List[Something]):
  # Emtpies the 'collection' list, thereby frees the reference to 'Something' objects
  
def critical_function():
  # Add multiple 'Something' objects to 'collection' list by using 'add_something' function
  # Once it is done adding the objects to the list, it then clears the list
```

* As seen above, `Something` references `SomethingNew` & vice versa. And collection stores list of multiple `Something` objects. On clearing `collection` list, reference to all the `Something` object is freed. But `Something` object is not yet freed from the memory. This is because `SomethingNew` is still refering to `Something` object and hence the reference count is not 0. Following table shows the reference count (RefCnt):

  | Step                                         | Object       | RefCnt | Notes                                                        |
  | -------------------------------------------- | ------------ | ------ | ------------------------------------------------------------ |
  | Something object is added to collection list | Something    | 2      | Referenced by `collection` object and by `SomethingNew` object |
  |                                              | SomethingNew | 1      | Referenced by `Something` object                             |
  | collection list is cleared                   | Something    | 1      | Referenced by `SomethingNew`                                 |
  |                                              | SomethingNew | 1      | Referenced by `Something` object                             |

* Although garbage collector will free up cyclic references as it runs periodically. One can also free it up by breaking the cyclic reference before emptying the `collection` list. This is done by changing the reference of `SomethingNew` object to a `None` object

### Improving performance using Python Interning

Following code shows a program which can be improved in performance by using Python's interning optimization:

```python
def compare_strings_old(n):
	# compares two very long strings
	# checks if character exists in char_list
	
def compare_strings_new(n):
	# sleeps for 6s
```

* `compare_strings_old` performs two operations (1) String comparison (2) Character existince in string. There two performed `n` number of times. `n` is set to a huge number to get performance benchmark on how much it takes for Python to perform these two operations
* `compare_strings_new` is a placeholder function with a sleep function for us to rewrite old function with improved performance
* Comparing contents of object with another is time consuming. By interning the the strings, this can be optimized as post interning if object contents are same, they are assigned same memory address. Hence we can just match memory addresses of the strings rather than matching its contents
* Existence check is made on the list of characters. List is mutable object, which compiler optimizes it to a tuple (immutable object) thereby improving performance. But it can further be improved by using a `set` datatype. As `set` hashes the characters just like a dictionary, existence check performance is improved significantly. Moreover, compiler optimized `set` (mutable object) to a frozenset (immutable object), which again increases the performance

### Summary

* Avoid cyclic references
* Use python interning feature for improved performance
* Use `set` instead of `list` for existence checks
* Use immutable objects as much as possible

