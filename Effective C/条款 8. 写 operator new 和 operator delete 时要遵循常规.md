# 条款 8. 写 operator new 和 operator delete 时要遵循常规

## 0. 总则

有关`operator new`和`operator delete`(以及他们的数组形式)的重写一定要遵循相应的原则，主要是内存分配程序要能够支持`new-handler`函
数并正确地处理了零内存请求，而内存释放程序能处理空指针。

## 1. `operator new`运行机制

`operator new`实际做起来主要完成的功能如下：

* 申请足够的内存；
* 要有正确的返回值；
* 可用内存不够时要调用出错处理函数；
* 处理好0字节内存请求的情况。

`operator new`实际上会不只一次地尝试着去分配内存，它要在每次失败后调用出错处理函数，
还期望出错处理函数能想办法释放别处的内存，因此执行完出错函数后会继续尝试分配内存。
只有在指向出错处理函数的指针为空的情况下，`operator new`才抛出异常。如果用代码表示其过程，大致如下：

```cpp
void* MyNew(size_t nSize)
{
    // 如果请求分配0字节内存时，也要返回一个合法指针，如此能为C++语言其它地方带来许多简便。
    if (nSize == 0)
    {
        nSize = 1;
    }

    void* pDat = NULL;
    // 循环尝试申请分配内存
    while (true)
    {
        // 内存申请
        pDat = malloc(nSize);

        // 分配成功
        if (pDat)
        {
            return pDat;
        }
        else
        {
            // 获取全局异常函数
            new_handler pNewHandle = std::set_new_handler(0);
            std::set_new_handler(pNewHandle);

            // 调用异常处理函数或抛出异常
            if (pNewHandle)
            {
                pNewHandle();
            }
            else
            {
                throw std::bad_alloc();
            }
        }
    }
}
```

## 2. `operator delete`运行机制

`operator delete`实际做起来主要完成的功能如下：

* 支持空指针删除；
* 内存删除释放。

归纳起来，如果用代码表示其过程，大致如下：

```cpp
void MyDelete(void* pMem)
{
    // 支持空指针
    if (pMem == NULL)
    {
        return;
    }

    // 内存释放
    free(pMem);
}
```

## 3. 后记

C++标准规定所有独立的(freestanding)类的大小都是非零值，此处与`new`大小为0的对象时默认处理为分配空间大小为1遥呼相应。编写类的`operator new`时需要注意类的继承关系，对于一个类 X 的`operator new`来说，函数内部的行为在涉及到对象的大小时，都是精确的`sizeof(X)`大小。由于存在继承，基类中的`operator new`可能会被调用去为一个子类对象分配内存，为避免情况复杂化，最简单的办法是把这个*错误*数量的内存分配请求转给标准`operator new`来处理，如：

```cpp
void * Base::operator new(size_t size)
{
    // 如果数量“错误”， 让标准 operator new 去处理这个请求
    if (size != sizeof(Base))
    {
        return ::operator new(size);
    }

    // 正常处理
}
```