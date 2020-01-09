1. Creating generators
**********************

Generators were introduced in Python a long time ago (PEP-255), with the idea of
introducing iteration in Python while improving the performance of the program (by using
less memory) at the same time.
The idea of a generator is to create an object that is iterable, and, while it's being iterated,
will produce the elements it contains, one at a time. The main use of generators is to save
memory—instead of having a very large list of elements in memory, holding everything at
once, we have an object that knows how to produce each particular element, one at a time,
as they are required.
This feature enables lazy computations or heavyweight objects in memory, in a similar
manner to what other functional programming languages (Haskell, for instance) provide. It
would even be possible to work with infinite sequences because the lazy nature of
generators allows for such an option.
A first look at generators
Let's start with an example. The problem at hand now is that we want to process a large list
of records and get some metrics and indicators over them. Given a large data set with
information about purchases, we want to process it in order to get the lowest sale, highest
sale, and the average price of a sale.
For the simplicity of this example, we will assume a CSV with only two fields, in the
following format:
<purchase_date>, <price>
...
We are going to create an object that receives all the purchases, and this will give us the
necessary metrics. We could get some of these values out of the box by simply using the
min() and max() built-in functions, but that would require iterating all of the purchases
more than once, so instead, we are using our custom object, which will get these values in a
single iteration.
[ 189 ]Using Generators
Chapter 7
The code that will get the numbers for us looks rather simple. It's just an object with a
method that will process all prices in one go, and, at each step, will update the value of each
particular metric we are interested in. First, we will show the first implementation in the
following listing, and, later on in this chapter (once we have seen more about iteration), we
will revisit this implementation and get a much better (and compact) version of it. For now,
we are settling on the following:
class PurchasesStats:
def __init__(self, purchases):
self.purchases = iter(purchases)
self.min_price: float = None
self.max_price: float = None
self._total_purchases_price: float = 0.0
self._total_purchases = 0
self._initialize()
def _initialize(self):
try:
first_value = next(self.purchases)
except StopIteration:
raise ValueError("no values provided")
self.min_price = self.max_price = first_value
self._update_avg(first_value)
def process(self):
for purchase_value in self.purchases:
self._update_min(purchase_value)
self._update_max(purchase_value)
self._update_avg(purchase_value)
return self
def _update_min(self, new_value: float):
if new_value < self.min_price:
self.min_price = new_value
def _update_max(self, new_value: float):
if new_value > self.max_price:
self.max_price = new_value
@property
def avg_price(self):
return self._total_purchases_price / self._total_purchases
def _update_avg(self, new_value: float):
self._total_purchases_price += new_value
[ 190 ]Using Generators
Chapter 7
self._total_purchases += 1
def __str__(self):
return (
f"{self.__class__.__name__}({self.min_price}, "
f"{self.max_price}, {self.avg_price})"
)
This object will receive all the totals for the purchases and process the required values.
Now, we need a function that loads these numbers into something that this object can
process. Here is the first version:
def _load_purchases(filename):
purchases = []
with open(filename) as f:
for line in f:
*_, price_raw = line.partition(",")
purchases.append(float(price_raw))
return purchases
This code works; it loads all the numbers of the file into a list that, when passed to our
custom object, will produce the numbers we want. It has a performance issue, though. If
you run it with a rather large dataset, it will take a while to complete, and it might even fail
if the dataset is large enough as to not fit into the main memory.
If we take a look at our code that consumes this data, it is processing the purchases , one at
a time, so we might be wondering why our producer fits everything in memory at once. It
is creating a list where it puts all of the content of the file, but we know we can do better.
The solution is to create a generator. Instead of loading the entire content of the file in a list,
we will produce the results one at a time. The code will now look like this:
def load_purchases(filename):
with open(filename) as f:
for line in f:
*_, price_raw = line.partition(",")
yield float(price_raw)
If you measure the process this time, you will notice that the usage of memory has dropped
significantly. We can also see how the code looks simpler—there is no need to define the
list (therefore, there is no need to append to it), and that the return statement also
disappeared.
In this case, the load_purchases function is a generator function, or simply a generator.
[ 191 ]Using Generators
Chapter 7
In Python, the mere presence of the keyword yield in any function makes it a generator,
and, as a result, when calling it, nothing other than creating an instance of the generator
will happen:
>>> load_purchases("file")
<generator object load_purchases at 0x...>
A generator object is an iterable (we will revisit iterables in more detail later on), which
means that it can work with for loops. Notice how we did not have to change anything on
the consumer code—our statistics processor remained the same, with the for loop
unmodified, after the new implementation.
Working with iterables allows us to create these kinds of powerful abstractions that are
polymorphic with respect to for loops. As long as we keep the iterable interface, we can
iterate over that object transparently.
Generator expressions
Generators save a lot of memory, and since they are iterators, they are a convenient
alternative to other iterables or containers that require more space in memory such as lists,
tuples, or sets.
Much like these data structures, they can also be defined by comprehension, only that it is
called a generator expression (there is an ongoing argument about whether they should be
called generator comprehensions. In this book, we will just refer to them by their canonical
name, but feel free to use whichever you prefer).
In the same way, we would define a list comprehension. If we replace the square brackets
with parenthesis, we get a generator that results from the expression. Generator
expressions can also be passed directly to functions that work with iterables, such as sum() ,
and, max() :
>>> [x**2 for x in range(10)]
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> (x**2 for x in range(10))
<generator object <genexpr> at 0x...>
>>> sum(x**2 for x in range(10))
285
[ 192 ]Using Generators
Chapter 7
Always pass a generator expression, instead of a list comprehension, to
functions that expect iterables, such as min() , max() , and sum() . This is
more efficient and pythonic.