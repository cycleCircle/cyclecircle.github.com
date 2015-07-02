---
layout: post
title: "C++ 构造函数和析构函数的特别技巧"
date: 2015-07-02 10:50:23 +0800
comments: true
categories: 
---

> ## new出来的空间是在堆里面的，直接声明的空间是在栈里面的，数据量大的对象尽量用new

**不可复制的类**

我们有时候可能不想让一些资源不能被复制，如一个cpu等等，但是C++会默认帮你添加一个复制构造函数
这样子就会破坏我们想要的规则，所以要禁止对象被复制，可以声明一个私有的复制构造函数。

``` c++ 
class CPU
{
    private:
        CPU(const CPU&); //private copy constructor
        CPU& operator = (const CPU&); // private copy assignment operator
}
```

只需要把复制构造函数声明为私有就可以了，不需要提供实现
当然，上面在遇到赋值的时候，还是不可以的，所以我们还要提供**私有赋值运算符的声明**


**单例模式**

``` c++ 

#include <iostream>
#include <string>
using namespace std;

class President
{
    private:
        // private default constructor
        President() {};

        // private copy constroctor
        President(const President&);

        // private assignment operator
        const President& operator=(const President&);

        string Name;

    public:
        static President& GetInstance()
        {
            static President OnlyInstance;

            return OnlyInstance;
        }

        string GetName();
        {
            return name;
        }

        void SetName(string InputName)
        {
            Name = InputName;
        }
}
```


**禁止在栈中实例化的类**

栈中的空间有限，所以如果一些包含数据大的类，应试禁止在栈上实例化，而只允许在堆上创建实例，
为此，关键在于将析构函数声明为私有的。

``` c++ 

#include <iostream>
using namespace std;

class MonsterDB
{
    private:
        ~MonsterDB() {}; // private destructor

    public:
        static void DestroyInstance(MonsterDB* pInstance)
        {
            delete pInstance;
        }
};

int main()
{
    MonsterDB* pMyDatabase = new MonsterDB();
    MonsterDB::DestroyInstance(pMyDatabase);

    return 0;
}
```

主要是把析构函数声明为私有的，然后再创建静态方法来释放new出来的空间，不然就会造成内存泄漏


