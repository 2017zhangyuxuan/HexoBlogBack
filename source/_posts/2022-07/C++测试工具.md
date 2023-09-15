---
title: C++测试工具
date: 2022-07-30 15:43:36
excerpt: TDD（Test-Driven Development)测试驱动开发是工程实践开发的一大技术点，因此本期将介绍C++中常用的测试工具，包括经典的GoogleTest，以及一些轻量级的开源测试工具。
index_img: https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202207301957269.png
categories: 
- [计算机知识,C++]
tags:
- C++
- googletest
---

# 前言

说来惭愧，断更了一个多月了，一方面的确是最近开始实习，时间变得紧张起来，另一方面也的确是自己偷懒了，松懈了不少。另外，比较难受的是，本来这篇博文已经在另一台电脑上写好了大概，准备上周就发的，但是因为被一些事情耽搁了，结果这周再去找的时候，结果发现原文找不到了，真的是老倒霉蛋了，所以只能硬着头皮重新写一篇 QAQ。

# gtest

那么首先来介绍GoogleTest（简称gtest），这是由google推出的C++测试框架，其功能之强大自然不必多说，接下来就简单介绍gtest的相关概念及其使用，更多的使用说明可以参考官方文档：https://google.github.io/googletest/primer.html

首先介绍一些术语概念，分别是 *Test*, *Test Case* and *Test Suite* ，这几个单词在gtest框架中的含义 与在[ISTQB](http://www.istqb.org/)（国际软件测试资质认证委员会）定义的含义有所不同，这里就介绍在gtest里的含义。因为历史原因，在gtest中， *Test* 认为是一个单元测试程序（指定输入，检测输出），而 *Test Cast* 被认为是一组相关的测试，现在，google开始用 *Test Suite* 来替换 *Test Case*。

此外，还有一些基本概念，例如在gtest中，使用断言（Assertion）判断一个条件成立与否，一个 Assertion 的结果可以是success, non-fatal failure, or fatal failure， 如果是fatal failure 则测试程序直接终止，而如果是non-fatal failure，则程序程序继续执行，只有当测试程序没有出现任何 failure 时，测试才算通过。一个 Test Suite可以包含多个 test，这些 test 可以共享一些对象和流程，可以放在 TestFixture class中。而一个测试程序可以包含多个 Test Suite。

刚刚提到了Assertion 断言，相关的API就是ASSERT_ * 和 EXPECT_ * ，如果ASSERT失败了则程序执行，而EXPECT失败了，则程序会继续执行下面的测试。可以使用TEST() 宏定义来创建一个简单的单元测试，同时gtest使用TestSuiteName来汇总test，所以具有相关性的test使用相同的TestSuiteName。

```C++
int func(int n) {...}

TEST(TestSuiteName, TestName1) {
  EXPECT_EQ(func(0), 0);
}

TEST(TestSuiteName, TestName2) {
  EXPECT_EQ(func(0), 0);
}
```

> 需要注意的是，**TestSuiteName 和 TestName 最好不要使用下划线**，因为在gtest源码中，会使用下划线将它们拼接成一个类名。这再简单介绍下gtest的实现吧，当我们使用TEST编写好一个test，该宏会将其展开为一个类，并声明一个TestBody函数，而TestBody的实现就是TEST里用户编写的逻辑。

而当TestSuite中有公共流程时，就可以提取出来，避免重复编写，这就是TestFixture。TestFixture需要使用TEST_F来定义test，其中`SetUp`函数在会在每次TEST_F执行前初始化一次，`TearDown` 函数则是每次在执行完之后执行，释放资源。一个简单例子如下：

```C++
#include "gtest/gtest.h"

class FuncTest : public ::testing::Test {
 protected:
  void SetUp() override {
     q1_.Enqueue(1);
     q2_.Enqueue(2);
     q2_.Enqueue(3);
  }

  void TearDown() override {
  }

  Queue<int> q0_;
  Queue<int> q1_;
  Queue<int> q2_;
};

TEST_F(FuncTest, IsEmptyInitially) {
  EXPECT_EQ(q0_.size(), 0);
}

TEST_F(FuncTest, DequeueWorks) {
  int* n = q0_.Dequeue();
  EXPECT_EQ(n, nullptr);

  n = q1_.Dequeue();
  ASSERT_NE(n, nullptr);    // 继续运行已无必要，失败时直接中断
  EXPECT_EQ(*n, 1);         // 继续运行暴露更多问题
  EXPECT_EQ(q1_.size(), 0);
  delete n;

  ....
}

int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
```

整体流程：

1. 缓存gtest flags的状态；

2. 为第一个test创建TestFixture对象；

3. gtest创建FuncTest对象（o1）；

4. o1.SetUp()；

5. 基于o1，运行第一个test（IsEmptyInitially）；

6. o1.TearDown；

7. 析构o1；

8. 在gtest flags中存储状态；

9. 重复上述操作，运行下一个test（DequeueWorks），一直到结束；



<p class='note note-info'>
  还值得一提的是，如果需要访问private & protected 成员和函数，可以添加编译选项 -fon-access-control，如果使用cmake的话，可以添加 ADD_COMPILE_OPTIONS(-fno-access-control) 
</p>



# gmock

gmock也是gtest测试框架中的重要组成部分，当我们测试一个模块的时候，可能涉及到和其他模块交互，此时可以将模块之间的接口mock，模拟交互过程，其作用类似白盒测试中的打桩的概念。（打桩是软件测试里单元测试的一种方法，单元测试涉及手工编写测试集、指定输入数据以及为缺少的函数提供桩函数。给桩函数提供返回值叫做打桩。）

关于gmock的使用也可以查阅官方文档：https://google.github.io/googletest/gmock_for_dummies.html

其基本使用方式为，当需要mock某一个类的成员方法（注意：需要声明为**虚函数**），定义一个派生类，然后继承需要mock的类，并声明`MOCK_METHOD` ，然后设置`EXPECT_CALL`指定其函数的返回行为。如下面的例子所示：

```C++
// origin file
class Origin {
    virutal ~Origin() {}
    virtual int Do(int x, int y) const = 0;
    virtual int Do2() const = 0;
};

// --------------------------------------------------------------

#include "gmock/gmock.h"

class Mock: public Origin {                       // 派生
    public:
        MOCK_METHOD(int, Do, (int x, int y), (override));
        MOCK_METHOD(int, Do2, (), (const, override));
};

using ::testing::Return;                          // 引入依赖函数
using ::testing::_;                               // 引入依赖函数

TEST(BarTest, DoesThis) {
  MockFoo foo;                                    // 构建mock对象

  ON_CALL(foo, Do2())                         // 设置默认行为,不要求被调用
      .WillByDefault(Return(1));

  EXPECT_CALL(foo, Do2(5, _))                   // 开始设置命中matcher(x=50)期望
      .Times(3)                                 // 期望被调用的次数
      .WillOnce(Return(100))                    // 期望的action
      .WillRepeatedly(Return(90));
  
  EXPECT_EQ(MyProductionFunction(&foo), "good");//调用使用mock对象的函数
                                                //可以检测函数结果
}                           // mock对象析构，gmock检测是否满足全部exception

........
```

> 如果没有显示指定`Times`
>
> - `WillOnce` / `WillRepeatedly` 都没有在 EXPECT_CALL 中被调用，time = 1
> - n 个 `WillOnce`，没有 `WillRepeatedly` (n >= 1)， time = n
> - n 个 `WillOnce`， 1个`WillRepeatedly` (n >= 0)，time = AtLeast(n)

## MOCK_METHOD

其基本结构为如下所示：

```C++
class MyMock {
    public:
        MOCK_METHOD(ReturnType, MethodName, (Args..), (Specs..));
}
```

ReturnTypeo为被mock函数的返回类型，MethodName为被mock的函数名，（Args...)为被mock函数的参数，可选参数（Specs...)为mock函数修饰参数，如override,const等。

