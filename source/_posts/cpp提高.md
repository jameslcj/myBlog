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

### 强制类型转换

- static_cast 同类型转换(编译时进行类型检测), 比如int->long
- reinterpret_cast<tpyeName>  不同类型之间转换 可以转换任意一个32bit整数，包括所有的指针和整数
- dynamic_cast<typeName> 多态之间转换
- const_cast<typeName> 去除只读属性

```cpp
void modifyP(const char *p) {
    char *p1 = NULL;
    p1 = const_cast<char *>(p);
    
    p1[0] = 'A';
}

int main(int argc, const char * argv[]) {
    
    char buf[10] = "aaaaaaaaa";
    
    modifyP(buf);
    
    cout << "buf:" << buf << endl;
    
    int a = 1;
    double b;
    char c;
    int* p = NULL;
    
//    p = static_cast<int *>(a);
    p = reinterpret_cast<int *>(a);
    
    cout << "p: " << p << endl;
    
    return 0;
}
```

## 异常处理

```cpp
class ExpObj {
public:
    ExpObj() {
        cout << "构造函数" << endl;
    }
    ~ExpObj() {
        cout << "析构函数" << endl;
    }
};

// throw是限定抛出什么类型的异常
void TestThrow() throw(int, char, ExpObj)
{
    throw ExpObj();
}

int main(int argc, const char * argv[]) {
    
    try {
        TestThrow();
    } catch (char *e) {
        cout << "throw char :" << e << endl;
    } catch (int e) {
        cout << "throw int :" << e << endl;
    } catch (ExpObj &e) { //如果不用引用来接受 会进行拷贝对象
        cout << "throw ExpObj :" << endl;
    } catch (...) {
        cout << "unkown exp"  << endl;
    }

    
    return 0;
}
```

### 自定义异常
```cpp
class MyExp : public exception {
public:
    MyExp(char * p) {
        this->m_p = p;
    }
    virtual const char * what() {
        cout << "MyExp: " << m_p << endl;
        return m_p;
    }
private:
    char* m_p;
};

void testMyExp() {
    throw MyExp("异常抛出");
}

int main(int argc, const char * argv[]) {
    try {
        testMyExp();
    } catch (MyExp &e) {
        e.what();
    }
    return 0;
}
```

### 通过多态实现自定义异常
```cpp
class TestThrowClass {
public:
    class expParent{
    public:
        virtual void printfExp() {
            cout << "expParent" << endl;
        }
    };
    class expSub1 : public expParent{
    public:
        virtual void printfExp() {
            cout << "expSub1" << endl;
        }
    };
    class expSub2 : public expParent{
    public:
        virtual void printfExp() {
            cout << "expSub2" << endl;
        }
    };
    
    void testMethod(int num) {
        if (num < 0) {
            throw expSub1();
        } else if (num > 0) {
            throw expSub2();
        } else {
            throw expParent();
        }
    }
};

int main(int argc, const char * argv[]) {
    TestThrowClass t;
    try {
        t.testMethod(-1);
        
    } catch (TestThrowClass::expParent &e) {
        e.printfExp();
    }
    return 0;
}
```

## IO
```cpp
#include "fstream"
class Teacher {
public:
    Teacher() {
        _age = 18;
        strcpy(_name, "");
    }
    Teacher(char *name, int age) {
        _age = age;
        strcpy(_name, name);
    }
    void printInfo() {
        cout << "name: " << _name << " age: " << _age << endl;
    }
private:
    int _age;
    char _name[32];
};
int main(int argc, const char * argv[]) {
    char *fname = "/Users/xxxx/Desktop/test.txt";
    ofstream fout(fname, ios::binary);
    if (!fout) {
        cout << "open file error";
        return -1;
    }
    Teacher t1("t1", 18);
    Teacher t2("t2", 19);
    fout.write((char*)&t1, sizeof(Teacher));
    fout.write((char*)&t2, sizeof(Teacher));
    fout.close();

    Teacher tmp;
    ifstream fin(fname);
    fin.read((char *)&tmp, sizeof(Teacher));
    tmp.printInfo();
    fin.close();
    return 0;
}
```

## STL
### vector
```cpp
#include "vector"
class Teacher {
public:
    Teacher() {
        _age = 18;
        strcpy(_name, "");
    }
    Teacher(char *name, int age) {
        _age = age;
        strcpy(_name, name);
    }
    void printInfo() {
        cout << "name: " << _name << " age: " << _age << endl;
    }
private:
    int _age;
    char _name[32];
}
void TestVector() {
    Teacher t1("t1", 18);
    Teacher t2("t2", 19);
    vector<Teacher> v;
    v.push_back(t1);
    v.push_back(t2);
    
    for (vector<Teacher>::iterator it = v.begin(); it != v.end(); it++) {
        it->printInfo();
    }
}

int main(int argc, const char * argv[]) {
    TestVector();
    return 0;
}
```

### string
```cpp
#include "string"
#include "algorithm"
void TestString() {
    string s1 = "hello world hello world hello world hello world ";
    string str = "hello";
    string str2 = "HELLO";
    int strlen = str2.length();
    int index = 0;
    while ((index = s1.find(str, index)) != string::npos) {
        s1.replace(index, strlen, str2);
        index += strlen;
    }
    cout << "s1: " << s1 << endl;
    
    str.append(" world");
    transform(str.begin(), str.end(), str.begin(), ::toupper);
    cout << "str: " << str << endl;
    
    str2.insert(0, "nihao, ");
    str2.insert(str2.length(), "!!!");
    transform(str2.begin(), str2.end(), str2.begin(), ::tolower);
    cout << "str2: " << str2 << endl;
}
int main(int argc, const char * argv[]) {
    TestString();
    return 0;
}

```