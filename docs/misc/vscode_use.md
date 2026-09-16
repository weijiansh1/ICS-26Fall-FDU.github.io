
---
title: VS Code 中的 C/C++：编译、运行、调试与 Git
---
# VS Code 中的 C/C++：编译、运行、调试与 Git

本文面向第一次使用，或刚开始使用 VS Code 进行 C/C++ 开发的同学。

本文不会把 VS Code 当作一个“点一下按钮就能运行代码”的黑盒，而是希望帮助大家理解：

- VS Code、编译器、调试器分别在做什么；
- C/C++ 程序是怎样从源代码变成可执行程序的；
- `gcc`、`g++`、`gdb` 分别有什么作用；
- VS Code 中几个看起来很像的 “Run” 到底有什么区别；
- 程序出错以后，应该怎样判断问题发生在哪一步；
- 如何使用 VS Code 的图形界面完成基本 Git 操作。

!!! info "本文默认的实验环境"

    本文默认你已经按照课程实验入门手册配置好 Linux / WSL 环境，并能够使用 VS Code 打开其中的文件。

    文中的 `gcc`、`g++`、`gdb` 等命令均指 Linux 环境中的工具。

    如果你使用 Windows 原生的 MinGW、MSVC，或 macOS 上的 Clang，部分命令和界面可能有所不同。

---

## 0. 先记住一件事：VS Code 不是编译器

这是理解后面所有内容最重要的一点。

VS Code 本质上是一个**代码编辑器（Editor）**。

它可以帮助我们：

- 编辑代码；
- 浏览文件；
- 调用终端；
- 调用编译器；
- 调用调试器；
- 管理 Git 仓库；
- 通过扩展增加更多功能。

但是：

> **VS Code 本身并不会编译 C/C++ 程序。**

真正负责 C/C++ 编译的，是 `gcc`、`g++`、Clang 等编译工具。

例如，一个最简单的 C 程序从源代码到运行，大致经历：

```text
main.c
  │
  │ gcc main.c -o main
  ▼
可执行文件 main
  │
  │ ./main
  ▼
程序运行
```

如果是 C++：

```text
main.cpp
  │
  │ g++ main.cpp -o main
  ▼
可执行文件 main
  │
  │ ./main
  ▼
程序运行
```

VS Code 中的各种 Run、Build、Debug 按钮，本质上都是在帮你调用这些工具。

!!! tip "遇到 VS Code 问题时，最重要的排错原则"

    **不要只看按钮有没有反应，要看它背后究竟执行了什么命令。**

---

# 1. 开始之前：确认工具是否可用

在 VS Code 中打开：

```text
Terminal → New Terminal
```

然后分别输入：

```bash
gcc --version
```

```bash
g++ --version
```

```bash
gdb --version
```

如果能够正常显示版本信息，说明对应工具已经安装，并且当前 Shell 可以找到它。

如果出现：

```text
command not found
```

说明当前环境中还没有正确安装或配置相应工具。

在 Ubuntu / WSL 中，可以使用：

```bash
sudo apt update
sudo apt install build-essential gdb
```

安装常用的 C/C++ 编译工具和 GDB。

!!! warning "确认自己到底在哪个环境里"

    Windows 用户尤其需要注意：

    VS Code 可以同时打开 Windows 本机目录和 WSL 中的目录。

    你应该确认当前终端究竟是：

    ``text     PowerShell / CMD     ``

    还是：

    ``text     bash / zsh in WSL     ``

    对本课程而言，除非实验文档另有说明，请优先在课程要求的 Linux 环境中完成实验。

---

# 2. 打开“文件夹”，而不是只打开一个文件

初学 VS Code 时，一个常见习惯是直接双击：

```text
main.c
```

然后开始写代码。

对于很小的文件这样做当然可以，但在正式实验中，更推荐：

```text
File → Open Folder
```

打开整个项目目录。

例如：

```text
hello/
├── main.c
├── input.txt
└── README.md
```

