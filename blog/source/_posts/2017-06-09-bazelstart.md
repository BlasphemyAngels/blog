---
title: goole开源构建工具bazel的简单使用
date: 2017-06-09 11:17:35
categories:
  - 工具
tags:
  - bazel
  - google
---


## 正文

### bazel概述

&ensp;&ensp;&ensp;bazel是一个构建代码工具，能对代码进行编译和测试，可以支持几乎任何语言。

&ensp;&ensp;&ensp;bazel通过一个名为BUILD的文件进行构建代码，BUILD文件中写有构建代码的规则，这些规则语法简单，很容易书写，我们可以通过书写这些规则来规定我们需要的代码构建方式，得到我们自己需要的结果。

<!--more-->

&ensp;&ensp;&ensp;下面展示了c语言的部分规则。

```python
cc_library(
    name = "hello-time",
    srcs = ["hello-time.cc"],
    hdrs = ["hello-time.h"],
)

cc_binary(
    name = "hello-world",
    srcs = ["hello-world.cc"],
    deps = [
        ":hello-time",
        "//lib:hello-greet",
    ],
)
```

&ensp;&ensp;&ensp;有的时候代码中是存在依赖关系的，观察上面的第二条构建规则，其中`deps`代表依赖，表示`hello-world`依赖于`：hello-time`和`//lib:hello-greet`。 `bazel`会根据`BUILD`文件生成一张依赖图，这张依赖图会一直保存在内存中，以让后面 `bazel`进行`增量构建`和`并行计算`使用。

下图是上面展示规则的依赖图

