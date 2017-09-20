#	C++ Programing Notes


## const

const: 用于既需要共享, 又要防止改变的数据. 包括:

1. 常对象; 必须进行初始化, 不能改变其中的内容. `const 类名 对象名(实参表)`, 例如: `const classA a(2,3);`
2. 常成员;
   - 常成员函数:  声明格式: `类型说明符 函数名(参数表) const;`, 如`void print() const`; 通过常对象只能调用它的常成员函数. `const`可以作为重载函数的区分. 对于非常对象也可以调用常成员函数, 不过优先调用非常成员函数的实现.
   - 常数据成员: 通过构造函数的初始化列表给对象的常数据成员赋初值, 之后貌似不能再改变. 常静态数据成员: `static const int b;`, 初始化与类的静态数据成员初始化相同, 在类外通过(类名::)初始化, 不能改变.
3. 常引用, 常用于作形参, 常引用所引用的对象不能被更新, 因此用做形参时不会意外地发生对实参的更改, 其声明形式为: `const 类型说明符 &引用名;`, 如`float dist(const Point &p1, const Point &2);` **在不希望改变实参的情况下, 应该尽可能使用常引用.** 常引用作形参的好处是: 一方面不需要传递整个对象, 可以提高效率, 另一方面又可以避免对实参的误改动.
4. 常数组: 数组元素不能被更新, `类型说明符 const 数组名[大小];`
5. 常指针: 指向常量的指针. 





## static

用static修饰的对象具有**静态生存期**, 这种生存期与程序的运行期相同, 也即只要程序在运行, 该对象就有效, 因此静态变量只在第一次进入函数时才会被初始化, 之后则一直有效; 另一种具有静态生存期的是在文件作用域中声明的对象;

与之对应的是**动态生存期**, 这种对象局部有效.

- 静态局部变量: 静态生存期, 全局寿命, 局部可见. (即在块中声明的静态变量, 只在当前块中可见, 但是在程序运行期间该变量一直有效,)
- 生存期与可见性: 一个描述对象何时有效, 一个描述对象是否可见.

类的静态数据成员: 为该类的所有对象共享, 静态数据成员具有静态生存期. 必须在类外定义和初始化, 用(::)来指明所属的类. 

类的静态函数成员: 类外代码可以使用类名和作用域操作符来调用静态成员函数. 静态成员函数主要用于处理该类的静态数据成员, 可以直接调用静态成员函数. 不能处理非静态数据成员(非静态数据成员属于该类的某一特定对象, 而不是这个类整体)

```cpp
class Point{
    pubilc:
      static void showCount() {
          cout << "Object count = " << count << endl;
      }
    ...;
    private:
      static int count;
}

int Point::count = 0;

int main()
{
    ...;
    Point a;
    Point::showCount();// 静态函数成员可以直接调用
    a.showCount();// 也可以通过某一对象调用,效果相同
    return 0;
}
```



## 数组与指针

数组名字是数组首元素的内存地址. 即等效于一个指针. 数组名是一个常量, 不能被赋值.

数组初始化时仅第一维的数组长度可以省略(前提是所有数组元素都给出来).

数组名可以作为函数形参, 其本质是地址. 在参数传递时本质上传递的是数组的首地址. 数组长度可以忽略第一维, 如: `void RowSum(int a[][4], int nRow);`

对象数组: 定义`类名 数组名[元素个数]`, 每个元素被创建时, 都会调用构造函数. 访问其中的对象中的成员可以通过`数组名[下标].成员名`

**基于范围的for循环**

```cpp
int array[3] = {1, 2, 4};
int *p;
for (p = array; p < array + sizeof(array)/sizeof(int); p++){
    *p += 2;
    std::cout << *p << std::endl;
}

for (int *ptr = array; ptr < array + sizeof(array)/sizeof(int); ptr++){
    *ptr += 2;
    std::cout << *ptr << std::endl;
}
//基于范围的for循环写法, 这里e定义为array中元素的引用, 可以通过这种方法对array中的元素进行操作
for (int & e:array){
    e += 2;
    std::cout << e << std::endl;
}
// 用这种方法, 没有取引用, 这里实际上只是对e, 也就是array的副本进行操作, 并不影响array本身的值
for (int e:array){
    e += 2;
    std::cout << e << std::endl;
}
```



指针: 内存地址, 用于间接访问内存单元. 指针变量: 用于存放地址的变量.

定义: `static int i = 3; static int *ptr = &i; cout << *ptr << endl; (output: 3)`



*****和**&**

`*`代表获取相应地址中存储的变量, 通过`*地址 `可以得到相应地址中存储的数据, 因此`*`后面跟着的通常为指针;

`&`代表取址, 通过 `&对象名` 可以获取相应对象的存储地址.

指针的初始化: `存储类型 数据类型 *指针名 = 初始地址;`. 简单理解可以理解为`数据类型 *`为一种新的数据类型, 即指针数据类型, `int *p = &a;` 指针的赋值: `指针名 = 地址`; `int *p; int a = 0; p = &a;`

注意: 用变量地址作为初值时, 该变量必须在指针初始化之前已声明过, 且变量类型应与指针类型一致, 即指针指向的地址必须是一个合法地址. 不要用一个内部非静态变量去初始化 static 指针.(因为该变量只有局部作用域, 跳出该局部后该变量的存储地址会被抹去, 此时指针指向的地址为空);

空指针: `int *p = 0; int *p = nullptr;`

存在void类型的指针, 但不存在void类型的变量(指针存储的是地址, 地址所占存储空间是明确的). void类型的指针可以被赋予任何类型对象的地址, 通过强制类型转换可以提取出void类型指针中对应的数据. 

```cpp
void *pv;
int i = 5;
pv = &i;
int *pint;
pint = static_cast<int*>(pv);
// int *pint = static_cast<int*>(pv);
cout << *pint << endl;
```



指向常量的指针: 不允许通过指针修改指针所指对象的值, 但可以改变指针, 可以指向其他对象;

指针类型的常量: 指针本身的值不能被改变, 也即指针与固定地址绑定, 可以改变所指对象的值.

这里指针常量好像定义与之前理解的是反的: `const int a;` 常数据, a不可更新; `const int *p;`并不是指针类型的常量, 而是指向常量的指针, 无法更新所指内容, 可以修改指针本身的地址.

```cpp
int a = 1;
int b = 4;
int *p = &a;
p = &b; // 可以修改指针指向b
*p = 7; // 可以修改指针所指对象的值, 即b的值

const int *p1 = &a; // p1是指向常量的指针
*p1 = 6; // error, 不允许通过指针修改值
a = 6; // right, 可以直接操作所指对象
p1 = &b; // right, 可以修改指针所指地址, 但是仍然不能通过指针操作b

int * const p2 = &a; // p2是指针类型的常量
*p2 = 5; // right, 可以对p2所指对象进行操作
p2 = &b; // wrong, 无法改变指针常量的值, 也即地址
```



数组指针: 指向数组的指针.

```cpp
int a[10];
int *ptr;
ptr = a; // 数组名字即为数组首元素的地址.
ptr = &a[0]; // 等效于上述用法, 定义指针指向数组首元素
// ptr与a完全等效; a[i]就是ptr[i], 也即*(ptr+i)或是*(a+i); 
// 可以写ptr++; 这里*(ptr++)等效于a[1]; 但是不能写a++, 因为a是数组首元素, 是常量
```

指针数组: 数组的元素是指针类型