也可以在终端中进入项目目录后：

```bash
code .
```

这样做的好处包括：

- VS Code 知道当前项目的根目录；
- Git 可以正确识别整个仓库；
- `.vscode/` 中的项目配置可以生效；
- 多文件项目更容易管理；
- 后续使用 Makefile、调试配置等更加自然。

---

# 3. C 程序是怎样编译和运行的？

假设我们有：

```c
#include <stdio.h>

int main(void) {
    printf("Hello, ICS!\n");
    return 0;
}
```

在终端中执行：

```bash
gcc hello.c -o hello
```

其中：

```text
gcc
```

表示调用 GCC；

```text
hello.c
```

是要编译的源文件；

```text
-o hello
```

表示将输出的可执行文件命名为 `hello`。

编译完成后：

```bash
./hello
```

即可运行：

```text
Hello, ICS!
```

因此最简单的流程就是：

```text
源代码
  ↓
编译
  ↓
可执行文件
  ↓
运行
```

---

# 4. C++ 为什么通常使用 `g++`？

假设有：

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, ICS!" << std::endl;
    return 0;
}
```

推荐使用：

```bash
g++ hello.cpp -o hello
```

然后：

```bash
./hello
```

对于刚开始接触 C/C++ 的同学，可以先记住：

| 文件                          | 推荐编译器 |
| ----------------------------- | ---------- |
| `.c`                        | `gcc`    |
| `.cpp` / `.cc` / `.cxx` | `g++`    |

!!! info "`gcc` 和 `g++` 到底是什么关系？"

    `gcc` 和 `g++` 都属于 GCC 工具链。

    `gcc` 并不是“完全不能处理 C++”。

    但是在链接 C++ 程序时，`g++` 会自动按照 C++ 程序的需要链接 C++ 标准库，因此对初学者来说：

    > **C 用 `gcc`，C++ 用 `g++`。**

    是最简单、也最不容易出错的习惯。

??? question "为什么用 `gcc` 编译 C++ 时会看到 `undefined reference to std::cout`？"

    例如：

    ``bash     gcc hello.cpp -o hello     ``

    可能出现：

    ``text     undefined reference to `std::cout'     ``

    此时源代码本身未必有问题。

    问题通常出现在**链接阶段**：C++ 标准库没有按照正常 C++ 编译流程被链接进来。

    对 `.cpp` 文件直接使用：

    ``bash     g++ hello.cpp -o hello     ``

    通常即可解决。

---

# 5. 编译并不只是“一步”

为了便于入门，我们常常说：

```text
源代码 → 编译 → 可执行文件
```

但实际上，一个典型的 C/C++ 程序还会经历几个阶段：

```text
源代码
  ↓
预处理 Preprocessing
  ↓
编译 Compilation
  ↓
汇编 Assembling
  ↓
目标文件 .o
  ↓
链接 Linking
  ↓
可执行文件
```

例如：

```c
#include <stdio.h>
```

首先需要经过预处理。

然后源代码会被编译、汇编，生成目标文件。

最后由链接器将目标文件和需要的库组合成最终的可执行程序。

!!! info "为什么要知道这些？"

    因为以后你会遇到不同类型的错误：

    ``text     syntax error     ``

    和：

    ``text     undefined reference     ``

    并不是同一类问题。

    能判断错误发生在哪一个阶段，会显著提高排错效率。

---

# 6. 编译错误、链接错误和运行错误

建议从现在开始逐渐建立下面的分类意识。

## 6.1 编译错误

例如：

```text
error: expected ';' before '}'
```

一般表示源代码本身无法被正确翻译。

常见原因：

- 少了分号；
- 括号不匹配；
- 类型使用错误；
- 使用了不存在的变量。

---

## 6.2 链接错误

例如：