![](data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIKICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8IS0tIEdlbmVyYXRlZCBieSBncmFwaHZpeiB2ZXJzaW9uIDIuMzYuMCAoMjAxNDAxMTEuMjMxNSkKIC0tPgo8IS0tIFRpdGxlOiBteWdyYXBoIFBhZ2VzOiAxIC0tPgo8c3ZnIHdpZHRoPSI0MDZwdCIgaGVpZ2h0PSIxOTBwdCIKIHZpZXdCb3g9IjAuMDAgMC4wMCA0MDYuMDAgMTkwLjAwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIj4KPGcgaWQ9ImdyYXBoMCIgY2xhc3M9ImdyYXBoIiB0cmFuc2Zvcm09InNjYWxlKDEgMSkgcm90YXRlKDApIHRyYW5zbGF0ZSg0IDE4NikiPgo8dGl0bGU+bXlncmFwaDwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9IndoaXRlIiBzdHJva2U9Im5vbmUiIHBvaW50cz0iLTQsNCAtNCwtMTg2IDQwMiwtMTg2IDQwMiw0IC00LDQiLz4KPCEtLSAvL21haW46aGVsbG8mIzQ1O3dvcmxkIC0tPgo8ZyBpZD0ibm9kZTEiIGNsYXNzPSJub2RlIj48dGl0bGU+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZDwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIHBvaW50cz0iMjY0LjI1LC0xODIgMTQ1Ljc1LC0xODIgMTQ1Ljc1LC0xNDYgMjY0LjI1LC0xNDYgMjY0LjI1LC0xODIiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMjA1IiB5PSItMTYwLjMiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCI+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZDwvdGV4dD4KPC9nPgo8IS0tIC8vbWFpbjpoZWxsbyYjNDU7dGltZSAtLT4KPGcgaWQ9Im5vZGUyIiBjbGFzcz0ibm9kZSI+PHRpdGxlPi8vbWFpbjpoZWxsbyYjNDU7dGltZTwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIHBvaW50cz0iMTE5LjUsLTExMCA4LjUsLTExMCA4LjUsLTc0IDExOS41LC03NCAxMTkuNSwtMTEwIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjY0IiB5PSItODguMyIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIj4vL21haW46aGVsbG8mIzQ1O3RpbWU8L3RleHQ+CjwvZz4KPCEtLSAvL21haW46aGVsbG8mIzQ1O3dvcmxkJiM0NTsmZ3Q7Ly9tYWluOmhlbGxvJiM0NTt0aW1lIC0tPgo8ZyBpZD0iZWRnZTEiIGNsYXNzPSJlZGdlIj48dGl0bGU+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZCYjNDU7Jmd0Oy8vbWFpbjpoZWxsbyYjNDU7dGltZTwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIGQ9Ik0xNzAuNTA4LC0xNDUuODc2QzE1MS41NiwtMTM2LjQ2OSAxMjcuODY4LC0xMjQuNzA4IDEwNy42NDQsLTExNC42NjciLz4KPHBvbHlnb24gZmlsbD0iYmxhY2siIHN0cm9rZT0iYmxhY2siIHBvaW50cz0iMTA5LjEzOCwtMTExLjUwMiA5OC42MjUxLC0xMTAuMTkgMTA2LjAyNiwtMTE3Ljc3MSAxMDkuMTM4LC0xMTEuNTAyIi8+CjwvZz4KPCEtLSAvL21haW46aGVsbG8mIzQ1O3dvcmxkLmNjIC0tPgo8ZyBpZD0ibm9kZTMiIGNsYXNzPSJub2RlIj48dGl0bGU+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZC5jYzwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIHBvaW50cz0iMjcyLC0xMTAgMTM4LC0xMTAgMTM4LC03NCAyNzIsLTc0IDI3MiwtMTEwIi8+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjIwNSIgeT0iLTg4LjMiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCI+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZC5jYzwvdGV4dD4KPC9nPgo8IS0tIC8vbWFpbjpoZWxsbyYjNDU7d29ybGQmIzQ1OyZndDsvL21haW46aGVsbG8mIzQ1O3dvcmxkLmNjIC0tPgo8ZyBpZD0iZWRnZTIiIGNsYXNzPSJlZGdlIj48dGl0bGU+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZCYjNDU7Jmd0Oy8vbWFpbjpoZWxsbyYjNDU7d29ybGQuY2M8L3RpdGxlPgo8cGF0aCBmaWxsPSJub25lIiBzdHJva2U9ImJsYWNrIiBkPSJNMjA1LC0xNDUuNjk3QzIwNSwtMTM3Ljk4MyAyMDUsLTEyOC43MTIgMjA1LC0xMjAuMTEyIi8+Cjxwb2x5Z29uIGZpbGw9ImJsYWNrIiBzdHJva2U9ImJsYWNrIiBwb2ludHM9IjIwOC41LC0xMjAuMTA0IDIwNSwtMTEwLjEwNCAyMDEuNSwtMTIwLjEwNCAyMDguNSwtMTIwLjEwNCIvPgo8L2c+CjwhLS0gLy9saWI6aGVsbG8mIzQ1O2dyZWV0IC0tPgo8ZyBpZD0ibm9kZTQiIGNsYXNzPSJub2RlIj48dGl0bGU+Ly9saWI6aGVsbG8mIzQ1O2dyZWV0PC90aXRsZT4KPHBvbHlnb24gZmlsbD0ibm9uZSIgc3Ryb2tlPSJibGFjayIgcG9pbnRzPSIzOTEuMjUsLTExMCAyOTAuNzUsLTExMCAyOTAuNzUsLTc0IDM5MS4yNSwtNzQgMzkxLjI1LC0xMTAiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMzQxIiB5PSItODguMyIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIj4vL2xpYjpoZWxsbyYjNDU7Z3JlZXQ8L3RleHQ+CjwvZz4KPCEtLSAvL21haW46aGVsbG8mIzQ1O3dvcmxkJiM0NTsmZ3Q7Ly9saWI6aGVsbG8mIzQ1O2dyZWV0IC0tPgo8ZyBpZD0iZWRnZTMiIGNsYXNzPSJlZGdlIj48dGl0bGU+Ly9tYWluOmhlbGxvJiM0NTt3b3JsZCYjNDU7Jmd0Oy8vbGliOmhlbGxvJiM0NTtncmVldDwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIGQ9Ik0yMzguMjY5LC0xNDUuODc2QzI1Ni40NjMsLTEzNi41MTIgMjc5LjE5MSwtMTI0LjgxNCAyOTguNjQsLTExNC44MDMiLz4KPHBvbHlnb24gZmlsbD0iYmxhY2siIHN0cm9rZT0iYmxhY2siIHBvaW50cz0iMzAwLjMxMywtMTE3Ljg3OCAzMDcuNjAzLC0xMTAuMTkgMjk3LjExLC0xMTEuNjU0IDMwMC4zMTMsLTExNy44NzgiLz4KPC9nPgo8IS0tIC8vbWFpbjpoZWxsbyYjNDU7dGltZS5jY1xuLy9tYWluOmhlbGxvJiM0NTt0aW1lLmggLS0+CjxnIGlkPSJub2RlNSIgY2xhc3M9Im5vZGUiPjx0aXRsZT4vL21haW46aGVsbG8mIzQ1O3RpbWUuY2Ncbi8vbWFpbjpoZWxsbyYjNDU7dGltZS5oPC90aXRsZT4KPHBvbHlnb24gZmlsbD0ibm9uZSIgc3Ryb2tlPSJibGFjayIgcG9pbnRzPSIxMjgsLTM4IDAsLTM4IDAsLTAgMTI4LC0wIDEyOCwtMzgiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNjQiIHk9Ii0yMi44IiBmb250LWZhbWlseT0iVGltZXMsc2VyaWYiIGZvbnQtc2l6ZT0iMTQuMDAiPi8vbWFpbjpoZWxsbyYjNDU7dGltZS5jYzwvdGV4dD4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNjQiIHk9Ii03LjgiIGZvbnQtZmFtaWx5PSJUaW1lcyxzZXJpZiIgZm9udC1zaXplPSIxNC4wMCI+Ly9tYWluOmhlbGxvJiM0NTt0aW1lLmg8L3RleHQ+CjwvZz4KPCEtLSAvL21haW46aGVsbG8mIzQ1O3RpbWUmIzQ1OyZndDsvL21haW46aGVsbG8mIzQ1O3RpbWUuY2Ncbi8vbWFpbjpoZWxsbyYjNDU7dGltZS5oIC0tPgo8ZyBpZD0iZWRnZTQiIGNsYXNzPSJlZGdlIj48dGl0bGU+Ly9tYWluOmhlbGxvJiM0NTt0aW1lJiM0NTsmZ3Q7Ly9tYWluOmhlbGxvJiM0NTt0aW1lLmNjXG4vL21haW46aGVsbG8mIzQ1O3RpbWUuaDwvdGl0bGU+CjxwYXRoIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIGQ9Ik02NCwtNzMuODEyOUM2NCwtNjYuMTEwMSA2NCwtNTYuODIzNCA2NCwtNDguMTQ5Ii8+Cjxwb2x5Z29uIGZpbGw9ImJsYWNrIiBzdHJva2U9ImJsYWNrIiBwb2ludHM9IjY3LjUwMDEsLTQ4LjAxOTUgNjQsLTM4LjAxOTYgNjAuNTAwMSwtNDguMDE5NiA2Ny41MDAxLC00OC4wMTk1Ii8+CjwvZz4KPCEtLSAvL2xpYjpoZWxsbyYjNDU7Z3JlZXQuY2Ncbi8vbGliOmhlbGxvJiM0NTtncmVldC5oIC0tPgo8ZyBpZD0ibm9kZTYiIGNsYXNzPSJub2RlIj48dGl0bGU+Ly9saWI6aGVsbG8mIzQ1O2dyZWV0LmNjXG4vL2xpYjpoZWxsbyYjNDU7Z3JlZXQuaDwvdGl0bGU+Cjxwb2x5Z29uIGZpbGw9Im5vbmUiIHN0cm9rZT0iYmxhY2siIHBvaW50cz0iMzk4LjUsLTM4IDI4My41LC0zOCAyODMuNSwtMCAzOTguNSwtMCAzOTguNSwtMzgiLz4KPHRleHQgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMzQxIiB5PSItMjIuOCIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIj4vL2xpYjpoZWxsbyYjNDU7Z3JlZXQuY2M8L3RleHQ+Cjx0ZXh0IHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjM0MSIgeT0iLTcuOCIgZm9udC1mYW1pbHk9IlRpbWVzLHNlcmlmIiBmb250LXNpemU9IjE0LjAwIj4vL2xpYjpoZWxsbyYjNDU7Z3JlZXQuaDwvdGV4dD4KPC9nPgo8IS0tIC8vbGliOmhlbGxvJiM0NTtncmVldCYjNDU7Jmd0Oy8vbGliOmhlbGxvJiM0NTtncmVldC5jY1xuLy9saWI6aGVsbG8mIzQ1O2dyZWV0LmggLS0+CjxnIGlkPSJlZGdlNSIgY2xhc3M9ImVkZ2UiPjx0aXRsZT4vL2xpYjpoZWxsbyYjNDU7Z3JlZXQmIzQ1OyZndDsvL2xpYjpoZWxsbyYjNDU7Z3JlZXQuY2Ncbi8vbGliOmhlbGxvJiM0NTtncmVldC5oPC90aXRsZT4KPHBhdGggZmlsbD0ibm9uZSIgc3Ryb2tlPSJibGFjayIgZD0iTTM0MSwtNzMuODEyOUMzNDEsLTY2LjExMDEgMzQxLC01Ni44MjM0IDM0MSwtNDguMTQ5Ii8+Cjxwb2x5Z29uIGZpbGw9ImJsYWNrIiBzdHJva2U9ImJsYWNrIiBwb2ludHM9IjM0NC41LC00OC4wMTk1IDM0MSwtMzguMDE5NiAzMzcuNSwtNDguMDE5NiAzNDQuNSwtNDguMDE5NSIvPgo8L2c+CjwvZz4KPC9zdmc+Cg==)