```cpp
int line1[3] = {3,4,5};
int line2[5] = {3,4,5};
int line3[4] = {3,4,5,6};

int *p[3] = { line1, line2, line3 }; // 指针数组, 这里line1~line3必须实现定义, 才会有合法地址
// 可以通过p[i][j]访问相应的元素
```

二维数组可以视为一种特殊的指针数组, 区别在于二维数组存储位置连续, 且每行元素个数相同, 而指针数组则没有这种限制.



指针作为函数参数: 可以双向传递; 要传递一组数据时, 只传首地址效率高, 实参是数组名时形参可以是指针.

`void splitFloat(float x, int *intPat, float * fracPart);`



指针类型的函数: 函数返回值是指针.

定义: `存储类型 数据类型 *函数名() { //函数体; };`

**注意: 不要将非静态局部地址作为函数的返回值.** 通常是主调函数中定义的数组, 在子函数中进行操作, 返回其中一个元素的地址. 或者可以在子函数中通过**动态内存分配new操作**取得的内存地址, 注意: 动态分配的内存不能忘记释放.

```cpp
int *newintvar();

int main()
{
  int *intptr = newintvar();
  *intptr = 5;
  delete intptr; //如果忘记在这里释放，会造成内存泄漏
  return 0;    
}

int *newintvar(){
    int *p = new int();
    return p; // 返回的地址指向的是动态分配的空间
}// 函数运行结束时, p中的地址仍有效
```



函数指针: `存储类型 数据类型 (*函数指针名)();`

```cpp
#include <iostream> using namespace std;
int compute(int a, int b, int(*func)(int, int)) {
    return func(a, b);
}

int sum(int a, int b){ // 求和
    return a + b;
}

int max(int a, int b){ // 最大值
    return ((a > b) ? a:b);
}

int min(int a, int b){ // 最小值
    return ((a > b) ? b:a);
}

int main()
{
  int a, b, res;
  cin >> a;
  cin >> b;
  res = compute(a, b, &max);
  res = compute(a, b, &min);
  res = compute(a, b, &sum);
}
```



对象指针: 指向一个对象; `类名 *对象指针名`

```cpp
Point a(5, 10);
Point *ptr;
ptr = &a;
```

访问对象成员: ptr->getx() 等效于 (*ptr).getx();



**this 指针**: 指向当前对象自己, 它隐含于类的每一个非静态成员函数中. 在调用类的非静态成员函数中, 实际上还隐含传递了一个this指针, 通过该指针, 系统才知道应该使用那个对象中的成员. 例如`getX()中return x; 本质上是return this->x;`



## 动态内存分配

动态申请内存操作符: `new`.

`new 类型名T (初始化参数列表);`; 在程序执行期间, 申请用于存放T类型对象的内存空间, 并按照初值列表赋初值. 如果申请成功, 返回T类型的指针, 指向新分配的内存.

释放内存操作符`delete`. delete操作的必须是new操作的返回值. 删除的是指针所指的对象, 该指针仍然存在, 可以重新赋值.



`new int(n)`; `new int[n]`; 前者是定义一个int类型的数组, 其初始值为n, 后者是定义一个长度为n的动态数组.



申请和释放动态数组

分配: `new 类型名T [数组长度]`, 数组长度可以是任何整数类型表达式, 在运行时计算

释放: `delete[] 数组名p`, 释放指针p所指向的数组, p必须是用new分配得到的数组首地址. 如果没有`[]`, 只释放了数组首地址.



动态创建多维数组: `new 类型名T[第1维长度][第2维长度]...;`

```cpp
#include <iostream>
using namespace std;
int main() {
  int (*cp)[9][8] = new int[7][9][8];
  for (int i = 0; i < 7; i++)
    for (int j = 0; j < 9; j++)
      for (int k = 0; k < 8; k++)
        *(*(*(cp + i) + j) + k) =（i * 100 + j * 10 + k);
  for (int i = 0; i < 7; i++) {
    for (int j = 0; j < 9; j++) {
      for (int k = 0; k < 8; k++)
        cout << cp[i][j][k] << " ";
      cout << endl;
    }
    cout << endl;
  }
delete[] cp;
return 0;
}
```



例如:

```cpp
#include <iostream>
#include <cassert>
using namespace std;
class Point { //类的声明同例6-16 … };
class ArrayOfPoints { //动态数组类
public:
  ArrayOfPoints(int size) : size(size) {
    points = new Point[size];
  }
  ~ArrayOfPoints() {
    cout << "Deleting..." << endl;
    delete[] points;
  }
  Point& element(int index) {
    assert(index >= 0 && index < size);
    return points[index];
  }
private:
  Point *points; //指向动态数组首地址
  int size; //数组大小
};
int main() {
  int count;
  cout << "Please enter the count of points: ";
  cin >> count;
  ArrayOfPoints points(count); //创建数组对象
  points.element(0).move(5, 0); //访问数组元素的成员
  points.element(1).move(15, 20); //访问数组元素的成员
  return 0;
}
```

这里`elemet`的返回类型为`Point&`, 是Point类的一个引用, 返回“引用”可以用来操作封装数组对象内部的数组元素.  就像主函数中对`element(0)`和`element(1)`进行了操作, 如果返回“值”则只是返回了一个“副本”, 通过“副本”是无法操作原来数组中的元素的.



## vector

`vector<元素类型> 数组对象名(数组长度)`

```cpp
#include <iostream>
#include <vector>
using namespace std;
//计算数组arr中元素的平均值
double average(const vector<double> &arr) // 形参为vector的引用
{
    double sum = 0;
    for (unsigned i = 0; i<arr.size(); i++)
    sum += arr[i];
    return sum / arr.size();
}

int main( int argc, char *argv[] ) {
    unsigned n;
    cout << "n = ";
    cin >> n;
    vector<double> arr(n); //创建数组对象
    cout << "Please input " << n << " real numbers:" << endl;
    for (unsigned i = 0; i < n; i++)
        cin >> arr[i];
    cout << "Average = " << average(arr) << endl;
    return 0;
}
```



基于范围的for循环配合auto举例:

```cpp
#include <vector>
#include <iostream>
int main()
{
  std::vector<int> v = {1,2,3};
  for(auto i = v.begin(); i != v.end(); ++i)// i实际上是一个迭代器
    std::cout << *i << std::endl;

  for(auto e : v)
    std::cout << e << std::endl;
}
```



**对象的浅层复制和深层复制**:

浅层复制: 实现对象间数据元素的一一对应复制.

深层复制: 当被复制的对象数据成员是指针类型时, 不是复制该指针成员本身, 而是将指针所指对象进行复制.

浅层复制的情况下, 复制对象和原始对象中的指针指向的是同一地址, 一个对象中对该地址内的数据进行修改, 另一个对象中的结果也随之改变.



## 移动构造函数

```cpp
将要返回的局部对象转移到主调函数，省去了构造和删除临时对象的过程。
#include<iostream>
using namespace std;
class IntNum {
public:
  IntNum(int x = 0) : xptr(new int(x)){ //构造函数
    cout << "Calling constructor..." << endl;
  }
  IntNum(const IntNum & n) : xptr(new int(*n.xptr)){//复制构造函数
    cout << "Calling copy constructor..." << endl;
  }
  IntNum(IntNum && n): xptr( n.xptr){ //移动构造函数, 将n的指针直接复制到新对象的指针, 即浅层复制
     n.xptr = nullptr; // 需要将之前的对象n的指针置为空, 这样n析构的时候delete空指针不会发生任何作用, 不会存在二次delete的问题
     cout << "Calling move constructor..." << endl;
  }
  ~IntNum(){ //析构函数
    delete xptr; 
    cout << "Destructing..." << endl;
  }
private:
  int *xptr;
};
//返回值为IntNum类对象
IntNum getNum() {
  IntNum a;
  return a;
}
int main() {
  cout << getNum().getInt() << endl; return 0;
}

运行结果：
Calling constructor...
Calling move constructor...
Destructing...
0
Destructing...
```



