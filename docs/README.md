---
home: false
sidebarDepth: 4
---


# Pysanity

## Auto Formatters

* Use [black](https://github.com/psf/black/) with default settings (max line length 88 characters).
* Use [flake8](https://github.com/PyCQA/flake8) to catch linting errors.
* Use [isort](https://github.com/timothycrosley/isort) to sort the imports.

:::warning

However, be careful while sorting imports of application modules. It might accidentally add circular dependencies. For example in Flask applications, changing the import order in flask `app/package/__init__.py` can mess up the codebase.

:::

```python
from flask import Blueprint

package = Blueprint("package", __name__)

from . import views
```

To avoid this, add `# isort:skip` like this:

```python
from flask import Blueprint

package = Blueprint("package", __name__)

from . import views  # isort:skip
```

::: tip

`Black` isn't compatible with `isort`. So it's better to run black after running isort.

:::

### Whitespaces

* Use soft tabs (space character) set to 4 spaces as per PEP8.

```python
# don't
def foo():
∙∙return bar

# don't
def foo():
∙return bar

# do
def foo():
∙∙∙∙return bar
```


### Naming Conventions

* Use *snake_case* when naming **variables**, **functions**, and **instances**. Use it for *file names* too as they'll be used in imports.

```python
# don't
import myMod

anOBJEct = {}
thisIsAnObject = {}
def ThisisAFunction():
    ...

# do
import my_mod

anobject = {}
this_is_an_object = {}
def this_is_a_function():
    ...
```

* **Global constants**, that are usually defined on a module level, should be named in all capital letters with underscores separating the words.

```python
# don't
global_constant = 50

# do
GLOBAL_CONSTANT = 50
```

* Use PascalCase only when naming **classes**.

```python
# don't
class exampleDummyFactory():
    # ...

fact = exampleDummyFactory()

# do
class ExampleDummyFactory():
    # ...

fact = ExampleDummyFactory()
```

* Use underscores before private **attributes**, **methods**, **variables** and **functions**.

```python
# private methods and attributes demo


class SomeThing:
    """A class for demonstration."""

    def __init__(self):
        # private attribute
        self._private_att = 10

    def _private_method(self):
        """This is a private method."""
        pass

    def method_x(self):
        # using the private attribute
        res_private_att = self._private_att
        # calling the private method
        res_private_method = self._private_method()
```

```python
# private function demo


def _private_function():
    pass


def public_function():
    # calling the private function
    res_private_function = _private_function()
```

* Avoid single letter names. Use descriptive and meaningful names - tell what the function does,
or what data type an object is. Use `description_object` instead of `object_description`.

```python
# don't
def a():
    # ...

# do
def analogy():
    # ...

# don't - no convention to know what data type it's
df_raw_data = pd.DataFrame(raw_data)
id_dct_num = {"a": 1, "b": 2}

# do - convention to tell data type by the last term
raw_data_df = pd.DataFrame(raw_data)
id_num_dct = {"a": 1, "b": 2}

# don't - meaningless names, lost context
LIST_1 = ["Jack", "Alice", "Emily"]
# ... many lines of code later
for item in LIST_1:
    add_person(item)

# do
NAME_LIST = ["Jack", "Alice", "Emily"]
# ... many lines of code later
for name in NAME_LIST:
    add_person(name)
```

* Use singular or base words in naming; avoid using plural and instead append singular with the data type.

```python
# don't
def moves_object(x, y):
    # ...

# do
def move_object(x, y):
    # ...

# don't - inconsistent naming for same data type and usage
teacher = ["Michael"]
students = ["Jack", "Alice", "Emily"]
books = pd.DataFrame({"title": ["lorem", "ipsum"]})

for t in teacher:
    add_human(t)

for student in students:
    add_human(student)

for book in books:
    add_item(book) # wrong; iterate column name instead of book

# do
teacher_list = ["Michael"]
student_list = ["Jack", "Alice", "Emily"]
book_df = pd.DataFrame({"title": ["lorem", "ipsum"]})

for teacher in teacher_list:
    add_human(teacher)

for student in student_list:
    add_human(student)

# naming as df suggests it'll be treated as a dataframe
for idx, book in book_df.iterrow():
    add_item(book)
```

## Coding Guideline

### Variables
* Avoid mutable variables in global scope

    **Why:**

    ==> They hide the dependencies between the functions and classes and it is really hard to see how different parts of code are related to each other. This makes the usage, refactoring and testing hard, because you have no idea in what state the global variables should be in order to run the code successfully. This means you have to understand the whole code base in order to modify a single point of global variable usage. If your code base is changing actively, you are new in the project or is so big that it is hard to remember it completely this means you have to bublesort through all N previous functions and classes that use the global variable in order to write the function 'N+1'. This makes the usage of the global variables literally exponentially worse way to program compared to the usage of technique called dependency injection in large software projects.

    ==> If you run tests in parallel the tests mess up the global state.

    ==> If you debug a deep chain of functions, it is not easy to pick all the variables you have to monitor.

    ==> They tend to waste developer time in the form of merge conflicts.

    ==> They tend to put more workload to the persons responsible for the variables or manager classes and functions which manipulate these variables because of their central role and might cause mental breakdown if the work load is too high

### Functions

* Avoid mutable data types as default function/method arguments.

```python
# don't
def make_list(val, lst=[]):
    lst.append(val)
    return lst


make_list(1)
# => [1]
make_list(2)
# => [1, 2], instead of the new init [2]

# do
def make_list(val, lst=None):
    if lst is None:
        lst = []
    lst.append(val)
    return lst


init_list(1)
# >> [1]

init_list(2)
# >> [2]
```

* Robert C. Martin's [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) encourages that a function should only have a single responsibility. That means, it should do one thing and one thing only. It massively improves refactorability.

    ==> There can be only one reason ever to change the function: if the way in which it does that thing must change.

    ==> It also becomes clear when a function can be deleted: if, when making changes elsewhere, it becomes clear the function's single responsibility is no longer needed, simply remove it.

```python
# don't
# this function calculates multiple things and print them out at the same time
# ideally these two responsibilities can be split into two functions


def calculate_and_print_stats(list_of_numbers):
    total = sum(list_of_numbers)
    mean = statistics.mean(list_of_numbers)
    median = statistics.median(list_of_numbers)
    mode = statistics.mode(list_of_numbers)

    print("-----------------Stats-----------------")
    print(f"SUM: {total}")
    print(f"MEAN: {mean}")
    print(f"MEDIAN: {median}")
    print(f"MODE: {mode}")
```

```python
# do
# these two functions do the same things but each of them
# only has a single responsibility


def calculate_statistics(list_of_numbers):
    """Calculates arithmatic sum, mean, median and mode."""

    total = sum(list_of_numbers)
    mean = statistics.mean(list_of_numbers)
    median = statistics.median(list_of_numbers)
    mode = statistics.mode(list_of_numbers)

    return total, mean, median, mode


def print_statistics(total, mean, median, mode):
    """Prints statistics on the console."""

    print("-----------------Stats-----------------")
    print(f"SUM: {total}")
    print(f"MEAN: {mean}")
    print(f"MEDIAN: {median}")
    print(f"MODE: {mode}")
```

* Strive to write **pure** or at least **idempotent** functions as much as possible.

==> **An idempotent function** always returns the same value given the same set of arguments, regardless of how many times it's called. The result does not depend on non-local variables, the mutability of arguments, or data from any I/O streams.

The following function is idempotent. The result of `square_num(5)` is always going to return `25`, regardless of how many times it's called:

```python
def square_num(number):
    """This is an idempotent function."""

    return number ** 2
```

This function takes in user input and is not idempotent.

```python
def square_num():
    """Return number_entered_by_user ** 2."""

    number = int(input("Enter a number: "))
    return number ** 2
```

**Why:**
Idempotent functions are easy to test because they're guaranteed to always return the same result when called with the same arguments. Testing is simply a matter of checking that the value returned by various different calls to the function return the expected value.

==> **A function is considered pure** if it's both idempotent and has no observable side effects. For example, if the idempotent version of square_func(number)above printed the result before returning it or mutated a variable outside of the function scope, it's still considered idempotent because while it accessed an I/O stream. However, it would not remain a pure function anymore.

```python
a_variable = 0


def square_num(number):
    """Idempotent but not pure."""

    sq_num = number ** 2
    a_variable += square_num

    return sq_num
```

**Why:** Pure functions are even easier to test than idempotent functions. They don't keep any footprint outside of the function scope. Also, they don't call any non-pure functions. It's basically a data-in-data-out pipeline.


### Imports

* Don't use wild card import

```python
# don't
from mod import *
from package.mod import *

# do
from mod import func_0
from package.mod import func_1
```

* Don't import unused modules (Flake8 will point out unused imports, make sure you remove them before committing).

* Sort the imports by import then from, and sort alphabetically (`isort` will automatically do this for you).

```python
# don't
from a_mod import foo
import e_mod
import b_mod
from z_mod import bar, baz

# do
import b_mod
import e_mod
from a_mod import foo
from z_mod import bar, baz
```

* Use absolute imports in your production code. Relative imports can be messy, particularly for shared projects where directory structure is likely to change. Consider this directory structure:

```bash
└── project
└── package1
    ├── module1.py
    └── module2.py
```

```python
# package1/module1

# don't
from .module2 import func

# do
from package1.module2 import func
```


### Exception Handling

* Don't write bare try-except block

```python
# don't
try:
    do_something()
except:
    pass

# don't
try:
    do_something()
except Exception:
    pass
```

In most cases, the caught exception type must be as specific as possible. Something like KeyError, or ConnectionTimeout, etc.

```python
# do
try:
    do_something()
# catch some very specific exception - KeyError, ValueError, etc.
except ValueError:
    pass
```

If some code path simply must broadly catch all exceptions - for example, the top-level loop for some long-running persistent process - then each such caught exception must write the full stack trace to a log or file, along with a timestamp. Not just the exception type and message, but the full stack trace.

```python
# do
def get_number():
    return int("foo")


try:
    x = get_number()
except ValueError:
    pass
except Exception:
    logging.exception("Caught an error", exec_info=True)
```


### Functional Paradigm

Although Python is regarded as a multi-paradigm language, it wasn't exactly [designed]((https://developers.slashdot.org/story/13/08/25/2115204/interviews-guido-van-rossum-answers-your-questions)) as a functional language and simply having `map`, `filter` and `reduce` functions doesn't make it one.

However, Python has in fact, functional capabilities. This is because functions are first class citizens here. They're expressions which can be evaluated at runtime. They can be passed as argument and you can return them from other functions. They can 'remember' things in closure.

On top of that functional programming is also a way to think. Avoiding side effects and mutations for example are things you can probably always benefit from in some way if its not too strictly and mechanically enforced. Here are a few guidelines that will keep you from abusing the functional aspects of python.

* Limit your usage of anonymized `lambda` functions to a minimum.

    **Why:** When error occurs in `lambda` expression, Python does not provide the function name in the traceback. Also, they're harder to read when the expression gets complicated.

```python
# don't
div_zero = lambda x: x / 0
div_zero(2)
```

The traceback in this case looks like this:

```python
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "<stdin>", line 1, in <lambda>
ZeroDivisionError: division by zero
```

* Don't assign a lambda expression (flake8 will complain if do so). Define a normal function if the need arises.

```python
# don't
divider = lambda some_list: list(map(lambda n: n // 2, some_list))

divider([1, 2, 3])
# >> [0, 1, 1]

# do
def divider(some_list):
    return [x // 2 for x in some_list]


divider([1, 2, 3])
# >> [0, 1, 1]
```

* Only use `map`, `filter`, `reduce` absolutely when you have to.

    **Why:** In almost all cases, they can be replaced with `list comprehension` and built in functions.

==> **Map Alternative:**

```python
# don't
customers = ["Alice", "Bob", "Frank", "Ann"]
result = map(lambda x: x[0], customers)

print(list(result))
# >> ['A', 'B', 'F', 'A']

# do
customers = ["Alice", "Bob", "Frank", "Ann"]
result = [x[0] for x in customers]

print(result)
# >> ['A', 'B', 'F', 'A']
```

==> **Filter Alternative:**

```python
# don't
customers = ["Alice", "Bob", "Frank", "Ann"]
result = filter(lambda x: x[0] == "A", customers)

print(list(result))
# >> ['Alice', 'Ann']

# do
customers = ["Alice", "Bob", "Frank", "Ann"]
result = [x for x in customers if x[0] == "A"]

print(result)
# >> ['Alice', 'Ann']
```

==> **Reduce Alternative:**

```python
# don't
from functools import reduce

print(reduce(lambda x, y: x + y, range(1, 6)))
# >> 15

# do
print(sum(range(1, 6)))
# >> 15
```



### Logging

Instantiate your logger in your package's `__init__.py` module. See how it's done in `requests` library [here.](https://github.com/kennethreitz/requests)

* Define a basic logging class

```python
# __init__.py
# demo of a global logger that can be called and used anywhere in your package

import logging

logging.getLogger(__name__)

logging.basicConfig(
    level=logging.INFO,
    format="\n(asctime)s [%(levelname)s] %(message)s",
    handlers=[logging.FileHandler("packg/debug.log"), logging.StreamHandler()],
)
```

* Import and use it like this

```python
# mod.py
# using the logging class defined in __init__.py

from . import logging

def dumb_div(a):
    try:
        res = a // 0
    except ValueError:
        res = a // 1
    except Exception:
        logging.exception("Exception Occured")
        res = None

    return res

dumb_div(5)
```

* For logging in production applications, use [Sentry's Logging](https://docs.sentry.io/platforms/python/logging/). You can view your logs in the Sentry Dashboard.

```python
# __init__.py

import logging
import sentry_sdk
from sentry_sdk.integrations.logging import LoggingIntegration

sentry_logging = LoggingIntegration(
    level=logging.INFO,  # Capture info and above as breadcrumbs
    event_level=logging.ERROR,  # Send errors as events
)
sentry_sdk.init(
    dsn="<sign-up-for-sentry-and-you-will-get-your-dsn>",
    integrations=[sentry_logging],
)
```

```python
# mod.py
# using the sentry logging class defined in __init__.py

from . import logging

def dumb_div(a):
        try:
            res = a // 0
        except ValueError:
            res = a // 1
        except Exception:
            logging.exception("Exception Occured")
            res = None

        return res

    dumb_div(5)
```

* In some cases you'll want to decouple exception handling and logger logics from your core logic. You can use Python's context manager decorator to achieve that.

```python
from . import logging
from contextlib import contextmanager


class Calculation:
    """Dummy class for demonstrating exception decoupling with contextmanager."""

    def __init__(self, a, b):
        self.a = a
        self.b = b

    @contextmanager
    def errorhandler(self):
        try:
            yield
        except ZeroDivisionError:
            print(
                f"Custom handling of Zero Division Error! Printing "
                "only 2 levels of traceback.."
            )
            logging.exception("ZeroDivisionError")

    def main_func(self):
        """Function that we want to save from nasty error handling logic."""

        with self.errorhandler():
            return self.a / self.b
```

```
# folder structure of the logging demo package
packg
├── debug.log
├── __init__.py
└── mod.py
```


### Patterns

#### Null Comparison

As PEP8 suggests, comparisons to singletons like `None` should always be done with `is` or `is not`, never the equality (`=` / `!=`) operators.

**Why:** The `==` operator compares the values of both the operands and checks for value equality. Whereas `is` operator checks whether both the operands refer to the same object or not.

```python
# don't
if val == None:
    # ...

if val != None:
    # ...
```

```python
# do
if val is None:
    # ...

if val is not None:
    # ...
```

#### Decorators

Instead of directly changing the source code, when possible, use [decorators](https://realpython.com/primer-on-python-decorators/) to change or monitor function/methods. Follow [this](https://stackoverflow.com/a/39335652/8963300) style taken from David Beazly's [Python Cookbook](https://www.oreilly.com/library/view/python-cookbook-3rd/9781449357337/) to write your decorators. This is a generalized format that has the least amount of nesting and can be used with or without parameter.

```python
from functools import partial, wraps


def decorator(func=None, foo="spam"):
    if func is None:
        return partial(decorator, foo=foo)

    @wraps(func)
    def wrapper(*args, **kwargs):
        # do something with `func` and `foo`, if you're so inclined
        pass

    return wrapper
```

This can be used with or without parameters.

```python
# applying decorator without any parameter
@decorator
def f(*args, **kwargs):
    pass


# applying decorator with extra parameter
@decorator(foo="buzz")
def f(*args, **kwargs):
    pass
```

Here is another example of a `retry` decorator that runs a callable multiple times in case it hits pre-specified exceptions.

```python
import logging
import time
from functools import partial, wraps


def retry(func=None, exception=Exception, n_tries=5, delay=5, backoff=1, logger=False):
    """Retry decorator with exponential backoff.

    Parameters
    ----------
    func : typing.Callable, optional
        Callable on which the decorator is applied, by default None
    exception : Exception or tuple of Exceptions, optional
        Exception(s) that invoke retry, by default Exception
    n_tries : int, optional
        Number of tries before giving up, by default 5
    delay : int, optional
        Initial delay between retries in seconds, by default 5
    backoff : int, optional
        Backoff multiplier e.g. value of 2 will double the delay, by default 1
    logger : bool, optional
        Option to log or print, by default False
    Returns
    -------
    typing.Callable
        Decorated callable that calls itself when exception(s) occur.
    Examples
    --------
    >>> import random
    >>> @retry(exception=Exception, n_tries=4)
    ... def test_random(text):
    ...    x = random.random()
    ...    if x < 0.5:
    ...        raise Exception("Fail")
    ...    else:
    ...        print("Success: ", text)
    >>> test_random("It works!")
    """

    if func is None:
        return partial(
            retry,
            exception=exception,
            n_tries=n_tries,
            delay=delay,
            backoff=backoff,
            logger=logger,
        )

    @wraps(func)
    def wrapper(*args, **kwargs):
        ntries, ndelay = n_tries, delay

        while ntries > 1:
            try:
                return func(*args, **kwargs)
            except exception as e:
                msg = f"{str(e)}, Retrying in {ndelay} seconds..."
                if logger:
                    logging.warning(msg)
                else:
                    print(msg)
                time.sleep(ndelay)
                ntries -= 1
                ndelay *= backoff

        return func(*args, **kwargs)

    return wrapper
```


### Testing

* Use [pytest](https://docs.pytest.org/en/latest/) to write your tests

* Strive to write many small [pure](https://stackoverflow.com/a/47245930/8963300) and [idempotent](https://stackoverflow.com/questions/1077412/what-is-an-idempotent-operation) functions, and minimize where mutations occur.

* Whenever you fix a bug, write a regression test. A bug fixed without a regression test is almost certainly going to break again in the future.

* Use direct assertions and explicit comparisons; avoid negations.

```python
# don't - other values can be falsy too: `[], 0, '', None`
assert not result
assert result_list

# do
assert result == False
assert len(result_list) > 0
```

### Flask

#### Project Structure

Try to follow **divisional structure** while designing your microservice APIs.
Here's an example of divisional structure in a Flask project.

```
yourapp/
    __init__.py
    admin/
        __init__.py
        views.py
        static/
        templates/
    home/
        __init__.py
        views.py
        static/
        templates/
    control_panel/
        __init__.py
        views.py
        static/
        templates/
    models.py
```

Read more on divisional structure [here.](https://exploreflask.com/en/latest/blueprints.html#divisional)


## Documentation Guideline

### Docstrings

* Use [numpy](https://numpydoc.readthedocs.io/en/latest/format.html#docstring-standard) style docstring. This is a good format that uses extra vertical space for maximum readability.

```python
def dumb_add(num0, num1, num2, num3, num4):
    """[summary]

    Parameters
    ----------
    num0 : [type]
        [description]
    num1 : [type]
        [description]
    num2 : [type]
        [description]
    num3 : [type]
        [description]
    num4 : [type]
        [description]

    Returns
    -------
    [type]
        [description]
    """
```

* Examples should be given in REPL style

```python
"""
Examples
--------
>>> dumb_add(1, 2, 3, 4, 5)
15

Comment explaining the second example

>>> dumb_add(6, 7, 8, 9, 10)
40
"""
```

* Single line docstring.

```python
def dumb_sub(num1, num0):
    """Subtracting num0 from num1."""
```

* Inline comments should start with lowercase letter.

```python
# going through the student list
for idx, student in enumerate(students_list):
    ...
```

* In case of documenting classes, you should mention the methods and attributes like this. The methods should be documented like the functions.

```python
class Dummy(ndarray):
    """
    Dummy classes for demonstration.

    Attributes
    ----------
    attribute_1 : float
        Set up your goal.

    Methods
    -------
    method_1(c=2)
        Runs a self-referential loop n times.
    method_2(n=1.0)
        Prints Nietzsche's n times.
    """
```


## The Holy Grail of Being Pythonic

* [Pythonic Code Review](https://access.redhat.com/blogs/766093/posts/2802001)
* [Writing Great Code](https://www.oreilly.com/library/view/the-hitchhikers-guide/9781491933213/ch04.html)
* [PEP 8 — the Style Guide for Python Code](https://pep8.org/#overriding-principle)
* [wemake-python-styleguide](https://github.com/wemake-services/wemake-python-styleguide)


## References

* [The Most Diabolical Python Antipattern - Real Python](https://realpython.com/the-most-diabolical-python-antipattern/)
* [Flask Project Structure - Explore Flask](https://exploreflask.com/en/latest/blueprints.html#divisional)
* [Python Style Guide -Kengz](https://github.com/kengz/python)
* [Django Style Guide](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/)
* [Write Better Python Functions - Jeff Knupp](https://jeffknupp.com/blog/2018/10/11/write-better-python-functions/)
* [Code Style, Hitchhiker's Guide to Python - Kenneth Reitz](https://docs.python-guide.org/writing/style/)
* [Primer on Python Decorators - Real Python](https://realpython.com/primer-on-python-decorators/)
* [Why did Guido want to remove map(), filter(), reduce(), and Lambda from Python 3?](https://www.quora.com/Why-did-Guido-want-to-remove-map-filter-reduce-and-Lambda-from-Python-3)
* [Fate of reduce in python 3000](https://blog.finxter.com/about-guidos-fate-of-reduce-in-python-3000/)
* [Difference between == and is operator in Python](https://www.geeksforgeeks.org/difference-operator-python/)
* [Null in Python: Understanding Python's NoneType Object](https://realpython.com/null-in-python/)
