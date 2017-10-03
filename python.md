## commands

### matplotlib toolkit

```python
from matplotlib import pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
```

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

### [isinstance](https://docs.python.org/2/library/functions.html#isinstance)

`isinstance(object, classinfo)` return `True` if `object` is an instance of `classinfo`.


### swap

```python
a, b = 1, 2
a, b = b, a #short form of swap
```

### string.join

```python
a = [1,2,3,4,5,6]
print(a)
# [1, 2, 3, 4, 5, 6]
print('{' + ', '.join([str(e) for e in a]) + '}')
# {1, 2, 3, 4, 5, 6}
```


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