## 字符串string

读取字符串:

```cpp
string str;
cin >> str; //从默认输入设备中读取字符串, 默认以空格为中断

// -----------getline
#include <string>
getline(cin, str);
getline(cin, str, ',')
  
// istream& getline (istream& is, string& str, char delim);
// is: 输入流对象, cin表示键盘输入, 也可以防止文本输入流. delim代表分隔符, 默认为换行符
// istream& getline (istream& is, string& str);
// istream被iostream, ifstream, istringstream继承
ifstream inFile("file.txt");
getline(inFile, str); 
```



### char* and string

```cpp
string str;
// string to char*
const char* ccstr = str.c_str();
char* cstr = new char[str.length()+1];
std::strcpy(cstr, str.c_str());

// char* to string
const char* ccstr = "test"; // ""代表字符串常量
string str(ccstr); // 字符串常量可以用来初始化string, 等效于 string str = "test"
string str = std::string(cstr); // char* 貌似可以直接初始化string
string str = cstr; // 貌似这样也是可以的
```





## 继承与派生

### 继承方式

继承方式: `public`, `protected`, `private`

`public`

- 基类的`public`和`protected`成员: **访问属性在派生类中保持不变**;
- 基类的`private`成员: 不可直接访问;
- 派生类中的成员函数可直接访问基类中的`public`和`protected`成员, 但不能直接访问基类的`private`成员;
- 通过派生类的对象: **只能访问`public`成员**.(参考第一条)

`private`

- 基类的`public`和`protected`成员: **都以`private`身份出现在派生类中**;
- 基类的`private`成员: 不可直接访问;
- 派生类中的成员函数可直接访问基类中的`public`和`protected`成员, 但不能直接访问基类的`private`成员;
- 通过派生类的对象: **不能直接访问从基类继承的任何成员**.(参考第一条)

`protected`

- 基类的`public`和`protected`成员: **都以`protected`身份出现在派生类中**;
- 基类的`private`成员: 不可直接访问;
- 派生类中的成员函数可直接访问基类中的`public`和`protected`成员, 但不能直接访问基类的`private`成员;
- 通过派生类的对象: **不能直接访问从基类继承的任何成员**.(参考第一条)

<u>**note**</u>: 派生类中全都包含基类的成员, 只是能否直接访问的区别.



`protected`成员的特点与作用: (与`private`类似, 区别体现在继承和派生)

- 对建立其所在类对象的模块来说, 它与`private`成员的性质相同.
- 对于其派生类来说, 它与`public`成员的性质相同.
- 既实现了数据隐藏, 又方便继承, 实现代码重用.

`protected`可以在派生类中直接访问(与`public`类似), 而对于类的对象来说, `protected`成员不能被直接访问(与`private`类似)



所有的`private`在类外都是不可直接访问的(即使是在派生类中).

在继承的过程中, 按照不同的继承方式, 基类中的`public`和`protected`成员会在派生类中改变其成员类型, 但是这两种成员在派生类中都是可以直接访问的.

通过类的对象只能访问类中的`public`成员, 判断基类中的成员能否通过类的对象访问需要注意该成员在派生类中的身份.

```cpp
// keyword_private.cpp  
class BaseClass {  
public:  
   // privMem accessible from member function  
   int pubFunc() { return privMem; }  
protected:
   int proFunc() { return privMem; }
private:  
   int privFunc() { return privMem; }
   int privMem;  
};  
  
class DerivedClass : public BaseClass {  
public:  
   void usePrivate( int i )  
      { privMem = i; }   // error, C2248: privMem not accessible from derived class  
   int usePublic() { return pubFunc(); } // pubFunc() accessible from derived class  
};  

class DerivedClass1 : protected BaseClass {  
public:  
   void usePrivate( int i )  
      { privMem = i; }   // error, C2248: privMem not accessible from derived class  
   int usePublic() { return pubFunc(); } // pubFunc() accessible from derived class  
};  

class DerivedClass2 : private BaseClass {  
public:  
   void usePrivate( int i )  
      { privMem = i; }   // error, C2248: privMem not accessible from derived class  
   int usePublic() { return pubFunc(); } // pubFunc() accessible from derived class  
};  
  
int main() {  
   BaseClass aBase;  
   DerivedClass aDerived;  
   DerivedClass1 aDerived1;  
   DerivedClass2 aDerived2;  
   aBase.privMem = 1;      // error, C2248: privMem is private in base class 
   aDerived.privMem = 1;   // error, C2248: privMem is private in derived class
   aDerived1.privMem = 1;  // error, C2248: privMem is private in derived class
   aDerived2.privMem = 1;  // error, C2248: privMem is private in derived class
  
   aBase.pubFunc();	  // 			   	   pubFunc() is public in base class  
   aDerived.pubFunc();	  // 			   pubFunc() is public in derived class  
   aDerived1.pubFunc();   // error, C2247: pubFunc() is protected in derived class  
   aDerived2.pubFunc();   // error, C2247: pubFunc() is private in derived class  
}  
```



### 基类与派生类

- **公有**派生类对象可以被当作基类的对象使用, 反之则不可.
  - 派生类的对象可以隐含转换为基类对象;
  - 派生类的对象可以初始化基类的引用;
  - 派生类的指针可以隐含转换为基类的指针.
- 通过基类对象名, 指针只能使用从基类继承的成员.



默认情况下, 基类的构造/析构函数不被继承, 派生类需要定义自己的构造函数, 且需要提供用于基类初始化的参数.

可以使用`using`语句继承基类的构造函数, 这种情况下, 缺点在于只能初始化从基类继承来的成员, 对于新增的成员无法在构造函数中进行初始化, 除非采用类内初始值进行初始化. `using B::B`

如果不继承基类构造函数, 派生类的构造函数需要保证:

- 派生类新增成员: 派生类定义构造函数进行初始化;
- 继承来的成员: 自动调用基类构造函数进行初始化;
- 为满足第二点要求, 派生类的构造函数需要给基类的构造函数传递参数.




单继承时构造函数的定义语法

```cpp
派生类名::派生类名(基类所需的形参，本类成员所需的形参):
基类名(参数表), 本类成员初始化列表
{
	//其他初始化；
}；
```



多继承时构造函数的定义语法

```cpp
派生类名::派生类名(参数表) : 
基类名1(基类1初始化参数表), 
基类名2(基类2初始化参数表), 
...
基类名n(基类n初始化参数表), 
本类成员初始化列表
{
        //其他初始化；
}；
```
在执行构造函数时, 先**按照派生类声明时的基类顺序调用各个基类的构造函数**, 再对本类成员进行初始化, 最后执行构造函数内部的其他函数体.

析构函数不被继承, 需要自行声明析构函数. **在析构函数中系统会自动隐式调用基类的析构函数**, 不需要用户自己显式调用. 在执行时, **先执行派生类析构函数的函数体再调用基类的析构函数**. 与构造函数顺序相反.(先析构最后定义的本类对象, 析构完全部本类对象, 再调用基类析构函数, 析构顺序与构造顺序相反)



