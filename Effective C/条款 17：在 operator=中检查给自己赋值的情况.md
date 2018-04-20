# 条款 17: 在 operator=中检查给自己赋值的情况

## 0. 总则

重写赋值操作函数时检查给自己赋值时必要的，其一可以减少内存拷贝以提升性能，其二可以杜绝因为删除自身而产生的不可预知的情况来保证重写的正确性。一个赋值运算符必须首先释放掉一个
对象的资源（去掉旧值），然后根据新值分配新的资源。在自己给自己赋值的情况下，释放旧的资源将是灾难性的，因为在分配新的资源时会需要旧的资源。

## 1. 检测对象相同的方法

* 对象值相同

```cpp
class CMyo
{
public:
    CMyo(int a) :nData(a) {}
    CMyo& operator=(const CMyo& v)
    {
        // 值判断对象是否相同
        if (v.nData == this->nData)
        {
            return *this;
        }

        nData = v.nData;
        return *this;
    }

private:
    int nData;
};
```

* 对象地址相同

```cpp
class CMyo
{
public:
    CMyo(int a) :nData(a) {}
    CMyo& operator=(const CMyo& v)
    {
        // 地址判断对象是否相同
        if (this == &v)
        {
            return *this;
        }

        nData = v.nData;
        return *this;
    }

private:
    int nData;
};
```

* 对象实现的id身份鉴别函数返回值相同 

```cpp
class CMyo
{
public:
    CMyo(int a) :nData(a) {}
    CMyo& operator=(const CMyo& v)
    {
        // 标识判断对象是否相同
        if (this->id() == v.id())
        {
            return *this;
        }

        nData = v.nData;
        return *this;
    }

    // 对象唯一性标识
    int id() const
    {
        return nData % 100;
    }

private:
    int nData;
};
```