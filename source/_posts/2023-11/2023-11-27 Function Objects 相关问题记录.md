---
title: Function Objects 相关问题记录
date: 2023-11-27 11:32:58
excerpt: 简单介绍了 Function Objects，以及在使用 std::bind, std::mem_fn, std::function 时传参遇到的问题。
index_img: https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271200911.jpeg
categories: 
- [计算机知识,C++]
tags: 
- C++
---



# 前言

此前在学习简单线程池时用到了 `std::bind` ，所以也学习了相关内容，其中也遇到了一些问题，特此记录一下。原本是打算放在“线程池”那篇文章里作为一个小章节，后来想想这里遇到的问题与原文关系并不是很密切，所以还是单独拆分出来写个小文章。

# Function Objects

Function objects 的定义概念可以直接参考 https://en.cppreference.com/w/cpp/utility/functional，简单翻译一下，就是可以调用 operator() 的对象被视作是 function objects，而相关联的函数调用`INVOKE`文档上也给出了详细说明。

![Function invoaction](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271133288.png)

这里简单总结一下，就是函数调用操作可以视为 `INVOKE(f, t1, t2, ... , tN)`，可以总结为如下三种情况：

- 如果 `f`是一个指向 `T` 类成员函数的指针，那么会根据 `t1` 的类型不同（类型为T或是T的派生类，对象引用，对象指针），采取对应的函数调用方式（**注意，这里的指针也可以是智能指针**）
- 如果 `f`是一个指向 `T` 类的成员变量，并且 `N` 为1，则等同于要去访问这个类的成员变量（也会根据`t1`类型做出适配的调用）
- 如果不是上述两种情况，则等同于 `f(t1, t2, ..., tN)`，进行一次函数调用，这里的 `f` 就是一个 FunctionObject



# 问题记录

然后在看 mem_fn , bind, function 相关实例代码时，对一些传参方式很疑惑，通过用不同的方式来传入对象、指针、引用，会有不同的输出表现，具体示例代码和输出结果如下所示，也可以直接看这个链接 https://compiler-explorer.com/z/n81o8bssP 。

```c++
#include <cstdio>
#include <functional>
#include <iostream>
using namespace std;
struct Foo
{
    Foo(int num) : num_(num) {}
    void print_add(int i)   { num_ += i;}
    int num_;
};

int main()
{
  
		Foo ff(1);
    auto f1 = std::mem_fn(&Foo::print_add);
    f1(ff, 1);
    cout << "mem_fn pass ff, after exec, ff num : " << ff.num_ << endl;
    f1(&ff, 1);
    cout << "mem_fn pass &ff, after exec, ff num : " << ff.num_ << endl;


    auto f2 = std::bind(&Foo::print_add, placeholders::_1, placeholders::_2);
    f2(ff, 1);
    cout << "bind use placeholder_1 to pass ff, after exec, ff num : " << ff.num_ << endl;

    std::function<void(int)> f3 = std::bind(&Foo::print_add, ff, placeholders::_1);
    f3(1);
    cout << "bind pass ff, after exec, ff num : " << ff.num_ << endl;

    std::function<void(int)> f4 = std::bind(&Foo::print_add, &ff, placeholders::_1);
    f4(1);
    cout << "bind pass &ff, after exec, ff num : " << ff.num_ << endl;

    auto f5 = std::bind(f1, placeholders::_1, placeholders::_2);
    f5(ff, 1);
    cout << "bind_mem_fn pass ff, after exec, ff num : " << ff.num_ << endl;

    auto f6 = std::bind(f1, placeholders::_1, placeholders::_2);
    f6(&ff, 1);
    cout << "bind_mem_fn pass &ff, after exec, ff num : " << ff.num_ << endl;


    std::function<void(Foo, int)> f7 = &Foo::print_add;
    f7(std::ref(ff), 1);
    cout << "function(Foo), pass ff, after exec, ff num : " << ff.num_ <<endl;

    std::function<void(Foo*, int)> f8 = &Foo::print_add;
    f8(&ff, 1);
    cout << "function(Foo*), pass ff*, after exec, ff num : " << ff.num_ <<endl;
}

```

