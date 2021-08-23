---
title: C++学习--Clion开发环境设置
date: 2021-07-08 23:32:25
tags: C++
---



##### 一、Windows gcc环境配置

首先下载MinGW，[官方下载地址](https://osdn.net/projects/mingw/releases/)，[百度网盘](https://pan.baidu.com/s/1uOuxXCSCAiixa7O4l_0j6A)的下载地址，提取码为4dmt ，下载完成后解压到某个路径后，配置对应环境变量映射到解压目录的bin目录。

验证：win+R 输入cmd，命令行输入 gcc+ -v，提示环境信息即成功

##### 二、配置编译器

默认已安装JetBrains 旗下Clion编辑器，新建C++项目后，Next后提示配置环境变量，选择MingGW安装目录会自动联想补充，Finish完成。

Clion会自动生成main.cpp文件

```c++
/*
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
*/
//提前声明命名空间写法
#include <iostream>
using namespace std;
int main(){
    cout << "Hello World!";//print hello world
    return 0;
}
```

解释下上面代码：

- C++ 语言定义了一些头文件，这些头文件包含了程序中必需的或有用的信息。上面这段程序中，包含了头文件 **<iostream>**。
- 下一行 **using namespace std;** 告诉编译器使用 std 命名空间。命名空间是 C++ 中一个相对新的概念。
- 下一行 **// main() 是程序开始执行的地方** 是一个单行注释。单行注释以 // 开头，在行末结束。
- 下一行 **int main()** 是主函数，程序从这里开始执行。
- 下一行 **cout << "Hello World";** 会在屏幕上显示消息 "Hello World"。
- 下一行 **return 0;** 终止 main( )函数，并向调用进程返回值 0。
