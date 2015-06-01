# Effective Python


--------

## 1. Pythonic Thinking


### Item 2: Follow the PEP8 Style Guide

* Use inline negation(`if a is not b`) instead of negation of positive 
  expressions (`if not a is b`);
* Do not check for empty values(like [] or '') by checking the length(if 
  len(somelist) == 0). Use `if not somelist` and assume empty values implicitly 
  evaluate on False;


### Item 3: Know the Differences Between `bytes`, `str`, and `unicode`

In Python 3, there are two types that represent sequences of characters: bytes 
and str. Instances of bytes contain raw 8-bit values. Instances of str contain 
Unicode characters.

In Python 2, there are two types that represent sequences of characters: str and 
unicode. In contrast to Python 3, iinstances of str contain raw 8-bit values. 
Instances of unicode contain Unicode characters.

In Python 2, unicode and str instances seem to be the same type when a str only 
contains 7-bit ASCII characters.
	* You can combine such a str and unicode together using the + operator.
	* You can compare such str and unicode instances using equality and 
	  inequality operators.
	* You can use unicode instances for format strings like "%s".
But in Python 3, bytes and str instances are never equivalent--not even the 
empty string--so you must be more deliberate about the types of character.

In Python 3, operations invalving file handles (returned by the `open` built-in 
function) default to UTF-8 encoding. In Python 2, file operations default to 
binary encoding. So if you want to read or write binary data to/from a file, 
always open the file using a binary mode(like "rb" or "wb").


### Item 4: Write Helper Functions Instead of Complex Expressions

Defaut value:
```python
red = my_values.get("red", [""])[0] or 0
```
The *opacity* case works because the value in the my_values dictionary is 
missing altogether. The behavior of the `get` method is to return its second 
argument if the key does not exist in the dictionary.

No:
```python
red = int(my_values.get('red', [''])[0] or 0)
```
Better: (For less complicated situations)
```python
red = my_values.get('red', [''])
red = int(red[0]) if red[0] else 0  # if/else conditional, Added in Python 2.5
```
Best: (Especially if you need to use this logic repeatedly)
```python
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        found = int(found[0])
    else:
        found = default
    return found
```

### Item 5: Know How to Slice Sequences

Slicing can be extended to any Python class that implements the `__getitem__` 
and `__setitem__` special methods.

If you leave out both the start and the end indexes when slicing, you will end 
up with a copy of the original list.
```python
b = a[:]
assert b == a and b is not a
```
If you assign a slice with no start or end indexes, you will replace its entire 
contents with a copy of what is referenced(instead of allocating a new list).


### Item 6: Avoid Using `start`, `end`, and `stride` in a Single Slice

Avoid using start, end and stride together in a single slice. If you need all 
three parameters, consider doing two assignments(one to slice, another to 
stride) or using *islice* from the *itertools* built-in module.


### Item 7: Use List Comprehensions Instead of map and filter

```python
even_squares = [x**2 for x in a if x % 2 == 0]
```

Dictionaries and sets have their own equivalents of list comprehensions. These 
make it easy to create derivative data structures when writing algorithms.
```python
chile_ranks = {'ghost': 1, 'habanero': 2, 'cayenne': 3}
rank_dict = {rank: name for name, rank in chile_ranks.items()}
chile_len_set = {len(name) for name in rank_dict.values()}
```

List comprehensions allow you to easily skip items from the input list, a 
behavior map does not support without help from filter.


### Item 8: Avoid More Than Two Expressions in List Comprehensions

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)
>>> [1, 2, 3, 4, 5, 6, 7, 8, 9]
squared = [[x**2 for x in row] for row in matrix]
```

List comprehensions also support multiple if conditions. Multiple conditions at 
the same loop level are an implicit and expression.
```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x % 2 == 0] # Equivalent
```


### Item 9: Consider Generator Expressions for Large Comprehensions

The problem with list comprehensions is that they may create a whole new list 
containing one item for each value in the input sequences. This is fine for 
small inputs, but for file which is absolutely enormous or perhaps a never-
ending network socket, list comprehensions are problematic.
```python
value = [len(x) for x in open('/tmp/my_file.txt')]
```

To solve this, Python provides generator expressions, it do not materialize the 
whole output sequence when they are run. Instead, generator expressions evaluate 
to an iterator that yields one item at a time from the expression.

A generator expression is created by putting list-comprehension-like syntax 
between `()` characters:
```python
it = (len(x) for x in open('/tmp/my_file.txt'))
print(next(it))
print(next(it))
```

Another powerful outcome of generator expressions is that they can be composed 
together.
```python
roots = ((x, x**0.5) for x in it)
```

Chaining generators like this executes very quickly in Python. When you are 
looking for a way to compose functionality that is operating on a large stream 
of input, generator expressions are the best tool for the job.

The obly gotcha is that the iterators returned by generator expressions are 
stateful, so you must be careful not to use them more than once.


### Item 10: Prefer `enumerate` Over `range`

No:
```python
for i in range(len(flavor_list)):
    flavor = flavor_list[i]
