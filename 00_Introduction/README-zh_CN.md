# 第 0 部分：引言

[TOC]

我决定开启一段编写编译器的旅程。过去我曾写过一些 [汇编器](https://github.com/DoctorWkt/pdp7-unix/blob/master/tools/as7)，也写过一个针对无类型语言的 [简单编译器](https://github.com/DoctorWkt/h-compiler)。但我从未写过能够编译自身的编译器。所以，这就是我此行的目标。

在这个过程中，我会记录我的工作，以便其他人可以跟随学习。这也有助于我理清自己的思路和想法。希望你我都能从中受益！



## 旅程目标

以下是我为这次旅程设定的目标与非目标：

+ 编写一个能自举的编译器。我认为如果编译器能编译自身，它就有资格称自己为一个 *真正的* 编译器。
+ 至少以一个真实的硬件平台为目标。我见过一些为假想机器生成代码的编译器。我希望我的编译器能在真实的硬件上运行。此外，如果可能，我希望编写编译器时能够支持针对不同硬件平台的多个后端。
+ 实践优先于研究。编译器领域有大量的研究。我想在这段旅程中从零开始，因此我会倾向于采用实用的方法，而非理论性过强的方法。话虽如此，有时我仍然需要引入（并实现）一些基于理论的内容。
+ 遵循 KISS 原则：保持简单明了！我肯定会在这里运用肯·汤普逊的原则："当有疑问时，采用蛮力。"
+ 通过许多小步骤达成最终目标。我会将旅程分解为许多简单的步骤，而不是大跨步前进。这将使编译器的每个新增部分都易于理解和消化。



## 目标语言

目标语言的选择很困难。如果我选择像 Python、Go 这样的高级语言，那么我将不得不实现一大堆库和类，因为它们是这些语言内置的。

我可以为 Lisp 这样的语言编写编译器，但这些语言 [可以很容易实现](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-039.pdf)。

相反，我选择了一个可靠的老牌语言，我将为一个 C 语言的子集编写编译器，这个子集要足够大，使得编译器能够编译自身。

C 语言只是比汇编语言高一级（对于 C 的某个子集而言，不是 [C18](https://en.wikipedia.org/wiki/C18_(C_standard_revision))），这将有助于将 C 代码编译成汇编语言的任务变得更容易一些。哦，而且我也喜欢 C。



## 编译器的基本工作

编译器的工作是将一种语言（通常是高级语言）的输入翻译成另一种不同的输出语言（通常比输入语言级别更低）。主要步骤是：

<center>
    <img src="Figs/编译器解析步骤.png" alt="图片无法加载！">
    <figcaption> 图 1&emsp; 编译器的解析步骤 </figcaption>
</center>


+ 进行 [词法分析](https://en.wikipedia.org/wiki/Lexical_analysis) 以识别词法元素。在几种语言中，`=` 与 `==` 是不同的，因此你不能只读取单个 `=`。我们称这些词法元素为 *词法标记*。

+ [解析](https://en.wikipedia.org/wiki/Parsing) 输入，即识别输入的语法和结构元素，并确保它们符合语言的 *语法*。例如，你的语言可能有这样的决策结构：

  ```c
  if (x < 23)
  {
  	print("x is smaller than 23\n");
  }
  ```

  > 但在另一种语言中，你可能会这样写：

  ```python
  if (x < 23):
      print("x is smaller than 23\n")
  ```

  > 这里也是编译器可以检测语法错误的地方，例如第一个 *print* 语句末尾缺少分号。

+ 对输入进行 [语义分析](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers))，即理解输入的含义。这实际上与识别语法和结构不同。例如，在英语中，一个句子可能具有 `<主语> <动词> <形容词> <宾语>` 的形式。以下两个句子结构相同，但含义完全不同：

  ```text
  David ate lovely bananas.
  Jennifer hates green tomatoes.
  ```

+ 将输入的含义 [翻译](https://en.wikipedia.org/wiki/Code_generation_(compiler))成另一种语言。在这里，我们将输入的一部分一部分地转换为更低级的语言。



## 资源

互联网上有很多关于编译器的资源。以下是我将参考的一些。

### 学习资源

如果你想从一些关于编译器的书籍、论文和工具开始，我强烈推荐这个列表：

+ Ahmad Alhour 整理的 [关于编译器、解释器和运行时的精选资源列表](https://github.com/aalhour/awesome-compilers)

### 现有编译器

虽然我将构建自己的编译器，但我计划参考其他编译器的思路，并可能借用它们的一些代码。以下是我正在参考的编译器：

+ Nils M Holm 编写的 [SubC](http://www.t3x.org/subc/)
+ Robert Swierczek 编写的 [Swieros C 编译器](https://github.com/rswier/swieros/blob/master/root/bin/c.c)
+ Fabrice Bellard 编写的 [fbcc](https://github.com/DoctorWkt/fbcc)
+ 同样由 Fabrice Bellard 等人编写的 [tcc](https://bellard.org/tcc/)
+ Yuichiro Nakada 编写的 [catc](https://github.com/yui0/catc)
+ Jim Huang 编写的 [amacc](https://github.com/jserv/amacc)
+ Ron Cain, James E. Hendrix 编写的 [Small C](https://en.wikipedia.org/wiki/Small-C)，以及其他人的衍生版本

特别地，我将大量借鉴 SubC 编译器的思路，并使用它的一些代码。



## 设置开发环境

假设你想加入这段旅程，以下是你需要准备的东西。我将使用 Linux 开发环境，所以请下载并设置你喜欢的 Linux 系统：我使用的是 Lubuntu 18.04。

我将以两个硬件平台为目标：Intel x86-64 和 32 位 ARM。我将使用运行 Lubuntu 18.04 的 PC 作为 Intel 目标平台，使用运行 Raspbian 的树莓派作为 ARM 目标平台。

在 Intel 平台上，我们需要一个现有的 C 编译器。所以，安装这个软件包（我给出 Ubuntu/Debian 的命令）：

```sh
sudo apt-get install build-essential
```

如果在普通的 Linux 系统上还需要任何其他工具，请告诉我。

最后，克隆一份这个 Github 仓库。



## 下一步

在我们编译器编写旅程的下一部分，我们将从扫描输入文件并查找构成语言词法元素的*词法标记*的代码开始。[下一步](../01_Scanner/Readme.md)

