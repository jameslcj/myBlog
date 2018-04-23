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

### list
```cpp
#include "list"
void printList(list<int> l) {
    for (list<int>::iterator it = l.begin(); it != l.end(); it++) {
        cout << *it << " ";
    }
    cout << endl;
}
void TestList() {
    list<int> l;
    for (int i = 0; i < 10; i++) {
        l.push_back(i);
    }
    printList(l);
    
    l.insert(l.begin(), -1);
    l.insert(l.begin(), -2);
    l.insert(l.begin(), -3);
    printList(l);
    
    l.erase(l.begin());
    printList(l);
    
    list<int>::iterator it1 = l.begin();
    list<int>::iterator it2 = l.begin();
    it2 ++;
    it2 ++; //必须一次次叠加不能直接+2
    l.erase(it1, it2);
    printList(l);
    
    l.remove(5);
    printList(l);
}
int main(int argc, const char * argv[]) {
    TestList();
    return 0;
}
```

### queue
```cpp
#include "queue"
void fillQueue(priority_queue<int> &q) {
    q.push(1);
    q.push(10);
    q.push(4);
    q.push(8);
}
void fillQueue2(priority_queue<int, vector<int>, less<int>> &q) {
    q.push(1);
    q.push(10);
    q.push(4);
    q.push(8);
}
void fillQueue3(priority_queue<int, vector<int>, greater<int>> &q) {
    q.push(1);
    q.push(10);
    q.push(4);
    q.push(8);
}

void printQueue(priority_queue<int> q) {
    while (!q.empty()) {
        int tmp = q.top();
        cout << tmp << " ";
        q.pop();
    }
    cout << endl;
}
void printQueue2(priority_queue<int, vector<int>, less<int>> q) {
    while (!q.empty()) {
        int tmp = q.top();
        cout << tmp << " ";
        q.pop();
    }
    cout << endl;
}
void printQueue3(priority_queue<int, vector<int>, greater<int>> q) {
    while (!q.empty()) {
        int tmp = q.top();
        cout << tmp << " ";
        q.pop();
    }
    cout << endl;
}
void TestQueue() {
    priority_queue<int> q1;//默认最大值优先队列
    priority_queue<int, vector<int>, less<int>> q2; //最大值优先队列
    priority_queue<int, vector<int>, greater<int> > q3;//最小值优先队列
    
    fillQueue(q1);
    fillQueue2(q2);
    fillQueue3(q3);
    
    printQueue(q1);
    printQueue2(q2);
    printQueue3(q3);
}
int main(int argc, const char * argv[]) {
    TestQueue();
    return 0;
}
```

### set/multiset
```cpp
#include "set"
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
    int getAge() const {
        return _age;
    }
    const char* getName() const {
        return _name;
    }
private:
    int _age;
    char _name[32];
};
struct CustOrder {
    bool operator()(const Teacher& left, const Teacher& right) {
        return left.getAge() < right.getAge();
    }
};
void insertSet(set<Teacher, CustOrder>&s, Teacher &t) {
    pair<set<Teacher, CustOrder>::iterator, bool> pair = s.insert(t);
    if (pair.second == true) {
        cout << t.getName() << " 插入成功" << endl;
    } else {
        cout << t.getName() << " 插入失败" << endl;

    }
}
void TestSet() {
    Teacher t1("t1", 18);
    Teacher t2("t2", 38);
    Teacher t3("t3", 28);
    Teacher t4("t4", 8);
    Teacher t5("t5", 18);
    Teacher t6("t6", 58);
    set<Teacher, CustOrder> s1;
    insertSet(s1, t1);
    insertSet(s1, t2);
    insertSet(s1, t3);
    insertSet(s1, t4);
    insertSet(s1, t5);
    insertSet(s1, t6);
    
    for (set<Teacher, CustOrder>::iterator it = s1.begin(); it != s1.end(); it++) {
        cout << "name: " << it->getName() << " age:" << it->getAge() << endl;
    }
}
int main(int argc, const char * argv[]) {
    TestSet();
    return 0;
}
```

