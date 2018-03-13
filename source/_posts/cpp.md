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

## 构造方法与析构方法
```cpp
class myClass {
    int _age;
public:
    myClass() {
        std::cout << "无参构造";
    }
    myClass(int age) {
        _age = age;
    }
    myClass(const myClass &obj) {
        _age = obj._age;
    }

    ~myClass() {
        std::cout << "我是析构方法" << std::endl;
    }
    void myClass::setAge(int age){
        _age = age;
    }
    int getAge() {
        return _age;
    }
};
int main(int argc, const char * argv[]) {
    myClass c(18);
    myClass d = c;
    myClass e;
    cout << "age: " << c.getAge() <<endl;
    cout << "age: " << d.getAge() <<endl;
    return 0;
}


```

### 当一个类含有变量是类并有有参构造方法
> 必须要初始化参数类的构造方法

```cpp
class c1 {
public:
    c1(int a) {
        cout << "c1 构造方法 " << endl;
    }
    ~c1() {
        cout << "c1 析构方法" << endl;
    }
};
class c2 {
public:
    c1 a1;
    c1 a2;
    c2(int m1, int m2, int m3) : a1(m2), a2(m3)
    {
        cout << "c2 构造方法 " << endl;
    }
    ~c2() {
        cout << "c2 析构方法" << endl;
    }
};

int main(int argc, const char * argv[]) {
    c2 a2(1, 2, 3);
    /**
    c1 构造方法
    c1 构造方法 
    c2 构造方法 
    c2 析构方法
    c1 析构方法
    c1 析构方法
    **/
    return 0;
}

```

### 构造方法里调构造方法问题
> 会直接初始化一个类, 然后直接析构掉

```cpp
class c3 {
public:
    int _a;
    int _b;
    int _c;
    c3(int a, int b) {
        cout << "2个参数构造方法" << endl;
        c3(a, b, 100);
    }
    c3 (int a, int b, int c) {
        cout << "3个参数构造方法" << endl;
        _a = a;
        _b = b;
        _c = c;
    }
    
    void printParmas() {
        cout << "_a: " << _a << " _b: " << _b << " _c: " << _c << endl;
    }
    
    ~c3() {
        cout << "析构方法被调用" << endl;
    }
};
int main(int argc, const char * argv[]) {
    c3 c(1, 2);
    c.printParmas();
    /***
    2个参数构造方法
    3个参数构造方法
    析构方法被调用
    _a: 0 _b: 0 _c: 0
    析构方法被调用
    **/
    return 0;
}
```

### 类中的static变量
> 同一个类的所有对象公用同一个static变量

```cpp
class c4 {
public:
    int a = 0;
    static int b;
    void coutParams() {
        cout << "a: " << a << " b: " << b << endl;
    }
    void addParams() {
        a ++;
        b ++;
    }
};
int c4::b = 0; //必须在外面声明
int main(int argc, const char * argv[]) {
    c4 c1, c2;
    c1.coutParams();
    c2.coutParams();
    c1.addParams();
    c2.addParams();
    c1.coutParams();
    c2.coutParams();
    return 0;
}


```

### 类中的static方法
> static方法中只能使用static变量

```cpp
class c5 {
public:
    int a = 0;
    static int b;
    //static方法中只能使用static变量
    static void coutParams() {
        cout << " b: " << b << endl;
    }
    void addParams() {
        a ++;
        b ++;
    }
};
int c5::b = 0;
int main(int argc, const char * argv[]) {
    c5 c1, c2;
    c1.coutParams();
    c1.addParams();
    c5::coutParams();
    c1.coutParams();
    return 0;
}
```

### 类的大小
> 下面这个类对象只占4字节而不是16字节, 他们是怎么分配内存的呢? 其实c5是用结构体实现的, 只有a变量是放在结构体c1的内存里, 静态变量b和静态方法coutParams都放在全局内存区, addParams放在代码内存区; 
那么问题来了, 那调用addParams时如何区别是哪个对象调用的呢? 其实cpp编译会将当前对象以一个变量名为`this`的变量传递给方法, 因此这也是为什么我们能再方法内使用`this`变量来调用对象的变量和方法