## EXPECT_CALL

给mock对象的命中matchers的method设定expection，需要在调用mock对象前设置。

```C++
EXPECT_CALL(mock_object, method(matchers...))
    .With(multi_argument_matcher)  // Can be used at most once
    .Times(cardinality)            // Can be used at most once
    .InSequence(sequences...)      // Can be used any number of times
    .After(expectations...)        // Can be used any number of times
    .WillOnce(action)              // Can be used any number of times
    .WillRepeatedly(action)        // Can be used at most once
    .RetiresOnSaturation();        // Can be used at most once
```

- matchers：单值matcher，_表示接受任意参数，内置的matcher宏详见[链接](https://google.github.io/googletest/reference/matchers.html)

- multi_argument_matcher： [多参数matcher](https://google.github.io/googletest/reference/matchers.html#MultiArgMatchers)，约束method的参数的形式（Lt, 第一个参数小于第二）

- Time：AnyNumber、AtLeast、Between ... 限定执行次数

- InSequence：指定mock函数的调用顺序。

- After：指定mock函数在特定函数后调用
- WillOnce: mock函数被调用时的单次[行为](https://google.github.io/googletest/reference/actions.html)
- WillRepeatedly: 所有后续的调用都命中的行为

> EXPECT_CALL(mock, Func())
>
> .WillOnce(Return(1))               // 第一次返回1               
>
> .WillRepeatedly(Return(2))     // 后续都返回2

- RetriesOnSaturation: 命中足够次数后忽略

> EXPECT_CALL(mock, do()).Times(AnyNumber());
>
> EXPECT_CALL(mock, do(9)).Time(2).RetriesOnSaturation();

## 处理未包裹的逗号

gmock的时候，如果函数原型中包含","信息，则会编译失败：

```C++
class MockM {
    public:
        MOCK_METHOD(std::pair<bool, int>, GetPair, ());
        MOCK_METHOD(bool, CheckMap, (std::map<int, double>, bool));
}
```

可选如下的解决方法：

1. ( ) 包裹

```Plaintext
class MockM {
    public:
        MOCK_METHOD((std::pair<bool, int>), GetPair, ());
        MOCK_METHOD(bool, CheckMap, ((std::map<int, double>), bool));
}
```

2. 定义alias

```Plaintext
class MockM {
    public:
        using BoolAndInt = std::pair<bool, int>;
        MOCK_METHOD(BoolAndInt, GetPair, ());
}
```

## 有序调用

代码中指定某些函数，按顺序执行。

```undefined
using ::testing::InSequence;

TEST(FooTest, test) {
    {
        InSequence seq;
        EXPECT_CALL(turtle, PenDown());
        EXPECT_CALL(turtle, Forward(100));
        EXPECT_CALL(turtle, PenUp());
    }
    ...
}
```



# 轻量级开源测试工具

因为GoogleTest依赖比较重，因此 [现代C++ Unit Test库](https://mp.weixin.qq.com/s/-6HMlVLSRdd6ycr5fSqhKw) 这篇文章介绍几个轻量易用的单元测试库，doctest（对应于gtest），fakeit（对应于gmock，同时还能集成到gtest中），nanobench（benchmark库），这几个开源库只需要引入头文件即可使用，非常方便。文章里也都有简单的使用介绍，或者看看github上的文档即可，这里就不再赘述了（~~实际上是不想再写第二遍了~~）。不过简单使用之后，个人感觉和gtest和gmock差别不是很大，如果项目不大或者此前也没怎么用过C++测试工具的话，这两个工具可以尝试一下的，当然直接去学习gtest和gmock也不亏，就是需要点成本罢了。

不过这个nanobench库还是值得学习了解的，因为此前我对性能测试相关的工具了解不多，下面列了下其官方文档上给的例子，这里做个简单的翻译。

```C++
#define ANKERL_NANOBENCH_IMPLEMENT
#include <nanobench.h>

int main() {
    double d = 1.0;
    ankerl::nanobench::Bench().run("some double ops", [&] {
        d += 1.0 / d;
        if (d > 5.0) {
            d -= 5.0;
        }
        ankerl::nanobench::doNotOptimizeAway(d);
    });
}
```




|               ns/op |                op/s |    err% |          ins/op |          cyc/op |    IPC |         bra/op |   miss% |     total | benchmark
|--------------------:|--------------------:|--------:|----------------:|----------------:|-------:|---------------:|--------:|----------:|:----------
|                7.52 |      132,948,239.79 |    1.1% |            6.65 |           24.07 |  0.276 |           1.00 |    8.9% |      0.00 | `some double ops`

其结果如上表所示，大概含义是上面执行的代码花费了7.52纳秒，因此大约每秒能执行133 millon 次，测量浮动率为1.1%，每次执行需要6.65条指令，24.07个CPU周期，因此IPC （指令/时钟）为0.276。该代码中只有1个分支，分支预测失败概率为8.9%，最后的total为执行时间，这里显示0.00 表示只花费一些毫秒。

> 需要注意的是，CPU的一些数据，如指令数，CPU周期，分支数与分支预测失败率只能在Linux获取，因为是通过perf events采集得到的。



# 总结



最后小结一下吧，在学校的时候，其实自己都没怎么关注过测试，随手写的一些项目代码，都是只要能正确实现功能就行，没有说要写什么测试代码来保证代码质量，但是到了实际生产工作中，单元测试就被重视起来了，
