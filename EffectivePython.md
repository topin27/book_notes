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
* You can compare such str and unicode instances using equality and inequality 
  operators.
* You can use unicode instances for format strings like "%s".

But in Python 3, bytes and str instances are never equivalent--not even the 
empty string--so you must be more deliberate about the types of character.

In Python 3, operations invalving file handles (returned by the `open` built-in 
function) default to UTF-8 encoding. In Python 2, file operations default to 
binary encoding. So if you want to read or write binary data to/from a file, 
always open the file using a binary mode(like "rb" or "wb").


### Item 4: Write Helper Functions Instead of Complex Expressions

Defaut value:

	red = my_values.get("red", [""])[0] or 0

The *opacity* case works because the value in the my_values dictionary is 
missing altogether. The behavior of the `get` method is to return its second 
argument if the key does not exist in the dictionary.

No:

	red = int(my_values.get('red', [''])[0] or 0)
	
Better: (For less complicated situations)

	red = my_values.get('red', [''])
	red = int(red[0]) if red[0] else 0  # if/else conditional, Added in Python 2.5

Best: (Especially if you need to use this logic repeatedly)

	def get_first_int(values, key, default=0):
	    found = values.get(key, [''])
	    if found[0]:
	        found = int(found[0])
	    else:
	        found = default
	    return found

### Item 5: Know How to Slice Sequences

Slicing can be extended to any Python class that implements the `__getitem__` 
and `__setitem__` special methods.

If you leave out both the start and the end indexes when slicing, you will end 
up with a copy of the original list.

	b = a[:]
	assert b == a and b is not a

If you assign a slice with no start or end indexes, you will replace its entire 
contents with a copy of what is referenced(instead of allocating a new list).


### Item 6: Avoid Using `start`, `end`, and `stride` in a Single Slice

Avoid using start, end and stride together in a single slice. If you need all 
three parameters, consider doing two assignments(one to slice, another to 
stride) or using *islice* from the *itertools* built-in module.


### Item 7: Use List Comprehensions Instead of map and filter

	even_squares = [x**2 for x in a if x % 2 == 0]

Dictionaries and sets have their own equivalents of list comprehensions. These 
make it easy to create derivative data structures when writing algorithms.

	chile_ranks = {'ghost': 1, 'habanero': 2, 'cayenne': 3}
	rank_dict = {rank: name for name, rank in chile_ranks.items()}
	chile_len_set = {len(name) for name in rank_dict.values()}

List comprehensions allow you to easily skip items from the input list, a 
behavior map does not support without help from filter.


### Item 8: Avoid More Than Two Expressions in List Comprehensions

	matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
	flat = [x for row in matrix for x in row]
	print(flat)
	>>> [1, 2, 3, 4, 5, 6, 7, 8, 9]
	squared = [[x**2 for x in row] for row in matrix]

List comprehensions also support multiple if conditions. Multiple conditions at 
the same loop level are an implicit and expression.

	a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
	b = [x for x in a if x > 4 if x % 2 == 0]
	c = [x for x in a if x > 4 and x % 2 == 0] # Equivalent


### Item 9: Consider Generator Expressions for Large Comprehensions

The problem with list comprehensions is that they may create a whole new list 
containing one item for each value in the input sequences. This is fine for 
small inputs, but for file which is absolutely enormous or perhaps a never-
ending network socket, list comprehensions are problematic.

	value = [len(x) for x in open('/tmp/my_file.txt')]

To solve this, Python provides generator expressions, it do not materialize the 
whole output sequence when they are run. Instead, generator expressions evaluate 
to an iterator that yields one item at a time from the expression.

A generator expression is created by putting list-comprehension-like syntax 
between `()` characters:

	it = (len(x) for x in open('/tmp/my_file.txt'))
	print(next(it))
	print(next(it))

Another powerful outcome of generator expressions is that they can be composed 
together.

	roots = ((x, x**0.5) for x in it)

Chaining generators like this executes very quickly in Python. When you are 
looking for a way to compose functionality that is operating on a large stream 
of input, generator expressions are the best tool for the job.

The obly gotcha is that the iterators returned by generator expressions are 
stateful, so you must be careful not to use them more than once.


### Item 10: Prefer `enumerate` Over `range`

No:

	for i in range(len(flavor_list)):
	    flavor = flavor_list[i]

Yes:

	for i, flavor in enumerate(flavor_list):
	    pass


### Item 11: Use `zip` to Process Iterators in Parallel

No:

	for i in range(len(names)):
	    count = letters[i]
	    if count > max_letters:
	        longest_name = names[i]
	        max_letters = count

Better:

	for i, name in enumerate(names):
	    count = letters[i]
	    if count > max_letters:
	        longest_name = name
	        max_letters = count

Best:

	for name, count in zip(names, letters):
	    if count > max_letters:
	        longest_name = name
	        max_letters = count

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

When your usage of `nonlocal` starts getting complicated, it is better to wrap 
your state in a helper class.

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

Unfortunately, Python 2 does not support the `nonlocal` keyword. You need to 
use a work-around that takes advantage of Pythons scoping rules.

	def sort_priority(numbers, group):
	    found = [False]
	    def helper(x):
	        if x in group:
	            found[0] = True
	            return (0, x)
	        return (1, x)
	    numbers.sort(key=helper)
	    return found[0]


