### Modules

* Modules can be imported using the `import` statement. When a module is imported:

  * System cache is checked first (`sys.modules`)
  * Otherwise, locate it from finders (e.g: `sys.meta_path`: gets list of finders). Module is retrieved (loaded) from loaders (returned by finders). `empty` module typed object is created. A reference to the module is added to the system cache (`sys.modules`) and the module is compiled. When the module is executed, it sets up the module's namespace

* Module Finders:

  ```
  In [2]: sys.meta_path
  Out[2]:
  [_frozen_importlib.BuiltinImporter, #--> finds built-ins, such as math
   _frozen_importlib.FrozenImporter,  #--> finds frozen modules
   _frozen_importlib_external.PathFinder] #--> file-based modules
  ```

* PathFinder: Finds file-based modules based on `sys.path` and package `__path__`

* Module Properties: `__spec__`, `__name__`, `__package__` (empty for modules), `__file__` (found by PathFinder)

* Modules may reside:

  * in the built-ins
  * in files on disk
  * they can even be pre-compiled, frozen or even inside zip archives
  * anywhere else that can be accessed by a finder and a loader
  * For file based modules (PathFinder), they must exist in path specified in `sys.path` or in a path specified by `<package>.__path__`

### Packages

* Packages are modules. They can contain modules, packages.

* If a module is a package, then it must have a value set for `__path__

* Packages represent a hierarchy of modules/packages

* How to define package?: Directory name becomes package name & `__init__.py` is where the code for package will go.

* If we don't have the `__init__.py` file, then Python will create an implicit namespace package

* Package will have `__path__` & `__file__` property defined:

  * `__path__` : file system directory path (absolute)
  * `__file__`: file system path to `__init__.py` (absolute)

* Module will also have above properties:

  * `__file__` : location of module code in file system
  * `__package__`: package the module code is located in (is empty string if module is located in application root)
  * `__path__`: only set if module is also a package. Location of the package (directory) in the file system

* Nested packages:

  ```
  app/
      module.py
      pack1/
          __init__.py
          module1.py
          module2.py
          pack1_1/
              __init__.py
              module1_1a.py
              module1_1b.py
  ```

  * module inside `pack1`: `pack1.module1`
  * package inside `pack1`: `pack1.pack1_1`
  * module inside `pack1.pack1_1`: `pack1.pack1_1.module1_1a`
  * `module.__file__`:`.../app/module.py`
  * `module:__path__`: Not set
  * `module.__package__`: `""`
  * `pack1.__file__`:`.../app/pack1/__init__.py`
  * `pack1.__path__`: `.../app/pack1/`
  * `pack1.__package__ `: `pack1`
  * Python will load all the modules referenced in the import statement and the modules in the files of the imported modules and store it in `sys.modules`

* `__all__`: List of public objects of a module, as interpreted by `import *`. It overrides the default of hiding everything that begins with an underscore.

* Example:

  ```python
  # date.py
  
  __all__ = ['is_date', 'constant', 'date_helper_1']
  
  constant = 3.14
  
  def is_date(arg):
      pass
  
  def date_helper_1():
      pass
  
  def date_helper_2():
      pass
  
  def constant():
      pass
  ```

  * Only `'is_date', 'constant', 'date_helper_1'` can be used. Other funcs are hidden.