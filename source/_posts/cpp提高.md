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
template <typename T>
class MyVertor {
//    friend ostream& operator<< <T>(ostream& out, const MyVertor& obj);
    friend ostream& operator<< (ostream& out, const MyVertor<T>& obj) {
        int size = obj.getSize();
        
        out << "operator << : ";
        
        for (int i = 0; i < size; i ++) {
            out << obj.m_space[i] << " ";
        }
        
        return out;
    }
public:
    
    int getSize() const
    {
        return m_size;
    }
    
    T& getSpace()
    {
        return m_space;
    }

public:
    MyVertor<T>(int size = 0)
    {
        m_size = size;
        m_space = new T[size];
    }
    MyVertor<T>(const MyVertor<T> &obj)
    {
        int size = obj.m_size;
        m_size = size;
        m_space = new T[m_size];
        for (int i = 0; i < size; i ++) {
            m_space[i] = obj.m_space[i];
        }
    }
    ~MyVertor<T>() {
        if (m_space != NULL) {
            delete [] m_space;
            m_space = NULL;
            m_size = 0;
        }
    }
//
public:
    T& operator[](int index)
    {
        return m_space[index];
    }

    MyVertor<T>& operator=(const MyVertor<T>& obj)
    {
        if (m_space != NULL) {
            delete[] m_space;
            m_space = NULL;
            m_size = 0;
        }
        int size = obj.getSize();
        m_size = size;
        m_space = new T[m_size];
        for (int i = 0; i < size; i ++) {
            m_space[i] = obj.m_space[i];
        }
        
        
        return *this;
    }

   

protected:
    int m_size;
    T* m_space = NULL;
    
};

//template <typename T>
//ostream& operator<< (ostream& out, const MyVertor<T>& obj) {
//    int size = obj.getSize();
//    
//    for (int i = 0; i < size; i ++) {
//        out << obj.m_space[i] << " ";
//    }
//    
//    return out;
//}

int main(int argc, const char * argv[]) {
    MyVertor<int> m1(10), m2(10);
    for (int i = 0; i < 10; i ++) {
        m1[i] = i;
        cout << m1[i] << " ";

    }
    
    cout << endl;
    
    m2 = m1;
    
    MyVertor<int> m3 = m1;
    
    cout << m3;    return 0;
}
```