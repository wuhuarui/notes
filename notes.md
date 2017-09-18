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




## 函数适配器

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

