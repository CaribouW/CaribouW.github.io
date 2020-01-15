---
title: c++新特性
date: 2020-01-15 10:14:27
tags: 整理
description: 虽说是新特性，但是C++ 11 & 14 已经推出了将近10年了.这次就好好整理一下
---

## What’s new in C++ 11?

###  storage duration specifier

多个C++版本都对变量存储时间的定义有严格的说明

- `auto` - *automatic* storage duration.

- `register` - *automatic* storage duration. Also hints to the compiler to place the object in the processor's register. (deprecated) ——*since C++ 17*

- `static` - *static* or *thread* storage duration and *internal* linkage

- `extern` - *static* or *thread* storage duration and *external* linkage.

- `thread_local` - *thread* storage duration

  The `thread_local` keyword is only allowed for objects declared at namespace scope, objects declared at block scope, and static data members. It indicates that the object has **thread storage duration**. It can be combined with `static` or `extern` to specify internal or external linkage (except for static data members which always have external linkage), respectively, but that additional `static` doesn't affect the storage duration.

- `mutable` - does not affect storage duration or linkage

  注意：`mutable` 主要是标识 **const对象** 中某些可变的成员，实现了从二进制的物理 **const** 到逻辑 **const** (外观不变)

### Variadic templates

可变类模板，在 `c++ reference` 里面以 *parameter pack* 来代替

```cpp
template<class... Types>			//class ... Types 是一个 pack 的声明	
void f(Types... args) {}			//Types... args 是
```

#### Pack Expansion

- `&args...` 代表的是参数扩展
- `&args`代表的是 *pack pattern* 本身

### Move semantics

> **凡是取地址（`&`）操作可以成功的都是左值，其余都是右值**
>
> - 等号左边的不一定是左值——可以通过操作符重载来让左部变成 **右值**

#### 右值引用

我们之前常见的都是 **左值引用**，指的是我们只能够将 **左值** 赋给一个引用。

-  `int& a = 1` 就是非法的。而我们也可以 `int const& i = 42;` 来进行一个 *tricky* 的躲避

  在 `c++11`中允许了右值引用的出现
```
int&& a = 3;
```

#### 移动语义

我们在进行 `Test(const Test& test)` 拷贝构造函数的执行过程中，会造成很多的拷贝浪费。当单独的一次拷贝构造函数 (特别是对指针数组这些资源)过程消耗特别大的时候，对于性能来说是很低的。

那么 `c++11`标准给出了 **移动拷贝构造函数** 和 **移动赋值操作符重载**。此外，还支持 `std::move()`来强制性让 **左值转化为右值**

#### 完美转发

我们想要实现

> 多级函数调用过程中
>
> - 如果变量是左值，那么它作为其他函数的参数的时候也应该是 **左值**
> - 如果变量是右值，那么它作为其他函数的参数的时候也应该是 **右值**

```cpp
template <typename T>
void func(T t) {
    cout << "in func" << endl;
}

template <typename T>
void relay(T&& t) {
    cout << "in relay" << endl;
    func(t);
}

int main() {
    relay(Test());
}
```

上面这个例子就是一个 **反例**。我们在传入 `func(t)` 的时候，其实调用了 **拷贝构造函数** (因为编译器把 **t** 当做了一个 **左值**)

> `std::forward<T>()` ，能够保留参数的左右值类型

### Value Category

不止左值右值那么简单

> Cpp reference:
>
> Each C++ [expression](https://en.cppreference.com/w/cpp/language/expressions) (an operator with its operands, a literal, a variable name, etc.) is characterized by two independent properties: a *type* and a *value category*. Each expression has some non-reference type, and each expression belongs to exactly one of the three primary value categories: *prvalue*, *xvalue*, and *lvalue*.