### Item 16: Consider Generators Instead of Returning Lists

No:

	def index_words(text):
	    result = []
	    if text:
	        result.append(0)
	    for index, letter in enumerate(text):
	        if letter == ' ':
	            result.append(index + 1)
	    return result

Yes:

	def index_words_iter(text):
	    if text:
	        yield 0
	    for index, letter in enumerate(text):
	        if letter == ' ':
	            yield index + 1

The iterator returned by the generator call can easily be converted to a list 
by passing it to the `list` built-in function

	result = list(index_words_iter(address))

The only gotcha of defining generators like this is that the callers must be 
aware that the iterators returned are stateful and can not be reused.


### Item 17: Be Defensive When Iterating Over Arguments

When a function takes a list of objects as a parameter, it is often important 
to iterate over that list multiple times.

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

The cause of this behavior is that an iterator obly produces its results a 
single time. If you iterate over an iterator or generator that has already 
raised a `StopIteration` exception, you will not get any results the second 
time around.

To solve this problem, you can explicitly exhaust an input iterator and keep a 
copy of its entire contents in a list.

	def normalize_copy(numbers):
	    numbers = list(numbers)
	    total = sum(numbers)
	    result = []
	    for value in numbers:
	        percent = 100 * value / total
	        result.append(percent)
	    return result

The problem with this approach is the copy of the input iterators contents 
counld be large. One way around this is to accept a function that returns a new 
iterator each time it is called.

	def normalize_func(get_iter):
	    total = sum(get_iter())
	    result = []
	    for value in get_iter():
	        percent = 100 * value / total
	        result.append(percent)
	    return result
	
	percentages = normalize_func(lambda: read_visits(path))

The better way to achieve the same result is to provide a new container class 
that implements the *iterator protocol*. The iterator protocol is how Python 
`for` loops and related expressions traverse the contents of a container type. 
When Python sees a statement like `for x in foo` it will actually call `iter(
foo)`.  The `iter` built-in function calls the `foo.__iter__` special method in 
turn. The `__iter__` method must return an iterator object(which itself 
implements the `__next__` special method). Then the `for` loop repeatedly calls 
the `next` built-in function on the iterator object until it is exhausted(and 
raise a `StopIteration` exception).

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

This works because the `sum` method in `normalize` will call 
`ReadVisits.__iter__` to allocate a new iterator object. The `for` loop to 
normalize the numbers will also call `__iter__` to allocate a seconde iterator 
object. Each of those iterators will be advanced exhausted independently.

You can detect that a value is an iterator(instead of a container) if calling 
`iter` on it twice produces the same result, which can then be progressed with 
the `next` built-in function.


### Item 18: Reduce Visual Noise with Variable Positional Arguments

Functions that accept `\*args` are best for situations where you know the number 
of inputs in the argument list will be reasonably small.

You can use the items from a sequence as the positional arguments for a function 
with the `\*` operator.

	def log(message, *values):
	    pass
	
	favorites = [7, 39, 99]
	log('Favorite colors', *favorites)      # * operator

Using the `\*` operator with a generator may cause your program to run out of 
memory and crash.


### Item 19: Provide Optional Behavior with Keyword Arguments

* Keyword arguments with default values make it easy to add new behaviors to a 
function, especially when the function has existing callers.


### Item 20: Use `None` and Docstrings to Specify Dynamic Default Arguments

	def log(message, when=datetime.now()):
	    print('%s: %s' % (when, message))
	log('Hi there!')
	sleep(1)
	log('Hi again')
	>>>2014-11-15   21:10:10.371432:    Hi  there!
	2014-11-15  21:10:10.371432:    Hi  again!

The timestamps are the same because `datetime.now()` is only executed a single 
time: when the function is defined. Default argument values are evaluated only 
once per module load, which usually happens when a program starts up.

	def log(message, when=None):
	    '''Log a message with a timestamp.
	    Args:
	        message: Message to print.
	        when: datetime of when the message occurred.
	            Defaults to the present time.
	    '''
	    when = datetime.now() if when is None else when
	    print('%s: %s' % (when, message))


### Item 21: Enforce Clarity with Keyword-Only Arguments

In Python 3, you can demand clarity by defining your functions with keyword-only 
arguments.

	def safe_division_c(number, divisor, *, 
	                    ignore_overflow=False,
	                    ignore_zero_division=False):
	    pass

Now, calling the function with positional arguments for the keyword arguments 
will not work.

Unfortunately, Python 2 does not have explicit syntax for specifying keyword-
only arguments like Python 3. But you can achieve the same behavior of raising 
`TypeError` for invalid function calls by using the `**` operator in argument 
lists.

	def safe_division_d(number, divisor, **kwargs):
	    ignore_overflow = kwargs.pop('ignore_overflow', False)
	    ignore_zero_division = kwargs.pop('ignore_zero_division', False)
	    if kwargs:
	        raise TypeError('Unexpected **kwargs: %r' % kwargs)


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

	class CountMissing(object):
	    def __init__(self):
	        self.added = 0
	    def missing(self):
	        self.added += 1
	        return 0
	counter = CountMissing()
	result = defaultdict(counter.missing, current)

Much better:

	class BetterCountMissing(object):
	    def __init__(self):
	        self.added = 0
	    def __call__(self):
	        self.added += 1
	        return 0
	counter = BetterCountMissing()
	assert callable(counter)
	result = defaultdict(counter, current)  # Relies on __call__

