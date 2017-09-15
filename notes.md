## 泛型程序设计及STL

STL: standard template library
- 容器 container
- 迭代器 iterator
- 函数对象 function object
- 算法 algorithms

`std::transform`, which defined in `<algorithm>` applies the given function to a range and stores the result in another range, beginning at d_first.
其函数原型有两种形式:
```cpp
template< class InputIt, class OutputIt, class UnaryOperation >
OutputIt transform( InputIt first1, InputIt last1, OutputIt d_first, UnaryOperation unary_op );

template< class InputIt1, class InputIt2, class OutputIt, class BinaryOperation >
OutputIt transform( InputIt1 first1, InputIt1 last1, InputIt2 first2, OutputIt d_first, BinaryOperation binary_op );
```
transform会遍历从起始迭代器`first1`到最终迭代器`last1`之间的元素, 按照`unary_op`或`binary_op`进行运算, 如果需要第二组输入参数, 需给出第二组数据的起始迭代器`first2`, 计算结果保存在输出容器中 需给出输出容器的起始迭代器`d_first`.
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

这里`unart_op`和`binary_op`存在多种形式, 一种是采用`<functional>`中的函数, 一种是用户自定义的函数,(函数可以定义在外部, 也可以直接书写)
如果写在内部, 其句式为: `[](形参表)->输出数据类型 {函数体;}` 

```cpp
// transform algorithm example
#include <iostream>     // std::cout
#include <algorithm>    // std::transform
#include <vector>       // std::vector
#include <functional>   // std::plus

int op_increase (int i) { return ++i; } // 用户自定义外部函数

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
