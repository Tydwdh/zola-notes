+++
title = "Makefile 基础：规则、变量与隐式规则"
date = 2026-03-25
description = "从目标、依赖、命令到变量和隐式规则，梳理 Makefile 的基本工作方式。"

[taxonomies]
tags = ["makefile", "build-system"]
+++
# 1.Makefile的语法
Makefile是由一系列规则组成。一个规则通常是这样的:
```Makefile
targets: prerequisites 
	command 
	command 
	command
```
- **targets**:这是文件的名字,用空格分隔。通常一个规则只有一个文件名。
- **command**：这是一系列用于构建项目的步骤。在命令前面需要一个Tab 键，而不是空格
- **prerequisites**：这也是文件的名字，用空格分隔。在这个命令运行之前这些文件就要存在。这个也被称为*依赖*
  
# 2.Make的精髓
让我们先引用一个例子：
```Makefile
hello: 
	echo "Hello, World" 
	echo "This line will print if the file hello does not exist."
```
让我们分析一下这个规则：
- 我们有一个目标叫做hello
- 这个目标有两个命令
- 这个目标没有依赖
当我们运行`make hello`时就会成功运行。

如果当前文件夹下已经有一个`hello`文件，make命令不会执行。

需要重点注意的是,hello既是一个目标也是一个文件。通常来说一个规则运行时同时会创建一个和目标一样名字的文件。但是这个例子中，`hello`并不会创建一个文件。

让我们创建一个更加典型的makefile——编译一个单独的C语言文件。在那之前，我们先创建一个简单C语言文件。
```c
int main()
{
    return 0;
}
```
接下来创建一个 `Makefile`文件
```makefile
main.exe: main.c
    gcc main.c -o main
```
当我们再次运行make时，将执行以下步骤:
- 选择第一个目标，因为第一个目标是默认目标。
- 它有一个依赖main.c
- Make决定它是否应该运行main.exe目标。它只会在main.exe不存在，或者blah.c比blah.exe更新的情况下运行

最后一步很关键，也是make的精髓所在。它要做的是判断自从上次编译main.exe之后，main.exe的先决条件是否发生了变化。也就是说，如果main.c被修改，运行make应该重新编译该文件。相反，如果 main.c 没有更改，则不应该重新编译它。

## 一个简单的小例子
```makefile
main.exe: main.o
	gcc main.o -o main 

main.o: main.c
	gcc -c main.c -o main.o 

main.c:
	echo "int main() { return 0; }" > main.c 
```
这个Makefile最终会运行所有三个目标。当你在终端中运行make时，它将通过一系列步骤构建一个名为main的程序：
1. Make选择目标main，因为第一个目标是默认目标。
2. main需要main.o，所以make搜索main.o目标。
3. main.o需要main.c，所以make搜索main.c目标。
4. main.c没有依赖，所以运行echo命令。
5. 所有main.o的依赖都完成后，然后运行gcc -c命令。
6. 所有main的依赖都完成后，运行顶部的gcc命令。
7. 就这样：main是一个编译过的C程序。


# 3.Make clean
`clean`经常作为一个目标来移除其他的输出项目，但它并不是一个makefile的关键字。
```makefile
main.c:
	echo "int main() { return 0; }" > main.c 
clean: 
	rm -f main.c
```
注意，有两个要求：
- 这不是第一个（默认）的目标，也不是一个依赖。这意味着除非你明确地调用make clean，否则它永远不会运行。
- 它并不是作为一个文件名来使用的。如果你碰巧有一个名为clean的文件，这个目标就不会运行（后面会解决这个问题）。

# 4.变量
变量只能是字符串。
以下是一个例子
```makefile
files = file1 file2
some_file: $(files)
	echo "Look at this variable: " $(files)
	type > some_file

file1:
	type > file1
file2:
	type > file2

clean:
	rm -f file1 file2 some_file
```
引用变量用`${}` 或 `$()`

# 5.目标
## all目标
有多个目标，你想让他们全都运行?做一个all目标。把它作为第一个目标，它将在默认情况下运行。
```makefile
all: one two three

one:
	type > one
two:
	type > two
three:
	type > three

clean:
	rm -f one two three
```
## 多个目标
当一个规则有多个目标时，将为每个目标运行命令。$@是一个包含目标名称的自动变量。
```makefile
all: f1.o f2.o

f1.o f2.o:
	echo $@
# 等价于:
# f1.o:
#	 echo f1.o
# f2.o:
#	 echo f2.o
```