当派生类与基类中有相同成员时:

- 若未特别限定, 则通过派生类对象使用的是派生类中的同名成员.
- 如要通过派生类对象访问基类中被隐藏的同名成员, 应使用基类名和作用域操作符`::`来限定.

```cpp
#include <iostream>
using namespace std;
class Base1 {   
public:
    int var;
    void fun() { cout << "Member of Base1" << endl; }
};
class Base2 {   
public:
    int var;
    void fun() { cout << "Member of Base2" << endl; }
};
class Derived: public Base1, public Base2 {
public:
    int var;    
    void fun() { cout << "Member of Derived" << endl; }
};

int main() {
    Derived d;
    Derived *p = &d;

  //访问Derived类成员
    d.var = 1;
    d.fun();    

    //访问Base1基类成员
    d.Base1::var = 2;
    d.Base1::fun(); 

    //访问Base2基类成员
    p->Base2::var = 3;
    p->Base2::fun();    

    return 0;
}
```



二义性, 上述问题不属于二义性, 因为存在同名隐藏(派生类成员优先级高一些?). 

如果从不同基类继承了同名成员, 但是在派生类中没有定义同名成员. “**派生类对象名或引用名.成员名**”, “**派生类指针->成员名**”访问成员存在二义性问题. 可以用“**派生类对象名或引用名.Base::成员名**”限定.



### 虚基类

如果一个派生类继承了多个基类, 而这些基类最终也继承了共同基类, 这时候如果要访问共同基类的成员, 会产生冗余, 并有可能带来不一致性.

- 虚基类声明
  - 以virtual说明基类继承方式
  - 例：`class B1:virtual public B`
- 作用
  - 主要用来解决多继承时可能发生的对同一基类继承多次而产生的二义性问题
  - 为最远的派生类提供唯一的基类成员，而不重复产生多次复制
- 注意：在第一级继承时就要将共同基类设计为虚基类。





虚基类及其派生类构造函数

- 建立对象时所指定的类称为**最远派生类**。
- 虚基类的成员是由最远派生类的构造函数通过调用虚基类的构造函数进行初始化的。
- **在整个继承结构中，直接或间接继承虚基类的所有派生类，都必须在构造函数的成员初始化表中为虚基类的构造函数列出参数**。如果未列出，则表示调用该虚基类的默认构造函数。
- **在建立对象时，只有最远派生类的构造函数调用虚基类的构造函数，其他类对虚基类构造函数的调用被忽略。**




### 虚函数

```cpp
#include <iostream>
using namespace std;

class Base1 {
public:
    virtual void display() const;  // 虚函数
};
void Base1::display() const {	// 虚函数在类外实现, 实现时不加virtual
    cout << "Base1::display()" << endl;
}

class Base2: public Base1 { 
public:
     virtual void display() const;  // 虚函数
};
void Base2::display() const {
    cout << "Base2::display()" << endl;
}
class Derived: public Base2 {
public:
     virtual void display() const;   // 虚函数
};
void Derived::display() const {
    cout << "Derived::display()" << endl;
}

void fun(Base1 *ptr) { 
    ptr->display(); 
}

int main() {    
    Base1 base1;
    Base2 base2;
    Derived derived;    
    fun(&base1);		// 调用Base1的display
    fun(&base2);		// 调用Base2的display
    fun(&derived);		// 调用Derived的display
    return 0;
}
```

对于上述例子, 如果`display`函数不是虚函数, 即没有`virtual`的话, 上述执行`fun`函数都是在调用`Base1`中的`display`. 这是由于编译器在编译`fun`过程中就需要进行链接, 在链接过程中必须要明确这里所用的`display`函数的地址, 编译器并不知道在运行时实际所给的是派生类的对象, 因此编译器只能按照形参中的对象进行编译, 这种对函数的调用叫做静态绑定或者静态链接, 静态链接就是在程序被执行之前函数调用是确定的, 这种函数在编译程序期间是固定的.

虚函数的功能是: 在编译中碰到调用的函数是虚函数时, 编译器会暂不进行链接, 而是等到运行过程中, 根据输入对象再确定需要调用的函数. 根据调用函数的对象的类型对程序中在任何给定指针中被调用的函数的选择, 这种操作被称为动态链接.

有些虚函数的定义为`=0`这类函数为纯虚函数, 纯虚函数必须在派生类中重载相应的函数才可以正常运行.  在基类中仅仅是声明一下.



- 虚函数的声明: `virtual 函数类型 函数名(形参表)`;
- 虚函数声明只能出现在类定义中的函数原型声明中, 而不能在成员函数实现的时候;
- C++中的虚函数是动态绑定的函数;
- 虚函数必须是非静态的成员函数，虚函数经过派生之后，就可以实现运行过程中的多态;
- 虚函数一般不声明为内联函数`inline`，因为对虚函数的调用需要动态绑定，而对内联函数的处理是静态的。
- 在派生类中可以对基类中的成员函数进行覆盖;
- 构造函数不能是虚函数, 一般成员函数和析构函数可以是虚函数;

虚成员函数: 

- 派生类可以不显式地用virtual声明虚函数，这时系统就会用以下规则来判断派生类的一个函数成员是不是虚函数：
  - 该函数是否与基类的虚函数有相同的名称、参数个数及对应参数类型；
  - 该函数是否与基类的虚函数有相同的返回值或者满足类型兼容规则的指针、引用型的返回值；
- 如果从名称、参数及返回值三个方面检查之后，派生类的函数满足上述条件，就会自动确定为虚函数。这时，派生类的虚函数便覆盖了基类的虚函数。
- 派生类中的虚函数还会隐藏基类中同名函数的所有其它重载形式。
- 一般习惯于在派生类的函数中也使用virtual关键字，以增加程序的可读性。



**虚析构函数**: `virtual ~ClassName();`

下面的例子中就需要声明虚析构函数. 因为在派生类的中增加了一个动态分配的成员, 这就需要在派生类的析构函数中删除它. 在调用`fun`函数中, 通过`delete b`会调用析构函数, 如果这里是静态链接的话(非虚函数), 那么只会调用基类的析构函数, 无法正常释放`p`的内存空间.

```cpp
#include <iostream>
using namespace std; 
class Base {
public:
    virtual ~Base(); // 虚函数
};
Base::~Base() {
    cout<< "Base destructor" << endl;
 }
class Derived: public Base{
public:
    Derived() { p = new int(0); };
    virtual ~Derived(); // 虚函数
private:
    int *p;
};

Derived::~Derived(){
    cout << "Derived destructor " << endl;
    delete p;
}

void fun(Base *b){
    delete b;
}
```

 

### 纯虚函数和抽象类

- 纯虚函数是一个在基类中声明的虚函数，它在该基类中没有定义具体的操作内容，要求各派生类根据实际需要定义自己的版本，纯虚函数的声明格式为: `virtual 函数类型 函数名(参数表) = 0;`
- 带有纯虚函数的类称为抽象类: `class 类名 { virtual 类型 函数名(参数表)=0; //其他成员…… }`


- 抽象类只能作为基类来使用; 不能定义抽象类的对象.





## 运算符重载

### 重载为成员函数

syntax

