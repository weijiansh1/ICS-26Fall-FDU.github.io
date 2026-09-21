---
title: VS Code 中的编译、运行与调试
---

# VS Code 中的编译、运行与调试

写好代码后，很多同学会先点击编辑器右上角的运行三角。它只是根据已安装的扩展和当前配置，替你执行一条或多条命令，并不会让代码“自动运行”。

对 C/C++ 而言，这通常意味着：先由编译器生成可执行文件，再运行该文件。

```text
main.c / main.cpp
        │
        │  gcc 或 g++
        ▼
   可执行文件 main
        │
        │  ./main
        ▼
     程序输出
```

VS Code 可以编辑代码、打开终端和启动调试器，但它不是 C/C++ 编译器；真正完成编译的是 `gcc` 或 `g++`。理解这条链路后，便能区分“运行三角没有反应”“程序未能编译”与“程序运行后出错”等不同问题。

!!! example "配图 1：右上角的运行三角"

    此处放一张 VS Code 编辑器截图，标出右上角的运行三角及其下拉箭头；鼠标悬停时最好能显示该按钮的提示文字。

    建议文件名：`assets/misc/vscode/run-button.png`

!!! note "先看它实际执行了什么"

    本文以 Microsoft 发布的 **C/C++** 扩展为准，使用其 `Run C/C++ File` 和 `Debug C/C++ File` 命令。若鼠标提示为 `Run Code`，那通常来自 Code Runner 扩展，可能在 Output 面板而非 Terminal 中运行；它的配置不在本文范围内。

    无论使用哪种按钮，都应观察实际执行的完整命令；它比按钮图标更能说明问题所在。

## 点击运行三角后，程序是怎样跑起来的

假设当前目录里有 `hello.c`：

```c
#include <stdio.h>

int main(void) {
    printf("Hello, ICS!\n");
    return 0;
}
```

如果不使用运行三角，在 VS Code 中打开 **Terminal → New Terminal**，可以手动执行同样的流程：

```bash
gcc hello.c -o hello
./hello
```

第一条命令没有输出，通常表示编译成功，并已生成 `hello`；第二条命令才会执行程序，输出应为：

```text
Hello, ICS!
```

!!! example "配图 2：终端中的编译与运行"

    此处放一张同时显示 `hello.c` 与底部 Terminal 的截图。终端应完整展示 `gcc hello.c -o hello`、`./hello` 和 `Hello, ICS!` 输出，使读者能把两条命令和结果对应起来。

    建议文件名：`assets/misc/vscode/hello-terminal.png`

### `./hello` 是什么意思？

`./hello` 不是 C 语言语法，而是 Linux Shell 中“运行当前目录下的 `hello` 文件”的写法：

```text
.       当前目录
/       路径分隔符
hello   刚刚生成的可执行文件
```

因此 `gcc hello.c -o hello` 生成 `hello` 后，`./hello` 就是在执行它。通常只输入 `hello` 并不能运行：Shell 会只在预先配置的 `PATH` 目录中寻找命令，而当前目录通常不在其中。写出 `./` 可以明确告诉 Shell：要运行的就是当前目录里的这个文件。

对于 C++ 文件，只需把 `gcc` 换为 `g++`：

```bash
g++ hello.cpp -o hello
./hello
```

## 让 VS Code 使用 `gcc` 或 `g++`

运行三角能否正确工作，前提是 VS Code 所在的 Linux/WSL 环境已经安装编译器。先在 VS Code 的 Terminal 中检查：

```bash
gcc --version
g++ --version
gdb --version
```

`gcc` 和 `g++` 能显示版本信息，说明当前环境可以编译 C/C++；`gdb` 则用于之后的调试。若出现 `command not found`，说明相应工具尚未安装或不在当前环境的 `PATH` 中。

接着在扩展市场安装 Microsoft 发布的 **C/C++** 扩展。打开一个 `.c` 或 `.cpp` 文件，点击运行三角旁边的下拉菜单，选择 **Run C/C++ File** 或 **Debug C/C++ File**。第一次运行时，VS Code 会列出检测到的编译器：

- 对 `.c` 文件，选择类似 `C/C++: gcc build and debug active file` 的选项；
- 对 `.cpp` 文件，选择类似 `C/C++: g++ build and debug active file` 的选项。

!!! example "配图 3：选择编译器"

    此处放运行三角下拉菜单和编译器选择列表的截图，突出 `gcc build and debug active file` 与 `g++ build and debug active file` 两个选项。

    建议文件名：`assets/misc/vscode/select-compiler.png`

选择后，C/C++ 扩展会在当前项目的 `.vscode/tasks.json` 中记录构建任务。以后再次点击运行三角时，它会按这个任务调用编译器；如果选错了，按 `Ctrl+Shift+P` 打开命令面板，执行 **Tasks: Configure Default Build Task**，重新选择即可。

