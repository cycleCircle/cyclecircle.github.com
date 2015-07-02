---
layout: post
title: "C++ 指针和引用"
date: 2015-07-01 14:28:58 +0800
comments: true
categories: 
---


指针是存储内存地址的变量，所以指针是一个变量，指针也占用内存空间，
因此指针是一种指向内存单元的特殊变量。

作为一种变量，指针也需要声明，*可以将指针声明为指向一个内存块，这种指针被称为void指针*。

``` c++ 
    PointedType * PointerVariableName;
```

**指针一定要初始化，不然就是指向一个随机的内存地址，引发不可预知的问题**

``` c++
    PointedType * PointerVariableName = NULL;
```

使用new和delete来进行内存分配和释放

**使用new分配出来的内存，一定要用delete来释放，不然就会造成内存泄漏**

``` c++
    Type* pointer = new Type;
    Type* pointers= new Type[NumElements];

    int* pNumber = new int;
    int* pNumbers = new int[10]; //分配了10个int的空间
```

使用delete释放new分配出来的内存空间

``` c++
    Type* pointer = new Type;
    delete pointer;

    int* pNumber = new int;
    delete pNumber;

    int* pNumbers = new int[10];
    delete[] pNumbers;
```

**delete只能释放new分配的内存，不能作用于其他**

const作用于指针

**指针指向的数据为常量，不能修改，但可以修改指针包含的地址**

``` c++
    int HoursInDay = 24;
    const int* pInteger = &HoursInDay;
    int MonthsInYear = 12;
    pInteger = &MonthsInYear; //OK
    *pInteger = 13; //error, cannot change data
    int* pAnotherPointerToInt = pInteger; //error cannot assign const to non-const
```

**指针包含的地址是常量，不能修改，但可以修改针指指向的数据**
    
``` c++
    int DaysInMonth = 30;
    int* const pDaysInMonth = & DaysInMonth;
    *pDaysInMonth = 31; //OK
    int DaysInLunarMonth = 28;
    pDaysInMonth = &DaysInLunarMonth; // error cannot change address
```

**指针包含的地址以及指向的值都不能修改**

``` c++   
    int HoursInDay = 24;
    const int* const pHoursInDay = &HoursInDay;
    *pHoursInDay = 25; //error cannot change data
    int DaysInMonth = 30;
    pHoursInDay = &DaysInMonth; //error cannot change address
```


引用是变量的别名。声明引用时，需要将其初始化为一个变量，因此引用只是另一种访问变量存储数据的方式

``` c++   
    VarType Original = value;
    VarType& ReferenceVariable = Original;
    
    #include <iostream>
    using namespace std;
    
    int main()
    {
        int Original = 30;
        cout << "Original = " << Original << endl;
        cout << "Original is at address: " << hex << &Original << endl;
        
        int& Ref = Original;
        cout << "Ref is at address: " << hex << &Ref << endl;
        
        int& Ref2 = Ref;
        cout << "Ref2 is at address: " << hex << &Ref2 << endl;
        cout << "Ref2 gets value, Ref2 = " << dec << Ref2 << endl;
        
        return 0;
    }
```

**引用主要用于函数参数的传递，这样不会进行参数的拷贝**


const作用于引用，可以禁止通过引用修改它指向的变量的值

``` c++ 
    int Original = 30;
    const int& ConstRef = Original;
    ConstRef = 40; // error cannot change value
    int& Ref2 = ConstRef; // error Ref2 is not const
    const int& ConstRef2 = ConstRef; //OK
```

