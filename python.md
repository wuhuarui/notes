## advanced note

### tree

`root node`, `branch`, `leaf`, `child`, `parent`...

[`Binary tree`](http://www.xuetangx.com/asset-v1:MITx+6_00_1x+sp+type@asset+block/treeSearch.py) 二叉树

[`Decision Tree`](http://www.xuetangx.com/courses/course-v1:MITx+6_00_1x+sp/courseware/Week_10/videosequence:Lecture_19/) 决策树

`Explicit search` 显式搜索; `Implicit search` 隐式搜索. 

[Depth-search first](https://en.wikipedia.org/wiki/Depth-first_search); [Breadth-first search](https://en.wikipedia.org/wiki/Breadth-first_search)



## commands

### Text rendering With LaTeX

using `r"$latex text$"`

### lambda

[python docs](https://docs.python.org/3/tutorial/controlflow.html)

Small anonymous functions can be created with the lambda keyword. This function returns the sum of its two arguments: lambda a, b: a+b.

返回一个函数类型的对象. 支持简单的匿名函数

```python
>>> def make_incrementor(n):
...     return lambda x: x + n
...
>>> f = make_incrementor(42)
>>> f(0)
42
>>> f(1)
43
```

### time

```python
import time
ts = time.time()
# some code here
print('time cost: {:.0f} secs'.format(time.time()-ts))
```

### matplotlib toolkit

```python
from matplotlib import pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
```

[text properties](https://matplotlib.org/api/text_api.html#matplotlib.text.Text)

#### savefig

```
savefig(fname, dpi=None, facecolor='w', edgecolor='w',
        orientation='portrait', papertype=None, format=None,
        transparent=False, bbox_inches=None, pad_inches=0.1,
        frameon=None)
```

Adjusting `dpi` to change the size of figure, adjusting `transparent` to remove the background pad.

`fig.savefig(SUMMARY_PATH + 'Scatter_vs_Stop.png', transparent=True, dpi=300)`

#### [rcParam](https://matplotlib.org/users/customizing.html)

```python
plt.rcParams['figure.figsize'] = [12 , 8]
plt.rcParams['figure.dpi'] = 72.0*2
```

#### [Pie chart](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.pie.html)

```python
sizes = [1,3,4,5,6,7,8] # percent of different parts, don't need to be normalized
explode = [0,0,0,0,0.1,0.1,0.1,0.1]
fig1, ax1 = plt.subplots()
ax1.pie(sizes, explode=explode, labels=labels, autopct='%1.2f%%',
        shadow=True, startangle=180)
ax1.axis('equal')  # Equal aspect ratio ensures that pie is drawn as a circle.
plt.title("$R59\ of\ ^{40}K\ in\ region\ of\ %.2fkeV$" % (peak[idxPeak]['mu']*1000.))
plt.show()
```

The fractional area of each wedge is given by x/sum(x). If sum(x) <= 1, then the values of x give the fractional area directly and the array will not be normalized.

If you want to show the actual value(rather than percentage), using `autopct` with some simple function.

autopct : None (default), string, or function, optional

If not None, is a string or function used to label the wedges with their numeric value. The label will be placed inside the wedge. If it is a format string, the label will be fmt%pct. If it is a function, it will be called.

The following example shows how to display actual value with `autopct`. 

```python
sizes = [1,2,3,4]
total = 10
fig1, ax1 = plt.subplots()
ax1.pie(sizes, labels=['a','b','c','d'], autopct=lambda x:'{:.0f}'.format(x*total/100), # x seems to be the percentage value
        shadow=True, startangle=180)
ax1.axis('equal') 
plt.show()
```

#### [stacked bar chart](https://matplotlib.org/examples/pylab_examples/bar_stacked.html)

```python
idx_material = [6, 4, 2, 5, 7, 10]
ind = range(len(idx_material))    # the x locations for the groups
width = 0.4          # the width of the bars: can also be len(x) sequence
ind1 = np.arange(-width/2., len(idx_material)-width/2.,1) # the middle postion of front part, minus width/2
ind2 = np.arange(width/2., len(idx_material)+width/2.,1)  # the middle postion of back part, plus width/2

fig, ax = plt.subplots()
fig.clf
p1 = ax.bar(ind1, nEStopped, width)
p2 = ax.bar(ind1, nEScatter, width, bottom = nEStopped)
p3 = ax.bar(ind2, nMStopped, width)
p4 = ax.bar(ind2, nMScatter, width, bottom = nMStopped)

ax.set_xlabel('material')
ax.set_ylabel('counts')
plt.xticks(ind, label)
plt.legend((p1[0], p2[0], p3[0], p4[0]), 
           ('e- stopped', 'e- scattered' , 'mu stopped','mu scattered'))
plt.show()
```

#### legend

```python
# examples to add legends in a plor
# method1
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [1, 2, 3],'ro')
ax.legend(['A simple line'],loc='upper center', shadow=True, fontsize='x-large')

# method 2
ax.plot([1, 2, 3], [1, 2, 3],'ro',label='A simple line')
ax.legend(loc='upper center', shadow=True, fontsize='x-large')

# change label
line, = ax.plot([1, 2, 3], [1, 2, 3], label='Inline label')
# Overwrite the label by calling the method.
line.set_label('Label via method')
ax.legend(loc='upper center', shadow=True, fontsize='x-large')

#method 3
plt.plot([1, 2, 3], [1, 2, 3],'ro',label='A simple line')
plt.legend(['A simple line'],loc='upper center', shadow=True, fontsize='x-large')
```

The arguments of `legend` should be a list of string rather than string, otherwise, the legend is only the first character of the string. For example: `plt.legend('A simple line')`, the plotted legend is just `A`.

#### subplots

`fig, ax = plt.subplots()` returns a [`matplotlib.figure.Figure` instance](https://matplotlib.org/api/_as_gen/matplotlib.figure.Figure.html#matplotlib.figure.Figure) and a [matplotlib.axes.Axes instance](https://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes). The get and set methods of the two instances are in the form of `fig.set_***` and `ax.get_***`.

fig : matplotlib.figure.Figure object

ax : Axes object (`fig, ax = plt.subplots()`) or array of Axes objects (`fig, ax = plt.subplots(2, 2)`).

```python
# another to add subplot and the Axes objects
fig = plt.figure()
ax = fig.add_subplot(111)

# equivalent but more general
fig.add_subplot(1,1,1)
```

Common commands:

```python
ax.legend([lsit of legend])
ax.set_xticks([list of xtick position])
ax.set_xticklabels([list of xtick label])
ax.set_title('title')
ax.set_xscale('log')
ax.set_xlabel('xlabel', **kwargs) # **kwargs : Text properties, for example: fontsize='large

fig.set_figheight(8.)
fig.set_figwidth(12.)
fig.set_dpi(144)
fig.savefig("/media/pxy10/Documents/G4Simulation/Argus/analysis/figures/CuShield.png",
            dpi = 288, transparent=True)
```

**`plt.tight_layout()` will adjust spacing between subplots to minimize the overlaps.**

#### [annotation](https://matplotlib.org/users/annotations.html)

#### [two scales](https://matplotlib.org/examples/api/two_scales.html)

```python
"""
===========================
Plots with different scales
===========================

Demonstrate how to do two plots on the same axes with different left and
right scales.

The trick is to use *two different axes* that share the same *x* axis.
You can use separate `matplotlib.ticker` formatters and locators as
desired since the two axes are independent.

Such axes are generated by calling the `Axes.twinx` method.  Likewise,
`Axes.twiny` is available to generate axes that share a *y* axis but
have different top and bottom scales.

The twinx and twiny methods are also exposed as pyplot functions.

"""

import numpy as np
import matplotlib.pyplot as plt

fig, ax1 = plt.subplots()
t = np.arange(0.01, 10.0, 0.01)
s1 = np.exp(t)
ax1.plot(t, s1, 'b-')
ax1.set_xlabel('time (s)')
# Make the y-axis label, ticks and tick labels match the line color.
ax1.set_ylabel('exp', color='b')
ax1.tick_params('y', colors='b')

ax2 = ax1.twinx()
s2 = np.sin(2 * np.pi * t)
ax2.plot(t, s2, 'r.')
ax2.set_ylabel('sin', color='r')
ax2.tick_params('y', colors='r')

fig.tight_layout()
plt.show()
```

### random

#### random.choice(seq)

Return a random element from the non-empty sequence seq. If seq is empty, raises IndexError.

### [neumerate(iterable, start=0)](https://docs.python.org/3/library/functions.html#enumerate)

Return an enumerate object. iterable must be a sequence, an iterator, or some other object which supports iteration. The `__next__()` method of the iterator returned by enumerate() returns a tuple containing a count (from start which defaults to 0) and the values obtained from iterating over iterable. Looks like a generator.

### [isinstance](https://docs.python.org/2/library/functions.html#isinstance)

`isinstance(object, classinfo)` return `True` if `object` is an instance of `classinfo`.

### enumetrate

### swap

```python
a, b = 1, 2
a, b = b, a #short form of swap
```

### [string](https://docs.python.org/2/library/string.html)

#### string.join

```python
a = [1,2,3,4,5,6]
print(a)
# [1, 2, 3, 4, 5, 6]
print('{' + ', '.join([str(e) for e in a]) + '}')
# {1, 2, 3, 4, 5, 6}
```

#### string.spilt(s\[, sep\[, maxsplit\]\])

Return a list of the words of the string s. If the optional second argument `sep` is absent or `None`, the words are separated by arbitrary strings of whitespace characters (space, tab, newline, return, formfeed). If the second argument `sep` is present and not None, it specifies a string to be used as the word separator. The returned list will then have one more item than the number of non-overlapping occurrences of the separator in the string. If maxsplit is given, at most maxsplit number of splits occur, and the remainder of the string is returned as the final element of the list (thus, the list will have at most maxsplit+1 elements). If maxsplit is not specified or -1, then there is no limit on the number of splits (all possible splits are made).

`'# dfdf  sdfas t *'.spilt()` returns `['#', 'dfdf', 'sdfas', 't', '*']`

`'# dfdf  sdfas t *'.spilt(' ')` returns `['#', 'dfdf', '', 'sdfas', 't', '*']`

`'# dfdf  sdfas t *'.spilt(None,3)` returns `['#', 'dfdf', 'sdfas', 't *']`

#### string.strip(s \[,chars\])

Return a copy of string s, whose leading and trailing characters removed. `chars` is a string, if `chars` is omitted or `None`, whitespace(including space, tab, etc.) will be removed. If given, remove the character in `chars`. 只有前后两端的字符会去除, 中间的字符不变. `" string test ".strip()` returns `"string test"`; `" #string* test* ".strip("#* ")` returns `"string* test"`.

#### string.replace(s, old, new\[, maxreplace\])

Return a copy of string s with all occurrences of substring old replaced by new. If the optional argument maxreplace is given, the first maxreplace occurrences are replaced.

`'# dfdf  sdfas t *'.replace(' ','')` returns `#dfdfsdfast*`

### class

```python
import math

def sq(x):
    return x*x

# object is the base class, a built_in class of python
class Coordinate(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def getX(self):
        # Getter method for a Coordinate object's x coordinate.
        # Getter methods are better practice than just accessing an attribute directly
        return self.x

    def getY(self):
        # Getter method for a Coordinate object's y coordinate
        return self.y

    def __str__(self):
        return "<"+str(self.x)+","+str(self.y)+">"

    def __eq__(self, other):
        # First make sure `other` is of the same type 
        assert type(other) == type(self)
        # Since `other` is the same type, test if coordinates are equal
        return self.getX() == other.getX() and self.getY() == other.getY()

    def distance(self,other):
        return math.sqrt(sq(self.x - other.x)
                         + sq(self.y - other.y))

    def __repr__(self):
        return 'Coordinate(' + str(self.x) + ', ' + str(self.y) + ')'

c = Coordinate(3,4)
Origin = Coordinate(0,0)
```

`__str__`: Called by str(object) and the built-in functions format() and print() to compute the “informal” or nicely printable string representation of an object. The return value must be a string object.

`__eq__`: implementation of operator `=`

`__repr__`: Called by the repr() built-in function to compute the “official” string representation of an object. It shows how to create an object.

`__lt__`: implementation of operator `<`, with this method implemented, one can use the method of `sort()`. Here `obj2 < obj1` is equal to `obj2.__lt__(obj1)`

#### inheritance subclass

```python
import datetime

class Person(object):
    def __init__(self, name):
        """create a person called name"""
        self.name = name
        self.birthday = None
        self.lastName = name.split(' ')[-1]

    def getLastName(self):
        """return self's last name"""
        return self.lastName

    def setBirthday(self,month,day,year):
        """sets self's birthday to birthDate"""
        self.birthday = datetime.date(year,month,day)

    def getAge(self):
        """returns self's current age in days"""
        if self.birthday == None:
            raise ValueError
        return (datetime.date.today() - self.birthday).days

    def __lt__(self, other):
        """return True if self's ame is lexicographically
           less than other's name, and False otherwise"""
        if self.lastName == other.lastName:
            return self.name < other.name
        return self.lastName < other.lastName

    def __str__(self):
        """return self's name"""
        return self.name

# me = Person("William Eric Grimson")
# her = Person("Cher")
# her.getLastName()
# plist = [me, her]
# for p in plist: print p
# plist.sort() # sort according to the method of `__lt__`
# for p in plist: print p

class MITPerson(Person): # inheritance subclass, inherited from the class of Person
    nextIdNum = 0 # next ID number to assign, belong to the whole class rather than a single object

    def __init__(self, name):
        Person.__init__(self, name) # initialize Person attributes
        # new MITPerson attribute: a unique ID number
        self.idNum = MITPerson.nextIdNum
        MITPerson.nextIdNum += 1

    def getIdNum(self):
        return self.idNum

    # sorting MIT people uses their ID number, not name!
    def __lt__(self, other):
        return self.idNum < other.idNum

p1 = MITPerson('Eric')
p2 = MITPerson('John')
p3 = MITPerson('John')
p4 = Person('John')

p4 < p1 # the comparision is valid, equal to p4.__lt__(p1), compare according to the method of Person class
p1 < p4 # the comparision is not valid, equal to p1.__lt__(p4), p1 is an object of MITPerson, the comparision should use the MITPerson method, but p4 don't have idNum.
```

#### hierarchy(层级)

接上

```python
class Student(MITPerson):
        pass

class UG(Student):
        def __init__(self, name, classYear):
                MITPerson.__init__(self, name)
                self.year = classYear
        
        def getClass(self):
                return selt.year

class Grad(Student):
        pass

class TransferStudent(Student):
        pass

def isStudent(obj):
        return isinstance(obj, Student)
```

The method `pass` means the subclass is just as same as superclass, no changes was made.

If one subclass wants to inherit from two superclasses,(multiple inheritance) use `class C(A, B)`. In this case, if there are same methods in class A and class B, named `sameMethod`, then create an instance of C, named `c`. With `c.sameMethod()`, the method in class A will be called. 简单来说就是C继承A和B, C中的方法和变量的调用过程中会依次从左往右, 从subclass(也就是C自身)到superclass(如果A,B还有母类,继续往上追溯), 在这个遍历的过程中, 第一次出现的方法或变量就是实际调用到的. 因此如果A, B中都含有`sameMethod`, 由于A先遍历, 会调用A中的方法.

### generator

Any procedure or method with a `yield` statement is called a generator.

```python
def genTest():
yield 1
yield 2
```

Generators have a `next()` method which starts/resumes execution of the procedure. Inside of generator:

- yield suspends execution and returns a value

- Returning from a generator raises a StopIteration exception

```python
foo = genTest()
foo.next() # return 1
foo.next() # return 2
foo.next() # Results in a StopIteration exception

for n in genTest():
        print(n)
# 1
# 2
```

```python
# a fancier example:
def genFib():
        fibn_1 = 1 #fib(n-1)
        fibn_2 = 0 #fib(n-2)
        while True:
                # fib(n) = fib(n-1) + fib(n-2)
                next = fibn_1 + fibn_2
                yield next
                fibn_2 = fibn_1
                fibn_1 = next
```

Generator allows one to generate each new objects as needed as part of another computation (rather than computing a very long sequence, only to throw most of it away while you do something on an element, then repeating the process). 例如有时产生一个sequence, 但是其len很大, 如果在使用的时候我们只关心其中的某个元素, 不放设置为generator, 否则返回一个很长的sequence, 占用内存较多且效率较低.

```python
def allStudents(self):
        if not self.isSorted:
                self.students.sort()
                self.isSorted = True
        return self.students[:]
#return copy of list of students

def allStudents(self):
        if not self.isSorted:
                self.students.sort()
                self.isSorted = True
        for s in self.students:
                yield s
```

syntax changes: python 2 `gen.next()`; python 3 `next(gen)`


### print

python2: `print a`

python3: `print(a)`

Print elements in one line: 

```python
dic = {'g': 5}
print('a', dic)
# only works in python3.X
print('a', end='')
print(' same line')
```

`print([object, ...], *, sep=' ', end='\n', file=sys.stdout)` python 3.x

### random

`import random`

`random.randint(a, b)` return random integer in the range of [a, b], endpoint is included;

`random.randrange(a, b[, step=1])` return random value in the range of \[a, b) with the step, endpoint is not included. a, b, step are in the type of integer.

### input

python2: raw_input()
python3: input()

接收用户输入的内容, 存储为字符串

```python
y = float(raw_input('Enter a number: '))
x = raw_input('Enter your name: ')
```

### assert

assert: same as C++
`assert type(x) == int and x>=0, 'x should be an integer bigger than 0'`

the first part `type(x) == int and x>=0` is the condition of this assertion.

the second part `'x should be an integer bigger than 0'` is the output message if the condition is `False`.


add comments of a function:
```python
def func1(x, y)；
“““this function is used to...;
    input: x should be an integer; y should be a float”””
   return x + y
```

### global

全局变量, 通过global声明, 在每个使用了该变量的函数内部都需要用global声明一下

```python
def fibMetered(x):
    global numCalls
    numCalls += 1
    if x == 0 or x == 1:
        return 1
    else:
        return fibMetered(x-1) + fibMetered(x-2)


def testFib(n):
    for i in range(n+1):
        global numCalls
        numCalls = 0
        print('fib of ' + str(i) + ' = ' + str(fibMetered(i)))
        print('fib called ' + str(numCalls) + ' times')
```

### tuples

In python, tuples can be defined with `(element1, element2...)`.

```python
# create a new tuple
t1 = (1, 'two', 3)
t2 = (t1, 'four')
t3 = ('five',) # the "," is used to show that 't3' is a tuple with one single element. If not defined with ",", t3 will be treated as a string
t4 =() # the empty tuple

# To add an element or a new tuple to a existing tuple, "+" can be used.
t3 = t1 + t2
t4 = t3 + (1,)

# iterating in a tuple
sum = 0
for i in t1；
    sum += i
```

### lists

difference between lists and tuples:

+ 'tuple' object does not support item assignment, which means the elements in tuple can not be changed.

+ `list = [1, 2, 4]`, list object uses "[]" rather than "()";

+ list with one single element: `list = [3]`, donot like what we did with tuple `tuple = (3,)`

Opreation "+" can help combine two lists. `newlist = list1 + list2`

```python
# make a clone of a list
Clone = list[:] # Clone is the copy of list, once it's created, it has no relationship with "list"
SameList = list # SameList is equal to list
```

**One can use function object as the element of list or the formal parameter of a function**

```python
def applyFuns(L, x):
    for f in L:
        print(f(x))

applyFuns([abs, int, fact, fib], 4)
```

#### list.sort

syntaxof sort() method: `list.sort(key=..., reverse=...)` or `sorted(list, key=..., reverse=...)`

Simplest difference between sort() and sorted() is: sort() doesn't return any value while, sorted() returns an iterable list. Another difference is that the list.sort() method is only defined for lists. **In contrast, the sorted() function accepts any iterable.**

`key` is a function object: function that serves as a key for the sort comparison.

```python
# take second element for sort
def takeSecond(elem):
    return elem[1]

# random list
random = [(2, 2), (3, 4), (4, 1), (1, 3)]

# sort list with key
random.sort(key=takeSecond)
```

Actually, an easier way to implement the method is:

```python
random.sort(key=lambda x:x[1])
```

#### zip and sort one list by another

```python
X = ["a", "b", "c", "d", "e", "f", "g", "h", "i"]
Y = [ 0,   1,   1,    0,   1,   2,   2,   0,   1]
Z = [x for x,_ in sorted(zip(X,Y), key=lambda x:x[1])]
```

[zip](https://docs.python.org/3/library/functions.html#zip), `zip` makes an iterator that aggregates elements from each of the iterables.

`for x,_ in sorted(zip(X,Y), key=lambda x:x[1])`, here `_` represents the second element. Since the element won't be used, using `_` rather than a variable.

### map

`Map` applies a function to all the items in an input_list: `map(function_to_apply, list_of_inputs)`

```python
items = [1, 2, 3, 4, 5]
squared = []
for i in items:
    squared.append(i**2)
# same code with "map"
items = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, items)) # "lamda x: x**2" is a function object

#  Instead of a list of inputs we can even have a list of functions!
def multiply(x):
    return (x*x)
def add(x):
    return (x+x)

funcs = [multiply, add]
for i in range(5):
    value = list(map(lambda x: x(i), funcs))
    print(value)

# Output:
# [0, 0]
# [1, 2]
# [4, 4]
# [9, 6]
# [16, 8]

# General form: an n-ary function and n collections of arguments
L1 = [1, 28, 36]
L2 = [2, 57, 9]
map(min, L1, L2)
```

### dictionaries

syntax: `dic = {'key1': value1, 'key2': value2, ...}`

```python
monthNumber = {1: 'Jan', 2: 'Feb', 'Feb': 2, 'Jan': 1}
# insertion
monthNumer['Mar'] = 3
# iteration
listofkey = []
for key in monthNumnber: # return the key inside the dictionary
    listofkey.append(key)
# listofkey is the same as
monthNumber.keys()
```

```python
def getFrequencyDict(sequence):
    """
    Returns a dictionary where the keys are elements of the sequence
    and the values are integer counts, for the number of times that
    an element is repeated in the sequence.

    sequence: string or list
    return: dictionary
    """
    # freqs: dictionary (element_type -> int)
    freq = {}
    for x in sequence:
        freq[x] = freq.get(x,0) + 1
    return freq
```

`dic.get(k[, d])` if k is a key of dictionary, return dic[k], else return d. The default value of d is None.

Keys in a dictionary can not be changed, but the value can be changed. In this way, a tuple can be applied as a key, but a list cannot.

`dic.copy()` return the copy of a dictionary.

`pop(key[, default])` If key is in the dictionary, remove it and return its value, else return default. If default is not given and key is not in the dictionary, a KeyError is raised.


### exception syntax

using `try`, `except`, `finally`, `raise`.

```python
def getRatios(v1, v2):
    """Assumes: v1 and v2 are lists of equal length of numbers
       Returns a list containing the meaningful values of v1[i]/v2[i]"""
    ratios = []
    for index in range(len(v1)):
        try:
            ratios.append(v1[index]/float(v2[index]))
        except ZeroDivisionError:
            ratios.append(float('NaN'))
        except:
            raise ValueError('getRatios called with bad arg...')
    return ratios
    

try:
    print(getRatios([1.,2.,7.,6.], [1.,2.,0.,3.]))
    print(getRatios([],[]))
    print(getRatios([1.,2.],[3.]))
except ValueError as e:
    print(e)
 ```
 
 
 
 ### with...as
 
 这个语法是用来代替传统的try...finally语法的。 
 
 with EXPRESSION [ as VARIABLE] WITH-BLOCK 
 
 基本思想是with所求值的对象必须有一个__enter__()方法，一个__exit__()方法。
 
 紧跟with后面的语句被求值后，返回对象的__enter__()方法被调用，这个方法的返回值将被赋值给as后面的变量。当with后面的代码块全部被执行完之后，将调用前面返回对象的__exit__()方法。
 
 ```python
 file = open("/tmp/foo.txt")
 try:
     data = file.read()
 finally:
     file.close()
 ```
 
 使用with...as的方式:
 
 ```python
 with open("/tmp/foo.txt") as file:  
     data = file.read()  
 ```


## Modules

One can group some parameters and functions into a single .py file, for example test.py. Then this file can be imported to the environment wherever you want to use the parameters or functions.
```python
import os
os.chdir("the directory of test.py")
# example1 of import
import test # import test.py, the objects inside test.py can be accessed via test.func1()

# example2 of import
from test import * # import all of the objects inside test.py, the objects can be used directly.

# example3 of import
from test import func1, func2 # import part of the objects inside test.py, the objects can be used directly.
```
