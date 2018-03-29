# 条款 9. 避免隐藏标准形式的 new

## 0. 总则

类设计时，如果需要重载`operator new`，请注意兼容标准写法，使得`new X();`合法，避免设计出的类不支持标准`new`的操作。

## 1. 类设计兼容标准`new`的支持

* 重载标准函数样式

```cpp
// 重载标准函数样式
static void* operator new(size_t nSzie, std::new_handler pNewHandle);
static void* operator new(size_t nSzie)
{
    return ::operator new(nSzie);
}
```

* 重载默认参数的函数

```cpp
// 重载默认参数的函数
static void* operator new(size_t nSzie, std::new_handler pNewHandle=NULL);
```