# 自动变量和通配符
## * 通配符
\*和%在Make中都称为通配符，但它们的含义完全不同。\*在文件系统中搜索匹配的文件名。我建议您总是将其包装在wildcard函数中，否则您可能会陷入下面描述的常见陷阱。
```makefile
# 错误的做法：尝试将所有的 .o 文件赋值给变量 thing_wrong
# '*' 通配符在这里并不会被扩展
thing_wrong := *.o # 不要这样做！'*' 不会被扩展

# 正确的做法：使用 wildcard 函数将所有的 .o 文件赋值给变量 thing_right
thing_right := $(wildcard *.o)

all: one two three four

# 这个目标会失败，因为 $(thing_wrong) 的值是字符串 "*.o"
one: $(thing_wrong)

# 如果当前目录下没有任何 .o 文件，*.o 将保持原样，而不会被扩展
two: *.o 

# 这个目标能正常工作，因为它正确地使用了 wildcard 函数来扩展 '*' 通配符
three: $(thing_right)

# 和规则 three 相同，也能正常工作
four: $(wildcard *.o)
```

## %通配符
`%`是一个通配符，它在Makefile中非常有用，但由于它可以用在各种情况中，所以有时会让人感到困惑。

- 当`%`用在"匹配"模式时，它可以匹配字符串中的一个或多个字符。这个匹配被称为"茎"。
- 当`%`用在"替换"模式时，它会取出匹配到的"茎"，并在字符串中进行替换。
- `%`最常用在规则定义和一些特定的函数中。

以下是一个例子来说明`%`的用法：

```makefile
# 规则定义
%.o: %.c
    echo $@

# 在这个例子中，% 用于匹配 .c 文件和 .o 文件的相同部分（即"茎"）。
# 当 make 命令需要构建一个 .o 文件时，它会查找与 .o 文件名相同（除了扩展名）的 .c 文件，
# 然后使用 gcc -c 命令将 .c 文件编译成 .o 文件。
```

## 自动变量
常用自动变量：
- **`$@`**：表示规则中的目标文件名。
- **`$<`**：表示规则中的第一个依赖文件名。
- **`$?`**：表示规则中所有比目标新的依赖文件列表。
- **`$^`**：表示规则中的所有依赖文件列表，这个列表中不包含重复的文件名。

# 有趣的一些规则
## 隐式规则 
以下是一些隐式规则的列表：

- 编译C程序：`n.o`会自动从`n.c`生成，使用的命令形式为 `\((CC) -c\)(CPPFLAGS) \((CFLAGS)\)^ -o $@`
- 编译C++程序：`n.o`会自动从`n.cc`或`n.cpp`生成，使用的命令形式为 `\((CXX) -c\)(CPPFLAGS) \((CXXFLAGS)\)^ -o $@`
- 链接单个对象文件：n会自动从n.o生成，通过运行命令 `\((CC)\)(LDFLAGS) \(^\)(LOADLIBES) \((LDLIBS) -o\)@`

隐式规则使用的重要变量包括：

- `CC`：编译C程序的程序；默认为cc
- `CXX`：编译C++程序的程序；默认为g++
- `CFLAGS`：传递给C编译器的额外标志
- `CXXFLAGS`：传递给C++编译器的额外标志
- `CPPFLAGS`：传递给C预处理器的额外标志
- `LDFLAGS`：当编译器应该调用链接器时，传递给编译器的额外标志

现在，让我们看看如何在不明确告诉Make如何进行编译的情况下，构建一个C程序：

```makefile
CC = gcc # 隐式规则的标志
CFLAGS = -g # 隐式规则的标志。打开调试信息

# 隐式规则#1：blah通过C链接器隐式规则构建
# 隐式规则#2：因为blah.c存在，所以blah.o通过C编译隐式规则构建
blah: blah.o

blah.c:
	echo "int main() { return 0; }" > blah.c

clean:
	rm -f blah*
```
