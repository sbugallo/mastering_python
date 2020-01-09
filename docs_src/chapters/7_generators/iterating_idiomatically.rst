2. Iterating idiomatically
**************************

n this section, we will first explore some idioms that come in handy when we have to deal
with iteration in Python. These code recipes will help us get a better idea of the types of
things we can do with generators (especially after we have already seen generator
expressions), and how to solve typical problems in relation to them.
Once we have seen some idioms, we will move on to exploring iteration in Python in more
depth, analyzing the methods that make iteration possible, and how iterable objects work.
Idioms for iteration
We are already familiar with the built-in enumerate() function that, given an iterable, will
return another one on which the element is a tuple, whose first element is the enumeration
of the second one (corresponding to the element in the original iterable):
>>> list(enumerate("abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]
We wish to create a similar object, but in a more low-level fashion; one that can simply
create an infinite sequence. We want an object that can produce a sequence of numbers,
from a starting one, without any limits.
An object as simple as the following one can do the trick. Every time we call this object, we
get the next number of the sequence ad infinitum:
class NumberSequence:
def __init__(self, start=0):
self.current = start
def next(self):
current = self.current
self.current += 1
return current
[ 193 ]Using Generators
Chapter 7
Based on this interface, we would have to use this object by explicitly invoking its next()
method:
>>> seq = NumberSequence()
>>> seq.next()
0
>>> seq.next()
1
>>> seq2 = NumberSequence(10)
>>> seq2.next()
10
>>> seq2.next()
11
But with this code, we cannot reconstruct the enumerate() function as we would like to,
because its interface does not support being iterated over a regular Python for loop, which
also means that we cannot pass it as a parameter to functions that expect something to
iterate over. Notice how the following code fails:
>>> list(zip(NumberSequence(), "abcdef"))
Traceback (most recent call last):
File "...", line 1, in <module>
TypeError: zip argument #1 must support iteration
The problem lies in the fact that NumberSequence does not support iteration. To fix this,
we have to make the object an iterable by implementing the magic
method __iter__() . We have also changed the previous next() method, by using the
magic method __next__ , which makes the object an iterator:
class SequenceOfNumbers:
def __init__(self, start=0):
self.current = start
def __next__(self):
current = self.current
self.current += 1
return current
def __iter__(self):
return self
[ 194 ]Using Generators
Chapter 7
This has an advantage—not only can we iterate over the element, we also don't even need
the .next() method any more because having __next__() allows us to use the
next() built-in function:
>>> list(zip(SequenceOfNumbers(), "abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]
>>> seq = SequenceOfNumbers(100)
>>> next(seq)
100
>>> next(seq)
101
The next() function
The next() built-in function will advance the iterable to its next element and return it:
>>> word = iter("hello")
>>> next(word)
'h'
>>> next(word)
'e' # ...
If the iterator does not have more elements to produce, the StopIteration exception is
raised:
>>> ...
>>> next(word)
'o'
>>> next(word)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
StopIteration
>>>
This exception signals that the iteration is over and that there are no more elements to
consume.
If we wish to handle this case, besides catching the StopIteration exception, we could
provide this function with a default value in its second parameter. Should this be provided,
it will be the return value in lieu of throwing StopIteration :
>>> next(word, "default value")
'default value'
[ 195 ]Using Generators
Chapter 7
Using a generator
The previous code can be simplified significantly by simply using a generator. Generator
objects are iterators. This way, instead of creating a class, we can define a function that
yield the values as needed:
def sequence(start=0):
while True:
yield start
start += 1
Remember that from our first definition, the yield keyword in the body of the function
makes it a generator. Because it is a generator, it's perfectly fine to create an infinite loop
like this, because, when this generator function is called, it will run all the code until the
next yield statement is reached. It will produce its value and suspend there:
>>> seq = sequence(10)
>>> next(seq)
10
>>> next(seq)
11
>>> list(zip(sequence(), "abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]
Itertools
Working with iterables has the advantage that the code blends better with Python itself
because iteration is a key component of the language. Besides that, we can take full
advantage of the itertools module (ITER-01). Actually, the sequence() generator we
just created is fairly similar to itertools.count() . However, there is more we can do.
One of the nicest things about iterators, generators, and itertools, is that they are
composable objects that can be chained together.
For instance, getting back to our first example that processed purchases in order to get
some metrics, what if we want to do the same, but only for those values over a certain
threshold? The naive approach of solving this problem would be to place the condition
while iterating:
# ...
def process(self):
for purchase in self.purchases:
if purchase > 1000.0:
...
[ 196 ]Using Generators
Chapter 7
This is not only non-Pythonic, but it's also rigid (and rigidity is a trait that denotes bad
code). It doesn't handle changes very well. What if the number changes now? Do we pass it
by parameter? What if we need more than one? What if the condition is different (less than,
for instance)? Do we pass a lambda?
These questions should not be answered by this object, whose sole responsibility is to
compute a set of well-defined metrics over a stream of purchases represented as numbers.
And, of course, the answer is no. It would be a huge mistake to make such a change (once
again, clean code is flexible, and we don't want to make it rigid by coupling this object to
external factors). These requirements will have to be addressed elsewhere.
It's better to keep this object independent of its clients. The less responsibility this class has,
the more useful it will be for more clients, hence enhancing its chances of being reused.
Instead of changing this code, we're going to keep it as it is and assume that the new data is
filtered according to whatever requirements each customer of the class has.
For instance, if we wanted to process only the first 10 purchases that amount to more than
1,000 , we would do the following:
>>> from itertools import islice
>>> purchases = islice(filter(lambda p: p > 1000.0, purchases), 10)
>>> stats = PurchasesStats(purchases).process() # ...
There is no memory penalization for filtering this way because since they all are generators,
the evaluation is always lazy. This gives us the power of thinking as if we had filtered the
entire set at once and then passed it to the object, but without actually fitting everything in
memory.
Simplifying code through iterators
Now, we will briefly discuss some situations that can be improved with the help of
iterators, and occasionally the itertools module. After discussing each case, and its
proposed optimization, we will close each point with a corollary.
[ 197 ]Using Generators
Chapter 7
Repeated iterations
Now that we have seen more about iterators, and introduced the itertools module, we
can show you how one of the first examples of this chapter (the one for computing statistics
about some purchases), can be dramatically simplified:
def process_purchases(purchases):
min_, max_, avg = itertools.tee(purchases, 3)
return min(min_), max(max_), median(avg)
In this example, itertools.tee will split the original iterable into three new ones. We
will use each of these for the different kinds of iterations that we require, without needing
to repeat three different loops over purchases .
The reader can simply verify that if we pass an iterable object as the purchases parameter,
this one is traversed only once (thanks to the itertools.tee function [see references]),
which was our main requirement. It is also possible to verify how this version is equivalent
to our original implementation. In this case, there is no need to manually raise ValueError
because passing an empty sequence to the min() function will do the same.
If you are thinking about running a loop over the same object more than
one time, stop and think if itertools.tee can be of any help.
Nested loops
In some situations, we need to iterate over more than one dimension, looking for a value,
and nested loops come as the first idea. When the value is found, we need to stop iterating,
but the break keyword doesn't work entirely because we have to escape from two (or
more) for loops, not just one.
What would be the solution for this? A flag signaling escape? No. Raising an exception?
No, this would be the same as the flag, but even worse because we know that exceptions
are not to be used for control flow logic. Moving the code to a smaller function and return
it? Close, but not quite.
The answer is, whenever possible, flat the iteration to a single for loop.
This is the kind of code we would like to avoid:
def search_nested_bad(array, desired_value):
coords = None
for i, row in enumerate(array):
[ 198 ]Using Generators
Chapter 7
for j, cell in enumerate(row):
if cell == desired_value:
coords = (i, j)
break
if coords is not None:
break
if coords is None:
raise ValueError(f"{desired_value} not found")
logger.info("value %r found at [%i, %i]", desired_value, *coords)
return coords
And here is a simplified version of it that does not rely on flags to signal termination, and
has a simpler, more compact structure of iteration:
def _iterate_array2d(array2d):
for i, row in enumerate(array2d):
for j, cell in enumerate(row):
yield (i, j), cell
def search_nested(array, desired_value):
try:
coord = next(
coord
for (coord, cell) in _iterate_array2d(array)
if cell == desired_value
)
except StopIteration:
raise ValueError("{desired_value} not found")
logger.info("value %r found at [%i, %i]", desired_value, *coord)
return coord
It's worth mentioning how the auxiliary generator that was created works as an abstraction
for the iteration that's required. In this case, we just need to iterate over two dimensions,
but if we needed more, a different object could handle this without the client needing to
know about it. This is the essence of the iterator design pattern, which, in Python, is
transparent, since it supports iterator objects automatically, which is the topic covered in
the next section.
Try to simplify the iteration as much as possible with as many
abstractions as are required, flatting the loops whenever possible.
[ 199 ]Using Generators
Chapter 7
The iterator pattern in Python
Here, we will take a small detour from generators to understand iteration in Python more
deeply. Generators are a particular case of iterable objects, but iteration in Python goes
beyond generators, and being able to create good iterable objects will give us the chance to
create more efficient, compact, and readable code.
In the previous code listings, we have been seeing examples of iterable objects that are
also iterators, because they implement both the __iter__() and __next__() magic
methods. While this is fine in general, it's not strictly required that they always have to
implement both methods, and here we'll show the subtle differences between
an iterable object (one that implements __iter__ ) and an iterator (that
implements __next__ ).
We also explore other topics related to iterations, such as sequences and container objects.
The interface for iteration
An iterable is an object that supports iteration, which, at a very high level, means that we
can run a for .. in ... loop over it, and it will work without any issues.
However, iterable does not mean the same as iterator.
Generally speaking, an iterable is just something we can iterate, and it uses an iterator to do
so. This means that in the __iter__ magic method, we would like to return an iterator,
namely, an object with a __next__() method implemented.
An iterator is an object that only knows how to produce a series of values, one at a time,
when it's being called by the already explored built-in next() function. While the iterator
is not called, it's simply frozen, sitting idly by until it's called again for the next value to
produce. In this sense, generators are iterators.
Python concept Magic method
Iterable
Iterator
Considerations
__iter__ They work with an iterator to construct the iteration logic.
These objects can be iterated in a for ... in ...: loop
__next__ Define the logic for producing values one at the time.
The StopIteration exception signals that the iteration is
over.
The values can be obtained one by one via the built-in next()
function.
[ 200 ]Using Generators
Chapter 7
In the following code, we will see an example of an iterator object that is not iterable—it
only supports invoking its values, one at a time. Here, the name sequence refers just to a
series of consecutive numbers, not to the sequence concept in Python, which will we
explore later on:
class SequenceIterator:
def __init__(self, start=0, step=1):
self.current = start
self.step = step
def __next__(self):
value = self.current
self.current += self.step
return value
Notice that we can get the values of the sequence one at a time, but we can't iterate over this
object (this is fortunate because it would otherwise result in an endless loop):
>>> si = SequenceIterator(1, 2)
>>> next(si)
1
>>> next(si)
3
>>> next(si)
5
>>> for _ in SequenceIterator(): pass
...
Traceback (most recent call last):
...
TypeError: 'SequenceIterator' object is not iterable
The error message is clear, as the object doesn't implement __iter__() .
Just for explanatory purposes, we can separate the iteration in another object (again, it
would be enough to make the object implement both __iter__ and __next__ , but doing
so separately will help clarify the distinctive point we're trying to make in this explanation).
Sequence objects as iterables
As we have just seen, if an object implements the __iter__() magic method, it means it
can be used in a for loop. While this is a great feature, it's not the only possible form of
iteration we can achieve. When we write a for loop, Python will try to see if the object
we're using implements __iter__ , and, if it does, it will use that to construct the iteration,
but if it doesn't, there are fallback options.
[ 201 ]Using Generators
Chapter 7
If the object happens to be a sequence (meaning that it implements __getitem__()
and __len__() magic methods), it can also be iterated. If that is the case, the interpreter
will then provide values in sequence, until the IndexError exception is raised, which,
analogous to the aforementioned StopIteration , also signals the stop for the iteration.
With the sole purpose of illustrating such a behavior, we run the following experiment that
shows a sequence object that implements map() over a range of numbers:
# generators_iteration_2.py
class MappedRange:
"""Apply a transformation to a range of numbers."""
def __init__(self, transformation, start, end):
self._transformation = transformation
self._wrapped = range(start, end)
def __getitem__(self, index):
value = self._wrapped.__getitem__(index)
result = self._transformation(value)
logger.info("Index %d: %s", index, result)
return result
def __len__(self):
return len(self._wrapped)
Keep in mind that this example is only designed to illustrate that an object such as this one
can be iterated with a regular for loop. There is a logging line placed in the __getitem__
method to explore what values are passed while the object is being iterated, as we can see
from the following test:
>>> mr = MappedRange(abs, -10, 5)
>>> mr[0]
Index 0: 10
10
>>> mr[-1]
Index -1: 4
4
>>> list(mr)
Index 0: 10
Index 1: 9
Index 2: 8
Index 3: 7
Index 4: 6
Index 5: 5
Index 6: 4
Index 7: 3
[ 202 ]Using Generators
Chapter 7
Index 8: 2
Index 9: 1
Index 10: 0
Index 11: 1
Index 12: 2
Index 13: 3
Index 14: 4
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0, 1, 2, 3, 4]
As a word of caution, it's important to highlight that while it is useful to know this, it's also
a fallback mechanism for when the object doesn't implement __iter__ , so most of the time
we'll want to resort to these methods by thinking in creating proper sequences, and not just
objects we want to iterate over.
When thinking about designing an object for iteration, favor a proper
iterable object (with __iter__ ), rather than a sequence that can
coincidentally also be iterated.