```text
undefined reference to `foo'
```

这通常表示：

> 编译器知道 `foo` 是什么，但最终找不到它的实现。

可能原因包括：

- 忘记编译某个 `.c` 文件；
- 忘记链接某个库；
- 函数只有声明，没有定义；
- C++ 错误地使用了不合适的链接方式。

---

## 6.3 运行错误

例如：

```text
Segmentation fault
```

说明：

> 程序已经成功生成并开始执行，但运行过程中发生了错误。

这时问题已经不是“能不能编译”，而是：

> 程序运行到了哪里？当时变量是什么？内存发生了什么？

这也是调试器发挥作用的地方。

---

# 7. 不要忽略 Warning

除了 Error，编译器还可能给出 Warning。

例如：

```c
#include <stdio.h>

int main(void) {
    int x;
    printf("%d\n", x);
    return 0;
}
```

建议在编译时开启更多警告：

```bash
gcc -Wall -Wextra main.c -o main
```

其中：

```text
-Wall
```

开启一组常用警告；

```text
-Wextra
```

开启额外警告。

对于实验代码，推荐逐渐养成这样的习惯：

```bash
gcc -Wall -Wextra -g main.c -o main
```

或者：

```bash
g++ -Wall -Wextra -g main.cpp -o main
```

!!! tip

    **能编译通过，不代表代码没有问题。**

    很多潜在 bug 都会先以 Warning 的形式出现。

    不要看到 Warning 就习惯性忽略。

---

# 8. VS Code 中为什么有这么多“运行”按钮？

这是初学者非常容易混淆的地方。

你可能会看到：

```text
Run Code
Run C/C++ File
Debug C/C++ File
Run Without Debugging
Start Debugging
```

它们并不是完全相同的功能。

---

## 8.1 最基础的方法：Terminal 手动运行

对于 C：

```bash
gcc main.c -o main
./main
```

对于 C++：

```bash
g++ main.cpp -o main
./main
```

这是最推荐大家首先掌握的方法。

原因很简单：

> **所有过程都看得见。**

如果程序不能运行，你可以明确知道：

- 编译器是谁；
- 输入文件是什么；
- 输出文件在哪里；
- 报错发生在哪一步。

---

## 8.2 `Run C/C++ File`

安装 Microsoft 的 **C/C++** 扩展后，可以直接运行当前 C/C++ 文件。

第一次运行时，VS Code 可能会询问使用哪个编译器，例如：

```text
C/C++: gcc build and debug active file
```

或者：

```text
C/C++: g++ build and debug active file
```

如果当前是 C++ 文件，通常应该选择 `g++`。

---

## 8.3 `Run Code`

`Run Code` 一般来自 **Code Runner** 扩展。

Code Runner 的本质是：

> 根据当前文件语言，自动执行一条事先配置好的命令。

例如 C++ 可能执行：

```bash
g++ main.cpp -o main && ./main
```

它很方便，但是也容易让初学者忽略背后的命令。

因此，如果 `Run Code` 出现奇怪问题，请先查看它究竟执行了什么。

??? question "为什么我的 `.cpp` 文件被 `gcc` 编译了？"

    正常情况下，应优先确认：

    1. 文件扩展名是否真的是 `.cpp`；
    2. VS Code 右下角是否将当前语言识别为 `C++`；
    3. Code Runner 的配置是否被修改；
    4. Workspace Settings 中是否存在项目级配置。

    最重要的是观察实际执行的命令。

    如果你看到：

    ``bash     gcc main.cpp ...     ``

    就应该继续追查：

    > 为什么当前运行配置选择了 `gcc`？

    而不是立刻修改源代码。

---

# 9. 为什么 `scanf` / `cin` 不能输入？

假设程序中有：

```c
int x;
scanf("%d", &x);
```

或者：

```cpp
int x;
std::cin >> x;
```

有时候点击 Code Runner 的 Run Code 后，会发现程序无法正常输入。

一个常见原因是：

> 程序运行在 Output 面板，而不是正常的交互式终端中。

如果你使用 Code Runner，可以在 Settings 中搜索：

```text
Code Runner: Run In Terminal
```

并启用它。

对应设置大致为：

```json
"code-runner.runInTerminal": true
```

这样程序会在 Terminal 中运行。

---

# 10. Run、Build、Debug 有什么区别？

可以先简单理解为：

| 操作  | 含义                   |
| ----- | ---------------------- |
| Build | 将源代码构建成程序     |
| Run   | 执行程序               |
| Debug | 在调试器控制下执行程序 |

例如：

```text
Build
  ↓
