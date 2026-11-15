# c++基础

- [c++基础](#c++基础)
    - [一、C/C++编译过程](#C/C++编译过程)
    - [二、动态库与静态库](#二动态库与静态库)
        - [socket编程的本质：文件操作](#socket编程的本质文件操作)
        - [socket函数详解](#socket函数详解)
        - [主机字节序与网路字节序](#主机字节序与网路字节序)
        - [网络通信流程](#网络通信流程)
        - [客户端与服务端封装例子](#客户端与服务端封装例子)
    - [三、windows的socket编程](#三windows的socket编程)




## 一、C/C++编译过程
编译过程一共分为：预处理->编译->汇编->链接，共四步，我将用命令行操作来详细分析过程。

### 1.预处理过程
- 命令
~~~ 
g++ -E main.cpp -o main.i
~~~
- 作用
  - 去掉注释
  - 预处理指令替换
  - 头文件替换
  - 生成.i文件

- 具体过程

~~~ cpp
//这是演示源码
#include <iostream>

#define MAXSIZE 100

#ifndef DEMO
    #define DEMO std::cout << "this is demo" << std::endl;
#endif

//这是一个demo
int main(){
    int a[MAXSIZE]= {};
    DEMO
    return 0;
}    
~~~

- 代码执行结果
![image1.png](image1.png)

- 结果解释
预处理完的.i文件和源文件十分相似，不同的是.i文件将"//这是演示源码"等注释去除
执行了"#ifndef #endif"命令,我们可以看到我们写的代码从原来的第十几行的位置变成了
第三万多行，这是应为预处理把"#include <iostream>"这个头文件替换成了头文件的源代码。

### 2.编译阶段
- 命令
~~~
g++ -S main.i -o main.s
~~~

- 作用
  - 检查代码报错
  - 将C/C++语言翻译成汇编语言
  - 生成.s文件

- 具体演示
  - 执行结果（生成汇编代码）
  - ![image2.png](image2.png)
  - 检查错误代码验证
  - ![image3.png](image3.png)
  - 错误代码执行结果
  - ![image4.png](image4.png)

- 结果解释
在执行预处理命令时编译器没有报错，在执行编译命令是报错说   
明检查代码错误是在编译阶段进行的。

### 3.汇编阶段
- 命令
~~~
g++ -c main.s -o main.o
~~~
- 作用
将.s文件汇编成二进制文件即.o文件

### 4.链接阶段
- 命令
~~~ cpp
g++ main.o -o mains
~~~
- 作用
连接产生.exe文件这是最简单的链接操作，只链接了我们写的mian.o和隐式链接了C++中一些标准头文件
例如<iostream>,链接过程和动态库和静态库密切相关，这里只简单讲。

## 二、动态库与静态库
我们在网上下载的各种工具包本质就是一个个静态库或是动态库，了解动态库和静态库的工作
原理能更好懂得如何配置不同的库。

### 1.静态库
- 特征:.a（Linux）或者.lib（windows）的文件
- 性质:由多个.o归档打包而成，链接时嵌入.exe文件中
- 优点：不用外部依赖.o或者.lib文件
- 缺点：当包更新时，需要重新编译链接包费时不灵活，且嵌入.exe文件中是其变臃肿。

- 静态库制作过程   

- - 源码
~~~cpp
//库函数头文件声明了函数体，使用库函数必须先包含头文件

#ifndef TEST_MYMATH_H
#define TEST_MYMATH_H

int add(int a, int b);

#endif //TEST_MYMATH_H

~~~
~~~ cpp
//函数的具体实现

#include "Mymath.h"

int add(int a, int b) {
    return a + b;
}
~~~
~~~
#将.cpp文件转为.o文件
g++ -c Mymath.cpp -o Mymath.o
#将.o文件打包成.a文件（后面可跟多个.o文件一起打包成一个.a文件），链接库的习惯命名要以lib开头.a结尾
ar rcs libMymath.a Mymath.o
#将main.cpp文件一步转为.o文件,链接Mymath库
g++ main.cpp -o main libMymath.a 
~~~
~~~ cpp
//包含与库关联的头文件就能使用库中的函数
#include <iostream>
#include "Mymath.h"

int main() {
    std::cout << add(1,2) << std::endl;
    return 0;
}
~~~

### 2.动态态库
- 特征:.so（Linux）或者.dll（windows）的文件
- 性质:共享目标文件，代码不会在链接阶段复制到可执行程序。程序运行时，由操作系统动态链接器加载到进程内存，多个进程可共享同一份库内存。
程序寻找动态库是先在当前目录下找，然后再到环境变量中找。
- 优点：库更新时只需要替换库文件，不用重新编译（前提是更新不改变函数的名字和接口）
- 缺点：在运行时找不到动态库才报错。
- 动态库的制作
~~~
# 1. 编译成位置无关的.o
g++ -c -fPIC Mymath.cpp -o Mymath.o
# 2. 打包生成动态库 libMymath.so
g++ -shared -o libMymath.so Mymath.o
# 链接动态库和静态库方式一样
g++ main.cpp -o main libMymath.so
~~~