&ensp;&ensp;&ensp;这种依赖图，可以通过bazel的命令来生成它。

&ensp;&ensp;&ensp;基于规则的构建能够生成正确的、可重用的构建结果，缓存的存在使
得bazel能够进行增量构建。

### 快速使用bazel

#### 创建工作区

&ensp;&ensp;&ensp;`bazel`是通过工作区为单位来工作的，要进行构建的一个任务的源码必须放在一个工作区内，这样在构建完成后就会在工作区内生成构建结果(如,`bazel-bin` 和`bazel-out`)。那么究竟什么是工作区呢？很简单，就是一个包含有名为`WORKSPACE`文件的目录，目录的位置在哪不重要，重要的是一定要包含一个名为`WORKSPACE`的文件，这个`WORKSPACE`文件可以是空白的，但是必须存在，也可以在其中加入一些控制内容，这个不是本文的范畴，所以不做讨论。

所以创建工作区即为：

```shell
mkdir test
cd test
touch WORKSPACE
```

#### 创建BUILD文件

&ensp;&ensp;&ensp;`bazel`使用`BUILD`来指定代码构建的动作，包括怎么构建，构建的结果，构建的源文件等。

&ensp;&ensp;&ensp;`BUILD`内一般包含一系列的规则声明。这些规则声明指定输入和输出，还有从输入到输出的计算如何进行。