main.c
  ↓ gcc
main
```

然后：

```text
Run
  ↓
./main
```

而 Debug 则更像：

```text
VS Code
  ↓
GDB
  ↓
main
```

程序仍然在运行，但现在调试器可以暂停程序、检查变量和控制执行流程。

---

# 11. 什么是调试？

假设你写了：

```c
#include <stdio.h>

int main(void) {
    int a = 10;
    int b = 0;

    int c = a / b;

    printf("%d\n", c);
    return 0;
}
```

直接 Run 时，你只能知道：

> 程序运行失败了。

但是使用调试器时，可以：

- 在某一行暂停程序；
- 一行一行执行；
- 查看变量的当前值；
- 进入函数内部；
- 查看函数调用关系；
- 查看程序究竟在哪一步出错。

在 GCC 工具链中，常用调试器是：

```text
GDB
```

---

# 12. 为什么调试时常常需要 `-g`？

例如：

```bash
gcc -g main.c -o main
```

或者：

```bash
g++ -g main.cpp -o main
```

其中：

```text
-g
```

表示在生成的程序中加入调试信息。

这些信息帮助 GDB 将底层机器指令重新对应到：

```text
main.c 第 10 行
变量 x
函数 foo
```

如果没有调试信息，你可能会发现：

- 断点无法正确对应；
- 看不到局部变量；
- 调用栈信息不完整；
- GDB 只能显示地址或汇编信息。

因此，调试程序时通常建议：

```bash
gcc -Wall -Wextra -g main.c -o main
```

---

# 13. VS Code 中最常用的调试操作

在代码左侧行号附近单击，可以添加一个红点：

```text
●
```

这叫做 **Breakpoint（断点）**。

当程序运行到这一行附近时，调试器会暂停。

常见调试操作包括：

| 操作      | 含义                               |
| --------- | ---------------------------------- |
| Continue  | 继续运行，直到下一个断点           |
| Step Over | 执行当前行，但不进入函数内部       |
| Step Into | 如果当前行调用函数，则进入函数内部 |
| Step Out  | 从当前函数返回到调用者             |
| Restart   | 重新开始调试                       |
| Stop      | 停止调试                           |

---

# 14. 调试时应该看哪些面板？

VS Code 左侧 Run and Debug 页面中，常见几个区域：

## Variables

查看当前作用域内变量。

例如：

```text
a = 10
b = 0
c = ...
```

---

## Watch

可以手动添加你关心的表达式。

例如：

```text
i
array[i]
ptr
*ptr
```

---

## Call Stack

显示当前函数是怎么被调用到的。

例如：

```text
main
└── foo
    └── bar
```

如果程序在 `bar()` 中崩溃，Call Stack 可以告诉你：

> `bar()` 是谁调用的？它又是谁调用的？

这对定位复杂程序中的错误非常重要。

---

## Breakpoints

集中管理当前设置的断点。

---

# 15. 一个推荐的调试思路

不要把调试理解为：

> “程序坏了 → 随便打几个断点看看。”

更推荐按照下面的流程：

```text
能够稳定复现错误
        ↓
缩小可能出错的代码范围
        ↓
在关键位置设置断点
        ↓
观察变量
        ↓
Step Over / Step Into
        ↓
