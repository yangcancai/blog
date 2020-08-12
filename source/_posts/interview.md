---
title: 面试记录
date: 2020-08-06 21:56:10
tags: 面试
categories:
- 面试
---
## 某大厂(一个小时)
### c++基础知识
#### static在哪些地方使用有什么作用
- 类的成员函数或者成员变量可以定义为static
- 所有对象共享静态成员数据,程序开始运行时分配空间，程序结束后释放空间
- 静态方法调用: 类名::函数名
- 静态方法只能调用静态成员
- 静态成员使用前必须初始化且初始化只能在类外,cpp文件中
- static函数和函数外的static变量作用域只在本文件,在cpp中定义。不要在.h中定义
```c++
// StaticDemo.h
#ifndef DEMO_CPP_STATICDEMO_H
#define DEMO_CPP_STATICDEMO_H
#include <iostream>
extern int GlobalData;// 所有include该头文件的cpp都能访问GlobalData
class StaticDemo {
public:
    static int GetData();
    static void SetData(const int& data);
    static void Test();
private:
    static int m_Data;
};

#endif //DEMO_CPP_STATICDEMO_H

// StaticDemo.cpp
#include "StaticDemo.h"
int GlobalData = 2;// 全局变量可以被别的cpp文件共享，需要在头文件添加extern int GlobalData;
int StaticDemo::m_Data = 10;// 只能在cpp定义类的静态变量
void StaticDemo::SetData(const int &data) {
    m_Data = data;
}
int StaticDemo::GetData() {
    return m_Data;
}
void StaticDemo::Test(){
        GlobalData = 3;
        StaticDemo s;
        // 类名访问静态成员函数推荐使用
        std::cout << StaticDemo::GetData() << " ";
        // 对象名访问静态函数
        s.SetData(30);
        std::cout << s.GetData() << " ";
        // 打印结果: 10 30
}
// main.cpp
#include "StaticDemo.h"
#include <iostream>
int main(int argc, char** argv){
   StaticDemo::Test();
   // 别的cpp文件的全局变量
   std::cout << GlobalData << std::endl;
   // 输出结果:3 
   return 0;
}
````
#### 构造函数和析构函数是否可以定义为virtual，为什么？
- virtual的作用就是为了父类的指针可以调用子类的方法
- 虚函数表指针初始化是在构造函数中完成的,所以构造函数不能定义为虚函数
- 如果是父类,析构函数一般都需要定义为virtual,否则删除父类指针的时候不会调用子类的析构函数
- 一般如果父类的方法需要子类重载的话都需要声明为virtual
- virtual会在类对象存储空间存放一个虚函数表的指针，该指针指向类的虚函数表空间
#### 栈和堆的区别
- 栈由操作系统自动分配和释放，结构和数据结构中的栈相似，先分配的内存地址比先分配的内存地址大
- 栈的生命周期，如何是全局变量程序结束释放，如果是局部变量函数结束释放
- 堆由开发人员手动分配和释放，如果开发人员不释放程序结束后由系统释放，堆的内存生长地址和栈相反由低到高
- 堆后申请的内存空间并不一定在先申请的内存空间的后面，即 p2 指向的地址并不一定大于 p1 所指向的内存地址，
  原因是先申请的内存空间一旦被释放，后申请的内存空间则会利用先前被释放的内存，从而导致先后分配的内存空间
  在地址上不存在先后关系 
#### stl有哪些数据结构，哈希结构的实现原理和map的实现原理 
#### 红黑树和平衡二叉树的区别
#### 描述c++代码的内存结构
- static members、virtual function members、nonstatic members
- class object 包括(虚函数表指针vptr、nonstatic data members of base class、nonstatic data members)
#### malloc和new的区别
- malloc只分配空间不进行成员初始化，不执行构造函数。不能调用虚函数
- new是对malloc的封装分配空间并对成员初始化，执行构造函数，生成虚函数表

### 网络相关
- 加密算法有哪些？
- aes有哪些工作模式，cbc向量的实现原理
- 签名算法和加密算法有什么区别，rsa是否可以作为签名算法
- 描述https交互流程
- tcp的状态有哪些
- tcp和http的区别
- 什么是彩虹表？
- 了解xss吗？

### linux相关
- 是否用过gdb，有哪些命令
- 如何查看内存泄漏使用什么工具
- 如何查看dump文件
- 使用什么工具抓包，tcpdump如何抓指定端口数据
- 如何查看应用使用的内存情况，top按内存排序命令是什么
- 网络编程IO复用有哪些模型？select、poll和epoll的区别
- 介绍linux系统内存模型
- 进程间通信有哪些方式
- 进程和线程的区别
### 数据库
- mysql有哪些存储引擎，myisam和innodb有什么区别
- 主键和唯一索引的区别
- 主键为什么要是递增的
- 事务的的底层执行流程
- 事务的隔离级别
- 事务有哪些特性
- 查表总条数语句，select count(*)和select count(name)效果是一样的吗

### rabbitmq
- rabbitmq有哪些组成部分
- 生产者如何保证消息投递成功
- broker如何保证数据投递成功
- 消费者处理数据失败后如何处理
- broker如何投递消息到不同的消费者
- 生产速度大于消费者速度策略处理，如何发现
- 一个节点down掉后如何处理，描述rabbitmq使用的分布式算法