下面就是一个简单的规则声明：

```shell
genrule(
  name = "hello",
  outs = ["hello_world.txt"],
  cmd = "echo Hello World > $@",
)
```
然后就能通过下面的命令构建它：

`bazel build :hello`

命令行出现如下信息：

```shell
INFO: Found 1 target...
Target //:hello up-to-date:
  bazel-genfiles/hello_world.txt
INFO: Elapsed time: 2.255s, Critical Path: 0.07s
```
&ensp;&ensp;&ensp;输出的目标我们可以通过它的标签（name的值）来使用。`bazel`的构建结果放在另一个目录中，`bazel-genfiles`目录只是一个符号文件，它的真实位置在别处。这样做的目的是不污染源代码目录树。

&ensp;&ensp;&ensp;一个规则可以使用其他规则输出作为输入，通过它的标签指定即可。如下：

```shell
genrule(
  name = "hello",
  outs = ["hello_world.txt"],
  cmd = "echo Hello World > $@",
)

genrule(
  name = "double",
  srcs = [":hello"],
  outs = ["double_hello.txt"],
  cmd = "cat $< $< > $@",
)
```
### 构建c++代码

&ensp;&ensp;&ensp;这里以c++为例展示一下用`bazel`构建程序的方法。

#### 创建工作区

&ensp;&ensp;&ensp;创作下面目录及文件形成工作区。

```shell
└── my-project
    ├── lib
    │   ├── BUILD
    │   ├── hello-greet.cc
    │   └── hello-greet.h
    ├── main
    │   ├── BUILD
    │   ├── hello-time.cc
    │   ├── hello-time.h
    │   └── hello-world.cc
    └── WORKSPACE
```
 
#### 写入源代码

```shell
cd ~/gitroot/my-project
mkdir ./main
cat > main/hello-world.cc <<'EOF'

#include "lib/hello-greet.h"
#include "main/hello-time.h"
#include <iostream>
#include <string>

int main(int argc, char** argv) {
  std::string who = "world";
  if (argc > 1) {
    who = argv[1];
  }
  std::cout << get_greet(who) <<std::endl;
  print_localtime();
  return 0;
}
EOF

cat > main/hello-time.h <<'EOF'

#ifndef MAIN_HELLO_TIME_H_
#define MAIN_HELLO_TIME_H_

void print_localtime();

#endif
EOF

cat > main/hello-time.cc <<'EOF'

#include "main/hello-time.h"
#include <ctime>
#include <iostream>

void print_localtime() {
  std::time_t result = std::time(nullptr);
  std::cout << std::asctime(std::localtime(&result));
}
EOF

mkdir ./lib
cat > lib/hello-greet.h <<'EOF'

#ifndef LIB_HELLO_GREET_H_
#define LIB_HELLO_GREET_H_

#include <string>

std::string get_greet(const std::string &thing);

#endif
EOF

cat > lib/hello-greet.cc <<'EOF'

#include "lib/hello-greet.h"
#include <string>

std::string get_greet(const std::string& who) {
  return "Hello " + who;
}
EOF
```

#### 编写BUILD文件



## 参考链接

* [bazel google documentation](https://bazel.build/versions/master/docs/bazel-overview.html)