找到“实际行为”和“预期行为”第一次不一致的位置
```

!!! tip "一个很有用的问题"

    调试时，不要只问：

    > “哪一行崩了？”

    还要问：

    > **“程序第一次开始偏离我的预期，是在哪一步？”**

    真正的 bug 往往发生在程序最终崩溃之前。

---

# 16. `.vscode` 文件夹是什么？

使用 VS Code 一段时间后，你可能会看到：

```text
.vscode/
├── tasks.json
├── launch.json
└── settings.json
```

这些文件记录的是**当前项目的 VS Code 配置**。

---

## 16.1 `tasks.json`

可以粗略理解为：

> **这个项目怎样 Build？**

例如：

```json
{
    "command": "gcc",
    "args": [
        "-g",
        "main.c",
        "-o",
        "main"
    ]
}
```

表示 VS Code 构建时会调用：

```bash
gcc -g main.c -o main
```

---

## 16.2 `launch.json`

可以粗略理解为：

> **这个项目怎样 Debug？**

例如它会指定：

- 要调试哪个程序；
- 使用什么调试器；
- 工作目录是什么；
- 程序参数是什么。

---

## 16.3 `settings.json`

记录当前 Workspace 的一些设置。

例如：

```json
{
    "editor.formatOnSave": true
}
```

!!! warning

    `tasks.json`、`launch.json`、`settings.json` 负责的事情不同。

    不要看到“编译器配置”几个字，就认为所有配置文件都会决定真正执行哪个编译命令。

    遇到问题时，最可靠的方法仍然是：

    > **查看实际执行的命令。**

---

# 17. 多文件程序

程序稍微复杂以后，通常不会只有一个 `.c` 文件。

例如：

```text
project/
├── main.c
├── add.c
└── add.h
```

其中：

```c
int add(int a, int b);
```

```c
#include "add.h"

int add(int a, int b) {
    return a + b;
}
```

```c
#include <stdio.h>
#include "add.h"

int main(void) {
    printf("%d\n", add(1, 2));
    return 0;
}
```

如果只执行：

```bash
gcc main.c -o main
```

可能出现：

```text
undefined reference to `add'
```

因为 `add()` 的实现在 `add.c` 中。

应该：

```bash
gcc main.c add.c -o main
```

或者先分别编译：

```bash
gcc -c main.c -o main.o
gcc -c add.c -o add.o
```

再链接：

```bash
gcc main.o add.o -o main
```

这也是理解链接过程的重要例子。

---

# 18. 为什么需要 Makefile？

当项目只有一个文件时：

```bash
gcc main.c -o main
```

很简单。

但是当项目变成：

```text
main.c
parser.c
lexer.c
utils.c
...
```

每次手动输入所有文件就会越来越麻烦。

这时通常会使用构建工具，例如：

```text
make
```

一个非常简单的 Makefile：

```makefile
main: main.c add.c
	gcc -Wall -Wextra -g main.c add.c -o main
```

然后只需要：

```bash
make
```

!!! info

    Makefile 并不是 VS Code 专属的东西。

    它描述的是：

    > **这个项目应该怎样被构建。**

    VS Code 只是可以进一步帮你调用 `make`。

后续实验如果提供了 Makefile，建议优先按照实验文档提供的构建方式进行，不要随意绕过它直接点击 Run Code。

---

# 19. AddressSanitizer：很好用的内存错误检查工具

C/C++ 中很多 bug 与内存有关，例如：

- 数组越界；
- use-after-free；
- heap buffer overflow；
- stack buffer overflow。

除了 GDB，还可以使用 AddressSanitizer。

例如：

```bash
gcc -g -fsanitize=address main.c -o main
```

然后正常运行：

```bash
./main
```

如果发生典型内存错误，它通常能够给出比普通：

```text
Segmentation fault
```

更具体的信息。

!!! tip

    可以粗略理解为：

    - **GDB**：程序运行到这里时，发生了什么？
    - **AddressSanitizer**：有没有典型的非法内存访问？

    两者并不是互相替代的关系。

---

# 20. 输入重定向：不要每次都手输测试数据

假设程序每次需要输入：

