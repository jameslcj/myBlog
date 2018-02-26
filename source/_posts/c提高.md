---
title: C提高
date: 2018-02-11 23:14:28
tags:
---
## 内存四区
- 栈区: 由编译器自动分配释放, 主要存放函数的参数值, 局部变量的值, 遵循先进后出原则
- 堆区: 程序猿malloc申请的内存存放区域
- 全局区: 存放全局变量和静态变量, 常量, 比如字符串常量
- 代码区: 存放函数体二进制代码

```c
int gl = 1;
int main(int argc, const char * argv[]) {
    char a = 'a';
    char b = 'b';
    int c = 1;
    char *p = "aaaa";
    char *p2 = (char*)malloc(sizeof(char) * 100);
    
    printf("a: %d , b: %d, c: %d, &p: %d, &p2: %d, p: %d, p2: %d, gl: %d\n", &a, &b, &c, &p, &p2, p, p2, &gl);
    //a: 1606416119 , b: 1606416118, c: 1606416112, &p: 1606416104, &p2: 1606416096, p: 7887, p2: 3186800, gl: 8436
    return 0;
}
```


```c
char* getStr1() {
    char *p = "hello";
    return p;
}
char* getStr2() {
    char *p = "hello";
    return p;
}

int main(int argc, const char * argv[]) {
    char *p1 = NULL;
    char *p2 = NULL;
    p1 = getStr1();
    p2 = getStr2();
    
    printf("p1: %d, p2: %d\n", p1, p2);
    //p1: 3936, p2: 3943

    return 0;
}

```
### memset(<#void *__b#>, <#int __c#>, <#size_t __len#>)
```c
char buff[100];
//刚申请的内存地址, 里面也许有脏数据, 可以通过memset初始化
memset(buff, 0, sizeof(buff));
```

## 指针
### arr + 1 与 &arr + 1的区别
```c
int arr[10];
printf("arr: %d, arr+1: %d, &arr: %d, $arr+1: %d", arr, arr + 1, &arr, &arr + 1); 
//arr: 1606416032, arr+1: 1606416036, &arr: 1606416032, $arr+1: 1606416072
//从上可知 arr+1会增加4字节, &arr+1会增加40字节, 及跳过数组的大小
```

### const char* p 与 char* const p 的区别
> const char*p 表示指针指针的内容不能修改
> char* const p 表示指针不能被修改
```c
void testConst(const char* p) {
    p[0] = 1;//报错
    p = 1;
}

void testConst(char* const p) {
    p[0] = 1;
    p = 1;//报错
}
```

### 3中二级指针模型 
#### 指针数组 第一种二级指针模型 
> 数组中的每个元素是指针, 指针指向全局区里的字符串常量

```c
int main(int argc, const char * argv[]) {
    // insert code here...
    char * array[] = {"aaaa", "bbbb"};
    int len = sizeof(array) / sizeof(array[0]);
    for (int i = 0; i < len; i ++) {
        printf("array[i] = %s\n", array[i]);
        printf("*(array+i) = %s\n\n", *(array+i));
    }
    return 0;
}

```

#### 第二种二级指针模型
> 所有数据都存放在栈区, array步长为4

```c
int main(int argc, const char * argv[]) {
    char array[3][5] = {"aaaa", "bbbb", "cccc"};
    for (int i = 0; i < 3; i++) {
       printf("%d-%s \n", i, array[i]);
    }
    return 0;
}
```

#### 第三种二级指针模型
> 数据都存放在堆区

```c
int main(int argc, const char * argv[]) {
    char** p = NULL;
    int num = 5;
    p = (char**)malloc(sizeof(char*)*num);
    for (int i = 0; i < num; i++) {
        *(p+i) = (char*)malloc(sizeof(char)*num);
        sprintf(p[i], "%d%d%d", i+1, i+2, i+1);
    }
    
    for (int i = 0; i < num; i++) {
        printf("%d-%s \n", i, p[i]);
    }
    return 0;
}
```