```
Yes:
```python
for i, flavor in enumerate(flavor_list):
    pass
```


### Item 11: Use `zip` to Process Iterators in Parallel

No:
```python
for i in range(len(names)):
    count = letters[i]
    if count > max_letters:
        longest_name = names[i]
        max_letters = count
```
Better:
```python
for i, name in enumerate(names):
    count = letters[i]
    if count > max_letters:
        longest_name = name
        max_letters = count
```
Best:
```python
for name, count in zip(names, letters):
    if count > max_letters:
        longest_name = name
        max_letters = count
```

There are two problems with the `zip` built-in. The first issue is that in 
Python 2 `zip` is not a generator, it will exhaust the supplied iterators and 
return a list of all the tuples it creates. If you want to `zip` very large 
iterators in Python 2, you should use `izip` from the `itertools` built-in 
module. The second issue is that `zip` behaves strangely if the input iterators 
are of different lengths.


### Item 12: Avoid `else` Blocks After `for` and `while` Loops

* The `else` block after a loop only runs if the loop body did not encounter a 
  `break` statement.
* Avoid using `else` blocks after loops because their behavior is not intuitive 
  and can be confusing.


### Item 13: Take Advantage of Each Block in `try/expect/else/finally`

* The `try/finally` compound statement lets you run cleanup code regardless of 
  whether exceptions were raised in the `try` block.
* The `else` block helps you minimize the amount of code in `try` blocks and 
  visually distinguish the success case from the `try/except` blocks.


---------------

## 2. Functions


### Item 14: Prefer Exceptions to Returning None

* Functions that return None to indicate special meaning are error prone because 
  None and other values(e.g., zero, the empty string) all evaluate to False in 
  conditional expressions.
* Raise exceptions to indicate special situations instead of returning None. 
  Expect the calling code to handle exceptions properly when they are documented


### Item 15: Know How Closures Interact with Variable Scope

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)
numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)
>>> [2, 3, 5, 7, 1, 4, 6, 8]
```
Python has specific rules for comparing tuples. It first compares items in index 
zero, then index one, then index two, and so on. This is why the return value 
from the `helper` closure causes the sort order to have two distinct groups.

Assigning a value to a variable works differently with reference a variable. If 
the variable is already defined in the current scope, then it will just take on 
the new value. If the variable does not exist in the current scope, then Python 
treats the assignment as a variable definition. The scope of the newly defined 
variable is the function that contains the assignment.

In Python 3, there is special syntax for getting data out of a closure. the `
nonlocal` statement is used to indicate that scope traversal should happen upon
assignment for a specific variable name. The only limit is that `nonlocal` will 
not traverse up to the module-level scope(to avoid polluting globals).
```python
def sort_priority3(numbers, group):
    found = False
    def helper(x):
        nonlocal found
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found
```

When your usage of `nonlocal` starts getting complicated, it is better to wrap 
your state in a helper class.
```python
class Sorter(object):
    def __init__(self, group):
        self.group = group
        self.found = False
    def __call__(self, x):
        if x in self.group:
            self.found = True
            return (0, x)
        return (1, x)
sorter = Sorter(group)
numbers.sort(key=sorter)
assert sorter.found is True
```

Unfortunately, Python 2 does not support the `nonlocal` keyword. You need to 
use a work-around that takes advantage of Pythons scoping rules.
```python
def sort_priority(numbers, group):
    found = [False]
    def helper(x):
        if x in group:
            found[0] = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found[0]
```


### Item 16: Consider Generators Instead of Returning Lists

No:
```python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result
```
Yes:
```python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1
```

The iterator returned by the generator call can easily be converted to a list 
by passing it to the `list` built-in function
```python
result = list(index_words_iter(address))
```

The only gotcha of defining generators like this is that the callers must be 
aware that the iterators returned are stateful and can not be reused.


### Item 17: Be Defensive When Iterating Over Arguments

When a function takes a list of objects as a parameter, it is often important 
to iterate over that list multiple times.
```python
def normalize(numbers):
    total = sum(numbers)    # first iterate the iterator
    result = []
    for value in numbers:   # second iterate the iterator
        percent = 100 * value / total
        result.append(percent)
    return result

def read_visits(data_path):
    with open(data_path) as f:
        for line i f:
            yield int(value)

it = read_visits('/tmp/my_numbers.txt')
percentages = normalize(it)
print(percentages)
>>> []      # Surprisingly!!
```
The cause of this behavior is that an iterator obly produces its results a 
single time. If you iterate over an iterator or generator that has already 
raised a `StopIteration` exception, you will not get any results the second 
time around.