```cpp
class c5 {
public:
    int a = 0;
    static int b;
    static void coutParams() {
        cout << " b: " << b << endl;
    }
    void addParams() {
        a ++;
        b ++;
    }
};
int c5::b = 0;
int main(int argc, const char * argv[]) {
    c5 c1;
    
    cout << "c1 size: " << sizeof(c1) << endl; // 4
    return 0;
}


// c5转换为c代码是如下
struct c5{
    int a = 0;
}
//代码区
void addParams(const c5 this) {
    a ++;
    b ++;
}
//静态区
static int b;
static void coutParams() {
    cout << " b: " << b << endl;
}
```

### new和delete
> new/delete与malloc/free 功能类似, 都是向堆区申请内存, 并且可以混用, 但是new会调用类的构造方法, delete会调用类的析构方法

```cpp
int main(int argc, const char * argv[]) {
   int * p1 = new int(11);
    char * p2 = new char[100];
    strcpy(p2, "hello world");
    c3* p3 = new c3(1, 2);
    cout << "p1: " << *p1 << endl;
    
    cout << "p2: " << p2 << endl;
    
    delete p1;
    delete []p2;
    delete p3;
    return 0;
}

```

### 类的const
> void setA() const ==> void setA(const c1 * const this)

```cpp
class c1 {
public:
    int a;
    void setA() const
    {
        // this->a = 100; 这里会报错 因为有const修饰
    }
    int getA() {
        return this->a;
    }
};

int main(int argc, const char * argv[]) {
    c1 c;
    c.setA();
    cout<< " a: " << c.getA() << endl;
    return 0;
}
```

### 方法返回类引用
```cpp
class C1 {
public:
    int a;
    C1(int a = 0) {
        this->a = a;
    }
    C1& add(C1 c2) {
        this->a = this->a + c2.a;
        
        return *this;
    }
    
    void printA() {
        cout << " a: " << this->a << endl;
    }
};

int main(int argc, const char * argv[]) {
    C1 c1(1), c2(2);
    c1.add(c2);
    c1.printA(); // a: 3
    c2.printA(); // a: 2
    return 0;
}

```

### 友元函数
> 通过friend申明为友元方法, 可以使用外部函数直接修改类的私有变量

```cpp
class C1 {
private:
    int a;
public:
    friend void modifyA(C1 &c, int _a);
    C1(int a = 0) {
        this->a = a;
    }
    C1& add(C1 c2) {
        this->a = this->a + c2.a;
        
        return *this;
    }
    
    void printA() {
        cout << " a: " << this->a << endl;
    }
};

void modifyA(C1 &c, int _a) {
    c.a = _a;
}

int main(int argc, const char * argv[]) {
    C1 c1(1);
    c1.printA();
    modifyA(c1, 100);
    c1.printA();
    return 0;
}
```

### 友元类
```cpp
class C1 {
private:
    int a;
public:
    friend class C2;
    C1(int a = 0) {
        this->a = a;
    }
    C1& add(C1 c2) {
        this->a = this->a + c2.a;
        
        return *this;
    }
    
    void printA() {
        cout << " a: " << this->a << endl;
    }
};

class C2 {
private:
    C1 c;
public:
    void setC1A(int _a) {
        c.a = _a;
    }
    void printA() {
        cout << " a: " << c.a << endl;
    }
};

int main(int argc, const char * argv[]) {
    C2 c2;
    c2.printA();
    c2.setC1A(100);
    c2.printA();
    return 0;
}
```

### 类的操作符重载
```cpp

class C3 {
    int* arr;
    int a = 0;
    //因为是私有变量所以需要将全局函数声明为友元函数
    friend C3 operator+(C3 &c1, C3 &c2);
    friend C3 operator++(C3 &c1, int);
public:
    C3(int a) {
        this->a = a;
        this->arr = new int[100];
    }

    void printA() {
        cout << "a: " << this->a << endl;
    }

    //第一种 用类方法声明
    C3 operator-(C3 &c2) {
        C3 tmp(0);
        tmp.a = this->a - c2.a;

        return tmp;
    }
    
    //一元操作符重载
    C3& operator++() {
        this->a++;
        
        return *this;
    }

    //后置--
    C3 operator--(int) {
        C3 tmp = *this;
        this->a--;
        
        return tmp;
    }

    int& operator[](int i) {
        this->arr[i] = i;
        
        return i;
    }
};
//第二种 使用全局函数声明
C3 operator+(C3 &c1, C3 &c2) {
    C3 tmp(0);
    tmp.a = c1.a + c2.a;

    return tmp;
}
C3 operator++(C3 &c1, int){
    C3 tmp = c1;
    c1.a ++;
    return tmp;
}
int main(int argc, const char * argv[]) {
    C3 c1(1), c2(2);

    C3 c3 = c1 + c2;
    c3.printA();
    C3 c4 = c1 - c2;
    c4.printA();
    ++c4;
    c4.printA();
    for (int i = 0; i < 5; i ++) {
        c4[i] = i;
        cout << c4[i] << endl;
    }
    return 0;
}
```

