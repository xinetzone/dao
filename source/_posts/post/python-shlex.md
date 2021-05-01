---
title: shlex --- 简单的词法分析
lang: zh-CN
tags: shlex
categories: Python
abbrlink: a3a9b0e7
date: 2021-04-13 22:28:30
---

参考自：[shlex --- 简单的词法分析](https://docs.python.org/zh-cn/3.10/library/shlex.html#shlex.shlex.commenters)

`shlex` 类使您可以轻松地为类似于 Unix Shell 的简单语法编写词法分析器。这对于编写迷你语言（例如，在 Python 应用程序的运行控制文件中）或解析带引号的字符串通常很有用。

## `shlex.split(s, comments=False, posix=True)`

使用类似于 shell 的语法拆分字符串。如果`comments`为 `False`（默认值），则将禁用对给定字符串中的注释的解析（将 shlex 实例的 `commenters` 属性设置为空字符串）。默认情况下，此函数在 POSIX 模式下运行，但如果 `posix` 参数为 `False`，则使用非 POSIX 模式。

## `shlex.join(split_command)`

连接列表 `split_command` 的标记并返回一个字符串。此函数是 `split` 的逆函数。

```python
from shlex import join
print(join(['echo', '-n', 'Multiple words']))
```

<output>
echo -n 'Multiple words'
</output>

返回的值将转义为 shell，以防止注入漏洞（请参阅 `quote`）。

## `shlex.quote(s)`

返回字符串的 shell 换码版本。返回的值是一个字符串，在无法使用列表的情况下，可以安全地用作 shell 命令行中的一个标记。

比如，下面的常用写法是不安全的：

```python
filename = 'somefile; rm -rf ~'
command = 'ls -l {}'.format(filename)
print(command)  # executed by a shell: boom!
```

<output>
ls -l somefile; rm -rf ~
</output>

`quote` 使您可以填补安全漏洞：

```python
from shlex import quote
command = 'ls -l {}'.format(quote(filename))
print(command)
remote_command = 'ssh home {}'.format(quote(command))
print(remote_command)
```

<output>
<div>
ls -l 'somefile; rm -rf ~'
</div>
<div>
ssh home 'ls -l '"'"'somefile; rm -rf ~'"'"''
</div>
</output>

可以看出，那些可能造成漏洞的字符被转义了。

该引用与 UNIX shell 和 `split()` 兼容：

```python
from shlex import split
remote_command = split(remote_command)
print(remote_command)
command = split(remote_command[-1])
print(remote_command)
```

<output>
<div>
['ssh', 'home', "ls -l 'somefile; rm -rf ~'"]
</div>
<div>
['ls', '-l', 'somefile; rm -rf ~']
</div>
</output>

## `class shlex.shlex(instream=None, infile=None, posix=False, punctuation_chars=False)`

`shlex` 实例或子类实例是词法分析器对象。初始化参数（如果存在）指定从何处读取字符。它必须是具有`read`和`readline`方法的类似于文件/流的对象，或者是字符串。如果未提供任何参数，则输入将从`sys.stdin`中获取。第二个可选参数是文件名字符串，用于设置`infile`属性的初始值。如果省略了`instream`参数或等于`sys.stdin`，则第二个参数默认为`"stdin"`。`posix`参数定义操作模式：如果`posix`不为 true（默认值），则 `shlex` 实例将在兼容模式下运行。在 POSIX 模式下运行时，`shlex` 将尝试尽可能接近 POSIX shell 解析规则。`punctuation_chars` 参数提供了一种使行为更接近于实际 shell 解析方式的方法。这可以采用许多值：默认值`False`保留了在 Python 3.5 及更早版本中看到的行为。如果设置为`True`，则将解析字符`();<>|&`：将这些字符的任何运行（考虑为标点符号）作为单个标记返回。如果设置为非空字符串，则这些字符将用作标点字符。出现在`punctuation_chars`中的`wordchars`属性中的任何字符都将从`wordchars`中删除。`punctuation_chars`只能在创建 `shlex` 实例时设置，以后不能修改。

## shlex 对象

`shlex` 实例具有以下方法：

暂更

##