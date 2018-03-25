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

### 类模板
```cpp
#include <iostream>
using namespace std;

template <typename T>
class MyVertor2 {
    friend ostream& operator<<(ostream& out, const MyVertor2<T> &obj)
    {
        
        out << "开始重装 <<: " ;
        int len = obj.getLen();
        for (int i = 0; i < len; i++) {
            out << obj.m_space[i] << " ";
        }
        
        out << endl;
        
        return out;
    }

protected:
    int m_len;
    T* m_space;
public:
    MyVertor2(int len = 0);
    MyVertor2(const MyVertor2& obj);
    ~MyVertor2();
public:
    T& operator[](int index);
    MyVertor2& operator=(const MyVertor2& obj);
    int getLen()const;
};

template <typename T>
MyVertor2<T>::MyVertor2(int len)
{
    this->m_len = len;
    this->m_space = new T[len];
}

template <typename T>
MyVertor2<T>::MyVertor2(const MyVertor2& obj)
{
    m_len = obj.m_len;
    m_space = new T[m_len];
    
    for (int i = 0; i < m_len; i++) {
        m_space[i] = obj.m_space[i];
    }
    
    return *this;
}

template <typename T>
MyVertor2<T>::~MyVertor2()
{
    if (m_space != NULL) {
        delete [] m_space;
        m_space = NULL;
        m_len = 0;
    }
}


template <typename T>
T& MyVertor2<T>::operator[](int index)
{
    return m_space[index];
}

template <typename T>
MyVertor2<T>& MyVertor2<T>::operator=(const MyVertor2& obj)
{
    if (m_space != NULL) {
        delete [] m_space;
        m_space = NULL;
        m_len = 0;
    }
    
    m_len = obj.m_len;
    m_space = new T[m_len];
    
    for (int i = 0; i < m_len; i++) {
        m_space[i] = obj.m_space[i];
    }

    return *this;
}

template <typename T>
int MyVertor2<T>::getLen() const
{
    return m_len;
}
class Human {
    
    friend ostream& operator<<(ostream& out, const Human& obj) {
        out << "name: " << obj.m_name << " age:" << obj.m_age << endl;
        return out;
    }
protected:
    char* m_name;
    int m_age;
public:
    Human() {
        m_age = 18;
        m_name = new char[0];
        strcpy(m_name, "");
    }
    Human(char *p_name, int age) {
        int len = strlen(p_name) + 1;
        m_name = new char[len];
        strcpy(m_name, p_name);
        m_age = age;
    }
    ~Human() {
        if (m_name != NULL) {
            delete m_name;
            m_name = NULL;
        }
    }
    Human& operator=(const Human& obj) {
        if (m_name != NULL) {
            delete m_name;
            m_name = NULL;
        }
        
        int len = strlen(obj.m_name) + 1;
        m_name = new char[len];
        strcpy(m_name, obj.m_name);
        
        m_age = obj.m_age;
        return *this;
    }
};

int main(int argc, const char * argv[]) {
    Human h1("h1", 1);
    Human h2("h2", 2);
    MyVertor2<Human> v1(1);
    v1[0] = h1;
    
    MyVertor2<Human> v2;
    v2 = v1;
    
    
    cout << v2[0];
    
    return 0;
}
```