To solve this problem, you can explicitly exhaust an input iterator and keep a 
copy of its entire contents in a list.
```python
def normalize_copy(numbers):
    numbers = list(numbers)
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

The problem with this approach is the copy of the input iterators contents 
counld be large. One way around this is to accept a function that returns a new 
iterator each time it is called.
```python
def normalize_func(get_iter):
    total = sum(get_iter())
    result = []
    for value in get_iter():
        percent = 100 * value / total
        result.append(percent)
    return result

percentages = normalize_func(lambda: read_visits(path))
```

The better way to achieve the same result is to provide a new container class 
that implements the *iterator protocol*. The iterator protocol is how Python 
`for` loops and related expressions traverse the contents of a container type. 
When Python sees a statement like `for x in foo` it will actually call `iter(
foo)`.  The `iter` built-in function calls the `foo.__iter__` special method in 
turn. The `__iter__` method must return an iterator object(which itself 
implements the `__next__` special method). Then the `for` loop repeatedly calls 
the `next` built-in function on the iterator object until it is exhausted(and 
raise a `StopIteration` exception).
```python
class ReadVisits(object):
    def __init__(self, data_path):
        self.data_path = data_path
    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)
visits = ReadVisits(path)
percentages = normalize(visits)
print(percentages)
>>> [11.53, 26.92, 61.53]
```
This works because the `sum` method in `normalize` will call 
`ReadVisits.__iter__` to allocate a new iterator object. The `for` loop to 
normalize the numbers will also call `__iter__` to allocate a seconde iterator 
object. Each of those iterators will be advanced exhausted independently.

You can detect that a value is an iterator(instead of a container) if calling 
`iter` on it twice produces the same result, which can then be progressed with 
the `next` built-in function.


### Item 18: Reduce Visual Noise with Variable Positional Arguments

Functions that accept `*args` are best for situations where you know the number 
of inputs in the argument list will be reasonably small.

You can use the items from a sequence as the positional arguments for a function 
with the `*` operator.
```python
def log(message, *values):
    pass

favorites = [7, 39, 99]
log('Favorite colors', *favorites)      # * operator
```

Using the `*` operator with a generator may cause your program to run out of 
memory and crash.


### Item 19: Provide Optional Behavior with Keyword Arguments

* Keyword arguments with default values make it easy to add new behaviors to a 
function, especially when the function has existing callers.


### Item 20: Use `None` and Docstrings to Specify Dynamic Default Arguments

```python
def log(message, when=datetime.now()):
    print('%s: %s' % (when, message))
log('Hi there!')
sleep(1)
log('Hi again')
>>>2014-11-15   21:10:10.371432:    Hi  there!
2014-11-15  21:10:10.371432:    Hi  again!
```
The timestamps are the same because `datetime.now()` is only executed a single 
time: when the function is defined. Default argument values are evaluated only 
once per module load, which usually happens when a program starts up.

```python
def log(message, when=None):
    '''Log a message with a timestamp.
    Args:
        message: Message to print.
        when: datetime of when the message occurred.
            Defaults to the present time.
    '''
    when = datetime.now() if when is None else when
    print('%s: %s' % (when, message))
```


### Item 21: Enforce Clarity with Keyword-Only Arguments

In Python 3, you can demand clarity by defining your functions with keyword-only 
arguments.
```python
def safe_division_c(number, divisor, *, 
                    ignore_overflow=False,
                    ignore_zero_division=False):
    pass
```
Now, calling the function with positional arguments for the keyword arguments 
will not work.

Unfortunately, Python 2 does not have explicit syntax for specifying keyword-
only arguments like Python 3. But you can achieve the same behavior of raising 
`TypeError` for invalid function calls by using the `**` operator in argument 
lists.
```python
def safe_division_d(number, divisor, **kwargs):
    ignore_overflow = kwargs.pop('ignore_overflow', False)
    ignore_zero_division = kwargs.pop('ignore_zero_division', False)
    if kwargs:
        raise TypeError('Unexpected **kwargs: %r' % kwargs)
```


---------

## 3. Classes and Inheritance


### Item 22: Prefer Helper Classes Over Bookkeeping with Dictionaries and Tuples

Pythons built-in dictionary and tuple types made it easy to keep going, adding 
layer to the internal bookkeeping. But you should avoid doing this for more 
than one level of nesting(i.e., avoid dictionaries that contain dictionaries).
It makes your code hard to read by other programmers and sets you up for a 
maintenance nightmare.

As soon as you realize the bookkeeping is getting complicated, break it all out 
into classes.

As soon as you find yourself going longer than a two-tuple, it is time to 
consider another approach.


### Item 23: Accept Functions for Simple Interfaces Instead of Classes

```python
class CountMissing(object):
    def __init__(self):
        self.added = 0
    def missing(self):
        self.added += 1
        return 0
