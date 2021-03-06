# 条款 1：尽量用 const 和 inline 而不用#define

## 0.总则

宏是一种语法糖，适合高级编码以帮助完善语言本身不足之处，而内联是为了弥补语言本身不足而引入以减少函数调用开销。

## 1. #define替换后会变成无符号，不利于编译调试

 #define内容会在程序编译前被预处理替换掉，故编译器是看不到#define对应的符号名，因此在程序编译报错时只会显示替换后的内容，而不会显示符号名，不易于编译出错调试，但是如果换成const定义的常量或者用enum，那么就可以避免这种情况了。

## 2.类的静态成员常量既可以就地初始化，也可以类外初始化

```cpp
class CDemo
{
public:
    // 静态成员常量
    static const int m_nNum4 = 2;   // 旧版本存在不兼容，不建议
    static const int m_nNum5;       // 建议使用

};
const int CDemo::m_nNum5 = 0;
```

上述代码中两种方式定义初始化静态成员常量均可以，但注意兼容性问题。

## 3.类成员变量常量

类变量常量内存地址说明，对象创建时:

* 普通变量会分配一个新地址;
* 普通常量也会分配一个新地址，且该地址与普通变量的地址是连续的;
* 静态变量不会分配新地址，对应的变量地址是程序编译时就已经写到可执行文件中，程序运行时就已经分配了内存，此时会复用该类静态区域内对应该静态变量的地址
* 静态常量也不会分配新地址，复用

## 4.思考

### 思考1：为何建议使用初始化列表？

类创建时，先创建类对象空间，然后执行构造函数，构造函数可分为初始化阶段和计算阶段:
所有该对象地址空间内的成员变量都会在初始化阶段初始化，即使该成员没有出现在构造函数的初始化列表中,
如此会导致不在初始化列表中的对象会调用默认的构造函数，建议使用初始化列表初始化，有利于提升程序性能
否则在构造函数内初始化(计算阶段)的对象会额外调用默认构造函数来初始化。

### 思考2：为何普通变量不能就地赋值？（推断）

因为类对象创建时的初始化阶段会初始化所有的对象，显式或隐式，故就地赋值会被覆盖。

### 思考3：为何常量必须在初始化列表赋值？（推断）

沿用思考2的描述，因为常量有且仅需初始化一次，故仅限于初始化列表初始化。

### 思考4：如何验证类的静态变量在类内只是声明，不是定义

```cpp
class A
{
public:
    static int x; // 声明语句a
};
int A::x = 1;  // 定义语句b，此处才是变量定义语句，允许不初始化
int main()
{
    A a; // ok
    std::cout << a.x; // 引用静态变量语句c，如果不在外面再定义x，链接时会报无法解析的外部符号类似的错误
    return 0;
}
```

上述代码当注释掉定义语句b时，会报错显示无法找到变量x的定义，但如果此时去掉对变量x的引用语句c，则程序编译运行正常，
故说明语句a仅是声明，语句b才是定义

### 思考5：为何静态变量类内只允许声明，不允许定义？（推断）

因为定义变量会创建内存空间，但由于静态变量地址不在类对象地址空间内，且有且仅在程序启动时分配空间，
故编写类的定义代码时只需要对静态变量的声明，声明一个到该变量到某个静态变量的关联关系。此外，如果为定义，则需要在每次创建对象时都重新分配一次内存，与静态变量的特性相矛盾。静态成员在程序运行的过程中只被初始化一次，所以每当类的对象创建时都去“初始化”它们没有任何意义，至少这会影响效率：既然是“初始化”，那为什么要去做多次？

### 思考6：为何静态变量在类内不允许就地赋值？

因静态变量在类内是声明，不是定义。

```cpp
 class CDemo
{
public:
    // 静态成员常量
    static const int m_nNum4 = 2; // 旧的编译器存在不谦容的情况
    static const int m_nNum5;     // 建议使用这种方法，标准化

};
const int CDemo::m_nNum5 = 0;
```

 上述代码中静态常量m_nNum4和m_nNum5的声明定义均正确。

## x. 测试代码(Visual Studio)

```cpp
#include <iostream>
using namespace std;
class CDemo
{
public:
    CDemo() :m_nNum2(8)
    {
    }
    ~CDemo()
    {

    }

public:
    // 成员变量
    int m_nNum1;
    const int m_nNum2;
    static int m_nNum3;
    static const int m_nNum4 = 2;
    static const int m_nNum5;

};
int CDemo::m_nNum3 = 0;
const int CDemo::m_nNum5 = 0;

int main()
{
    CDemo d1, d2;
    const int *p, *q;

    p = &d1.m_nNum1;
    q = &d2.m_nNum1;
    cout << "m_nNum1:" << p << " " << q << std::endl;
    p = &d1.m_nNum2;
    q = &d2.m_nNum2;
    cout << "m_nNum2:" << p << " " << q << std::endl;
    p = &d1.m_nNum3;
    q = &d2.m_nNum3;
    cout << "m_nNum3:" << p << " " << q << std::endl;
    p = &d1.m_nNum4;
    q = &d2.m_nNum4;
    cout << "m_nNum4:" << p << " " << q << std::endl;
    p = &d1.m_nNum5;
    q = &d2.m_nNum5;
    cout << "m_nNum5:" << p << " " << q << std::endl;
    return 0;
}
```

**输出结果:**

```output
m_nNum1:00CFFDE0 00CFFDD8
m_nNum2:00CFFDE4 00CFFDDC
m_nNum3:00354468 00354468
m_nNum4:00353278 00353278
m_nNum5:00353298 00353298
```