# 条款 16: 在 operator=中对所有数据成员赋值

## 0. 总则

在重写类的赋值函数时，必须对对象的每一个数据成员赋值，不仅需要注意对当前类成员（尤其是新增成员）的逐项赋值，还要注意对基类对象赋值，因为已经重写了赋值函数时，系统不再会为该类对象默认完成类对象本身或者基类部分数据的默认赋值操作，因此要么不重写，要么重写上上下下，不要遗漏。

## 1. 本章测试

```cpp
#include <iostream>
#include <sstream>

class A
{
public:
    A(int x) :a(2016),b(x){}
    void DebugInfo()
    {
        std::cout << "a=" << a << " b=" << b << std::endl;
    }
    void Set(int m, int n)
    {
        a = m;
        b = n;
    }
private:
    int a;
    int b;
};

class B :public A
{
public:
    B() :A(2018)
    {
        b = __LINE__;
        c = __LINE__;
    }
    B& operator=(const B& x)
    {
        if (this == &x) return *this;

        // 对象父类部分赋值
        // 1.推荐：强制转化父类对象类型赋值
        static_cast<A&>(*this) = x;
        // 2.不推荐：调用父类的的赋值函数，因为某些编译器不兼容
        // A::operator =(x);

        // 当前类成员赋值，逐项赋值
        b = x.b;
        c = x.c;

        return *this;
    }
    void DebugInfo()
    {
        A::DebugInfo();
        std::cout << "b=" << b << " c=" << c << std::endl;
    }
public:
    int b;
    int c;
};

class C :public A
{
public:
    C() :A(2019)
    {
        b = __LINE__;
        c = __LINE__;
    }
    C& operator=(const C& x)
    {
        if (this == &x) return *this;

        // 无对象父类部分赋值

        // 当前类成员赋值，逐项赋值
        b = x.b;
        c = x.c;

        return *this;
    }

    void DebugInfo()
    {
        A::DebugInfo();
        std::cout << "b=" << b << " c=" << c << "\n" << std::endl;
    }

public:
    int b;
    int c;
};


int main()
{
    B* pb = new B();
    C* pc = new C();
    B b1;
    C c1;

    // 重新赋值后
    pb->Set(5401, 5402);
    pc->Set(5401, 5402);
    b1 = *pb;
    c1 = *pc;
    // 因为C的赋值操作没有重写对象基类部分的数据赋值
    // 导致赋值时基类部分值未变

    std::cout << "重新赋值后:\n*pb:" << std::endl;
    pb->DebugInfo();

    std::cout << "*pc:" << std::endl;
    pc->DebugInfo();

    std::cout << "b1:" << std::endl;
    b1.DebugInfo();

    std::cout << "c1:" << std::endl;
    c1.DebugInfo();

    return 0;
}
```

输出结果

```output
重新赋值后:
*pb:
a=5401 b=5402
b=27 c=28
*pc:
a=5401 b=5402
b=61 c=62

b1:
a=5401 b=5402
b=27 c=28
c1:
a=2016 b=2019
b=61 c=62
```

上述示例中，重新赋值后c1的基类成员a和b的值并未在赋值时更新，因此因要注意重写上上下下。