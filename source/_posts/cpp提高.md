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

### 不同类型静态变量
> 不同类型的静态变量 各分配内存

```cpp
template <typename T>
class TestA {
public:
    static T a;
};

//这句必须要申明
template <typename T>
T TestA<T>::a = 0;

int main(int argc, const char * argv[]) {
    // insert code here...
    TestA<int> a;
    a.a = 10;
    TestA<char> b;
    b.a = 'a';
    
    cout << "TestA<int>::a " << TestA<int>::a << " TestA<char>::a " << TestA<char>::a << endl;//TestA<int>::a 10 TestA<char>::a a

    return 0;
}

```