When you need a function to maintain state, consider defining a class that 
provides the `__call__` method instead of defining a stateful closure.


### Item 24: Use `@classmethod` Polymorphism to Construct Objects Generically

In Python, not only do the objects support polymorphism, but the classes do as 
well.

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

The huge issue is the `mapreduce` function is not generic at all. The best way 
to solve this problem is with `@classmethod` polymorphism. This is exactly like 
the instance method polymorphism I used for `InputData.read`, except that it 
applies to whole classes instead of their constructed objects.

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

Now you can write other `GenericInputData` and `GenericWorker` classes as you 
wish and not have to rewrite any of the glue code.


### Item 25: Initialize Parent Classes with `super`

The old way to initialize a parent class from a child class is to directly call 
the parent classs `__init__` method with the child instance.

	class MyBaseClass(object):
	    def __init__(self, value):
	        self.value = value
	
	class MyChildClass(MyBaseClass):
	    def __init__(self):
	        MyBaseClass.__init__(self, 5)

This approach works fine for simple hierarchies but breaks down in many cases. 
If your class is affected by multiple inheritance, calling the superclasses `
__init__` methods directly can lead to unpredictable behavior.

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

To solve these problems, Python 2.2 added `super` built-in function and defined 
the *method resolution order*(MRO).

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

When I call `GoodWay(5)`, it in turn calls `TimesFiveCorrect.__init__`, which 
calls `PlusTwoCorrect.__init__`, which calls `MyBaseClass.__init__`. Once this 
reaches the top of the diamond, then all of the initialization methods actually 
do  their work in the opposite order from how their `__init__` functions were 
called.

But the syntax is a bit verbose. Python 3 fixes these issues:

	class Explicit(MyBaseClass):
	    def __init__(self, value):
	        super(__class__, self).__init__(value * 2)
	
	class Implicit(MyBaseClass):
	    def __init__(self, value):
	        super().__init__(value * 2)
	
	assert Explicit(10).value == Implicit(10).value


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

	bar = [1, 2, 3]
	bar[0]

it will be interpreted as:

	bar.__getitem__(0)

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

Specifying a `setter` on a property also lets you perform type checking and 
validation on values passed to your class.

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

You can even use `@property` to make attributes from parent classes immutable.

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


### Item 30: Consider `@property` Instead of Refactoring Attributes

The built-in `@property` decorator makes it easy for simple accesses of an 
instances attributes to act smarter. One advanced but common use of `@property` 
is transitioning what was once a simple numerical attribute into an on-the-fly 
calculation.

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


### Item 31: use Descriptors for Reusable `@property` Methods

The big problem with the `@property` built-in is the methods ti decorates can 
not be reused for multiple attributes of the same class. They also can not be 
reused by unrelated classes.

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

Also, this approach is not general. It is gets tedious.

The better way to do this in Python is to use a *descriptor*. the descriptor 
protocol defines how attribute access is interpreted by the language. A descr-
iptor class can provide `__get__` and `__set__` methods that let you reuse the 
grade validation behavior without any boilerplate.

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

* Reuse the behavior and validation of `@property` methods by defining your own 
  descriptor classes.
* Use `WeakKeyDictionary` to ensure that your descriptor classes do not cause 
  memory leaks.


### Item 32: Use `__getattr__`, `__getattribute__`, and `__setattr__` for Lazy Attributes

If your class defines `__getattr__`, that method is called every time an attri-
bute can not be found in an objects instance dictionary.

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

Python has another language hook called `__getattribute__`. This special method 
is called every time an attribute is accessed on an object, even in cases where 
it **does** exist in the attribute dictionary. This enables you to do things 
like check global transaction state on every property access.

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

The problem is that `__getattribute__` accesses `self._data`, which causes `
__getattribute__` to run again, which accesses `self._data` again, and so on.
The solution is to use the `super().__getattribute__` method on your instance 
to fetch values from the instance attribute dictionary.

	class DictionaryDB(object):
	    def __init__(self, data):
	        self._data = data
	    def __getattribute__(self, name):
	        data_dict = super().__getattribute__('_data')
	        return data_dict[name]

Similarly, you will need `__setattr__` methods that modify attributes on an 
object to use `super().__setattr__`.


### Item 33: Validate Subclasses with Metaclasses

One of the simplest applications of metaclasses is verifying that a class was 
defined correctly. When you are building a complex class hierarchy, you may 
want to enforce sytle, require overriding methods, or and so on. Using 
metaclasses for validation can raise errors much earlier. A metaclass is 
defined by inheriting from `type`.

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

The metaclass has access to the name of the class, the parent classes it 
inherits form, and all of the class attributes that were defined in the classs 
body.

Python 2 has slightly different syntax and specifies a metaclass using the `
__metaclass__` class attribute.

	class Meta(type):
	    def __new__(meta, name, bases, class_dict):
	        # ...
	
	class MyClassInPython2(object):
	    __metaclass__ = Meta
	    # ...

You can add functionality to the `Meta.__new__` method in order to validdate 
all of the parameters of a class before it is defined.

The `__new__` method of metaclasses is run after the `class` statements entire 
body has been processed.


### Item 34: Register Class Existence with Metaclasses

