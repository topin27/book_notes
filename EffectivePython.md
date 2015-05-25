# Effective Python

-----

## Pythonic Thinking

### Item 2: Follow the PEP8 Style Guide

* Use inline negation(`if a is not b`) instead of negation of positive 
  expressions (`if not a is b`);
* Do not check for empty values(like [] or '') by checking the length(if 
  len(somelist) == 0). Use `if not somelist` and assume empty values implicitly 
  evaluate on False;


### Item 3: Know the Differences Between bytes, str, and unicode

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


### Item 6: Avoid Using start, end, and stride in a Single Slice

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


### Item 22: Prefer Helper Classes Over Bookkeeping with Dictionaries and 
             Tuples

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
