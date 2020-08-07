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
- 栈和堆的区别
- 构造函数和析构函数是否可以定义为virtual，为什么？
- stl有哪些数据结构，哈希结构的实现原理和map的实现原理 
- 红黑树和平衡二叉树的区别
- 描述c++代码的内存结构
### 网络相关
- 加密算法有哪些？
- aes有哪些工作模式，cbc向量的实现原理
- 签名算法和加密算法有什么区别，rsa是否可以作为签名算法
- 描述https交互流程
- tcp的状态有哪些
- tcp和http的区别

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
