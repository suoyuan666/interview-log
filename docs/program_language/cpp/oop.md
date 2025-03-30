# OOP

所有 C++ 中和面向对象相关的都在这里

## 1. 介绍一下虚函数

虚函数特殊的成员函数，它允许在基类和派生类之间实现动态多态性。简单来说，虚函数使得在运行时能够根据对象的实际类型调用相应的函数，而不是根据指针或引用的静态类型。

虚函数定义时加上 `= 0` 表示该函数是纯虚函数，该类不提供虚函数的定义，只是提供了该接口，需要继承它的类去实现它。

对于 C++ 来说，编译器会为包含虚函数的类生成一个虚函数表，这个表存储了虚函数的地址，也就是存储了函数指针。这些类都有个虚指针，其指向虚函数表，在正式调用时，程序会通过该指针找到虚函数表的函数，再通过类型转换正确的调用它。

可以参考下边的代码理解虚函数的实现，对于下边的 C++ 代码：

```cpp
class base {
  public:
  virtual void test();
};

class ov : base {
  public:
  virtual void test() override;
};

auto main() -> int {
  base var{};
  ov ele{};
  var.test();
  ele.test();
  return 0;
}
```

它可以被翻译成下边的 C 代码：

```c
/*************************************************************************************
 * NOTE: This an educational hand-rolled transformation. Things can be incorrect or  *
 * buggy.                                                                            *
 *************************************************************************************/
void __cxa_start(void);
void __cxa_atexit(void);
typedef int (*__vptp)();

struct __mptr
{
    short  d;
    short  i;
    __vptp f;
};

extern struct __mptr* __vtbl_array[];


typedef struct base
{
  __mptr * __vptrbase;
} base;

void testbase(base * __this);

inline base * Constructor_base(base * __this)
{
  __this->__vptrbase = __vtbl_array[0];
  return __this;
}


typedef struct ov
{
  __mptr * __vptrbase;
} ov;

void testov(ov * __this);

inline ov * Constructor_ov(ov * __this)
{
  Constructor_base((base *)__this);
  __this->__vptrbase = __vtbl_array[1];
  return __this;
}


int __main(void)
{
  base var;
  Constructor_base((base *)&var);
  ov ele;
  Constructor_ov((ov *)&ele);
  (*((void (*)(base *))((&var)->__vptrbase[0]).f))((((base *)(char *)(&var)) + ((&var)->__vptrbase[0]).d));
  (*((void (*)(ov *))((&ele)->__vptrbase[0]).f))((((ov *)(char *)(&ele)) + ((&ele)->__vptrbase[0]).d));
  return 0;
  /* ele // lifetime ends here */
  /* var // lifetime ends here */
}

int main(void)
{
  __cxa_start();
  int ret = __main();
  __cxa_atexit();
  return ret;
  /* ret // lifetime ends here */
}

__mptr __vtbl_base[1] = {0, 0, (__vptp)testbase};
__mptr __vtbl_ov[1] = {0, 0, (__vptp)testov};

__mptr * __vtbl_array[2] = {__vtbl_base, __vtbl_ov};

void __cxa_start(void)
{
}

void __cxa_atexit(void)
{
}
```

这份翻译来自 [C++ Insights](https://cppinsights.io/)

从这个 C 代码中可以更好的理解虚函数的实现，可以看到每个类都有一个虚指针，而全局存在一个虚函数表，对象在构造函数的时候会初始化这个虚指针，让它指向虚函数表的元素，之后再调用的时候直接用虚指针去找到对应的函数。
