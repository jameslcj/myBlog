---
title: c提高
date: 2018-02-11 23:14:28
tags:
---
## 内存四区
- 栈区: 由编译器自动分配释放, 主要存放函数的参数值, 局部变量的值, 遵循先进后出原则
- 堆区: 程序猿malloc申请的内存存放区域
- 全局区: 存放全局变量和静态变量, 常量, 比如字符串常量
- 代码区: 存放函数体二进制代码


```
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

## 指针
### arr + 1 与 &arr + 1的区别
```c
int arr[10];
printf("arr: %d, arr+1: %d, &arr: %d, $arr+1: %d", arr, arr + 1, &arr, &arr + 1); 
//arr: 1606416032, arr+1: 1606416036, &arr: 1606416032, $arr+1: 1606416072
//从上可知 arr+1会增加4字节, &arr+1会增加40字节, 及跳过数组的大小
```

