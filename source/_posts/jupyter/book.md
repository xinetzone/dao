---
layout: jupyter
title: Books with Jupyter
date: 2021-05-20 16:53:25
tags: jupyter-book
---

Jupyter Book 是一个开源项目，用于从计算中构建漂亮的、具有出版质量的书籍和文档。

以下是木星之书的一些特点：

- [在 Markdown 中编写出版质量的内容](https://jupyterbook.org/file-types/markdown.html)：你可以使用 Jupyter Markdown，也可以使用带有[发布功能](https://jupyterbook.org/content/myst.html)的 Markdown 的扩展版本。这包括对富文本语法的支持，如[引用和交叉引用](https://jupyterbook.org/content/citations.html)、[数学和等式](https://jupyterbook.org/content/math.html)，以及[图像](https://jupyterbook.org/content/figures.html)。
- [把内容写在 Jupyter Notebook 上](https://jupyterbook.org/file-types/notebooks.html)：这允许您在书中包含您的代码和输出。您也可以[完全在 Markdown 中编写](https://jupyterbook.org/file-types/myst-notebooks.html)笔记本，当您构建您的书时执行。
- [执行并缓存你的书的内容](https://jupyterbook.org/content/execute.html)：对于 `.ipynb` 和 Markdown 笔记本，执行代码并将最新的输出插入到您的书中。此外，[缓存和重用输出](https://jupyterbook.org/content/execute.html#execute-cache)以供以后使用。
- [将笔记本输出插入内容中](https://jupyterbook.org/content/code-outputs.html#content-code-outputs)：在构建文档时生成输出，并将它们与跨页面的内容一起插入。
- [增加你的书的互动性](https://jupyterbook.org/interactive/launchbuttons.html)：您可以[切换单元格可见性](https://jupyterbook.org/interactive/hiding.html)，包括来自 Jupyter 的[交互式输出](https://jupyterbook.org/interactive/interactive.html)，并与 Binder 等在线服务连接。
- [生成各种输出](https://jupyterbook.org/start/build.html)：这包括单页和多页的网站，以及 [PDF 输出](https://jupyterbook.org/advanced/pdf.html)。
- [使用简单的命令行界面构建书籍](https://jupyterbook.org/reference/cli.html)：您可以使用如下命令快速生成图书：`jupyter-book build mybook/`

安装很简单：`pip install -U jupyter-book`。

其他资源：

- [Jupyter Book Gallery](http://gallery.jupyterbook.org/)：从别人创作的 Jupyter Book 中获得灵感。
- [Contribute to Jupyter Book](https://jupyterbook.org/contribute/intro.html)：通过遵循贡献指南，找到需要解决的问题。查看功能投票排行榜以获得灵感。

## 一个小的示例项目

这是 Jupyter Book 创建的一本书的一个[简短例子](https://executablebooks.github.io/quantecon-mini-example/docs/index.html)。

展出的一些功能包括：

* [Jupyter Notebook-style inputs and outputs](https://executablebooks.github.io/quantecon-mini-example/docs/python_by_example.html#version-1)

* [citations](https://executablebooks.github.io/quantecon-mini-example/docs/about_py.html#bibliography)

* [numbered equations](https://executablebooks.github.io/quantecon-mini-example/docs/python_by_example.html#another-application)

* [numbered figures](https://executablebooks.github.io/quantecon-mini-example/docs/getting_started.html#jupyter-notebooks) with captions and cross-referencing

源文件可以在 [GitHub 的 docs 目录](https://github.com/executablebooks/quantecon-mini-example/)中找到。这些文件都是在 [MyST Markdown](https://jupyterbook.org/content/myst.html) 中编写的，这是 Jupyter Notebook Markdown 的扩展，允许额外的科学标记。它们也可以直接写成 Jupyter Notebook。

### 创建一个 demo

您可以通过以下步骤在命令行本地构建这本书：

1. 确保你安装了最新版本的 [Anaconda Python](https://www.anaconda.com/distribution/)。
2. 克隆包含演示图书源文件的存储库

```shell
git clone https://github.com/executablebooks/quantecon-mini-example
cd quantecon-mini-example
```

3. 安装从[environment.yml](https://github.com/executablebooks/quantecon-mini-example/blob/master/environment.yml) 中运行此特定示例中的代码所需的 Python 库。这包括最新版本的 Jupyter Book:

```shell
conda env create -f environment.yml
conda activate qe-mini-example
```

4. 在源文件上运行 Jupyter Book：

```shell
jupyter-book build ./mini_book
```

5. 通过浏览器查看结果- try(例如，firefox)

```shell
firefox mini_book/_build/html/index.html
```

(或者直接双击 HTML 文件)

现在您可能想尝试编辑 `mini_book/docs` 中的文件，然后重新构建。

## 创建你的第一本书

在本教程中，我们将介绍 Jupyter Book 生态系统的基础知识，并逐步引导您创建、构建和出版您的第一本书。

### 概览

本节简要概述了构建 Jupyter Book 的主要组件和步骤。有关更深入的信息，请参阅本指南的其他页面。

### Jupyter Book 的命令行界面

Jupyter Book 使用命令行界面来执行各种操作。例如，建造和清洁书籍。您可以运行以下命令查看哪些选项在您的控制下：

```shell
jupyter-book --help
```

```shell
Usage: jupyter-book [OPTIONS] COMMAND [ARGS]...

  Build and manage books with Jupyter.

Options:
  --version   Show the version and exit.
  -h, --help  Show this message and exit.

Commands:
  build   Convert your book's or page's content to HTML or a PDF.
  clean   Empty the _build directory except jupyter_cache.
  config  Inspect your _config.yml file.
  create  Create a Jupyter Book template that you can customize.
  myst    Manipulate MyST markdown files.
  toc     Command-line for sphinx-external-toc.
```

有关 CLI 的更完整信息，请参见[命令行界面](https://jupyterbook.org/reference/cli.html)。

### 书籍构建过程

构建一本木星之书大致包括以下步骤：

1. **创建你的书的内容**。您可以用文件夹、文件和配置的集合来构造您的书。参见 [木星之书的解剖](https://jupyterbook.org/start/overview.html#anatomy-of-a-book)。
2. **构建你的书**。使用 Jupyter Book 的命令行界面，您可以将页面转换为 HTML 或 PDF 图书。参见 [构建你的书](https://jupyterbook.org/start/build.html)。
3. 在网上出版你的书。一旦你的书建好了，你就可以和别人分享了。最常见的是构建 HTML，并将其作为公共网站托管。参见[在线出版你的书](https://jupyterbook.org/start/publish.html)。

### 木星之书的解剖

你需要做三件事来制作一本木星之书：

- 配置文件：`_config.yml`
- 目录文件：`_toc.yml`
- 你的书的内容

例如，考虑下面的文件夹结构，它组成了一个简单的 Jupyter Book。

```shell
mybookname/
├── _config.yml
├── _toc.yml
├── landing-page.md
└── page1.ipynb
```

#### Book 的配置（`_config.yml`）

您的书的所有配置都在一个名为 `_config.yml` 的 YAML 文件中。

您可以为您的图书定义元数据(比如它的标题)，添加图书标识，打开不同的“交互式”按钮(比如一个 [Binder](https://jupyterbook.org/reference/glossary.html#term-Binder) 按钮，用于从 Jupyter Book 构建的页面)，等等。

下面是一个简单的 `_config.yml` 示例文件：

```ymal
# in _config.yml
title: "My book title"
logo: images/logo.png
execute:
  execute_notebooks: "off"
```

- `title`：定义书的标题。它将会被展示在左侧的侧边栏。
- `logo`：定义图书标识的图像文件的路径(它也会显示在侧边栏中)。
- `execute`：包含控制[执行和缓存](https://jupyterbook.org/content/execute.html)的配置选项集合。`execute_notebooks: "off"` 告诉 Jupyter Book 不要执行任何它在构建书时发现的计算内容。默认情况下，Jupyter Book 会执行并缓存所有的图书内容。

更多关于 `_config.yml`：你可以用 `_config.yml` 文件做更多的事情。例如，您可以[添加源存储库按钮](https://jupyterbook.org/basics/repository.html#source-repository-button)或添加[交互式数据可视化](https://jupyterbook.org/interactive/interactive.html)。获取`_config.yml`的完整字段列表，请参见[配置参考](https://jupyterbook.org/customize/config.html)。

#### Book 的目录（`_toc.yml`）

Jupyter Book 使用你的目录来定义你的书的结构。例如，你的章节，分章节等等。

这是一个包含一组页面的 YAML 文件，每个页面都链接到书中的一个文件。下面是上面显示的两个内容文件的示例。

```ymal
# In _toc.yml
- file: landing-page
- file: page1
```

`_toc.yml` 中的每一项都指向单个文件。链接应该是相对于你的书的文件夹，没有扩展名。你可以将 TOC 文件的最顶层想象成书籍的章节(不包括登录页)。每一章的标题将从你文件的标题中推断出来。

第一个文件指定图书的登录页(在本例中，它是一个 markdown 文件)。登录页（landing page）是书籍内容层次结构中最高的页面。第二个文件指定图书的内容页(在本例中，它是一个 Jupyter Notebook)。

您可以使用 `_toc.yml` 文件 指定更复杂的图书配置。例如，您可以指定部件、节和控制自定义标题。有关书的目录文件的更多信息，请参见[结构化书的页面](https://jupyterbook.org/customize/toc.html)。

#### Book content

一组文本文件构成了你的书的内容。这些文件可以是几种类型中的一种，例如 markdown (`.md`)、Jupyter notebook (`.ipynb`)或 reStructuredText (`.rst`)文件(请参阅[内容源文件的类型](https://jupyterbook.org/file-types/index.html)以获得完整列表)。

在上面的例子中，列出了两个文件:一个 markdown 文件和一个 Jupyter Notebook。我们将在下一节中介绍它们。

### 创建你的书的源文件

现在我们理解了书的结构，让我们创建一个示例书来学习。

### 快速生成示例书

Jupyter Book 附带了一本轻量级的示例书来帮助你理解一本书的结构。运行以下命令创建一个示例书：

```shell
jupyter-book create mynewbook/
```

这将生成一个迷你的 Jupyter Book，你可以在本地构建和探索。它会为您做出一些决定，您可以在`_config.yml`中探索这本书的配置及其在`_toc.yml`中的结构化。把这本书当作灵感，或者作为你工作的起点。

### 研究你的书的内容文件

首先，请注意至少有两种不同类型的内容文件：markdown 文件(以 `.md` 结尾)和 Jupyter Notebooks 文件(以 `.ipynb` 结尾)。

我们将在下面逐一讨论。

#### Markdown files (`.md`)