```text
1 2 3 4 5
```

可以把它保存到：

```text
input.txt
```

然后：

```bash
./main < input.txt
```

程序会把 `input.txt` 当作标准输入。

类似地：

```bash
./main > output.txt
```

可以把标准输出保存到 `output.txt`。

这对重复测试程序非常方便。

---

# 21. VS Code 中使用 Git

VS Code 自带 Git 集成。

如果当前文件夹本身是一个 Git 仓库，可以打开左侧：

```text
Source Control
```

看到文件变化。

!!! info "Git 和 GitHub 不是一回事"

    **Git** 是版本控制工具。

    **GitHub** 是代码托管平台。

    即使完全不使用 GitHub，也可以在本地使用 Git。

---

# 22. 一个最基本的 Git 工作流

假设你修改了：

```text
main.c
```

Source Control 中可能出现：

```text
Changes
  M main.c
```

其中 `M` 表示 Modified。

一个常见工作流是：

```text
修改代码
   ↓
查看 Diff
   ↓
Stage Changes
   ↓
填写 Commit Message
   ↓
Commit
   ↓
Push / Sync
```

---

# 23. 查看 Diff

点击 Source Control 中修改过的文件，可以看到修改前后的差异。

例如：

```diff
- printf("hello\n");
+ printf("Hello, ICS!\n");
```

这是 Git 最重要的用途之一：

> **你可以清楚知道自己到底改了什么。**

建议在 Commit 之前养成查看 Diff 的习惯。

---

# 24. Stage 是什么？

Git 通常不是：

```text
修改 → Commit
```

而是：

```text
修改
 ↓
Stage
 ↓
Commit
```

Stage 可以理解为：

> **选择哪些修改进入下一次 Commit。**

在 VS Code 中，可以点击文件旁边的 `+`：

```text
Changes
  main.c   +
```

将修改加入 Staged Changes。

---

# 25. Commit Message 怎么写？

Commit Message 应该描述：

> 这一次提交做了什么。

例如：

```text
fix buffer boundary check
```

或者：

```text
add command parser
```

不推荐：

```text
update
```

```text
123
```

```text
final final really final
```

好的 Commit Message 可以帮助未来的你快速理解代码历史。

---

# 26. Pull、Push 和 Sync

可以先简单理解为：

```text
Push
本地 → 远端
```

```text
Pull
远端 → 本地
```

VS Code 还可能显示：

```text
Sync Changes
```

它通常会帮助你同步本地与远端状态。

!!! warning

    在不理解当前仓库状态时，不建议看到任何按钮都直接点。

    特别是：

    - Pull 前先确认自己有没有未提交修改；
    - Push 前先确认自己提交了什么；
    - 遇到冲突时不要随意选择 Accept Both。

---

# 27. Branch 是什么？

Branch（分支）可以理解为：

> 从某个代码状态出发，创建一条独立的开发路线。

例如：

```text
main
  |
  |------ feature-a
  |
  |------ debug-test
```

在 VS Code 左下角通常可以看到当前分支名称。

点击后可以：

- 切换分支；
- 创建新分支；
- 查看已有分支。

对于课程实验，如果实验文档对分支有要求，应优先按照实验文档执行。

---

# 28. Merge Conflict 是什么？

假设同一行代码：

本地修改为：

```c
printf("local\n");
```

远端修改为：

```c
printf("remote\n");
```

Git 无法自动判断应该保留哪个，于是产生冲突。

VS Code 可能提供：

```text
Accept Current
Accept Incoming
Accept Both
Compare Changes
```

!!! danger "不要把 Accept Both 当成万能按钮"

    冲突的本质不是：

    > “Git 坏了。”

    而是：

    > **Git 不知道哪一种代码才符合你的真实意图。**

    所以你需要先理解两边分别修改了什么，再决定最终代码。

---

# 29. 如何撤销一次错误修改？

VS Code Source Control 中，可以对尚未提交的修改使用：

```text
Discard Changes
```

