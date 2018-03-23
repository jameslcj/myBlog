---
title: cpp提高
date: 2018-02-23 09:57:52
tags:
---

## 模板<泛型>
> cpp编译器对模板进行两次编译, 根据程序中用到的类型生成对应的函数

```cpp
template <typename T>
void mySwap(T &a, T &b) {
    a = a ^ b;
    b = a ^ b;
    a = a ^ b;
}

int main(int argc, const char * argv[]) {
    // insert code here...
    int a = 1;
    int b = 2;
    char c = 'c';
    char d = 'd';
    
    mySwap<int>(a, b);
    cout << " a: " << a << " b: " << b << endl;
    mySwap<char>(c, d);
    cout << " c: " << c << " d: " << d << endl;
    return 0;
}
```