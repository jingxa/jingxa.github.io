---
title: char*的疑问
date: 2018-06-27 19:15:32
tags:
	- C++
	- C
categories:
	- C++
---

在使用char\* 传递参数的时候，以为char\* 传递的是指针；

```C++
#include <iostream>
#include<string>
#include <cstring>
#include<vector>

using namespace std;


void printNoref(char * s){
    cout<<"非引用传值函数中s 地址:"<<&s<<endl;
    cout<<"s 指向的地址:"<<(int*)s<<endl;
    s++;
    cout<<"str:"<<s<<endl;
}


void printref(char *& s){
    cout<<"非引用传值函数中s 地址:"<<&s<<endl;
    cout<<"s 指向的地址:"<<(int*)s<<endl;
    (s)++;
    cout<<"ref str:"<<s<<endl;
}



int main() {
    string s ( "123456789");
    char* str = new char[s.size() + 1];
    strcpy(str, s.c_str());
    
    // 真实地址
    cout<<"真实 str 地址："<<&str<<endl;
    cout<<"str 指向的地址："<<(int*)str<<endl;  
    
    
    //非引用传值
    printNoref(str);
    cout<<"str 指向的地址:"<<(int*)str<<" 值 str:"<<str<<endl;
    cout<<"================"<<endl;
    
    // 引用传值
    printref(str);
    cout<<"str 指向的地址:"<<(int*)str<<" 值 str:"<<str<<endl;
    


    return 0;
}
```

结果：

```
真实 str 地址：0x7ffcf092ee08
str 指向的地址：0x1481c20
非引用传值函数中s 地址:0x7ffcf092ede8
s 指向的地址:0x1481c20
str:23456789
str 指向的地址:0x1481c20 值 str:123456789
================
引用传值函数中s 地址:0x7ffcf092ee08
s 指向的地址:0x1481c20
ref str:23456789
str 指向的地址:0x1481c21 值 str:23456789
```

- `char * s`传递的是s地址中存储的指针地址的复制，
- `char*& s`传递的是指向s真实地址的指针；


---