Another common use of metaclasses is to automatically register types in your 
program.

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

Now, I can deserialize an arbitrary JSON string without having to know which 
class it contains. The problem with this approach is that you can forget to 
call `register_class`.

Metaclasses enable this by intercepting the `class` statement when subclasses 
are defined.

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

Using metaclasses for class regisration ensures that you will never miss a 
class as long as the inheritance tree is right.


### Item 35: Annotate Class Attributes with Metaclasses

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

To eliminate the redundancy, use the metaclass.

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


-----------

## 5. Concurrency and Parallelism

Concurrency is when a computer does many different things **seemingly** at the 
same time. On a computer with one CPU core, the operating system will rapidlly 
change which program is running on the single processor.

Parallelism is **actually** doing many different things at the same time. 
Computers with multiple CPU cores can execute multiple programs simultaneously.
Each CPU core runs the instructions of a separate program.

The key difference between parallelism and concurrency is *speedup*. When two 
distinct paths of execution in a program make forward progress in parallel, the 
time it takes to do the total work is cut in half; the speed of execution is 
faster by a factor of two. In  contrast, concurrent programs may run thousands 
of separate paths of execution seemingly in parallel but provide no speedup for 
the total work.


### Item 36: Use `subprocess` to Manage Child Processes

Child processes started by Python are able to run in parallel, enabling you to 
use Python to consume all of the CPU cores of your machine and maximize the 
throughput of your programs.

	proc = subprocess.Popen(['echo', 'hello from the child!'], 
	                        stdout=subprocess.PIPE)
	out, err = proc.communicate()
	print(out.decode('utf-8'))
	>>> hello from the child!

Child processes will run independently from their parent process, the Python 
interpreter. Their status can be polled periodically while Python does other 
work.

	proc = subprocess.Popen(['sleep', '0.3'])
	while proc.poll() is None:
		print('working...')
	print('exit status: ', proc.poll())
	>>> 
	working...
	working...
	exit status 0

Decoupling the child process from the parent means that the parent process is 
free to run many child processes in parallel.

	def run_sleep(period):
	    proc = subprocess.Popen(['sleep', str(period)])
	    return proc
	procs = []
	for _ in range(10):
	    proc = run_sleep(0.1)
	    procs.append(proc)
	for proc in procs:
	    proc.communicate()  # Get the result of the command and wait for the end.

You can also pipe data from your Python program into a subprocess and retrieve 
its output. This allows you to utilize other programs to do work in parallel.

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


### Item 38: Use Lock to Prevent Data Races in Threads

* Even though Python has a GIL, you are still responsible for protecting against 
  data races between the threads in your programs.
* Your programs will corrupt their data structures if you allow multiple threads 
  to modify the same objects without locks.
* The `Lock` class in the `threading` built-in module is Pythons standard mutual 
  exclusion lock implementation.


### Item 39: Use `Queue` to Coordinate Work Between Threads

A pipeline works like an assembly line used in manufacturing. Pipelines have 
many phases in serial with a specific function for each phase. New pieces of 
work are constantly added to the begining of the pipeline. Each function can 
operate concurrently on the piece of work in its phase. The work moves forward 
as each function completes until there are no phases remaining. This approach 
is especially good for work that includes blocking I/O or subprocesses--
activities that can easily be parallelized using Python.

For example, say you want to build a system that will take a constant stream of 
images from your digital camera, resize them, and then add them to a photo 
gallery online.

	class MyQueue(object):
	    def __init__(self):
	        self.items = deque()
	        self.lock = Lock()
	    def put(self, item):
	        with self.lock:
	            self.items.append(item)
	    def get(self):
	        with self.lock:
	            self.items.popleft()
	
	class Worker(Thread):
	    def __init__(self, func, in_queue, out_queue):
	        super().__init__()
	        self.func = func
	        self.in_queue = in_queue
	        self.out_queue = out_queue
	        self.polled_count = 0
	        self.work_done = 0
	    def run(self):
	        while True:
	            self.polled_count += 1
	            try:
	                item = self.in_queue.get()
	            except IndexError:
	                sleep(0.01)     # No work to do
	            else:
	                result = self.func(item)
	                self.out_queue.put(result)
	                self.work_done += 1
	
	download_queue = MyQueue()
	resize_queue = MyQueue()
	upload_queue = MyQueue()
	done_queue = MyQueue()
	threads = [
	    Worker(download, download_queue, resize_queue)
	    Worker(resize, resize_queue, upload_queue)
	    Worker(upload, upload_queue, done_queue)
	]
	for thread in threads:
	    thread.start()
	for _ in range(1000):
	    download_queue.put(object())    # Use plain `object` instance as a proxy 
	                                    # for the real data
	while len(done_queue.items) < 1000:
	    # Do something useful while waiting
	    # ...

When the worker functions vary in speeds, an earlier phase can prevent progress 
in later phases, backing up the pipeline. This causes later phases to starve 
and constantly check their input queues for new work in a tight loop. The 
outcome is that worker threads waste CPU time doing nothing useful.

There are three more problems that you should also avoid:

1. Determining that all of the input work is complete requires yet another busy 
   wait on the `done_queue`.
2. In `Worker` the `run` method will execute forever in its busy loop.
3. If the first phase makes rapid progress but the second phase makes slow 
   progress, then the queue connecting the first phase to the second phase will 
   constantly increase in size.

