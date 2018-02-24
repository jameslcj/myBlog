---
title: cpp
date: 2018-02-23 09:54:22
tags: 
---
## 引用
### 别名引用
> 别名引用实质就是一个常量指针 `Type& variable == Type* const variable`

```cpp
int main(int argc, const char * argv[]) {
    int a = 10;
    int &b = a;

    a = 20;
    
    std::cout << "a: " << a << std::endl; //20
    std::cout << "b: " << b << std::endl; // 20
    
    return 0;
}

```

### 引用可做函数返回值
```cpp
int& getA() {
    static int a = 1;
    cout << "a: " << a << endl;
    return a;
}

int main(int argc, const char * argv[]) {
    getA() = 10;
    getA() = 100;
    return 0;
}
```

### 常量引用
> `const Type& variable == const Type* const variable`

```cpp
int main(int argc, const char * argv[]) {
    int a = 1;
    const int &b = a;
    const int &c = 1;
    a = 10;
    // b = 100; //报错
    cout << "a: " << a << " b: " << b << " c:" << c << endl;
    
    return 0;
}
```

## 函数
### inline内联函数
> inline函数可以减少函数进栈出栈所消耗的性能, 一般用于一些简单的逻辑

#### define与inline函数的区别
```cpp
#define FUNC(a, b) ((a) < (b) ? (a) : (b))
inline int FUNC2(int a, int b) {
    return a < b ? a : b;
}

int main(int argc, const char * argv[]) {
    int a = 1;
    int b = 3;
//    int c = FUNC(++a, b); // ((a++) < (b) ? (a++) : (b)) a :3 b:3 c: 3
//    cout << "a :" << a << " b:" << b << " c: " << c << endl;
    
    
    int c = FUNC2(++a, b); //((a++) < (b) ? (a) : (b))  a :2 b:3 c: 2
    cout << "a :" << a << " b:" << b << " c: " << c << endl;
    
    return 0;

```