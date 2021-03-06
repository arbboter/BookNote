# 条款 5：对应的 new 和 delete 要采用相同的形式

## 0. 总则

因为`new`和`new[]`两种内存分配的机制存在差异，因此不可以用随便使用`delete`和`delete[]`销毁对象，必须采取相匹配的形式创建对象和销毁对象，否则会产生不可预知的各种情况。因此，如果你调用`new`时用了`[]`，调用`delete`时也要用`[]`,如果调用`new`时没有用`[]`，那调用`delete`时也不要用`[]`。

## 1. new和delete工作机制

用`new`的时候会发生两件事，首先内存被分配(通过 operator new 函数，类似C的malloc)，然后为被分配的内存调用一个或多个构造函数；
用`delete`的时候，也有两件事发生，首先为将被释放的内存调用一个或多个析构函数，然后释放内存(类似C的free)，决定是调用一个还是多个构造析构函数取决于操作类型是否带`[]`。

## 2. typedef类型对象的管理

> 如果你调用`new`时用了`[]`，调用`delete`时也要用`[]`,如果调用`new`时没有用`[]`，那调用`delete`时也不要用`[]`。
因此，`typedef`类型的对象的内存管理同样需要遵循这一原则，而不能仅看类型表象。比如：

```cpp
// 身份三要素用三行表示
typedef std::string PersonId[3];

// 创建一个身份三要素对象
string* p = new PersonId();

// 错误写法，因为PersonId表示string[3]
delete p;
// 正确写法
delete[] p;
```

上述示例中，由于`PersonId`的类型实质上是数组类型`string[3]`，所以创建虽然使用不带`[]`的new，但是销毁时确必须是带`[]`的`delete`。

由于此类typedef的情况在实际编写过程中很容易混淆，因此建议对需要自己管理的对象不要使用`typedef`，减少编写和维护的成本。