### 数组类型指针
#### 第一种
```c
int main(int argc, const char * argv[]) {
    // insert code here...
    typedef int (myArray)[5];//定义了一个数组类型
    //myArray arr; //int arr[5];
    myArray* p = NULL;
    int arr[5];
    for (int i = 0; i < 5; i ++) {
        arr[i] = i + 1;
    }
    p = &arr;
    for (int i = 0; i < 5; i ++) {
        printf("(*p)[%d]: %d\n", i, (*p)[i]);
    }
    return 0;
}
```
#### 第二种
```c
int main(int argc, const char * argv[]) {
    typedef int (*arrayTypeP)[5];
    arrayTypeP p = NULL;
    int arr[5];
    for (int i = 0; i < 5; i ++) {
        arr[i] = i + 1;
    }
    p = &arr;
    for (int i = 0; i < 5; i ++) {
        printf("(*p)[%d]: %d\n", i, (*p)[i]);
    }
    return 0;
}
```

#### 第三种
```c
int main(int argc, const char * argv[]) {
    int (*p)[5];
    int arr[5];
    for (int i = 0; i < 5; i ++) {
        arr[i] = i + 1;
    }
    p = &arr;
    for (int i = 0; i < 5; i ++) {
        printf("(*p)[%d]: %d\n", i, (*p)[i]);
    }
    return 0;
}
```

### 函数指针
```c
typedef void (*myFunc)(int a, int b);
void func1(int a, int b) {
    printf("a: %d, b: %d\n", a, b);
}
int main(int argc, const char * argv[]) {
    myFunc p = func1;
    p(1, 2);
    return 0;
}
```

## 结构体
> 一旦定义好结构体, 其对应的内存也分配完毕, 可以根据偏移量算出各变量的地址, 一旦声明完毕, 就不要轻易修改变量位置
### 结构体的定义
```c
struct Teacher {
    char name[32];
    int age;
};//struct Teacher tearch;

typedef struct Teacher2 {
    char name[32];
    int age;
}t;// t tearch;

struct Student {
    char name[32];
    int age;
}s1, s2;//定义类型 同时定义变量

struct {
    char name[32];
    int age;
}s3 = {"s3", 18};//定义匿名类型 同时定义变量

int main(int argc, const char * argv[], char ** env) {
    struct Teacher t1 = {"t1", 18};
    t t2 = {"t2", 18};
    return 0;
}
```

### 结构体的操作
> t1.age t->age `.`和`->` 没有操作内存, 都是在cpu中, 相对于t1进行偏移量寻址操作

```c
int main(int argc, const char * argv[], char ** env) {
    struct Teacher t1 = {"t1", 18};
    t t2 = {"t2", 18};
    t* p = NULL;
    p = &t2;
    printf("t2.age: %d\n", t2.age);
    
    p->age = 20;
    
    printf("t2.age: %d\n", t2.age);
    printf("p->name: %s\n", p->name);
    
    return 0;
}

```

## 其他
### 数组首地址与数组地址的区别?
> c是数组的首元素的地址 c+1 步长为4字节
> &c是数组的地址 &c+1 步长为200*4字节

```c
int main(int argc, const char * argv[]) {
    int c[200] = {0};//编译时将所有的值设置为0
    memset(c, 0, sizeof(c));//运行时, 显示的将所有的值设置为0
    
    printf("c => %d \n c+1 => %d \n &c => %d \n &c+1 => %d ", c, c+1, &c, &c+1);
    /**
        c => 1606415264 
        c+1 => 1606415268 
        &c => 1606415264 
        &c+1 => 1606416064
    **/
    return 0;
}
```
### 定义数组类型
```c
int main(int argc, const char * argv[]) {
    // insert code here...
    typedef int (myArray)[5];//定义了一个数组类型
    myArray arr; //int arr[5];
    for (int i = 0; i < 5; i ++) {
        printf("%d\n", arr[i]);
    }
    return 0;
}
```

### 程序默认参数
```c
int main(int argc, const char * argv[], char ** env) {
    //传递参数个数
    printf("argc: %d\n", argc);
    //调用程序是 传递是参数, 默认会传递文件路径
    for (int i = 0; i < argc; i++) {
        printf("%s\n", argv[i]);
    }
    //打印环境变量
    while (*env ++) {
        printf("%s\n", *env);
    }
    return 0;
}
```

### 字符串结束标志
> 字符串结束标志位 '\0', NULL, 0
> 本质就是0