The `Queue` class from the `queue` built-in module provides all of the 
functionality you need to solve these problems.

Queue eliminates the busy waiting in the worker by making the `get` method 
block until new data is available.

	queue = Queue()
	def consumer():
	    print('Consumer waiting')
	    queue.get()
	    print('Consumer done')
	thread = Thread(target=consumer)
	thread.start()
	
	print('Producer putting')
	queue.put(object())
	thread.join()
	print('Producer done')
	>>> Consumer waiting
	Producer putting
	Consumer done
	Producer done

To solve the pipeline backup issue, the `Queue` class lets you specify the 
maximum amount of pending work you will allow between two phases.

	queue = Queue(1)    # Buffer size of 1
	def consumer():
	    time.sleep(0.1)
	    queue.get()
	    print('Consumer got 1')
	    queue.get()
	    print('Consumer got 2')
	thread = Thread(target=consumer)
	thread.start()
	queue.put(object())
	print('Producer put 1')
	queue.put(object()) # Will be blocked because of the `time.sleep()`
	print('Producer put 2')
	thread.join()
	print('Producer done')
	>>> 
	Producer put 1
	Consumer got 1
	Producer put 2
	Consumer got 2
	Producer done

The `Queue` class also track the progress of work using the `task_done` method. 

	in_queue = Queue()
	def consumer():
	    work = in_queue.get()   # Done second
	    # Doing work
	    # ...
	    in_queue.task_done()    # Done third
	Thread(target=consumer).start()
	in_queue.put(object())      # Done first
	in_queue.join()             # Done fourth

Put all of these together:

	class CloseableQueue(Queue):
	    SENTINEL = object()
	    def close(self):
	        self.put(self.SENTINEL)
	    def __iter__(self):
	        while True:
	            item = self.get()
	            try:
	                if item is self.SENTINEL:
	                    return
	                yield item
	            finally:
	                self.task_done()
	
	class StoppableWorker(Thread):
	    def __init__(self, func, in_queue, out_queue):
	        # ...
	    def run(self):
	        for itme in self.in_queue:
	            result = self.func(item)
	            self.out_queue.put(result)
	
	download_queue = CloseableQueue()
	# ...
	threads = [
	    StoppableWorker(download, download_queue, resize_queue)
	    # ...
	]
	
	for thread in threads:
	    thread.start()
	for _ in range(1000):
	    download_queue.put(object())
	download_queue.close()
	download_queue.join()
	resize_queue.close()
	resize_queue.join()
	upload_queue.close()
	upload_queue.join()


### Item 40: Consider Coroutines to Run Many Functions Concurrently

Threre are three big problems with threads:

1. They require special tools to coordinate with each other safely.
2. Threads require a lot of memory, about 8MB per executing thread.
3. Threads are costly to start.

Python can work around these issues with *coroutines*. Conroutines let you have 
many seemingly simultaneous functions in your Python programs. The cost of 
starting a generator coroutine is a function call. Once active, they each use 
less than 1KB of memory until they are exhausted.

Coroutines work by enabling the code consuming a generator to `send` a value 
back into the generator function after each `yield` expression.

	def my_coroutine():
	    while True:
	        received = yield
	        print('Received: ', received)
	it = my_coroutine()
	next(it)  # Prime the coroutine, prepare the generator to receive the value.
	it.send('First')
	it.send('Second')
	>>> Received: First
	Received: Second

To implement a generator coroutine that yields the minimum value it has been 
sent so far.

	def minimize():
	    current = yield
	    while True:
	        value = yield current
	        current = min(value, current)
	it = minimize()
	next(it)
	print(it.send(10))
	print(it.send(4))
	print(it.send(22))
	print(it.send(-1))


### Item 41: Consider `concurrent.futures` for True Parallelism

The `multiprocessing` built-in module, easily accessed via the `concurrent.
futures` built-in module, eables Python to utilize multiple CPU cores in 
parallel by running additional interpreters as child process. These child 
processes are separate from the main interpreter, so their global interpreter 
locks are also seperate. Each child can fully utilize one CPU core.

The power of `multiprocessing` is best accessed through the `concurrent.futures` 
built-in module and its simple `ProcessPoolExecutor` class. The advanced parts 
of the `multiprocessing` module should be avoided because they are so complex.


--------

## 6. Built-in Modules


### Item 42: Define Function Decorators with `functools.wraps`

Python has special syntax for *decorators* that can be applied to functions. 
Decorators have the ability to run additional code before and after any cals to 
the functions they wrap. This allows them to access and modify input arguments 
and return values. This functionality can be useful for enforcing semantics, 
debugging, registering functions, and more.

	def trace(func):
	    def wrapper(*args, **kwargs):
	        result = func(*args, **kwargs)
	        print('%s(%r, %r)->%r' % (func.__name__, args, kwargs, result))
	        return result
	    return wrapper
	
	@trace
	def fibonacci(n):
	    if n in (0, 1):
	        return n
	    return (fibonacci(n - 2), + fibonacci(n - 1))

The `@` symbol is equivalent to `fibonacci = trace(fibonacci)`. but:

	print(fibonacci)
	>>> <function trace.<locals>.wrapper at 0x107f7ed08>

