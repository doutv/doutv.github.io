---
title: learning-notes-november
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
categories:
  - - Algorithm
date: 2020-11-29 00:27:50
summary:
---

# 移动语义及拷贝优化
来源：
- https://theonegis.github.io/cxx/C-%E7%A7%BB%E5%8A%A8%E8%AF%AD%E4%B9%89%E5%8F%8A%E6%8B%B7%E8%B4%9D%E4%BC%98%E5%8C%96/
- https://jonasdevlieghere.com/guaranteed-copy-elision/

- lvalue & 可寻址的变量
- rvalue && 不可寻址的字面常量或者是在表达式求值过程中创建的可寻址的无名临时对象

## 移动构造copy constructor和移动赋值move constructor
```cpp
struct Foo {
  Foo() { cout << "Constructed" << endl; }
  Foo(const Foo &) { cout << "Copy-constructed" << endl; } // lvalue
  Foo(Foo &&) { cout << "Move-constructed" << endl; } // rvalue
  ~Foo() { cout << "Destructed" << endl; }
};
```
拷贝构造函数中是对传进来的对象进行了实实在在的拷贝工作；而移动构造函数中只是对传进来的对象进行了所有权的转让，即掏空传进来的对象，然后把所有权转给当前对象。

## std::move
编译器只对右值引用才能调用转移构造函数和转移赋值函数，而所有命名对象都只能是左值引用。如果已知一个命名对象不再被使用而想对它调用转移构造函数和转移赋值函数，也就是把一个左值引用当做右值引用来使用，应该使用`std::move`，这个函数以非常简单的方式将左值引用转换为右值引用。

`std::move`的实现即使将一个对象强制转型为右值引用类型的对象而已，并不做任何移动工作。

## 拷贝优化Copy Elision
减少不必要的拷贝构造
+ RVO Regular Return Value Optimization: 
  + If you return an object by value, the compiler is allowed to elude copying.
  + ``` 
    Foo f() {
    return Foo();
    }
    ```
+ NRVO Named Return Value Optimization: 
  + ```
    Foo f() {
        Foo foo;
        return foo;
    }
    ```
+ Guaranteed copy elision
  + 为什么需要 Guaranteed copy elision ？因为要写 copy constructor 和 move constructor
  + The main problem is that, without guaranteed elision, you cannot get rid of the move and copy constructor, because elision might not take place.
  + https://zhuanlan.zhihu.com/p/22821671

# 类型转换
- `const_cast` 转型掉（移除）常量性或易变性
- `static_cast` 一般类型转换，用隐式和用户定义转换的组合在类型间转换
- `dynamic_cast` 类指针转换，沿继承层级向上、向下及侧向，安全地转换到其他类的指针和引用
- `reinterpret_cast` 强制转换，通过重新解释底层位模式在类型间转换

# overload operator
参考：https://zh.cppreference.com/w/cpp/language/operators
pay attention to signatures and parameters
- +
  - 复制赋值 copy assignment
    - `T& T::operator=(const T&)`
  - 移动赋值 move assignment
    - `T& T::operator=(T&&)`
- +=
- =

# ub undefined behavior

