---
layout: post
title: "编写一个好的C++类以及对象的复制问题"
date: 2015-07-02 09:56:22 +0800
comments: true
categories: 
---


当一个类包含一个指针成员，它指向动态分配的内存（这些内存是在构造函数中使用new分配的，并在析构
函数中使用delete进行释放）。复制这个类的对象时，将复制其指针成员，但不复制指针的缓冲区，这称为
浅复制，容易发生不可预知的问题

所以我们需要使用复制构造函数确保深复制

``` c++ 
#include <iostream>
using namespace std;

class MyString
{
    private:
        char* Buffer;

    public:
        MyString(const char* InitialInput)
        {
            if(InitialInput != NULL)
            {
                Buffer = new char [strlen(InitialInput) + 1];
                strcpy(Buffer, InitialInput);
            }
            else 
            {
                Buffer = NULL;
            }
        }

        //copy constroctor
        MyString(const MyString& CopySource)
        {
            if(CopySource.Buffer != NULL)
            {
                Buffer = new char [strlen(CopySource.Buffer) + 1];

                strcpy(Buffer, CopySource.Buffer);
            }
            else
            {
                Buffer = NULL;
            }
        }

        //destructor
        ~MyString()
        {
            if(Buffer != null)
            {
                delete[] Buffer;
            }
        }

        int GetLength()
        {
            return strlen(Buffer);
        }

        const char* GetString()
        {
            return Buffer;
        }
};

void UseMyString(MyString Input)
{
    cout << "String buffer in MyString is " << Input.GetLength();
    cout << " characters long" << endl;

    cout << "Buffer contains: " << Input.GetString() << endl;
    return;
}

int main()
{
    MyString SayHello("hello from string class");

    UseMyString(SayHello);

    return 0;
}
```

> 由于C++的特征和需求，有些情况下对象会自动被复制，常见就是在函数参数的传递时，这样子就会频繁
> 的调用复制构造函数，如果复制构造函数的开销大的话，那样就会造成性能问题，所以对于C++ 11来说
> 可以编写移动构造函数来解决这个问题，具体查阅C++ 11的相关信息。

对于上面这个类，当使用赋值运算进行复制时，还是会进行浅复制的，所以还要重载赋值运算符

``` c++ 

        // copy assignment operator
        MyString& operator= (const MyString& CopySource)
        {
            if((this != &CopySource) && (CopySource.Buffer != NULL))
            {
                if(Buffer != NULL)
                {
                    delete[] Buffer;
                }

                Buffer = new char[strlen(CopySource.Buffer) + 1];

                strcpy(Buffer, CopySource.Buffer);
            }
```

通过在复制构造函数声明中使用const，可确保复制构造函数不会修改指向的源对象，另外，**复制构造函数的
参数必须按引用传递**，否则调用它时将复制实参的值，导致于源数据进行浅复制，这样就死循环了。

编写类时应注意的事项

* 类包含原始指针成员（`char*`等）时，务必编写复制构造函数和复制赋值运算符
* 编写复制构造函数时，务必将接受源对象的参数声明为const引用
* 务必将类成员声明为std::string和智能指针类（而不是原始指针），因为它们实现了复制构造函数，可减少工作量
* 除非万不得已，不要将类成员声明为原始指针