The solution is to use the wraps helper function from the `functools` built-in 
module. Applying it to the `wrapper` function will copy all of the important 
metadata about the inner function to the outer function.

	def trace(func):
	    @wraps(func)
	    def wrapper(*args, **kwargs):
	        # ...
	    return wrapper
	@trace
	def fibonacci(n):
	    # ...


### Item 43: Consider `contextlib` and `with` Statements for Reusable `try/finally` Behavior

	def my_function():
	    logging.debug('Some debug data')
	    logging.error('Error log here')
	    logging.debug('More debug data')
	my_function()
	>>> Error log here

The default log level for my program is `WARNING`, so only the error message 
will print to screen when I run the function.

	@contextmanager
	def debug_logging(level):
	    logger = logging.getLogger()
	    old_level = logger.getEffectiveLevel()
	    logger.setLevel(level)
	    try:
	        yield
	    finally:
	        logger.setLevel(old_level)
	
	with debug_logging(logging.DEBUG):
	    print('Inside:')
	    my_function()
	print('After:')
	my_function()
	>>> 
	Inside:
	Some debug data
	Error log here
	More debug data
	After:
	Error log here

Using `with` targets:

	@contextmanager
	def log_level(level, name):
	    logger = logging.getLogger(name)
	    old_level = logger.getEffectiveLevel()
	    logger.setLevel(level)
	    try:
	        yield logger
	    finally:
	        logger.setLevel(old_level)
	
	with log_level(logging.DEBUG, 'my-log') as logger:
	    logger.debug('This is my message!')
	    logging.debug('This will not print')
	>>> This is my message!


### Item 44: Make `pickle` Reliable with `copyreg`

The `pickle` built-in module can serialize Python objects into a stream of 
bytes and deserialize bytes back into objects.

The `copyreg` module lets you register the functions responsible for 
serializing Python objects, allowing you to control the behavior of `pickle` 
and make it more reliable.

* The `pickle` built-in module is only useful for serializing and deserializing 
  objects between trusted programs.
* Use the `copyreg` built-in module with `pickle` to add missing attribute 
  values, allow versioning of classes., and provide stable import paths.


### Item 45: Use `datetime` Instead of `time` for Local Clocks

* Avoid using the `time` module for translating between different time zones.
* Use the `datetime` built-in module along with the `pytz` module to reliably 
  convert between times in different time zones.
* Always represent time in UTC and do conversions to local time as the final 
  step before presentation.


### Item 46: Use Built-in Algorithms and Data Structures

#### Double-ended Queue

The `deque` class from the `collections` module is a double-ended queue. It 
provides constant time operations for inserting or removing items from its 
begining or end.

	fifo = deque()
	fifo = append(1)
	x = fifo.popleft()

#### Orderd Dictionary

Standard dictionaries are unordered. That means a `dict` with the same keys and 
values can result in different orders of iteration.

The `OrderdDict` class from the `collections` module is a special type of 
dictionary that keeps track of the order in which its keys were inserted.

#### Default Dictionary

One problem with dictionaries is that you can not assume any keys are already 
present.

	stats = {}
	key = 'my_counter'
	if key not in stats:
	    stats[key] = 0
	stats[key] += 1

The `defaultdict` class from the `collections` module simplifies by automatic-
ally storing a default value when a key does not exist.

	stats = defaultdict(int)
	stats['my_counter'] += 1

#### Heap Queue

Heaps are useful data structures for maintaining a priority queue.

	a = []
	heappush(a, 5)
	heappush(a, 3)
	heappush(a, 7)
	heappush(a, 4)
	print(heappop(a), heappop(a), heappop(a), heappop(a))
	>>> 3 4 5 7

#### Bisection

Searching for an item in a `list` takes linear time proportional to its length 
when you call the `index` method.

	x = list(range(10**6))
	i = x.index(991234)

The `bisect` modules functions, such as `bisect_left`, provide an efficient 
binary search through a sequence of sorted items.

	i = bisect_left(x, 991234)

#### Iterator Tools

The `itertools` functions fall into three main categories:

* Linking iterators together
* Filtering items from an iterator
* Combinations of items from iterators


### Item 47: Use `decimal` When Precision Is Paramout

The `Decimal` class provides fixed point math of 28 decimal points by default. 
It can go even higher if required. It is ideal for situations that require high 
precision and exact rounding behavior, such as computations of monetary values.


### Item 48: Know Where to Find Community-Built Modules

pip.


--------

## 7. Collaboration


### Item 49: Write Docstrings for Every Function, Class, and Module

#### Documenting Modules

The first line of docstring should be a single sentence describing the modules 
purpose.

	#!/usr/bin/env python3
	"""Library for testing words for various linguistic patterns.
	
	Testing how words relate to each other can be tricky sometimes!
	This module provides easy ways to determine when words you've found have 
	special properties.
	
	Available functions:
	- palindrome: Determine if a word is a palindrome.
	- check_anagram: Determine if two words are anagrams.
	...
	"""
	# ...

#### Documenting Classes

	class Player(object):
	    """Represents a player of the game.
	    
	    Subclasses may override the ‘tick’ method to provide custom animations for 
	    the player’s movement depending on their power level, etc.
	    
	    Public attributes:
	    - power: Unused power-ups (float between 0 and 1).
	    - coins: Coins found during the level (integer).
	    """
	    # ...