但请注意：

!!! danger

    `Discard Changes` 会丢弃当前未提交修改。

    在点击之前，务必确认这些内容确实不需要。

如果修改比较重要但暂时不想提交，可以进一步了解：

```text
git stash
```

---

# 30. 推荐安装哪些 VS Code 扩展？

对于本课程，建议保持扩展尽量精简。

可以考虑：

### C/C++

Microsoft 官方 C/C++ 扩展。

主要提供：

- IntelliSense；
- 代码补全；
- 调试支持；
- C/C++ 语言相关功能。

---

### WSL

Windows + WSL 用户推荐。

可以直接使用 Windows 上的 VS Code 操作 WSL 中的项目。

---

### Remote - SSH

如果你需要连接远程 Linux 服务器，可以使用。

---

### Code Runner

可选。

它可以很方便地快速运行当前文件，但不建议把它当作唯一的 C/C++ 编译方式。

尤其当实验使用：

```text
Makefile
多个源文件
特殊编译参数
```

时，请优先按照实验要求构建程序。

---

# 31. VS Code 中几个容易混淆的概念

| 名称        | 它是什么                             |
| ----------- | ------------------------------------ |
| VS Code     | 编辑器 / 开发环境前端                |
| Terminal    | VS Code 中嵌入的终端                 |
| Shell       | Bash、Zsh、PowerShell 等命令解释环境 |
| gcc         | 常用 C 编译驱动程序                  |
| g++         | 常用 C++ 编译驱动程序                |
| gdb         | 调试器                               |
| Make        | 构建工具                             |
| Git         | 版本控制工具                         |
| GitHub      | Git 仓库托管平台                     |
| Code Runner | VS Code 扩展，用于快捷运行代码       |

---

# 32. 常见问题速查

??? question "`gcc: command not found`"

    首先执行：

    ``bash     which gcc     ``

    或：

    ``bash     gcc --version     ``

    如果确实没有安装，可以在 Ubuntu / WSL 中安装：

    ``bash     sudo apt update     sudo apt install build-essential     ``

---

??? question "`gdb: command not found`"

    Ubuntu / WSL 中可以：

    ``bash     sudo apt install gdb     ``

---

??? question "为什么程序能编译，但运行时 Segmentation fault？"

    这已经属于运行阶段的问题。

    常见原因：

    - 数组越界；
    - 空指针；
    - 野指针；
    - use-after-free；
    - 非法内存访问。

    建议：

    ``bash     gcc -g main.c -o main     gdb ./main     ``

    或使用：

    ``bash     gcc -g -fsanitize=address main.c -o main     ``

---

??? question "为什么头文件下面有红色波浪线，但 `gcc` 又能正常编译？"

    红色波浪线通常来自 VS Code 的 IntelliSense。

    编译则由真正的编译器完成。

    两者使用的配置不一定完全相同。

    因此：

    > **编辑器报红 ≠ 编译器一定报错。**

    遇到这种问题时，以实际编译命令的结果作为重要判断依据，再检查 IntelliSense 的配置。

---

??? question "为什么我改完代码，运行结果还是旧的？"

    可以检查：

    1. 文件是否已经保存；
    2. 是否重新编译；
    3. 当前运行的是不是另一个目录下的同名程序；
    4. VS Code 当前工作目录是否与你想的一致。

    可以使用：

    ``bash     pwd     ls     ``

    查看当前目录和文件。

---

??? question "为什么 Breakpoint 是灰色的？"

    可能原因：

    - 程序还没有以 Debug 模式运行；
    - 编译时没有加入 `-g`；
    - 当前源文件与正在调试的可执行程序不是同一份；
    - 调试配置错误。

---

??? question "为什么实验给了 Makefile，我点击 Run Code 却出问题？"

    因为 Run Code 通常只根据当前文件执行一个简单命令。

    但实验项目可能需要：

    - 多文件编译；
    - 特殊宏；
    - 特殊库；
    - 特定编译选项。

    如果实验提供：

    ``text     Makefile     ``

    请优先阅读实验文档并使用：

    ``bash     make     ``

    等课程规定的构建方式。