输出结果：

```
mem_fn pass ff, after exec, ff num : 2
mem_fn pass &ff, after exec, ff num : 3
bind use placeholder_1 to pass ff, after exec, ff num : 4
bind pass ff, after exec, ff num : 4
bind pass &ff, after exec, ff num : 5
bind_mem_fn pass ff, after exec, ff num : 6
bind_mem_fn pass &ff, after exec, ff num : 7
function(Foo), pass ff, after exec, ff num : 7
function(Foo*), pass ff*, after exec, ff num : 8
```

---

那么根据输出结果不难发现如下几个现象：

1. 对于 mem_fn 也就是代码里的 f1 ，无论传入的是 ff 还是 &ff ，执行后都是能正常修改 ff 对象中的 num_ 值。
2. 对于 bind，如果提前绑定了 ff，则执行后并没有修改 ff 对象中的 num_ 值；而如果使用 placeholder 或者 传入 &ff ，则也可以正常修改。
3. 对于 function，如果指定第一个参数为 Foo，此时传入 ff，执行后也没有修改 ff 对象中的 num_ 值。

所以我当时就挺困惑的，为什么会有上述的差异，对应的原理又是什么？后来也是查阅了一天，并最终在 StackOverflow 上提问后，得到了解答，具体可以看这个链接 [StackOverflow 提问](https://stackoverflow.com/questions/77542768/mem-fn-bind-function-difference-when-passing-parameters/77542820?noredirect=1#comment136703728_77542820) 。

# 分析思考

简单来说，相似的传参方式却得到了不同的结果，如代码里的 `f1(ff,1)` 和 `f7(ff,1)`，这其实是由于传参时一个是 pass by reference，一个是 pass by value 导致的。下面会进一步地对上述三点进行分析。

1. 对于 `mem_fn` 的表现，也就是 `f1` 和 `f2`，他们传值方式都是 pass by reference，这一点可以在 https://en.cppreference.com/w/cpp/utility/functional/mem_fn 得到证实，可以看到 `operator()(Arg&&... args)`形参格式为 `Args&&` 引用格式，然后进行完美转发再执行函数调用，那么此时就又可以联想之前提到的 `INVOKE` 调用规则。因为是引用，所以传入`ff`对象后执行调用能够直接修改对象，传入`&ff`调用也是修改 `ff` 对象本身。

![Member function](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271133362.png)

2. 对于 `std::bind`，返回一个 function object，对该 object 传入的参数都是拷贝或者移动，除非使用 `std::ref` 或者 `std::cref`表示传入引用，所以传入 `ff` 或者 `&ff`时都是进行拷贝，在真正 `INVOKE`时 `f3` 不能修改`ff` 对象，`f4` 可以。当但使用 `placeholders` 时，可以看到对应说明，会将对应传入的参数转发，转变成 && 引用类型，因此这是传入是个引用，所以 `f2` 的调用能够修改 `ff` 对象。

![](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271133403.png)

![Notes](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271133452.png)

![placeholders](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271133508.png)

3. 对于 `std::function`，可以看到对应的 `operator()`接受参数的类型是 `Args...` 而不是 mem_fn 的 `Args&&...`，注意到这两者细微的差别了么？这里其实就隐含说明传入的参数都是拷贝传入的，所以 `f7` 执行后并不会影响 `ff` 对象。

![std::function operator()](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202311271133587.png)



# 总结

此次遇到的问题，简单来说，就是对于某个结果现象不理解，或者说是不符合预期而产生疑惑，但其实从这个现象，再结合一下C++知识，不难想到一个是值传递一个是引用传递，但我当时就是没有找到对应的佐证，来说明为什么这里是值传递或是引用传递。后来也是仔细翻阅了 cppreference 上的说明，才看到证据，不过限于我个人能力，可能上述的分析也仅仅是我个人的推测，自身也没有很透彻地理解，所以如果哪里理解有误，还请联系我进行改正。