counter = CountMissing()
result = defaultdict(counter.missing, current)
```
Much better:
```python
class BetterCountMissing(object):
    def __init__(self):
        self.added = 0
    def __call__(self):
        self.added += 1
        return 0
counter = BetterCountMissing()
assert callable(counter)
result = defaultdict(counter, current)  # Relies on __call__
```

When you need a function to maintain state, consider defining a class that 
provides the `__call__` method instead of defining a stateful closure.


### Item 24: Use `@classmethod` Polymorphism to Construct Objects Generically

In Python, not only do the objects support polymorphism, but the classes do as 
well.

```python
class InputData(object):
    def read(self):
        raise NotImplementedError

class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path
    def read(self):
        return open(self.path).read()

class Worker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None
    def map(self):
        raise NotImplementedError
    def reduce(self, other):
        raise NotImplementedError

class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')
    def reduce(self, other):
        self.result += other.result

def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))

def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers

def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads:
        thread.start()
    for thread in threads:
        thread.join()
    first, rest = workers[0], workers[1:]
    for worker in rest:
        first.reduce(worker)
    return first.result

def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)

# connect all of the pieces together in a function to run each step.
with TemporaryDirectory() as tmpdir:
    write_test_files(tmpdir)
    result = mapreduce(tmpdir)
```
The huge issue is the `mapreduce` function is not generic at all. The best way 
to solve this problem is with `@classmethod` polymorphism. This is exactly like 
the instance method polymorphism I used for `InputData.read`, except that it 
applies to whole classes instead of their constructed objects.
```python
class GenericInputData(object):
    def read(self): 
        raise NotImplementedError
    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(GenericInputData):
    # ...
    def read(self):
        return open(self.path).read()
    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir)
            yield cls(os.path.join(data_dir, name))

class GenericWorker(object):
    # ...
    def map(self):
        raise NotImplementedError
    def reduce(self, other):
        raise NotImplementedError
    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers

class LineCountWorker(GenericWorker):
    # ... as above.

def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)

with TemporaryDirectory() as tmpdir:
    write_test_files(tmpdir)
    confg = {'data_dir': tmpdir}
    result = mapreduce(LineCountWorker, PathInputData, config)
```
Now you can write other `GenericInputData` and `GenericWorker` classes as you 
wish and not have to rewrite any of the glue code.


### Item 25: Initialize Parent Classes with `super`

The old way to initialize a parent class from a child class is to directly call 
the parent classs `__init__` method with the child instance.
```python
class MyBaseClass(object):
    def __init__(self, value):
        self.value = value

class MyChildClass(MyBaseClass):
    def __init__(self):
        MyBaseClass.__init__(self, 5)
