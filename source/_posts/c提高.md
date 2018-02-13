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

## 数组
### 数组首地址与数组地址的区别?
> c是数组的首元素的地址 c+1 步长为4字节
> &c是数组的地址 &c+1 步长为200*4字节

```c
int c[200] = {0};//编译时将所有的值设置为0
memset(c, 0, sizeof(c));//运行时, 显示的将所有的值设置为0
```

