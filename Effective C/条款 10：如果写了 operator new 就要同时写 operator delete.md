# 条款 10. 如果写了 operator new 就要同时写 operator delete

## 0. 总则

类设计时，如果已重写`operator new`，请务必重写`operator delete`，一般重写内存管理函数为了自定义内存管理，对于内存空间或程序性能有所提升。

## 1. 空间极简类示例

缺省的`operator new`和`operator delete`具有非常好的通用性，它的这种灵活性也使得在某些特定的场合下，可以进一步改善它的性能。尤其在那些需要动态分配大量的但很小的对象的应用程序里，情况更是如此。假如如果机器内存有限，目标类占用空间极小，且需要创建大量这样的对象，那么就会发现最后创建出来的内存使用率很低，相对而言类空间大小为1，但是用于内存管理所分配的空间大小可能是5或者更多，因此这种情况下就有必要重写该类的内存管理函数，且设计好的话，还会大大提升系统性能，代码示例如下：

```cpp
class CBookRef
{
public:
    char    szDesc[2048];
};

class CBook
{
public:
    static void* operator new(size_t nSize);
    static void  operator delete(void* pMem, size_t nSize);

private:
    static CBook* m_pFreeHead;
    static const size_t m_nMemNum;

private:
    union
    {
        CBook*      pNext;
        CBookRef*   pRef;
    };
};
// 静态常量对象个数
const size_t CBook::m_nMemNum = 1024*1024*1024;
CBook* CBook::m_pFreeHead = NULL;

void* CBook::operator new(size_t nSize)
{
    // 大小不符，标准处理
    if (nSize != sizeof(CBook))
    {
        return ::operator new(nSize);
    }

    CBook* pThis = m_pFreeHead;
    if (m_pFreeHead == NULL)
    {
        // 标准new出预分配对象大小，类似内存池
        m_pFreeHead = static_cast<CBook*>(::operator new(sizeof(CBook)*m_nMemNum));
        // 组装空闲链表
        pThis = m_pFreeHead;
        for (size_t i = 0; i < m_nMemNum-1; i++)
        {
            pThis[i].pNext = pThis + i + 1;
        }
        pThis[m_nMemNum - 1].pNext = NULL;
    }

    // 可能存在返回NULL，因此该种new需要判断返回结果是否不为NULL才安全
    pThis = m_pFreeHead;
    m_pFreeHead = m_pFreeHead->pNext;

    return pThis;
}

void CBook::operator delete(void* pMem, size_t nSize)
{
    // NULL值处理
    if (pMem == NULL)
    {
        return;
    }

    if (sizeof(CBook) != nSize)
    {
        return ::operator delete(pMem);
    }
    else
    {
        CBook* p = static_cast<CBook*>(pMem);
        p->pNext = m_pFreeHead;
        m_pFreeHead = p;
    }
}
```