#### Documenting Functions

	def find_anagrams(word, dictionary):
	    """Find all anagrams for a word.
	    
	    This function only runs as fast as the test for membership in the 
	    ‘dictionary’ container. It will be slow if the dictionary is a list and 
	    fast if it’s a set.
	    
	    Args:
	        word: String of the target word.
	        dictionary: Container with all strings that are known to be actual 
	            words.
	    
	    Returns:
	        List of anagrams that were found. Empty if none were found.
	    """
	    # ...


### Item 50: Use Package to Organize Modules and Provide Stable APIs

*Packages* are modules that contain other modules.

When you are writing an API for wider consumption, you will want to provide 
stable functionality that does not change between releases. To ensure that 
happens, it is important to hide your internal code organization from external users. 
This enables you to refactor and improve your packages internal modules without 
breaking existing users.

Python can limit the surface area exposed to API consumers by using the `__all__`
special attribute of a module or package. The value fo `__all__` is a list of 
every name to export from the module as part of its public API. When consuming 
code does `from foo import *`, only the attributes in `foo.__all__` will be 
imported from `foo`. If `__all__` is not present in `foo`, then only public 
attributes, those without a leading underscore, are imported.

Define the `models` module of `mypackage` to contain the representation of 
projectiles:

	# models.py
	__all__ = ['Projectile']
	
	class Projectile(object):
	    def __init__(self, mass, velocity):
	        self.mass = mass
	        self.velocity = velocity
	
	# Also define a utils module in mypackage to perform operations on the 
	# Projectile instances.
	
	# utils.py
	from . models import Projectile
	
	__all__ = ['simulate_collision']
	
	def _dot_product(a, b):
	    # ...
	
	def simulate_collision(a, b):
	    # ...
	
	# To provide all of the public parts of this API as a set of attributes 
	# that are available on the "mypackage" module. This will allow 
	# downstream consumers  to always import directly from mypackage instead 
	# of imprting from "mypackage.models" or "mypackage.utils". This 
	# ensures that the API consumers code will continue to work even if the 
	# internal organization of "mypackage" changes.
	
	# To do this with Python packages, you need to modify the "__init__.py" 
	# file in the "mypackage" directory. This file actually becomes the 
	# contents of the "mypackage" module when it is imported.
	
	# __init__.py
	__all__ = []
	from . models import *
	__all__ += models.__all__
	from . utils import *
	__all__ += utils.__all__
	
	# api_consumer.py
	from mypackage import *
	a = Projectile(1.4, 3)
	b = Projectile(3, 1.6)
	after_a, after_b = simulate_collision(a, b)

However, if you are building an API for use between your own modules, the 
functionality of `__all__` is probably unnecessary and should be avoided.


### Item 51: Define a Root EXception to Insulate Callers from APIs

Having a root exception in a module makes it easy for consumers of your API to 
catch all of the exceptions that you raise on purpose.

* Defining root exceptions for your modules allows API consumers to insulate 
  themselves from you API.
* Catching root exceptions can help you find bugs in code that consumes an API.
* Catching the Python `Exception` base class can help you find bugs in API 
  implementations.
* Intermediate root exceptions let you add more specific types of exceptions in 
  the future without breaking your API consumers.


### Item 52: Know How to Break Circular Dependencies

	# dialog.py
	import app
	
	class Dialog(object):
	    def __init__(self, save_dir):
	        self.save_dir = save_dir
	    # ...
	save_dialog = Dialog(app.prefs.get('save_dir'))
	def show():
	    # ...
	
	# app.py
	import dialog
	
	class Prefs(object):
	    # ...
	    def get(self, name):
	        # ...
	
	prefs = Prefs()
	dialog.show()

It is a circular dependency. If you try to use the `app` module from your main 
program, you will get an exception when you import it.

When a module is imported, here is what Python actually does in depth-first 
order:

1. Searches for your module in locations from `sys.path`
2. Loads the code from the module and ensures that it compiles
3. Creates a corresponding empty module object
4. Inserts the module into `sys.modules`
5. Runs the code in the module object to define its contents