---

??? question "为什么 Git 页面突然出现很多不认识的文件？"

    可能是：

    - 编译生成的可执行文件；
    - `.o` 目标文件；
    - VS Code 配置文件；
    - 临时文件。

    不需要加入版本控制的生成文件，通常应该写入：

    ``text     .gitignore     ``

    例如：

    ``gitignore     *.o     main     ``

    但不要看到陌生文件就直接忽略，先确认它是什么。

---

# 33. 推荐的排错顺序

如果一个 C/C++ 程序在 VS Code 中无法运行，可以按照下面的顺序排查：

```text
① 当前文件和目录对吗？
        ↓
② gcc / g++ 是否可用？
        ↓
③ 手动命令行能否编译？
        ↓
④ 编译错误、链接错误还是运行错误？
        ↓
⑤ VS Code 实际执行了什么命令？
        ↓
⑥ tasks.json / Code Runner 配置是否正确？
        ↓
⑦ 如果是运行 bug，使用 GDB / Sanitizer
```

其中最重要的一步是：

> **先确认命令行能不能工作，再判断是不是 VS Code 配置的问题。**

---

# 34. 一个推荐的学习路线

如果你刚开始使用 VS Code，可以按下面的顺序学习。

## 第一阶段：先把程序跑起来

掌握：

```bash
gcc main.c -o main
./main
```

以及：

```bash
g++ main.cpp -o main
./main
```

---

## 第二阶段：学会看错误

能够区分：

```text
编译错误
链接错误
运行错误
```

并开始使用：

```text
-Wall
-Wextra
```

---

## 第三阶段：学会调试

掌握：

```text
Breakpoint
Continue
Step Over
Step Into
Variables
Watch
Call Stack
```

以及：

```bash
-g
gdb
```

---

## 第四阶段：学会管理项目

了解：

```text
多文件编译
Makefile
.vscode/
Git
Branch
Diff
```

---

# 35. 最后：不要把 IDE 当作魔法

如果只记住本文的一件事，希望是：

> **不要只学习“怎么按 VS Code 的按钮”，而要理解按钮背后调用了什么工具。**

当你点击 Run 时，背后可能是：

```bash
gcc ...
```

或者：

```bash
g++ ...
```

当你点击 Debug 时，背后可能是：

```text
GDB
```

当你点击 Source Control 时，背后是：

```text
Git
```

VS Code 把这些工具集中到了一个图形界面中，使它们更方便使用。

但是理解这些工具本身，才是真正具有迁移性的能力。

以后即使你换成：

```text
Vim
Neovim
CLion
Visual Studio
Cursor
```

C/C++ 的编译、链接、调试和版本控制原理仍然是一样的。

---

## 附：工具速查表

| 我想做什么              | 常用工具                   |
| ----------------------- | -------------------------- |
| 编辑代码                | VS Code                    |
| 编译 C                  | `gcc`                    |
| 编译 C++                | `g++`                    |
| 调试                    | `gdb` / VS Code Debugger |
| 查内存错误              | AddressSanitizer           |
| 管理多文件构建          | `make`                   |
| 管理代码版本            | Git                        |
| 托管 Git 仓库           | GitHub                     |
| Windows 使用 Linux 环境 | WSL                        |
| 连接远程服务器          | SSH / Remote - SSH         |

!!! tip "最后一个建议"

    遇到问题向助教或 AI 求助时，最好同时提供：

    1. 你想完成什么；
    2. 当前操作系统 / WSL / 虚拟机环境；
    3. 完整的编译命令；
    4. 完整报错信息；
    5. 能够复现问题的代码；
    6. 你已经尝试过什么。

    相比一句：

    > “为什么我的 VS Code 跑不了？”

    这些信息会让问题容易定位得多。