### map/multimap
```cpp
#include "map"
#include "string"

void TestMap() {
    map<string, int> m1;
    m1.insert(pair<string, int>("m1", 18));
    m1.insert(make_pair("m2", 19));
    m1.insert(map<string, int>::value_type("m3", 20));
    m1["m4"] = 21;
    
    while (!m1.empty()) {
        map<string, int>::iterator it = m1.begin();
        cout << "name: " << it->first << " age: " << it->second << endl;
        m1.erase(it);
    }
    
    
    Teacher t1("t1", 18);
    Teacher t2("t2", 19);
    Teacher t3("t3", 20);
    Teacher t4("t4", 21);
    multimap<string, Teacher> m2;
    m2.insert(make_pair("js", t1));
    m2.insert(make_pair("js", t2));
    m2.insert(make_pair("cpp", t3));
    m2.insert(make_pair("cpp", t4));
    
    for (multimap<string, Teacher>::iterator it = m2.begin(); it != m2.end(); it ++) {
        cout << "type: " << it->first << " info: name:" << it->second.getName() << " age: " << it->second.getAge() << endl;
    }
}

int main(int argc, const char * argv[]) {
    TestMap();
    return 0;
}
```

### 算法
#### for_each
```cpp
template <typename T>
class ShowElement {
public:
    int num = 0;
    void operator()(T &t) {
        cout << t << endl;
        num++;
    }
    
    void showNum() {
        cout << "num: " << num << endl;
    }
};

template <typename T>
void showElementFunc(T &t) {
    cout << t << endl;
}

void testCallBack() {    vector<int> v;
    v.push_back(1);
    v.push_back(3);
    v.push_back(5);
    
    for_each(v.begin(), v.end(), ShowElement<int>());
    for_each(v.begin(), v.end(), showElementFunc<int>);
    
    //for_each函数是值传递不是引用传递
    ShowElement<int> showElement;
    ShowElement<int> showElement2 = for_each(v.begin(), v.end(), showElement);
    showElement.showNum(); //0
    showElement2.showNum();//3
}

int main(int argc, const char * argv[]) {
    testCallBack();
    return 0;
}
```

#### 谓词/find_if 
```cpp
template <typename T>
class IsDiv {
public:
    T div;
    IsDiv(T &t) {
        div = t;
    }
    
    bool operator()(T &t) {
        return (t % div == 0);
    }
};

void testPred() {
    vector<int> v;
    for (int i = 33; i <= 66; i++) {
        v.push_back(i);
    }
    
    int div = 5;
    
    vector<int>::iterator it =  find_if(v.begin(), v.end(), IsDiv<int>(div));
    if (it == v.end()) {
        cout << "没有找到能被 " << div << " 整除的数" << endl;
    } else {
        cout << "能被 " << div << " 整除的数是 " << *it << endl;
    }
}

int main(int argc, const char * argv[]) {
    testPred();
    return 0;
}

```

### functional
> for_each 的回调函数返回值可以是void, transform 必须有返回值

```cpp
#include "functional"
int increase(int num) {
    return num + 10;
}
template <typename T>
void printV(vector<T> v) {
    cout << "vector: ";
    for (int i = 0; i < v.size(); i++) {
        cout << v[i] << " ";
    }
    cout << endl;
}
void testFunc() {
    vector<int> v;
    for (int i = 0; i < 10; i ++) {
        v.push_back(i);
    }
    
    int num = 4;
    int count = count_if(v.begin(), v.end(), bind2nd(greater<int>(), num));
    cout << "大于 " << num << " 的个数为 " << count << endl;
    
    count = count_if(v.begin(), v.end(), bind2nd(modulus<int>(), num));
    cout << "能被 " << num << " 整除的个数为 " << count << endl;
    
    count = count_if(v.begin(), v.end(), not1(bind2nd(modulus<int>(), num)));
    cout << "不能被 " << num << " 整除的个数为 " << count << endl;
    
    transform(v.begin(), v.end(), v.begin(), increase);
    printV(v);
}
int main(int argc, const char * argv[]) {
    testFunc();
    return 0;
}
```