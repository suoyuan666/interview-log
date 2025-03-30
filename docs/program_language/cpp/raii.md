# RAII

本篇包含 C++ 中有关 RAII 思想的部分

> RAII（Resource Acquisition Is Initialization，资源获取即初始化）是 C++ 中一种重要的编程惯用法，它将资源的获取与对象的生命周期绑定在一起，从而实现自动的资源管理。
>
> RAII 的实现主要依赖于构造函数和析构函数，从而实现资源的自动获取和释放。RAII 主要用来防止内存泄露。

## 1. 了解智能指针吗

- 智能指针是一种类模板，它封装了原始指针，并自动管理所指向的内存。
- 智能指针通过在对象销毁时自动释放内存，从而避免了手动释放内存可能导致的内存泄漏。
- 智能指针的类型
    - `std::unique_ptr`
        - `unique_ptr` 独占它所指向的对象，即同一时间只能有一个 `unique_ptr` 指向该对象
        - `unique_ptr` 不可被复制，只能被移动(move)
    - `std::shared_ptr`
        - `shared_ptr` 允许多个 `shared_ptr` 共享同一个对象
        - `shared_ptr` 通过引用计数跟踪有多少 `shared_ptr` 指向同一个对象
        - 最后指向对象的 `shared_ptr` 被销毁时，该对象才会被销毁
    - `std::weak_ptr`
        - 类似 `std::shared_ptr`，但没有引用计数
        - 通常和 `shared_ptr` 一起用，来解决 `shared_ptr` 的循环引用问题

- shared_ptr 存在循环引用的隐患，示例代码如下:

```cpp
#include <iostream>
#include <memory>

class A;
class B;

class A {
public:
    std::shared_ptr<B> bPtr;
};

class B {
public:
    std::shared_ptr<A> aPtr;
};

int main() {
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();

    a->bPtr = b;
    b->aPtr = a;

    return 0;
}
```

这里的问题是 `a` 和 `b` 会互相引用，从而无法被销毁。解决方案就是把 `std::shared_ptr<A> aPtr` 改成 `std::weak_ptr<A> aPtr`。`weak_ptr` 并不会增加其引用计数。