```cpp
函数类型 operator 运算符 (形参)
{
    ...
}
// 可以把'operator 运算符'整体作为一个函数的函数名
// 形参个数 = 原操作数个数 - 1; 双目运算符(如 a + b)形参个数为1, 单目运算符(如 ++a )形参个数为0;
// 后置的++ --除外( 形参个数 = 原操作数个数)
```

双目运算符: 

用B表示运算符(B可以是+, - 等等). 要实现`obj1 B obj2`的表达式, 如 `a + b`, 需要在obj1对应的类中重载运算符B, 其形参为obj2对应的类. 重载后, `obj1 B obj2`就相当于`obj1.operator B(obj2)`.

考虑到, 通常运算符不对obj2进行改动, 常将运算符重载为常成员函数, const类型.

```cpp
class Complex{
    public:
      Complex(double r = 0., double i = 0.): real(r), image(i) {}
      Complex operator + (const Complex& c2) const;
   private:
      double real, image;
}
Complex Complex::operator + (const Complex& c2) const{
    return Complex(real+c2.real, image+c2.image);
}
```

单目运算符:

- 前置, 如++obj, --obj; 无形参, 重载为obj的成员函数即可
- 后置, obj++, obj--; 为了与前置区分, 增加了一个int类型的形参, 该形参无实际意义, 只是作为区分.

从下面的例子可以看出: `this`是指向当前对象的指针, 前置的`++`返回的是`*this`也即该对象. 而后置的`++`返回的是`old`, 这里对当前对象`*this`进行了前置`++`的操作, 也即当前对象已经完成了加1的操作, 但是返回的是加1之前的对象的copy, 也即`old`. 这就意味着: 前置++的返回值比后置++的返回值大1, 尽管此时原本的对象都已经加了1.

```cpp
int i = 10;
int j = i++;
int a = 10;
int b = ++a;
// i = 11, a = 11
// j = 10, b = 11
```



```cpp
class Clock{
    public:
      Clock(int hour = 0, int minute = 0, int second = 0);
      Clock& operator ++ (); // 前置
      Clock& operator ++ (int); // 后置
    private:
      int hour, minute, second;
}

// 前置
Clock& Clock::operator ++ ()
{
  second++;
  if (second > 60){
      second -= 60;
      minute++;
      if (minute > 60){
          minute -= 60;
          hour = (hour + 1)%24;
      }
  }
  return *this; // 返回的是加1之后的该对象
}

// 后置
Clock Clock::operator ++ (int) {
    Clock old = *this; // this
    ++(*this);
    return old;
}
```



### 重载为非成员函数

有些运算符不能重载为成员函数, 例如二元运算符的左操作数不是对象, 或者是不能由我们重载运算符的对象.

运算符重载为非成员函数的规则:

- 函数的形参代表依自左至右次序排列的各操作数。
- 重载为非成员函数时
- 参数个数=原操作数个数（后置++、--除外）
- 至少应该有一个自定义类型的参数。
- 后置单目运算符 ++和--的重载函数，形参列表中要增加一个int，但不必写形参名。
- **如果在运算符的重载函数中需要操作某类对象的私有成员, 可以将此函数声明为该类的友元。**

补充: 友元函数: 友元函数是在**类声明中**由关键字friend修饰说明的**非成员函数**，在它的函数体中能 够通过对象名访问 private 和 protected成员. 友元函数在类中声明, 但是是非成员函数.

```cpp
//8_3.cpp
#include <iostream>
using namespace std;

class Complex {
public:
    Complex(double r = 0.0, double i = 0.0) : real(r), imag(i) { }  
    friend Complex operator+(const Complex &c1, const Complex &c2);
    friend Complex operator-(const Complex &c1, const Complex &c2);
    friend ostream & operator<<(ostream &out, const Complex &c);
  // 这里ostream不是我们可以操作的对象, 只能采用非成员函数的形式重载
private:    
    double real;  //复数实部
    double imag;  //复数虚部
};

Complex operator+(const Complex &c1, const Complex &c2){
    return Complex(c1.real+c2.real, c1.imag+c2.imag); 
}
Complex operator-(const Complex &c1, const Complex &c2){
    return Complex(c1.real-c2.real, c1.imag-c2.imag); 
}

ostream & operator <<(ostream &out, const Complex &c){
    out << "(" << c.real << ", " << c.imag << ")";
    return out;
}

int main() {    
    Complex c1(5, 4), c2(2, 10), c3;    
    cout << "c1 = " << c1 << endl;
    cout << "c2 = " << c2 << endl;
    c3 = c1 - c2;   //使用重载运算符完成复数减法
    cout << "c3 = c1 - c2 = " << c3 << endl;
    c3 = c1 + c2;   //使用重载运算符完成复数加法
    cout << "c3 = c1 + c2 = " << c3 << endl;
    return 0;
}
```



## 模板

### 函数模板

syntax

```cpp
template <class T>  //定义函数模板, 用T代替数据类型
void outputArray(const T *array, int count) {
    for (int i = 0; i < count; i++)
        cout << array[i] << " "; //如果数组元素是类的对象，需要该对象所属类重载了流插入运算符“<<”
    cout << endl;
}
```



### 类模板

使用类模板使用户可以为类声明一种模式, 使得类中的某些数据成员、某些成员函数的参数、某些成员函数的返回值，能取任意类型（包括基本类型的和用户自定义类型）.

需要在模板类定义和类外成员函数定义前增加句式: 	`template <class T>`; 之后用抽象数据类型T代替模板类中的数据类型, 在类声明和定义过程中用`classname<T>`作为类名.

syntax

```cpp
#include <iostream>
#include <cstdlib>
using namespace std;
struct Student {
  int id;       //学号
  float gpa;    //平均分
}; 

template <class T> // 模板类声明前加该句式
class Store {	   // 类模板：实现对任意类型数据进行存取; 在该类中需要替换的数据类型用T代替
private:
    T item; // item用于存放任意类型的数据
    bool haveValue;  // haveValue标记item是否已被存入内容
public:
    Store();
    T &getElem();   //提取数据函数
    void putElem(const T &x);  //存入数据函数
};

template <class T>  // 在模板类外定义的成员函数前也需要增加该句式
Store<T>::Store(): haveValue(false) { } 

template <class T>
T &Store<T>::getElem() {
    //如试图提取未初始化的数据，则终止程序
    if (!haveValue) {   
        cout << "No item present!" << endl;
        exit(1);    //使程序完全退出，返回到操作系统。
    }
    return item;        // 返回item中存放的数据 
}

template <class T>
void Store<T>::putElem(const T &x) {
    // 将haveValue 置为true，表示item中已存入数值   
    haveValue = true;   
    item = x;           // 将x值存入item
}

int main() {
    Store<int> s1, s2;  // 定义int类型的类对象
    s1.putElem(3);  
    s2.putElem(-7);
    cout << s1.getElem() << "  " << s2.getElem() << endl;

    Student g = { 1000, 23 };
    Store<Student> s3; // 定义自定义类型的类对象
    s3.putElem(g); 
    cout << "The student id is " << s3.getElem().id << endl;

    Store<double> d;
    cout << "Retrieving object D... ";
    cout << d.getElem() << endl;
   //d未初始化，执行函数D.getElement()时导致程序终止
    return 0;
}
```



### 数组类

静态数组: 具有固定元素个数的群体, 大小在编译时就确定了, 运行时无法修改;

动态数组: 不定长度, vector就是典型的动态数组类.

下面的例子中介绍了如何自定义数组类：