## 类继承
类继承也有三种修饰符public, protected, private
- public继承, 就是默认继承
- protected继承, public属性和方法都会变成protected
- private继承, public, protected属性和方法都会变成private
### 属性的重写
```cpp
class Parent {
public:
    int a;
protected:
    int b;
private:
    int c;
public:
    void printAll() {
        cout << "a: " << a << " b:" << b << " c:" << c << endl;
    }
};
class Son : public Parent{
public:
    int b;
    void printAll2() {
        cout << "a: " << a << " b:" << b  << endl;
    }
};

int main(int argc, const char * argv[]) {
    Son s1;
    s1.a = 1;
    s1.b = 2;
    s1.Parent::a = 2;
    s1.printAll(); //a: 2 b:0 c:0
    s1.printAll2(); //a: 1 b:2

    return 0;
}

```

### 继承的构造方法和析构方法执行顺序
```cpp
class Object {
public:
    int o1;
    int o2;
    Object(int o1, int o2) {
        this->o1 = o1;
        this->o2 = o2;
        cout << "Object 构造方法:  o1: " << o1 << " o2: " << o2 << endl;
    }
    ~Object() {
        cout << "Object 析构方法:  o1: " << o1 << " o2: " << o2 << endl;
    }
};
class Parent: public Object {
public:
    int a;
protected:
    int b;
private:
    int c;
public:
    Parent(int a, int b) : Object(a + 1, b + 1) {
        this->a = a;
        this->b = b;
        cout << "Parent 构造方法:  a: " << a << " b: " << b << endl;
    }
    ~Parent() {
        cout << "Parent 析构方法:  a: " << a << " b: " << b << endl;
    }
    void printAll() {
        cout << "a: " << a << " b:" << b << " c:" << c << endl;
    }
};
class Son : public Parent{
public:
    int b;
    Object o1;
    Object o2;
    Son(int a, int b) : o2(a, 2), Parent(a+1, b + 1), o1(a, b) {
        cout << "Son 构造方法:  a: " << a << " b: " << b << endl;
    }
    ~Son() {
        cout << "Son 析构方法" << endl;
    }
    void printAll2() {
        cout << "a: " << a << " b:" << b  << endl;
    }
};

int main(int argc, const char * argv[]) {
    Son s1(1, 2);
    /**
     Object 构造方法:  o1: 3 o2: 4
    Parent 构造方法:  a: 2 b: 3
    Object 构造方法:  o1: 1 o2: 2
    Object 构造方法:  o1: 1 o2: 2
    Son 构造方法:  a: 1 b: 2
    Son 析构方法
    Object 析构方法:  o1: 1 o2: 2
    Object 析构方法:  o1: 1 o2: 2
    Parent 析构方法:  a: 2 b: 3
    Object 析构方法:  o1: 3 o2: 4
    **/
    return 0;
}
```

### 虚继承与继承二义性
> 如下, B1与B2都继承了A, C又同时继承了B1和B2, 这样就产生了二义性, 
为了防止产生二义性应该给B1, B2添加virtual, 表示虚继承, 这样A的构造函数只会被执行一次,
虚继承会给类多加了8字节的数据;
如果在2个类中出现同样名称的变量, 只能通过 `obj.class::var`的方式来解决二义性

```cpp
cclass A {
public:
    int a;
    A() {
        cout << "I am A" << endl;
    }
};

class B1 : virtual public A {
public:
    int b;
    
};

class B2 : virtual public A {
public:
    int b;
};

class C4: public B1, public B2 {
public:
    int c;
};

int main(int argc, const char * argv[]) {
    C4 c;
    c.a = 1;
    c.B1::b = 2;
    c.B2::b = 3;
    cout << "sizeof(A): " << sizeof(A) << endl;
    cout << "sizeof(B1): " << sizeof(B1) << endl;
    cout << "sizeof(B2): " << sizeof(B2) << endl;
    cout << "sizeof(C4): " << sizeof(C4) << endl;
    /**
    I am A
    sizeof(A): 4
    sizeof(B1): 16
    sizeof(B2): 16
    sizeof(C4): 40
    /*
    return 0;
}
```