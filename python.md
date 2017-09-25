## commands

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
`assert type(x) == int and x>=0`

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
t1 = (1, 'two', 3)
t2 = (t1, 'four')
t3 = ('five',) # the "," is used to show that 't3' is a tuple with one single element. If not defined with ",", t3 will be treated as a string
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