```cpp
#ifndef ARRAY_H
#define ARRAY_H
#include <cassert>

template <class T>  //数组类模板定义
class Array {
private:
    T* list;        //用于存放动态分配的数组内存首地址
    int size;       //数组大小（元素个数）
public:
    Array(int sz = 50);     //构造函数
    Array(const Array<T> &a);   //复制构造函数
    ~Array();           //析构函数
    Array<T> & operator = (const Array<T> &rhs);    //重载"=“
    T & operator [] (int i); //重载"[]”, 这里希望[]返回的是左值, 也即通过array[i]可以对array中的数据进行操作, 因此需要返回引用
    const T & operator [] (int i) const;     //重载"[]”常函数
    operator T * ();        //重载指针转换运算符
    operator const T * () const;
    int getSize() const;        //取数组的大小
    void resize(int sz);        //修改数组的大小
};

template <class T> Array<T>::Array(int sz) {//构造函数
    assert(sz >= 0);//sz为数组大小（元素个数），应当非负
    size = sz;  // 将元素个数赋值给变量size
    list = new T [size];    //动态分配size个T类型的元素空间
}

template <class T> Array<T>::~Array() { //析构函数
    delete [] list;
}

template <class T> 
Array<T>::Array(const Array<T> &a) {    //复制构造函数(深层)
    size = a.size;     //从对象x取得数组大小，并赋值给当前对象的成员
    list = new T[size]; // 动态分配n个T类型的元素空间
    for (int i = 0; i < size; i++)     //从对象X复制数组元素到本对象 
        list[i] = a.list[i];
}

//重载"="运算符，将对象rhs赋值给本对象。实现对象之间的整体赋值
template <class T>
Array<T> &Array<T>::operator = (const Array<T>& rhs) {
    if (&rhs != this) {//如果本对象中数组大小与rhs不同，则删除数组原有内存，然后重新分配
        if (size != rhs.size) {
            delete [] list; //删除数组原有内存
            size = rhs.size;    //设置本对象的数组大小
            list = new T[size];  //重新分配size个元素的内存
        }
        //从对象X复制数组元素到本对象  
        for (int i = 0; i < size; i++)
            list[i] = rhs.list[i];
    }
    return *this;   //返回当前对象的引用
}
//重载下标运算符，实现与普通数组一样通过下标访问元素，具有越界检查功能
template <class T>
T &Array<T>::operator[] (int n) {
    assert(n >= 0 && n < size);  //检查下标是否越界
    return list[n];       //返回下标为n的数组元素
}
template <class T>
const T &Array<T>::operator[] (int n) const {
    assert(n >= 0 && n < size);  //检查下标是否越界
    return list[n];       //返回下标为n的数组元素
//重载指针转换运算符，将Array类的对象名转换为T类型的指针
template <class T>
Array<T>::operator T * () { // 语法规定指针转换运算符的函数声明中没有返回值
    return list;    //返回当前对象中私有数组的首地址
}

//取当前数组的大小
template <class T>
int Array<T>::getSize() const {
    return size;
}

// 将数组大小修改为sz
template <class T>
void Array<T>::resize(int sz) {
    assert(sz >= 0);    //检查sz是否非负
    if (sz == size) //如果指定的大小与原有大小一样，什么也不做
        return;
    T* newList = new T [sz];    //申请新的数组内存
    int n = (sz < size) ? sz : size;//将sz与size中较小的一个赋值给n
    //将原有数组中前n个元素复制到新数组中
    for (int i = 0; i < n; i++)
        newList[i] = list[i];
    delete[] list;      //删除原数组
    list = newList; // 使list指向新数组
    size = sz;  //更新size
}
// 可以看到, resize操作过程中需要频繁的删除再分配内存空间, 效率很低, 因此不建议频繁的重新分配内存空间, 典型的做法是给定一个长度, 每次增加数据时判断是否越界, 如果越界, 长度翻倍
#endif  //ARRAY_H
  
```



### 链表

链表是一种动态数据结构，可以用来表示顺序访问的线性群体。

- 链表是由系列**结点**组成的，结点可以在运行时动态生成。
- 每一个结点包括**数据域**和指向链表中下一个结点的**指针**(即下一个结点的地址)如果链表每个结点中只有一个指向后继结点的指针，则该链表称为单链表。
- 对于数组，每次插入或删除一个元素，数组中的其他元素也需要进行移动，链表的优势在于插入/删除一个元素，其他元素可以维持不变.
- 数组可以通过下标访问每个元素, 链表必须通过上一个元素(结点)才可以访问下一个.

```cpp
#ifndef NODE_H
#define NODE_H
template<class T>
  class Node{
      private:
        Node<T> *next; // 指向后继结点的指针
      public:
        T data;
        Node(const T& data, Node<T> *next = 0); // 构造函数
        void insertAfter(Node<T>* p); // 在当前结点后插入新结点p
        Node<T> *deleteAfter(); // 删除当前结点的后继结点, 并返回后继结点的地址
        Node<T> *nextNode(); // 获取后继结点的地址
        const Node<T> *nextNode() const; // 获取后继结点的地址
  };

template<class T>
  Node<T>::Node(const T&data, Node<T> *next) : data(data), next(next) {}

template<class T>
  void Node<T>::insertAfter(Node<T>* p){
    p->next = next; // 将p结点中的指针指向当前结点的后继结点
    next = p; // 当前结点中的指针应该指向p
}

template<class T>
  Node<T> * Node<T>::deleteAfter(){
    Node<T> *tmpNode = next;
    if (next==0) // 判断是否是最后一个结点
      return 0;
    next = tmpNode->next; // 当前结点指向下下个结点
    return tmpNode;
}

template<class T>
  Node<T> * Node<T>::nextNode(){
      return next;
  }

template<class T>
  const Node<T> * Node<T>::nextNode() const{
      return next;
  }
#endif
```



## I/O流

istream: operator: >>

ostream: standard output stream(cout): operator: <<