!!! note "配置文件各自负责什么"

    `tasks.json` 决定构建时实际执行的命令；`launch.json` 决定调试时运行哪个程序、使用哪个调试器；`c_cpp_properties.json` 主要服务于代码补全和头文件红线。后者即使配置不正确，也不等同于 `gcc` 一定无法编译。

!!! tip "推荐的编译命令"

    练习时建议使用：

    ```bash
    gcc -Wall -Wextra -g hello.c -o hello
    ```

    `-Wall -Wextra` 让编译器提醒常见问题；`-g` 则为调试器保留源代码和变量信息。它们不会改变程序原本要做的事情。

这里先介绍终端命令，而不是直接使用 Run Code：终端会显示实际执行的命令和完整报错。先确认命令行流程无误，再使用图形界面的按钮，通常更容易定位问题。

## 编译、链接和运行不是一回事

一条看似简单的 `gcc hello.c -o hello`，实际包含编译和链接两个阶段；随后 `./hello` 才是运行阶段。不同阶段的报错含义也不同。

| 看到的现象 | 它发生在 | 可以先检查 |
| --- | --- | --- |
| `error: expected ...` | 编译 | 分号、括号、拼写、类型 |
| `undefined reference to ...` | 链接 | 是否遗漏源文件或库 |
| `Segmentation fault` | 运行 | 数组下标、指针、内存访问 |

例如项目有 `main.c` 和 `add.c`，而 `main.c` 调用了在 `add.c` 中实现的函数时，应当写：

```bash
gcc main.c add.c -o main
```

如果只编译 `main.c`，编译器能看懂函数声明，但链接器找不到函数实现，便会报告 `undefined reference`。

实验如果提供了 Makefile，应优先使用实验文档指定的命令：

```bash
make
./程序名
```

Makefile 已经记录了所有源文件和编译选项。此时不建议用 Code Runner 只运行当前文件。

## 当程序能运行，但答案不对

这才是调试器最有用的场景。下面的程序本意是计算 `1 + 2 + ... + n`：

```c linenums="1"
#include <stdio.h>

int sum_to(int n) {
    int sum = 0;
    for (int i = 1; i < n; ++i) {
        sum += i;
    }
    return sum;
}

int main(void) {
    printf("%d\n", sum_to(5));
    return 0;
}
```

它能正常编译并运行，却输出 `10`，而非预期的 `15`。这说明问题不在 VS Code 或编译命令，而在程序的运行过程。

不必急着修改代码。先在第 6 行 `sum += i;` 左侧设置断点，再打开左侧 **Run and Debug**，选择 C/C++ 调试配置并启动。

!!! example "配图 4：设置断点并启动调试"

    此处放上述程序第 6 行设置断点、左侧打开 Run and Debug 的截图。应能看出断点位置和启动调试的位置。

    建议文件名：`assets/misc/vscode/set-breakpoint.png`

程序每次停在断点时，观察 **Variables**：

| 执行次数 | `i` | 执行后 `sum` |
| --- | --- | --- |
| 第一次 | 1 | 1 |
| 第二次 | 2 | 3 |
| 第三次 | 3 | 6 |
| 第四次 | 4 | 10 |

继续执行后，循环直接结束了：当 `i` 变成 5 时，条件 `i < n` 不再成立，因此 5 没有被加进去。把第 5 行改为：

```c
for (int i = 1; i <= n; ++i) {
```

重新编译、运行后，结果应为 `15`。

调试器不会替你“自动修复 bug”，而是让变量变化和控制流程变得可见，帮助你确认程序从哪一步开始偏离预期。

!!! example "配图 5：观察变量并单步执行"

    此处放程序暂停在断点时的截图，展示编辑器中的断点、Variables 内的 `i` 和 `sum`、Call Stack，以及顶部的 Step Over 按钮。

    建议文件名：`assets/misc/vscode/inspect-variables.png`

## VS Code 中最常用的调试操作

- **Continue**：继续运行到下一个断点。
- **Step Over**：执行当前行，但不进入当前行调用的函数。
- **Step Into**：若当前行调用函数，则进入该函数内部。
- **Step Out**：执行完当前函数，回到调用它的位置。
- **Stop**：结束本次调试。

初学时最常用的是断点、Variables 和 Step Over。先检查变量是否符合预期，再决定是否需要进入某个函数；不必一开始就使用所有面板和按钮。

## 一个实用的排查顺序

程序“跑不了”或“结果不对”时，可以按这个顺序处理：

```text
确认正在编辑和运行的是同一份文件
        ↓
在终端中执行完整的编译命令
        ↓
判断是编译错误、链接错误，还是运行错误
        ↓
若能运行但结果错误，在关键位置设置断点
        ↓
观察变量和执行流程，找到第一次不符合预期的位置
```

向他人求助时，请提供完整的编译或运行命令、完整报错、运行环境，以及已经尝试过的操作。这些信息比一句“VS Code 跑不了”更有助于定位问题。
