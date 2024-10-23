---
title: qt4与qt5中QString::compare方法差异
date: 2024-10-23 18:04:07
tags:
- Qt
- Linux
categories: 
- Linux
---

## 背景

隔壁组的同事在调试qt代码的时候遇到了一个bug，该bug发生前的代码大致如下

```cpp
// 比较用户输入的密码与dv123456是否一致
int match = QString::compare("dv123456",userInputPwd());
// 一致则解锁
if(match == 0) {
    /* unlock */
}
```

该段代码一直运行正常

后面另一个同事为了增加另一个密码兼容，增加了如下代码

> 这里的代码写法肯定有问题的，不能使用&=符，直接两个compare结果单独判断是否为零即可

```cpp
// 比较用户输入的密码与dv123456或者412345是否一致
int match = QString::compare("dv123456",userInputPwd());
match &= QString::compare("412345",userInputPwd());
// 一致则解锁
if(match == 0) {
    /* unlock */
}
```

大部分情况下该代码可以正常运行，但是今天发现如果用户输入`dv123456`的前几个字符，或者`412345`的前几个字符,比如`d`,`4`,则一样会进入`match == 0`部分代码。

我刚好在旁边看着这个bug的触发，调出了qt的源码查看QString部分的源码查看compare实现

## 探究

### qt4 compare实现

由于我们组的芯片方案SDK中用的是qt4.8.6,所以我打开的是qt4源码中的compare实现，代码截取如下

```cpp
int QString::compare(const QString &other) const
{
    return ucstrcmp(constData(), length(), other.constData(), other.length());
}
// 设置了大小写敏感进入该函数
int QString::compare(const QString &other, Qt::CaseSensitivity cs) const
{
    if (cs == Qt::CaseSensitive)
        return ucstrcmp(constData(), length(), other.constData(), other.length());
    return ucstricmp(d->data, d->data + d->size, other.d->data, other.d->data + other.d->size);
}


// Unicode case-sensitive comparison
static int ucstrcmp(const QChar *a, int alen, const QChar *b, int blen)
{
    if (a == b && alen == blen)
        return 0;
    int l = qMin(alen, blen);
    int cmp = ucstrncmp(a, b, l);
    return cmp ? cmp : (alen-blen);
}

// Unicode case-sensitive compare two same-sized strings
static int ucstrncmp(const QChar *a, const QChar *b, int l)
{
    while (l-- && *a == *b)
        a++,b++;
    if (l==-1)
        return 0;
    return a->unicode() - b->unicode();
}
```

qt4中这个compare的实现还是比较简单易懂的，在调用`compare(a,b)`的时候，只会比较较短字符串的长度。如果遍历完成后都相同，则返回a和b的长度差，如果有不同，则返回不同字符的unicode代码差值

以`compare("dv123456","dv")`为例`ucstrncmp`中`l`为2,在while循环中遍历完后可以看到长度为2的子串都是相等的，那么`ucstrncmp`将会返回0，`ucstrcmp`则会返回`alen-blen`,最终返回应该是4。

而如果是`compare("412345","dv")`,在while中无法遍历完成，最终`ucstrncmp`将会返回4和d的ascii码差值，查表可知4的ascii码十进制是52，d为100，最后应该是-48

### 实际验证

为了验证该推论，在上述bug代码中增加打印如下

```cpp
// 比较用户输入的密码与dv123456是否一致
int match1 = QString::compare("dv123456",userInputPwd());
int match2 = QString::compare("412345",userInputPwd());
int match = match11 & match2;
qDebug() << "match1 = " << match1;
qDebug() << "match2 = " << match2;
qDebug() << "match  = " << match;
// 一致则解锁
if(match == 0) {
    /* unlock */
}
```

在输入框输入dv后，打印结果却有所出入

```cpp
match1 = 1
match2 = -48
match  = 0
```

首先match为0则是因为1的二进制与-48的二进制刚好是0，则会误进入unlock代码段中。而match1是1则让我百思不得其解，而我手动在我的qt4环境中写了demo打印发现确实应该是4和-48，demo如下

```cpp
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QString s1 = "dv123456";
    QString s2 = "412345";
    int ret = 0;

    QString test = "dv";  // s1: ret = 4
                          // s2: ret = -48

    ret = QString::compare(s1,test);
    qDebug() << "s1: ret = " << ret;

    ret = QString::compare(s2,test);
    qDebug() << "s2: ret = " << ret;

    return a.exec();
}
```

### qt5 compare实现

后面与同事沟通发现，他们那款芯片的sdk采用的是Qt5，于是我猜测是compare在qt4和qt5的实现不一致，于是拉下一个qt5的源码查看，截取如下

```cpp
int QString::compare(const QString &other, Qt::CaseSensitivity cs) const Q_DECL_NOTHROW
{
    return qt_compare_strings(*this, other, cs);
}

static int qt_compare_strings(QStringView lhs, QStringView rhs, Qt::CaseSensitivity cs) Q_DECL_NOTHROW
{
    if (cs == Qt::CaseSensitive)
        return ucstrcmp(lhs.begin(), lhs.size(), rhs.begin(), rhs.size());
    else
        return ucstricmp(lhs.begin(), lhs.end(), rhs.begin(), rhs.end());
}

static int ucstrcmp(const QChar *a, size_t alen, const char *b, size_t blen)
{
    const size_t l = qMin(alen, blen);
    const int cmp = ucstrncmp(a, reinterpret_cast<const uchar*>(b), l);
    return cmp ? cmp : lencmp(alen, blen);
}

// Unicode case-sensitive compare two same-sized strings
static int ucstrncmp(const QChar *a, const QChar *b, size_t l)
{
    /* 代码过长，不截取 */
}
```

可以看到前面几部分代码基本逻辑相同，在`ucstrncmp`也只是增加了许多平台优化相关，qt4和qt5最大的差别在于ucstrcmp的return语句。

```cpp
// qt4
return cmp ? cmp : (alen-blen);

// qt5
return cmp ? cmp : lencmp(alen, blen);

Q_DECL_CONSTEXPR int lencmp(Number lhs, Number rhs) Q_DECL_NOTHROW
{
    return lhs == rhs ? 0 :
           lhs >  rhs ? 1 :
           /* else */  -1 ;
}
```

可以看到qt5中不再使用alen-blen的方式，而且返回0，1，-1表示`compare(a,b)`的字符串长短关系，a比b长则返回1，相等则为0，a比b短为-1。

这也就是造成了qt4和qt5中同一个方法的结果差异所在。

## 解决

`compare`函数设计更多是为了用于字符串排序的工作，这种判断字符串是否完全相等的情况应该直接使用`QString`重写的`==`运算符,返回bool更加方便做判断运算

```cpp
// qt4
bool QString::operator==(const QString &other) const
{
    if (d->size != other.d->size)
        return false;

    return qMemEquals(d->data, other.d->data, d->size);
}

// qt5
bool operator==(const QString &s1, const QString &s2) Q_DECL_NOTHROW
{
    if (s1.d->size != s2.d->size)
        return false;

    return qt_compare_strings(s1, s2, Qt::CaseSensitive) == 0;
}
```