```
This approach works fine for simple hierarchies but breaks down in many cases. 
If your class is affected by multiple inheritance, calling the superclasses `
__init__` methods directly can lead to unpredictable behavior.

```python
class TimesFive(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value *= 5

class PlusTwo(MyBaseClass):
    def __init__(self, value):
        MyBaseClass.__init__(self, value)
        self.value += 2

class ThisWay(TimesFive, PlusTwo):
    def __init__(self, value):
        TimesFive.__init__(self, value)
        PlusTwo.__init__(self, value)   # reset back the 'value' to 5

foo = ThisWay(5)
print('(5 * 5) + 2 = 27 but is ', foo.value)
>>> (5 * 5) + 2 = 27 but is 7
```

To solve these problems, Python 2.2 added `super` built-in function and defined 
the *method resolution order*(MRO).
```python
class TimesFiveCorrect(MyBaseClass):
    def __init__(self, value):
        super(TimesFiveCorrect, self).__init__(value)
        self.value *= 5

class PlusTwoCorrect(MyBaseClass):
    def __init__(self, value):
        super(PlusTwoCorrect, self).__init__(value)
        self.value += 2

class GoodWay(TimesFiveCorrect, PlusTwoCorrect):
    def __init__(self, value):
        super(GoodWay, self).__init__(value)

foo = GoodWay(5)
print('5 * (5 + 2) = 35 and is ', foo.value)
>>> 5 * (5 + 2) = 35 and is 35  # not (5 * 5) + 2 = 27!!!
from pprint import pprint
pprint(GoodWay.mro())
>>> [<class ‘__main__.GoodWay’>,
<class  ‘__main__.TimesFiveCorrect’>,
<class  ‘__main__.PlusTwoCorrect’>,
<class  ‘__main__.MyBaseClass’>,
<class  ‘object’>]
```
When I call `GoodWay(5)`, it in turn calls `TimesFiveCorrect.__init__`, which 
calls `PlusTwoCorrect.__init__`, which calls `MyBaseClass.__init__`. Once this 
reaches the top of the diamond, then all of the initialization methods actually 
do  their work in the opposite order from how their `__init__` functions were 
called.

But the syntax is a bit verbose. Python 3 fixes these issues:
```python
class Explicit(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value * 2)

class Implicit(MyBaseClass):
    def __init__(self, value):
        super().__init__(value * 2)

assert Explicit(10).value == Implicit(10).value
```


### Item 26: Use Multiple Inheritance Only for Mix-in Utility Classes

As other programming language, it is better to avoid multiple inheritance alto-
gether. But if you find yourself desiring the convenience and encapsulation 
that comes with multiple inheritance, consider writing a *mix-in* instead. A 
mix-in is a small class that only defines a set of additional methods that a 
class should provide. Mix-in classes do not define their own instance attribu-
tes nor require their `__init__` constructor to be called.


### Item 27: Prefer Public Attribute Over Private Ones

In Python, there are only two types of attribute visibility for a classs 
attribute: *public* and *private*.

As you would expect with private fields, a subclass can not access its parent 
classs private fields.

The only time to seriously consider using private attributes is when you are 
worried about naming conflicts with subclasses. This is primarily a concern 
with classes that are part of a public API; the subclasses are out of your 
control, so you can not refactor to fix the problem. To reduce the risk of this 
happening, you can use a private attribute in the parent class to ensure that 
there are no attribute names that overlap with child classes.


### Item 28: Inherit from `collections.abc` for Custom Container Types

Python implements its container behaviors with instance methods that have 
special names. When you access a sequence item by index:
```python
bar = [1, 2, 3]
bar[0]
```
it will be interpreted as:
```python
bar.__getitem__(0)
```

```python
class FrequencyList(list):  # a container type that subclass from built-in list
    def __init__(self, members):
        super().__init__(members)
    def frequency(self):
        counts = {}
        for item in self:
            counts.setdefault(item, 0)
            counts[item] += 1
        return counts
foo = FrequencyList(['a', 'b', 'a', 'c', 'b', 'a', 'd'])
print('Length is ', len(foo))
foo.pop()
print('After pop: ', repr(foo))
print('Frequency: ', foo.frequency())
>>> 
Length  is  7
After   pop:    [‘a’,   ‘b’,    ‘a’,    ‘c’,    ‘b’,    ‘a’]
Frequency:  {‘a’:   3,  ‘c’:    1,  ‘b’:    2}
```
But if you want to provide an object that feels like a `list`, but it is not a 
`list` subclass, you need to implements `__getitem__`, `__len__` and so on.

Defining your own container types is much harder than it looks. To avoid this
differently throughout the Python universe, the built-in `collections.abc` 
module defines a set of abstract base classes that provide all of the typical 
method for each container type.


-------------

## 4. Metaclasses and Attributes


### item 29: use Plain Attributes Instead of Get and Set Methods

In Python, however, you almost never need to implement explicit setter or 
getter methods. Instead, you should always start your implementations with 
simple public attributes. These make operations like incrementing in place 
natural and clear.

If you decide you need special behavior when an attribute is set, you can migr-
ate to the `@property` decorator and its coresponding `setter` attribute.
```python
class Resistor(object):
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 0
        self.current = 0

class VoltageRessitance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0
    @property
    def voltage(self):
        return self._voltage
    @voltage.setter
    def voltage(self, voltage):
        self._voltage = voltage
        self.current = self._voltage / self.ohms

r2 = VoltageResistance(1e3)
print(‘Before: %5r amps’ % r2.current)
r2.voltage = 10
print(‘After: %5r amps’ % r2.current)
>>>
Before:     0 amps
After:   0.01 amps
```

Specifying a `setter` on a property also lets you perform type checking and 
validation on values passed to your class.
```python
class BoundedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
    @property
    def ohms(self):
        return self._ohms
    @ohms.setter
    def ohms(self, ohms):
        if ohms <= 0:
            raise ValueError('%f ohms must be > 0' % ohms)
        self._ohms = ohms

r3 = BoundedResistance(1e3)
r3.ohms = 0
>>> ValueError: 0.000000 ohms must be > 0
BoundedResistance(-5)
>>> ValueError: -5.000000 ohms must be > 0
```

You can even use `@property` to make attributes from parent classes immutable.
```python
class FixedResistance(Resistor):
    # ...
    @property
    def ohms(self):
        return self._ohms
    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError('Can not set attribute')
        self._ohms = ohms

r4 = FixedResistance(1e3)
r4.ohms = 2e3
>>> AttributeError: Can’t set attribute
```


### Item 30: Consider `@property` Instead of Refactoring Attributes

The built-in `@property` decorator makes it easy for simple accesses of an 
instances attributes to act smarter. One advanced but common use of `@property` 
is transitioning what was once a simple numerical attribute into an on-the-fly 
calculation.

```python
class Bucket(object):
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quota = 0
        self.quota_consumed = 0
    def __repr__(self):
        return ('Bucket(max_quota=%d, quota_consumed=%d)' % 
                (self.max_quota, self.quota_consumed))
    @property
    def quota(self):
        return self.max_quota - self.quota_consumed
    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        if amount == 0:
            self.quota_consumed = 0
            self.max_quota = 0
        elif delta < 0:
            assert self.quota_consumed == 0
            self.max_quota = amount
        else:
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta
```


### Item 31: use Descriptors for Reusable `@property` Methods

The big problem with the `@property` built-in is the methods ti decorates can 
not be reused for multiple attributes of the same class. They also can not be 
reused by unrelated classes.
```python
class Exam(object):
    def __init__(self):
        self._writing_grade = 0
        self._math_grade = 0
    @staticmethod
    def _check_grade(value):
        if not (0 <= value <= 100):
            raise ValueError(‘Grade must be between 0 and 100’)
    @property
    def writing_grade(self):
        return self._writing_grade
    @writing_grade.setter
    def writing_grade(self, value):
        self._check_grade(value)
        self._writing_grade = value
    @property
    def math_grade(self):
        return self._math_grade
    @math_grade.setter
    def math_grade(self, value):
        self._check_grade(value)
        self._math_grade = value
```
Also, this approach is not general. It is gets tedious.

The better way to do this in Python is to use a *descriptor*. the descriptor 
protocol defines how attribute access is interpreted by the language. A descr-
iptor class can provide `__get__` and `__set__` methods that let you reuse the 
grade validation behavior without any boilerplate.
```python
class Grade(object):
    def __init__(self):
        self._value = WeakKeyDictionary()
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._value.get(instance, 0)
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError('Grade must be between 0 and 100')
        self._value[instance] = value

class Exam(object):
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

exam = Exam()
exam.writing_grade = 40 #==> "Exam.__dict__['writing_grade'].__set__(exam, 40)"
print(exam.writing_grade) #==> "Exam.__dict__['writing_grade'.__get__(exam, Exam)]"
```

* Reuse the behavior and validation of `@property` methods by defining your own 
  descriptor classes.
* Use `WeakKeyDictionary` to ensure that your descriptor classes do not cause 
  memory leaks.


### Item 32: Use `__getattr__`, `__getattribute__`, and `__setattr__` for Lazy Attributes

If your class defines `__getattr__`, that method is called every time an attri-
bute can not be found in an objects instance dictionary.
```python
class LazyDB(object):
    def __init__(self):
        sef.exists = 5
    def __getattr__(self, name):
        value = 'Value for %s' % name
        setattr(self, name, value)
        return value
data = LazyDB()
print('Before: ', data.__dict__)
print('foo:    ', data.foo)
print('After: ', data.__dict__)
>>>
Before: {'exists': 5}
foo:    Value for foo
After:  {'exists': 5, 'foo': 'Value for foo'}
```

Python has another language hook called `__getattribute__`. This special method 
is called every time an attribute is accessed on an object, even in cases where 
it **does** exist in the attribute dictionary. This enables you to do things 
like check global transaction state on every property access.
```python
class ValidatingDB(object):
    def __init__(self):
        self.exists = 5
    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        try:
            return super().__getattribute__(name)
        except AttributeError:
            value = 'Value for %s' % name
            setattr(self, name, value)
            return value

data = ValidatingDB()
print('exists: ', data.exists)
print('foo:    ', data.foo)
print('foo:    ', data.foo)
>>>
Called __getattribute__(exists)
exists: 5
Called __getattribute__(foo)
foo:    Value for foo
Called __getattribute__(foo)
foo:    Value for foo
```

Python code implementing generic functionality often relies on the `hasattr` 
built-in function to determine when properties exist, and the `getattr` built-
in function to retrieve property values. These functions also look in the 
instance dictionary for an attribute name before calling `__getattr__`.

Unlike retrieving an attribute with `__getattr__` and `__getattribute__`, there 
is no need for two separate methods. The `__setattr__` method is always called 
every time an attribute is assigned on an instance(either directly or through 
the `setattr` built-in function).

The problem with `__getattribute__` and `__setattr__` is that they are called on 
every attribute access for an object, even when you may not want that to happen.
```python
class BrokenDictionaryDB(object):
    def __init__(self, data):
        self._data = {}
    def __getattribute__(self, name):
        print('Called __getattribute__(%s)' % name)
        return self._data[name]
data = BrokenDictionaryDB({'foo': 3})
data.foo
>>>
Called __getattribute__(foo)
Called __getattribute__(_data)
Called __getattribute__(_data)
...
Traceback ...
RuntimeError: maximum recursion depth exceeded
```
The problem is that `__getattribute__` accesses `self._data`, which causes `
__getattribute__` to run again, which accesses `self._data` again, and so on.
The solution is to use the `super().__getattribute__` method on your instance 
to fetch values from the instance attribute dictionary.
```python
class DictionaryDB(object):
    def __init__(self, data):
        self._data = data
    def __getattribute__(self, name):
        data_dict = super().__getattribute__('_data')
        return data_dict[name]
```
Similarly, you will need `__setattr__` methods that modify attributes on an 
object to use `super().__setattr__`.


### Item 33: Validate Subclasses with Metaclasses

One of the simplest applications of metaclasses is verifying that a class was 
defined correctly. When you are building a complex class hierarchy, you may 
want to enforce sytle, require overriding methods, or and so on. Using 
metaclasses for validation can raise errors much earlier. A metaclass is 
defined by inheriting from `type`.
```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        print((meta, name, bases, class_dict))
        return type.__new__(meta, name, bases, class_dict)

class MyClass(object, metaclass=Meta):
    stuff = 123
    def foo(self):
        pass

>>>
(<class ‘__main__.Meta’>,
 ‘MyClass’,
 (<class ‘object’>,),
 {‘__module__’: ‘__main__’,
  ‘__qualname__’: ‘MyClass’,
  ‘foo’: <function MyClass.foo at 0x102c7dd08>,
  ‘stuff’: 123})
```
The metaclass has access to the name of the class, the parent classes it 
inherits form, and all of the class attributes that were defined in the classs 
body.

Python 2 has slightly different syntax and specifies a metaclass using the `
__metaclass__` class attribute.
```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        # ...

class MyClassInPython2(object):
    __metaclass__ = Meta
    # ...
```

You can add functionality to the `Meta.__new__` method in order to validdate 
all of the parameters of a class before it is defined.

The `__new__` method of metaclasses is run after the `class` statements entire 
body has been processed.


### Item 34: Register Class Existence with Metaclasses

Another common use of metaclasses is to automatically register types in your 
program.

```python
class BetterSerializable(object)
    def __init__(self, \*args):
        sefl.args = args
    def serialize(self):
        return json.dumps({'class': self.__class__.__name__,
                           'args': self.args,})
    def __repr__(self):
        # ...
registry = {}
def register_class(target_class):
    registry[target_class.__name__] = target_class
def deserialize(data):
    params = json.loads(data)
    name = params['class']
    target_class = registry(name)
    return target_class(\*params['args'])

class EvenBetterPoint2D(BetterSerializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y
register_class(EvenBetterPoint2D)

point = EvenBetterPoint2D(5, 3)
print('Before: ', point)
data = point.serialize()
print('Serialized: ', data)
after = deserialize(data)
print('After: ', after)
```
Now, I can deserialize an arbitrary JSON string without having to know which 
class it contains. The problem with this approach is that you can forget to 
call `register_class`.

Metaclasses enable this by intercepting the `class` statement when subclasses 
are defined.
```python
class Meta(type):
    def __new__(meta, name, bases, class_dir):
        cls = type.__new__(meta, name, bases, class_dict)
        register_class(cls)
        return cls
class RegisteredSerializable(BetterSerializable, metaclass=Meta):
    pass
class Vector3D(RegisteredSerializable):
    def __init__(self, x, y, z):
        super().__init__(x, y, z)
        self.x, self.y, self.z = x, y, z
```
Using metaclasses for class regisration ensures that you will never miss a 
class as long as the inheritance tree is right.


### Item 35: Annotate Class Attributes with Metaclasses

```python
class Field(object):
    def __init__(self, name):
        self.name = name
        self.internal_name = '_' + self.name
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, '')
    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
class Customer(object):
    first_name = Field('first_name')
    last_name = Field('last_name')
    prefix = Field('prefix')
    sufix = Field('sufix')

foo = Customer()
print('Before: ', repr(foo.first_name), foo.__dict__)
foo.first_name = 'Euclid'
print('After: ', repr(foo.first_name), foo.__dict__)
>>>
Before: '' {}
After: 'Euclid' {‘_first_name’: ‘Euclid’}
```

To eliminate the redundancy, use the metaclass.
```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        for key, value in class_dict.items():
            if isinstance(value, Field):
                value.name = key
                value.internal_name = '_' + key
        cls = type.__new__(meta, name, bases, class_dict)
        return cls
class DatabaseRow(object, metaclass=Meta)
    pass
class Field(object):
    def __init__(self):
        # These will be assigned by the metaclass.
        self.name = None
        self.internal_name = None
    # ...
class BetterCustomer(DatabaseRow):
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()
foo = BetterCustomer()
print('Before: ', repr(foo.first_name), foo.__dict__)
foo.first_name = 'Euler'
print('After: ', repr(foo.first_name), foo.__dict__)
>>>
Before: '' {}
After: 'Euler' {'_first_name': 'Euler'}
```


-----------

## 5. Concurrency and Parallelism


### Item 36: Use `subprocess` to Manage Child Processes

Child processes started by Python are able to run in parallel, enabling you to 
use Python to consume all of the CPU cores of your machine and maximize the 
throughput of your programs.
```python
proc = subprocess.Popen(['echo', 'hello from the child!'], 
                        stdout=subprocess.PIPE)
out, err = proc.communicate()
print(out.decode('utf-8'))
>>> hello from the child!
```

Child processes will run independently from their parent process, the Python 
interpreter. Their status can be polled periodically while Python does other 
work.
```python
proc = subprocess.Popen(['sleep', '0.3'])
while proc.poll() is None:
	print('working...')
print('exit status: ', proc.poll())
>>> 
working...
working...
exit status 0
```

Decoupling the child process from the parent means that the parent process is 
free to run many child processes in parallel.
```python
def run_sleep(period):
    proc = subprocess.Popen(['sleep', str(period)])
    return proc
procs = []
for _ in range(10):
    proc = run_sleep(0.1)
    procs.append(proc)
for proc in procs:
    proc.communicate()  # Get the result of the command and wait for the end.
```

You can also pipe data from your Python program into a subprocess and retrieve 
its output. This allows you to utilize other programs to do work in parallel.
```python
def run_openssl(data):
    env = os.environ.copy()
    env['password'] = b'\xe23u\n\xdoq13s\x11'
    proc = subprocess.Popen(['openssl', 'enc', '-des3', '-pass', 'env:password'], 
                            env = env, stdin=subprocess.PIPE, 
                            stdout=subprocess.PIPE)
    proc.stdin.write(data)
    proc.stdin.flush()  # Ensure the child gets input
    return proc
def run_md5(input_stdin):
    proc = subprocess.Popen(['md5'], stdin=input_stdin, stdout=subprocess.PIPE)
    return proc
input_procs = []
hash_procs = []
for _in range(3):
    data = os.urandom(10)
    proc = run_openssl(data)
    input_procs.append(proc)
    hash_proc = run_md5(proc.stdout)
    hash_procs.append(hash_proc)
for proc in input_procs:
    proc.communicate()
for proc in hash_procs:
    out, err = proc.communicate()
    print(out.strip())
```

Use the `timeout` parameter with `communicate` to avoid deadlocks and hanging 
child processes.


### Item 37: Use Threads for Blocking I/O, Avoid for Parallelism

Pythons bytecode interpreter has state that must be maintained and coherent 
wihle the Python program executes. Python enforces coherence with a mechanism 
called the *global interpreter lock*(GIL).

Although Python supports multiple threads of execution, the GIL causes only one 
of them to make forward progress at a time. This means that when you reach for 
threads to do parallel computation and speed up your Python programs, you will 
be sorely disappointed.

There are ways to get CPython to utilize multiple cores, but it does not work 
with the standard `Thread` class.

The GIL prevents Python code from running in parallel, but it has no negative 
effect on system calls. This works because Python threads release the GIL just 
before they make system calls and reqcquire the GIL as soon as the system call 
are done.

So, why Python still support threads at all? Two good reasons:
1. Muiltple threads make it easy for your program to seem like it is doing 
   multiple things at the same time. Managing the juggling act of simultaneous 
   task si difficult to implement yourself. CPython ensures a level of fairness 
   between Python threads of execution, even though only one of them makes 
   forward progress at a time due to the GIL.
2. Python supports threads is to deal with blocking I/O, which happens when 
   Python does certain types of system calls.

Things to Remember:
* Python threads can not run bytecode in parallel on multiple CPU cores because 
  of the GIL.
* Python threads are still useful despite the GIL because they provide an easy 
  way to do multiple things at seemingly the same time.
* Use Python threads to make multiple system calls in parallel. This allows you 
  to do blocking I/O at the same time as computation.


--------

## 7. Collaboration


### Item 51: Define a Root EXception to Insulate Callers from APIs

* Defining root exceptions for your modules allows API consumers to insulate 
  themselves from you API.
* Catching root exceptions can help you find bugs in code that consumes an API.
* Catching the Python `Exception` base class can help you find bugs in API 
  implementations.
* Intermediate root exceptions let you add more specific types of exceptions in 
  the future without breaking your API consumers.
