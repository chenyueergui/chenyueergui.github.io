---
title: c++/efficiency_c++
published: 2025-05-27
description: 'effiency write in the modern c++'
image: ''
tags: [cplusplus, style， book]
category: 'cplusplus'
draft: false 
lang: ''
---

# 序言

本专题记录有关c++ 的高效编程习惯

# c 语言 -> c++ 语言的转变

## item 1 使用inline 和 const 来代替 #define
#define 这样的宏是c语言的特性，会有诸多不太友好的方面，如
`#define Apple 1`这样的语句可能会导致后续程序在1 处报错，但是全文却找不到Apple, 
`#define max(a, b) {(a) > (b)? (a): (b)}`这样的语句可能对于`max(++a, b + 10)`这样的语句有不同的结构，因此在类内，尽量使用`static const`来代替`#define`, 在类外定义函数的时候，尽量使用`inline` + `template`的形式来代替`#define`

```c++
template<typename T>
inline const T& max(const T& a, const T& b) {
    return a > b? a: b;
}
```

## item2 使用 `cin` `cout` 而不是`scanf` `printf`

scanf printf 这样的是上世纪c语言的产物，会要求你记住很多%d, %x, %u 之类很`weird`的事情，使用cin , cout 可以让你忽略这些细节，只需要为类实现 <<, >> 操作即可

> [!NOTE]
> 注意这里的<<, >> 并不是member function 而是friend function

```c++
class Rational {
    public:
        Rational(int numerator = 0, intdenominator = 1);
    private:
        int n, d;
    friend ostream& operator<<(ostream& s, const Rational& r);
};

ostream& operator<<(ostream& s, const Rational& r) {
    s << r.n << "/" << r.d;
    return s;
}
```

## item3 使用`new` and `delete` 而不是`malloc` 和 `free`

malloc 和 free 不会调用constructors 和 destructors 会导致很多意想不到的问题，而new, delete 会调用constructors 和 destructors 从而使得申请内存，释放内存更加合适


## item4 尽量使用c++风格的comments

```c++
if (a > b) {
    // int temp = a; // swap a and b
    // a = b;
    // b = temp;
}
```

上述的写法使用c++风格的注释不存在问题， 但是如果使用c语言风格的注释就会出现语法错误

```c
if (a > b) {
    /*
    int temp = a; /* swap a and b */
    a = b;
    b = temp;
    */
}
```

未完待续
---