Format tags prototype is **%\[flags\]\[width\]\[.precision\]\[length\]specifier**, which is explained in [fprintf format格式](https://www.tutorialspoint.com/c_standard_library/c_function_fprintf.htm).



流对象与文件对象建立连接, 通过操作流对象对文件对象产生作用. cout/cin作为标准输入输出设备也被处理为文件.

### 输出流

- ostream: 输出流类的基类
- ofstream: 文件输出流类
- ostringstream: 字符串输出流类

预先定义的输出流对象

- cout 标准输出
- cerr 标准错误输出，没有缓冲，发送给它的内容立即被输出。
- clog 类似于cerr，但是有缓冲，缓冲区满时被输出。


标准输出换向

```cpp
// 将cout输出到文件b.out中
ofstream fout("b.out"); // 
streambuf*  pOld  =cout.rdbuf(fout.rdbuf());  // 将cout定向到fout指定的文件b.out中
//…
cout.rdbuf(pOld); // 重置cout
```



输出格式的控制: 通过插入运算符`<<`和操纵符(manipulator)实现.

操纵符包括: `setprecision()`, `hex()`, `setw`, `width`等, 其中只 `setw`, `width`仅影响紧随其后的输出项, 其他操纵符保持有效直到发生改变.

```cpp
// 设置对齐方式
#include <iomanip> //setiosflags()
cout << setiosflags(ios_base::left) // 左对齐
     << setw(6) << "tommy"
     << resetiosflags(ios_base::left) // 取消左对齐设置
     << setw(10) << 4 << endl;
```



setiosflags的参数是该流的格式标志值，可用按位或（|）运算符进行组合

**setiosflags的参数**

- ios_base::skipws 在输入中跳过空白 。
- ios_base::left 左对齐值，用填充字符填充右边。
- ios_base::right 右对齐值，用填充字符填充左边（默认对齐方式）。
- ios_base::internal 在规定的宽度内，指定前缀符号之后，数值之前，插入指定的填充字符。
- ios_base::dec 以十进制形式格式化数值（默认进制）。
- ios_base::oct 以八进制形式格式化数值 。
- ios_base::hex 以十六进制形式格式化数值。
- ios_base::showbase 插入前缀符号以表明整数的数制。
- ios_base::showpoint 对浮点数值显示小数点和尾部的0 。
- ios_base::uppercase 对于十六进制数值显示大写字母A到F，对于科学格式显示大写字母E 。
- ios_base::showpos 对于非负数显示正号（“+”）。
- ios_base::scientific 以科学格式显示浮点数值。
- ios_base::fixed 以定点格式显示浮点数值（没有指数部分） 。
- ios_base::unitbuf 在每次插入之后转储并清除缓冲区内容。


```cpp
#include <iostream>
#include <iomanip>
 
int main()
{
    std::cout <<  std::resetiosflags(std::ios_base::dec) 
              <<  std::setiosflags(  std::ios_base::hex
                                   | std::ios_base::uppercase
                                   | std::ios_base::showbase) << 42 << '\n';
}
```



**精度**

- 浮点数输出精度的默认值是6，例如：3466.98。
- 要改变精度：setprecision操纵符（定义在头文件iomanip中）。
- 如果不指定fixed或scientific，精度值表示有效数字位数。
- 如果设置了`ios_base::fixed`或`ios_base::scientific`精度值表示小数点之后的位数。

```cpp
#include <iostream>
#include <iomanip>
using namespace std;

int main(){
    double value = 45.324;
    cout << setprecision(4) << value << endl; // 45.32
  
    cout << setiosflags(ios_base::fixed);
    cout << setprecision(2) << value << endl; // 45.32
    cout << resetiosflags(ios_base::fixed);
  
    cout << setiosflags(ios_base::scientific);
    cout << setprecision(2) << value << endl; // 4.53e+01
    cout << resetiosflags(ios_base::scientific);
    return 0;
}
```



**向字符串输出**

将字符串作为输出流的目标, 可以实现将其他数据类型转换为字符串的功能.

`ostringstream`

- 支持ofstream类的除open、close外的所有操作
- str函数可以返回当前已构造的字符串

```cpp
//11_6.cpp
#include <iostream>
#include <sstream>
#include <string>
using namespace std;

//函数模板toString可以将各种支持“<<“插入符的类型的对象转换为字符串。
template <class T>
inline string toString(const T &v) {
    ostringstream os;   //创建字符串输出流
    os << v;        //将变量v的值写入字符串流
    return os.str();    //返回输出流生成的字符串
}

int main() {
    string str1 = toString(5);
    cout << str1 << endl;
    string str2 = toString(1.2);
    cout << str2 << endl;
    return 0;
}
```



### 输入流

- istream类最适合用于顺序文本模式输入; cin是其实例. 也是后两个输入流类的基类.
- ifstream类支持磁盘文件输入.
- istringstream类从内存的字符串中输入.

提取预算符`>>`是从输入流对象中提取字节最容易的方法.

输入流相关的函数:

- open 把该流与一个特定磁盘文件相关联。
- get 功能与提取运算符（>>）很相像，主要的不同点是get函数在读入数据时包括空白字符。
- getline 功能是从输入流中读取多个字符，并且允许指定输入终止字符，读取完成后，从读取的内容中删除终止字符。
- read 从一个文件读字节到一个指定的内存区域，由长度参数确定要读的字节数。当遇到文件结束或者在文本模式文件中遇到文件结束标记字符时结束读取。 通常与输出流中的write函数对应使用.
- seekg 用来设置文件输入流中读取数据位置的指针。
- tellg 返回当前文件读指针的位置。
- close 关闭与一个文件输入流关联的磁盘文件。

```cpp
#include <iostream>
#include <string>
using namespace std;
int main() {
    string line;
    cout << "Type a line terminated by 't' " << endl; 
    getline(cin, line, 't'); // 为输入流指定一个终止字符't', 读取内容不包括终止字符
    cout << line << endl;
    return 0;
} 

int main() {
    int values[] = { 3, 7, 0, 5, 4 };
    ofstream os("integers", ios_base::out | ios_base::binary);
    // 写入二进制文件
    // reinterpret_cast强制类型转换, sizeof(values)返回values占的字节
    os.write(reinterpret_cast<char *>(values), sizeof(values));
    os.close();

    ifstream is("integers", ios_base::in | ios_base::binary);
    if (is) {
        // 跳过3个int字节长度
        is.seekg(3 * sizeof(int));
        int v;
        // 读取二进制文件
        is.read(reinterpret_cast<char *>(&v), sizeof(int));
        cout << "The 4th integer in the file 'integers' is " << v << endl;
    } else {
        cout << "ERROR: Cannot open file 'integers'." << endl;
    }
    return 0;
}
// 遍历文件， 找到其中0所在的位置
int main() {
    ifstream file("integers", ios_base::in | ios_base::binary);
    if (file) {
        while (file) {//读到文件尾file为0
            streampos here = file.tellg(); // 获取当前所在的位置
            int v;
            file.read(reinterpret_cast<char *>(&v), sizeof(int));
            if (file && v == 0) 
            cout << "Position " << here << " is 0" << endl;
        }
    } else {
        cout << "ERROR: Cannot open file 'integers'." << endl;
    }
    file.close();
    return 0;
}
```



istringstream

```cpp
#include <iostream>
#include <iomanip>
#include <sstream>
 
int main()
{
    std::string input = "41 3.14 false hello world";
    std::istringstream stream(input);
    int n;
    double f;
    bool b;
 
    stream >> n >> f >> std::boolalpha >> b; // 通过std::boolalpha可以读写bool量
    std::cout << "n = " << n << '\n'
              << "f = " << f << '\n'
              << "b = " << std::boolalpha << b << '\n';
 
    // extract the rest using the streambuf overload
    stream >> std::cout.rdbuf();
    std::cout << '\n';
}
```



### 输入/输出流

- iostream: 继承自istream和ostream, 是下面两个输入/输出流类的基类
- fstream
- stringstream



## 泛型程序设计及STL

STL: standard template library
- 容器 container
- 迭代器 iterator
- 函数对象 function object
- 算法 algorithms

**note** 通常尾迭代器都是指向最后一个元素的下一个位置, 也即尾迭代器`iter = c.end()`是不存在对应的元素, 可以通过`--iter`得到结尾的元素.



`std::transform`, which defined in `<algorithm>` applies the given function to a range and stores the result in another range, beginning at d_first.
其函数原型有两种形式:

```cpp
template< class InputIt, class OutputIt, class UnaryOperation >
OutputIt transform( InputIt first1, InputIt last1, OutputIt d_first, UnaryOperation unary_op );

template< class InputIt1, class InputIt2, class OutputIt, class BinaryOperation >
OutputIt transform( InputIt1 first1, InputIt1 last1, InputIt2 first2, OutputIt d_first, BinaryOperation binary_op );
```
transform会遍历从起始迭代器`first1`到最终迭代器`last1`之间的元素, 按照`unary_op`(一元函数)或`binary_op`(二元函数)进行运算, 如果需要第二组输入参数, 需给出第二组数据的起始迭代器`first2`, 计算结果保存在输出容器中 需给出输出容器的起始迭代器`d_first`.
其实现过程大致如下:
```cpp
//first
template<class InputIt, class OutputIt, class UnaryOperation>
OutputIt transform(InputIt first1, InputIt last1, OutputIt d_first, 
                   UnaryOperation unary_op)
{
    while (first1 != last1) {
        *d_first++ = unary_op(*first1++);
    }
    return d_first;
}

// second
template<class InputIt1, class InputIt2, 
         class OutputIt, class BinaryOperation>
OutputIt transform(InputIt1 first1, InputIt1 last1, InputIt2 first2, 
                   OutputIt d_first, BinaryOperation binary_op)
{
    while (first1 != last1) {
        *d_first++ = binary_op(*first1++, *first2++);
    }
    return d_first;
}
```

这里`unart_op`和`binary_op`本质上是函数对象(一元和二元), 存在多种形式, 一种是采用`<functional>`中的函数, 一种是用户自定义的函数,(函数可以定义在外部, 也可以直接书写)
如果写在内部, 其句式为: `[](形参表)->输出数据类型 {函数体;}` ; 此外重载了`()`运算符的实例也是函数对象.

```cpp
// transform algorithm example
#include <iostream>     // std::cout
#include <algorithm>    // std::transform
#include <vector>       // std::vector
#include <functional>   // std::plus

int op_increase (int i) { return ++i; } // 用户自定义外部函数对象

class IncreaseClass{  //定义IncreaseClass类
public:
  //重载操作符operator()
    int operator() (int i) const { return ++i; }   
};

int main () {
  std::vector<int> foo;
  std::vector<int> bar;

  // set some values:
  for (int i=1; i<6; i++)
    foo.push_back (i*10);                         // foo: 10 20 30 40 50

  bar.resize(foo.size());                         // allocate space

  std::transform (foo.begin(), foo.end(), bar.begin(), op_increase); // 采用用户自定义的函数
                                                  // bar: 11 21 31 41 51

  // std::plus adds together its two arguments:
  std::transform (foo.begin(), foo.end(), bar.begin(), foo.begin(), std::plus<int>()); // 采用STL中的函数
                                                  // foo: 21 41 61 81 101
  

  // std::plus adds together its two arguments:
  std::transform (foo.begin(), foo.end(), foo.begin(), [](int a)->int { return ++a;}); // 采用自定义函数
                                                  // foo: 22 42 62 82 102
  

  // std::plus adds together its two arguments:
  std::transform (foo.begin(), foo.end(), foo.begin(), IncreaseClass()); 
                                                  // foo: 23 43 63 83 103
                                                  
  std::cout << "foo contains:";
  for (std::vector<int>::iterator it=foo.begin(); it!=foo.end(); ++it)
    std::cout << ' ' << *it;
  std::cout << '\n';

  return 0;
}
```

迭代器和sort函数的用法如下:

```cpp
#include <algorithm>
#include <functional>
#include <array>
#include <iostream>
 
int main()
{
    std::array<int, 10> s = {5, 7, 4, 2, 8, 6, 1, 9, 0, 3}; // 这里array可以用vector直接替代
    // 对于vector: vector<int> v; 用法与array相同
    // 对于数组来说 int a[10]; s.begin() 等效于 a, s.end() 等效于 a+10;
 
    // sort using the default operator<
    std::sort(s.begin(), s.end());
    for (auto a : s) {
        std::cout << a << " ";
    }   
    std::cout << '\n';
 
    // sort using a standard library compare function object
    std::sort(s.begin(), s.end(), std::greater<int>());
    for (auto a : s) {
        std::cout << a << " ";
    }   
    std::cout << '\n';
 
    // sort using a custom function object
    struct {
        bool operator()(int a, int b) const
        {   
            return a < b;
        }   
    } customLess;
    std::sort(s.begin(), s.end(), customLess);
    for (auto a : s) {
        std::cout << a << " ";
    }   
    std::cout << '\n';
 
    // sort using a lambda expression 
    std::sort(s.begin(), s.end(), [](int a, int b) {
        return b < a;   
    });
    for (auto a : s) {
        std::cout << a << " ";
    } 
    std::cout << '\n';
}
```



### `std::istream_iterator`和`std::ostream_iterator`

`std::istream_iterator`是输入流迭代器, `std::ostream_iterator`是输出流迭代器, 二者配合使用可以较为方便的对容器中的数据进行操作.

```cpp
#include <iostream>
#include <iterator>
#include <algorithm>
using namespace std;

int main()
{
  istream_iterator<int> i1(cin); // 通过cin构造输入流迭代器i1, 代表从标准输入设备输入
  istream_iterator<int> i2;	   // 默认构造, 代表输入流的结束位置
  vector<int> s1(i1, i2);		   // 通过输入流迭代器从标准输入流中输入数据
  copy(s1.begin(), s1.end(), ostream_iterator<int>(cout, " "));
  // 将容器s1中的数据输出到输出流迭代器中, 这里采用cout, 即标准输出设备, 其中" "为间隔符
  return 0;
}
```



通过与容器所关联的迭代器类型可以将容器分为**可逆容器**和**随机访问容器**, 可逆容器存在逆向迭代器`rbegin()`和`rend()`分别指向容器尾和容器首, 随机访问容器则可以通过操作符`[]`获取相应顺序下的元素. 典型的随机访问容器包括: `vector`和`deque`. 此外从容器的元素组织方式可以分为顺序容器和关联容器.



### 顺序容器

顺序容器: 通过容器中的相对位置进行操作.

- vector
- deque
- list
- forward_list
- array

STL所提供的顺序容器各有所长也各有所短，我们在编写程序时应当根据我们对容器所需要执行的操作来决定选择哪一种容器。

- 如果需要执行大量的随机访问操作，而且当扩展容器时只需要向容器尾部加入新的元素，就应当选择向量容器vector；
- 如果需要少量的随机访问操作，需要在容器两端插入或删除元素，则应当选择双端队列容器deque；
- 如果不需要对容器进行随机访问，但是需要在中间位置插入或者删除元素，就应当选择列表容器list或forward_list；
- 如果需要数组，array相对于内置数组类型而言，是一种更安全、更容易使用的数组类型。



顺序容器的适配器

- stack栈, 后入先出
- queue队列, 先入先出
- priority_queue优先级队列, 通过用户自定义操作符`<`进行队列内元素比较, 优先pop出最"大"的那个元素. 详见例10-8.




### 函数适配器

- 绑定适配器：bind1st、bind2nd
  - 将n元函数对象的指定参数绑定为一个常数，得到n-1元函数对象
- 组合适配器：not1、not2
  - 将指定谓词的结果取反
- 函数指针适配器：ptr_fun
  - 将一般函数指针转换为函数对象，使之能够作为其它函数适配器的输入。
  - 在进行参数绑定或其他转换的时候，通常需要函数对象的类型信息，例如bind1st和bind2nd要求函数对象必须继承于binary_function类型。但如果传入的是函数指针形式的函数对象，则无法获得函数对象的类型信息。
- 成员函数适配器：ptrfun、ptrfun_ref
  - 对成员函数指针使用，把n元成员函数适配为n + 1元函数对象，该函数对象的第一个参数为调用该成员函数时的目的对象
  - 也就是需要将“object->method()”转为“method(object)”形式。将“object->method(arg1)”转为二元函数“method(object, arg1)