The problem with a circular dependency is that the attributes of a module are not 
defined until the code for those attributes has executed(after step #5). But 
the module can be loaded with the `import` statement immediately after it is 
inserted into `sys.modules`(after step #4).

There are three other ways to break circular dependencies:

#### Reordering Imports

	# app.py
	class Prefs(object):
	    # ...
	prefs = Prefs()
	
	import dialog
	dialog.show()

Although this avoids the `AttributeError`, it goes against the PEP 8 style 
guide. The style guide suggests that you always put imports at the top of your 
Python files. Thus, you should avoid import reordering to solve your circular 
dependency issues.

#### Import, Configure, Run

A second solution to the circular imports problem is to have your modules 
minimize side effects at import time. You have your modules only define 
functions, classes, and constants. You avoid actually running any functions at 
import time.

	# dialog.py
	import app
	
	class Dialog(object):
	    # ...
	
	save_dialog = Dialog()
	
	def show():
	    # ...
	
	def configure():
	    save_dialog.save_dir = app.prefs.get('save_dir')
	
	# app.py
	import dialog
	
	class Prefs(object):
	    # ...
	
	prefs = Prefs()
	
	def configure():
	    # ...
	
	# main.py
	import app
	import dialog
	
	app.configure()
	dialog.configure()
	
	dialog.show()

#### Dynamic Import

The third-and often simplest-solution to the circular imiports problem is to 
use an `import` statement within a function or method. This is called a *dynamic 
import* because the module import happens while the program is running.

	# dialog.py
	class Dialog(object):
	    # ...
	
	save_dialog = Dialog()
	
	def show():
	    import app
	    save_dialog.save_dir = app.prefs.get('save_dir')
	
	# app.py
	import dialog
	
	class Prefs(object):
	    # ...
	
	prefs = Prefs()
	dialog.show()


### Item 53: Use Virtual Environments for Isolated and Reproducible Dependencies

`pyvenv` allows you to create isolated versions of the Python environment. Using 
`pyvenv`, you can have many different versions of the same package installed on 
the same system at the same time without conflicts.

#### The `pyvenv` Command

	$ pyvenv /tmp/myproject
	$ cd /tmp/myproject
	$ ls
	bin     include     lib     pyvenv.cfg
	$ source bin/activate
	(myproject)$ which python3
	/tmp/myproject/bin/python3
	(myproject)$ pip3 install pytz
	(myproject)$ deactivate
	$ which python3
	/usr/local/bin/python3

#### Reproducing Dependencies

Eventually, you may want to copy your environment somewhere else.

	(myproject)$ pip3 freeze > requirements.txt
	$ pyvenv /tmp/otherproject
	$ cd /tmp/otherproject
	$ source bin/activate
	(otherproject)$ pip3 install -r /tmp/myproject/requirements.txt


----------

8. Production


### Item 54: Consider Module-Scoped Code to Configure Deployment Environments

	# dev_main.py
	TESTING = True
	import db_connection
	db = db_connection.Database()
	
	# prod_main.py
	TESTING = False
	import db_connection
	db = db_connection.Database()
	
	# db_connection.py
	import __main__
	
	class TestingDatabase(object):
	    # ...
	class RealDatabase(object):
	    # ...
	
	if __main__.TESTING:
	    Database = TestingDatabase
	else:
	    Database = RealDatabase

	# db_connection.py
	import sys
	class Win32Database(object):
	    # ...
	class PosixDatabase(object):
	    # ...
	
	if sys.platform.startwith('win32'):
	    Database = Win32Database
	else:
	    Database = PosixDatabase


### Item 55: Use `repr` Strings for Debugging Output

The `print` function outputs a human-readable string version of whatever you 
supply it. The problem is that the human-readble string for a value does not make 
it clear what the actual type of the value is.

	print(5)
	>>> 5
	print('5')
	>>> 5

If you are debugging a program with `print`, these type differences matter.

The `repr` built-n function returns the *printable representation* of an object,
which should be its most clearly understandable string representation.

	a = '\x07'
	print(repr(a))
	>>> '\x07'
	print(repr(5))
	>>> 5
	print(repr('5'))
	>>> '5'

Passing the value from `repr` to the `eval` built-in function should result in 
the same Python object you start with.

	b = eval(repr(a))
	assert b == a

For dynamic Python objects, the default human-readable string value is the same 
as the `repr` value. Unfortunately, the default value of `repr` ofr `object` 
instance is not especially helpful.

	class OpaqueClass(object):
	    def __init__(self, x, y):
	        # ...
	obj = OpaqueClass(1, 2)
	print(obj)
	>>> <__main__.OpaqueClass object at 0x107990ba8>

This output can not be passed to the `eval` function, and it says nothing about 
the instance fields of the object.

If you have control of the class:

	class BetterClass(object):
	    def __init__(self, x, y):
	        # ...
	    def __repr__(self):
	        return 'BetterClass(%d, %d)' % (self.x, self.y)
	obj = BetterClass(1, 2)
	print(obj)
	>>> BetterClass(1, 2)

If you don not have control over the class definition:

	obj = OpaqueClass(4, 5)
	pritn(obj.__dict__)
	>>> {'y': 5, 'x': 4}


### Item 56: Test Everything with `unittest`

* You can define tests by subclassing `TestCase` and defining one method per 
  behavior you would like to test. Test methods on `TestCase` classes must start 
  with the word `test`.
* It is important to write both unit tests(for isolated functionality) and 
  integration tests(for modules that interact).


### Item 57: Consider Interactive Debugging with `pdb`

	def complex_func(a, b, c):
	    # ...
	    import pdb; pdb.set_trace() # A single line so can be commented easily.

command: "bt", "up", "down"


### Item 58: Profile Before Optimizing

Python provides a built-in *profiler* for determining which parts of a program 
are responsible for its execution time. This lets you focus your optimization 
efforts on the biggest sources of trouble and ignore parts of the program that 
do not impact speed.

Python provides two built-in profilers, one that is pure Python(profile) and 
another that is a C-extension module(cProfile). The `cProfile` built-in module 
is better because of its minimal impact on the performance of your program 
while it is being profiled.


### Item 59: Use `tracemalloc` to Understand Memory Usage and Leaks

In practice, programs eventually do run out of memory due to held references. 

The first way to debug memory usage is to ask the `gc` built-in module to list 
every object currently known by the garbage collector.

The `gc` module can help you understand which objects exist, but it has no 
infomation about how they were allocated.

Python 3.4 introduces a new `tracemalloc` built-in module, makes it possible to 
connect an object back to where it was allocated.
