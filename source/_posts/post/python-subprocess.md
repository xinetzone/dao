---
title: subprocess --- 子进程管理
lang: zh-CN
abbrlink: 17ff065b
date: 2021-04-19 16:23:22
tags: subprocess
categories: Python 编程
---

`subprocess` 具有可访问的 I/O 流的子进程。该模块允许您生成进程，连接到其 `input/output/error` 管道并获取其返回码。主要 API 有：

1. `run(...)`：运行命令，等待命令完成，然后返回 `CompletedProcess` 实例。
2. `Popen(...)`：用于在新进程中灵活执行命令的类。

常量：

1. `DEVNULL`：指示应使用 `os.devnull` 的特殊值。
2. `PIPE`：指示应创建管道的特殊值。
3. `STDOUT`：指示 `stderr` 应该转到 `stdout` 的特殊值。

过时的 API（函数 `run()` ，`call()`，`check_call()` 和 `check_output()` 是 `Popen` 类的包装。直接使用 `Popen` 能够对如何运行命令以及如何处理输入输出流提供更多的控制。例如，通过对 `stdin`，`stdout` 以及 `stderr` 传递不同的参数，可以达到模仿 `os.popen()` 的效果）：

- `call(...)`：运行命令，等待它完成，然后获取其返回码。
- `check_call(...)`：与 `call()` 相同，但如果返回码不为 0，则会引发 `CalledProcessError()`
- `check_output(...)`：与 `check_call()` 相同，但返回 `stdout` 的内容，而不是返回码。
- `getoutput(...)`：在 shell 中运行命令，等待它完成，然后返回输出。
- `getstatusoutput(...)`：在 shell 中运行命令，等待它完成，然后返回 `(exitcode, output)` 元组。

## Popen

由于 `subprocess` 中的各种 API 均与 `Popen` 相关，所以，我先了解该对象。

class `Popen` 在新进程中执行子程序。有如下参数：

1. `args`：字符串或程序参数序列。`args` 被所有调用需要，应当为一个字符串，或者一个程序参数序列。提供一个参数序列通常更好，它可以更小心地使用参数中的转义字符以及引用（例如允许文件名中的空格）。如果传递一个简单的字符串，则 `shell` 参数必须为 `True` （见下文）或者该字符串中将被运行的程序名必须用简单的命名而不指定任何参数。
2. `bufsize`：在创建 `stdin/stdout/stderr` 管道文件对象时作为 `open()` 函数的 `buffering` 参数提供。
    - `0` 表示不使用缓冲区 （读取与写入是一个系统调用并且可以返回短内容）
    - `1` 表示行缓冲（只有 `universal_newlines=True` 时才有用，例如，在文本模式中）
    - 任何其他正值表示使用一个约为对应大小的缓冲区
    - 负的 `bufsize` （默认）表示使用系统默认的 `io.DEFAULT_BUFFER_SIZE`。
3. `executable`：要执行的替换程序。这很少需要。当 `shell=True`， `executable` 替换 `args` 指定运行的程序。但是，原始的 `args` 仍然被传递给程序。大多数程序将被 `args` 指定的程序作为命令名对待，这可以与实际运行的程序不同。在 POSIX， `args` 名作为实际调用程序中可执行文件的显示名称，例如 `ps`。如果 `shell=True`，在 POSIX， `executable` 参数指定用于替换默认 shell `/bin/sh` 的 shell。在 POSIX 上 `executable` 形参可以接受一个 path-like object。在Windows 上 `executable` 形参可以接受一个字节串和 path-like object。
4. `stdin`，`stdout` 和 `stderr`：它们分别指定执行程序的标准输入，标准输出和标准错误文件句柄。合法的值有 `PIPE`， `DEVNULL`， 一个存在的文件描述符（一个正整数），一个存在的 文件对象 以及 `None`。 `PIPE` 表示应创建一个新的对子进程的管道。`DEVNULL` 表示使用特殊的 `os.devnull` 文件。使用默认的 `None`，则不进行成定向；子进程的文件流将继承自父进程。另外， `stderr` 可设为 `STDOUT`，表示应用程序的标准错误数据应和标准输出一同捕获。
5. `preexec_fn`：（仅适用于 POSIX）在执行子进程之前要在子进程中调用的对象。（仅 POSIX）如果 `preexec_fn` 被设为一个可调用对象，此对象将在子进程刚创建时被调用。
6. `close_fds`：控制文件描述符（file descriptors）的关闭或继承。如果 `close_fds` 为真，所有文件描述符除了 0, 1, 2 之外都会在子进程执行前关闭。而当 `close_fds` 为 false 时，文件描述符遵守它们继承的标志，如 [文件描述符的继承](https://docs.python.org/zh-cn/3/library/os.html#fd-inheritance) 所述。
7. `shell`：如果为 true，则将通过 shell 执行该命令。
8. `cwd`：在执行子进程之前设置当前目录。如果 `cwd` 不为 `None`，此函数在执行子进程前会将当前工作目录改为 `cwd`。`cwd` 可以是一个字符串、字节串或 路径类对象。特别地，当可执行文件的路径为相对路径时，此函数会相对于 *cwd* 来查找 `executable` (或 `args` 中的第一项)。
9. `env`：定义新进程的环境变量。如果 `env` 不为 `None`，则必须为一个为新进程定义了环境变量的字典；这些用于替换继承的当前进程环境的默认行为。如果指定， `env` 必须提供所有被子进程需求的变量。在 Windows，为了运行一个 [side-by-side assembly](https://en.wikipedia.org/wiki/Side-by-Side_Assembly)，指定的 `env` **必须** 包含一个有效的 `SystemRoot`。
10. `text`：如果为 true，则使用给定的 `encoding`。（如果设置）对 stdin，stdout 和 stderr 进行解码，否则使用系统默认值。
11. `universal_newlines`：`text` 的别名，为向后兼容而提供。
12. `startupinfo` 和 `creationflags`（仅适用于 Windows）
13. `restore_signals` （仅适用于 POSIX）
14. `start_new_session`（仅适用于 POSIX）
15. `group`（仅适用于 POSIX）
16. `extra_groups`（仅适用于 POSIX）
17. `user`（仅适用于 POSIX）
18. `umask`（仅适用于 POSIX）
19. `pass_fds`（仅适用于 POSIX）
20. `encoding` 和 `errors`：用于文件对象 `stdin`，`stdout` 和 `stderr` 的文本模式编码和错误处理。

该实例有属性：`stdin`, `stdout`, `stderr`, `pid`, `returncode`。

实例创建：

```python
class Popen:
    _child_created = False  # Set here since __del__ checks it

    def __init__(self, args, bufsize=-1, executable=None,
                 stdin=None, stdout=None, stderr=None,
                 preexec_fn=None, close_fds=True,
                 shell=False, cwd=None, env=None, universal_newlines=None,
                 startupinfo=None, creationflags=0,
                 restore_signals=True, start_new_session=False,
                 pass_fds=(), *, user=None, group=None, extra_groups=None,
                 encoding=None, errors=None, text=None, umask=-1, pipesize=-1):
```

`subprocess` 模块的底层的进程创建与管理由 `Popen` 类处理。它提供了很大的灵活性，因此开发者能够处理未被便捷函数覆盖的不常见用例。

在新进程中执行子程序。在 POSIX 上，该类使用类似于 `os.execvpe()` 的行为来执行子程序。在 Windows 上，该类使用 Windows `CreateProcess()` 函数。`Popen` 的参数如下：

`args` 应当是一个程序参数的序列或者是一个单独的字符串或 [path-like object](https://docs.python.org/zh-cn/3.10/glossary.html#term-path-like-object)。默认情况下，如果 `args` 是序列则要运行的程序为 `args` 中的第一项。如果 `args` 是字符串，则其解读依赖于具体平台，如下所述。 请查看 `shell` 和 `executable` 参数了解其与默认行为的其他差异。除非另有说明，否则推荐以序列形式传入 `args`。

<div class="w3-pale-red">
警告：为了获得最大的可靠性，请为可执行文件使用完全限定的路径。要在 PATH 上搜索不合格的名称，请使用 shutil.which()。在所有平台上，建议再次传递 sys.executable 来启动当前的 Python 解释器，并使用 -m 命令行格式来启动已安装的模块。
</div>

向外部函数传入序列形式参数的一个例子如下:

```python
Popen(["/usr/bin/git", "commit", "-m", "Fixes a bug."])
```

在 POSIX，如果 `args` 是一个字符串，此字符串被作为将被执行的程序的命名或路径解释。但是，只有在不传递任何参数给程序的情况下才能这么做。

<div class="w3-card-2 w3-pale-blue w3-padding">
<i>注解</i>：将 shell 命令拆分为参数序列的方式可能并不很直观，特别是在复杂的情况下。<code>shlex.split()</code> 可以演示如何确定 <code>args</code> 适当的拆分形式：

```python
Type "help", "copyright", "credits" or "license" for more information.
>>> import shlex, subprocess
>>> command_line = input()
/bin/vikings -input eggs.txt -output "spam spam.txt" -cmd "echo '$MONEY'"
>>> args = shlex.split(command_line)
>>> print(args)
['/bin/vikings', '-input', 'eggs.txt', '-output', 'spam spam.txt', '-cmd', "echo '$MONEY'"]
>>> p = subprocess.Popen(args) # Success!
```

特别注意，由 shell 中的空格分隔的选项（例如 -input）和参数（例如 eggs.txt ）位于分开的列表元素中，而在需要时使用引号或反斜杠转义的参数在 shell（例如包含空格的文件名或上面显示的 echo 命令）是单独的列表元素。
</div>

在 Windows，如果 `args` 是一个序列，他将通过一个在 [Windows 上将参数列表转换为一个字符串](https://docs.python.org/zh-cn/3.10/library/subprocess.html#converting-argument-sequence) 描述的方式被转换为一个字符串。这是因为底层的 CreateProcess() 只处理字符串。

`Popen` 对象支持通过 `with` 语句作为上下文管理器，在退出时关闭文件描述符并等待进程：

```python
with Popen(["ifconfig"], stdout=PIPE) as proc:
    log.write(proc.stdout.read())
```

### Popen 对象

Popen 类的实例拥有以下方法：

1. `Popen.poll()`：检查子进程是否已被终止。设置并返回 `returncode` 属性。否则返回 `None`。
2. `Popen.wait(timeout=None)`：等待子进程被终止。设置并返回 `returncode` 属性。如果进程在 `timeout` 秒后未中断，抛出一个 `TimeoutExpired` 异常，可以安全地捕获此异常并重新等待。
    > 注解：当 `stdout=PIPE` 或者 `stderr=PIPE` 并且子进程产生了足以阻塞 OS 管道缓冲区接收更多数据的输出到管道时，将会发生死锁。当使用管道时用 `Popen.communicate()` 来规避它。
    > 注解：此函数使用了一个 busy loop （非阻塞调用以及短睡眠）实现。使用 `asyncio` 模块进行异步等待： 参阅 [asyncio.create_subprocess_exec](https://docs.python.org/zh-cn/3/library/asyncio-subprocess.html#asyncio.create_subprocess_exec)。
3. `Popen.communicate(input=None, timeout=None)` 与进程交互：将数据发送到 `stdin`。从 `stdout` 和 `stderr` 读取数据，直到抵达文件结尾。等待进程终止并设置 `returncode` 属性。可选的 `input` 参数应为要发送到下级进程的数据，或者如果没有要发送到下级进程的数据则为 `None`。如果流是以文本模式打开的，则 `input` 必须为字符串。在其他情况下，它必须为字节串。

`communicate()` 返回一个 `(stdout_data, stderr_data)` 元组。如果文件以文本模式打开则为字符串；否则字节。注意如果你想要向进程的 `stdin` 传输数据，你需要通过 `stdin=PIPE` 创建此 `Popen` 对象。类似的，要从结果元组获取任何非 `None` 值，你同样需要设置 `stdout=PIPE` 或者 `stderr=PIPE`。

如果进程在 `timeout` 秒后未终止，一个 `TimeoutExpired` 异常将被抛出。捕获此异常并重新等待将不会丢失任何输出。如果超时到期，子进程不会被杀死，所以为了正确清理一个行为良好的应用程序应该杀死子进程并完成通讯。

```python
proc = subprocess.Popen(...)
try:
    outs, errs = proc.communicate(timeout=15)
except TimeoutExpired:
    proc.kill()
    outs, errs = proc.communicate()
```

> 注解：内存里数据读取是缓冲的，所以如果数据尺寸过大或无限，不要使用此方法。

4. `Popen.send_signal(signal)`：将信号 `signal` 发送给子进程。如果进程已完成则不做任何操作。

>注解：在 Windows， `SIGTERM` 是一个 [`terminate()`](https://docs.python.org/zh-cn/3/library/subprocess.html#subprocess.Popen.terminate) 的别名。`CTRL_C_EVENT` 和 `CTRL_BREAK_EVENT` 可以被发送给以包含 `CREATE_NEW_PROCESS` 的 `creationflags` 形参启动的进程。

5. `Popen.terminate()`：停止子进程。在 POSIX 操作系统上，此方法会发送 `SIGTERM` 给子进程。在 Windows 上则会调用 Win32 API 函数 `TerminateProcess()` 来停止子进程。
6. `Popen.kill()`：杀死子进程。在 POSIX 操作系统上，此函数会发送 `SIGKILL` 给子进程。在 Windows 上 `kill()` 则是 `terminate()` 的别名。

以下属性也是可用的：

- Popen.args：传递给 `Popen` -- 一个程序参数的序列或者一个简单字符串
- `Popen.stdin`：如果 `stdin` 参数为 PIPE，此属性是一个类似 `open()` 返回的可写的流对象。如果 `encoding` 或 `errors` 参数被指定或者 `universal_newlines` 参数为 `True`，则此流是一个文本流，否则是字节流。如果 `stdin` 参数非 `PIPE`， 此属性为 `None`。
- `Popen.stdout`：如果 `stdout` 参数是 `PIPE`，此属性是一个类似 o`pen()` 返回的可读流。从流中读取子进程提供的输出。如果 `encoding` 或 `errors` 参数被指定或者 `universal_newlines` 参数为 `True`，此流为文本流，否则为字节流。如果 `stdout` 参数非 ·，此属性为 `None`。
- `Popen.stderr`：如果 `stderr` 参数是 `PIPE`，此属性是一个类似 `open()` 返回的可读流。从流中读取子进程提供的输出。如果 `encoding` 或 `errors` 参数被指定或者 `universal_newlines` 参数为 `True`，此流为文本流，否则为字节流。如果 `stderr` 参数非 `PIPE`，此属性为 `None`。

<div class="w3-pale-red">
警告：使用 <code>communicate()</code> 而非 <code>.stdin.write</code>， <code>.stdout.read</code> 或者 <code>.stderr.read</code> 来避免由于任意其他 OS 管道缓冲区被子进程填满阻塞而导致的死锁。
</div>

7. `Popen.pid`：子进程的进程号。注意如果你设置了 `shell` 参数为 `True`，则这是生成的子 `shell` 的进程号。
8. `Popen.returncode`：此进程的退出码，由 `poll()` 和 `wait()` 设置（以及直接由 `communicate()` 设置）。一个 `None` 值 表示此进程仍未结束。一个负值 `-N` 表示子进程被信号 `N` 中断 (仅 POSIX).

## run

`subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None, **other_popen_kwargs)` 运行被 `arg` 描述的指令。等待指令完成，然后返回一个 [CompletedProcess](https://docs.python.org/zh-cn/3.10/library/subprocess.html#subprocess.CompletedProcess) 实例。以上显示的参数仅仅是最简单的一些，下面 [常用参数](https://docs.python.org/zh-cn/3.10/library/subprocess.html#frequently-used-arguments) 描述（因此在缩写签名中使用仅关键字标示）。完整的函数头和 Popen 的构造函数一样，此函数接受的大多数参数都被传递给该接口。（`timeout`, `input`, `check` 和 `capture_output` 除外）。

如果 `capture_output` 设为 `true`，`stdout` 和 `stderr` 将会被捕获。在使用时，内置的 `Popen` 对象将自动用 `stdout=PIPE` 和 `stderr=PIPE` 创建。`stdout` 和 `stderr` 参数不应当与 `capture_output` 同时提供。如果你希望捕获并将两个流合并在一起，使用 `stdout=PIPE` 和 `stderr=STDOUT` 来代替 `capture_output`。

`timeout` 参数将被传递给 [Popen.communicate()](https://docs.python.org/zh-cn/3.10/library/subprocess.html#subprocess.Popen.communicate)。如果发生超时，子进程将被杀死并等待。 `TimeoutExpired` 异常将在子进程中断后被抛出。

`input` 参数将被传递给 `Popen.communicate()` 以及子进程的标准输入。如果使用此参数，它必须是一个字节序列。如果指定了 `encoding` 或 `errors` 或者将 `text` 设置为 `True`，那么也可以是一个字符串。当使用此参数时，在创建内部 `Popen` 对象时将自动带上 `stdin=PIPE`，并且不能再手动指定 `stdin` 参数。

如果 `check` 设为 `True`, 并且进程以非零状态码退出, 一个 `CalledProcessError` 异常将被抛出。这个异常的属性将设置为参数, 退出码, 以及标准输出和标准错误, 如果被捕获到。

如果 `encoding` 或者 `error` 被指定, 或者 `text` 被设为 `True`, 标准输入, 标准输出和标准错误的文件对象将通过指定的 `encoding` 和 `errors` 以文本模式打开, 否则以默认的 `io.TextIOWrapper` 打开。`universal_newline` 参数等同于 `text` 并且提供了向后兼容性。默认情况下, 文件对象是以二进制模式打开的。

如果 `env` 不是 `None`, 它必须是一个字典, 为新的进程设置环境变量; 它用于替换继承的当前进程的环境的默认行为. 它将直接被传递给 `Popen`。

例如：

```python
subprocess.run(["ls", "-l"])  # doesn't capture output
```

<output class="xin-term">
CompletedProcess(args=['ls', '-l'], returncode=0)
</output >

```python
subprocess.run("exit 1", shell=True, check=True)
```

<output class="xin-term">
Traceback (most recent call last):
  ...
subprocess.CalledProcessError: Command 'exit 1' returned non-zero exit status 1
</output >

```python
subprocess.run(["ls", "-l", "/dev/null"], capture_output=True)
```

<output class="xin-term">
CompletedProcess(args=['ls', '-l', '/dev/null'], returncode=0,
stdout=b'crw-rw-rw- 1 root root 1, 3 Jan 23 16:23 /dev/null\n', stderr=b'')
</output >

如果仅仅是为了运行一个外部命令而不用交互，类似 `os.system()`，可以使用 `run()` 方法。比如：

```python
# subprocess_os_system.py
import subprocess

completed = subprocess.run(['ls', '-l'])
print('returncode:', completed.returncode)
```

命令行参数被作为一个字符串列表传入，这样能够避免转义引号以及其他会被 `shell` 解析的特殊字符。`run()` 方法返回一个 `CompletedProcess` 实例，包含进程退出码以及输出等信息。

```shell
$ python subprocess_os_system.py
```

<output class="xin-term">
__pycache__
a.py
celeba.py
loader.py
test.py
vision.py
returncode: 0
</output >

设置 `shell` 参数为 `True` 会导致 `subprocess` 创建一个新的中间 `shell` 进程运行命令。默认的行为是直接运行命令。

```python
# subprocess_shell_variables.py
import subprocess

completed = subprocess.run('echo $HOME', shell=True)
print('returncode:', completed.returncode)
```

使用中间 `shell` 意味着在运行该命令之前处理命令字符串的变量，`glob` 模式以及其他特殊的 `shell` 功能。

> 使用 `run()` 而没有传递 `check=True` 等价于调用 `call()`，它仅仅返回进程的退出码。给 `run()` 方法传递 `check=True` 等价于调用 `check_all()`。

由 `run()` 启动的进程的标准输入输出渠道绑定在了父进程上。那就意味着调用程序不能捕获命令的输出。给 `stdout` 和 `stderr` 参数传递 `PIPE` 可以捕获输出用于后续处理。

```python
# subprocess_run_output.py
import subprocess

completed = subprocess.run(
    ['ls', '-1'],
    stdout=subprocess.PIPE,
)
print('returncode:', completed.returncode)
print('Have {} bytes in stdout:\n{}'.format(
    len(completed.stdout),
    completed.stdout.decode('utf-8'))
)
```

`ls -1` 命令成功运行了，所以它打印到标准输出的文本被捕获并返回了。

<output class="xin-term">
returncode: 0
Have 55 bytes in stdout:
__pycache__
a.py
celeba.py
loader.py
test.py
vision.py
</output>

> 传入 `check=True` 以及设置 `stdout` 为 `PIPE` 等价于使用 `check_output()`。

下个例子在子 `shell` 中运行了一些命令。在命令出错退出之前消息被发送到了标准输出和错误输出。

```python
# subprocess_run_output_error.py
import subprocess

try:
    completed = subprocess.run(
        'echo to stdout; echo to stderr 1>&2; exit 1',
        check=True,
        shell=True,
        stdout=subprocess.PIPE,
    )
except subprocess.CalledProcessError as err:
    print('ERROR:', err)
else:
    print('returncode:', completed.returncode)
    print('Have {} bytes in stdout: {!r}'.format(
        len(completed.stdout),
        completed.stdout.decode('utf-8'))
    )
```

标准错误输出被打印到了控制台，但是标准错误输出被隐藏了。

<output class="xin-term">
to stdout; echo to stderr ; exit 1
returncode: 0
Have 0 bytes in stdout: ''
</output>

为了阻止 `run()` 运行命令产生的错误消息打印到控制台，设置 `stderr` 参数为常量 `PIPE`。

```python
# subprocess_run_output_error_trap.py
import subprocess

try:
    completed = subprocess.run(
        'echo to stdout; echo to stderr 1>&2; exit 1',
        shell=True,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
    )
except subprocess.CalledProcessError as err:
    print('ERROR:', err)
else:
    print('returncode:', completed.returncode)
    print('Have {} bytes in stdout: {!r}'.format(
        len(completed.stdout),
        completed.stdout.decode('utf-8'))
    )
    print('Have {} bytes in stderr: {!r}'.format(
        len(completed.stderr),
        completed.stderr.decode('utf-8'))
    )
```

这个例子没有设置 `check=True`，所以命令的输出被捕获并且打印。

<output class="xin-term">
returncode: 0
Have 0 bytes in stdout: ''
Have 36 bytes in stderr: 'to stdout; echo to stderr ; exit 1\r\n'
</output>

为了捕获当使用 `check_output()` 产生的错误消息时，设置 `stderr` 为 `STDOUT`，并且这些消息将与该命令的其余输出合并。

```python
# subprocess_check_output_error_trap_output.py
import subprocess

try:
    output = subprocess.check_output(
        'echo to stdout; echo to stderr 1>&2',
        shell=True,
        stderr=subprocess.STDOUT,
    )
except subprocess.CalledProcessError as err:
    print('ERROR:', err)
else:
    print('Have {} bytes in output: {!r}'.format(
        len(output),
        output.decode('utf-8'))
    )
```

输出顺序可能会变化，取决于对标准输出流的缓冲方式以及打印的数据量。

<output class="xin-term">
Have 28 bytes in output: 'to stdout; echo to stderr \r\n'
</output>

**抑制输出**：某些情况下，输出不应该被展示和捕获，使用 `DEVNULL` 抑制输出流。这个例子抑制了标准输出流和错误输出流。

```python
# subprocess_run_output_error_suppress.py

import subprocess

try:
    completed = subprocess.run(
        'echo to stdout; echo to stderr 1>&2; exit 1',
        shell=True,
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
    )
except subprocess.CalledProcessError as err:
    print('ERROR:', err)
else:
    print('returncode:', completed.returncode)
    print('stdout is {!r}'.format(completed.stdout))
    print('stderr is {!r}'.format(completed.stderr))
```

`DEVNULL` 的名字来自于 Unix 特殊的设备文件，`/dev/null`，当读时直接响应文件结束，写时接收但忽略任何数量的输入。

<output class="xin-term">
stdout is None
stderr is None
</output>

### 与进程单向通信

为了去运行一个进程以及读取所有它的输出，设置 `stdout` 的值为 `PIPE` 并且调用 `communicate()`。

```python
# subprocess_popen_read.py
import subprocess

print('read:')
proc = subprocess.Popen(
    ['echo', '"to stdout"'],
    stdout=subprocess.PIPE,
)
stdout_value = proc.communicate()[0].decode('utf-8')
print('stdout:', repr(stdout_value))
```

这个类似于 `popen()` 的工作方式，除了读取由 `Popen` 实例内部管理。

<output class="xin-term">
read:
stdout: '"to stdout"\n'
</output>

为了设置一个管道允许调用者向其写入数据，设置 `stdin` 为 `PIPE`。

```python
# subprocess_popen_write.py
import subprocess

print('write:')
proc = subprocess.Popen(
    ['cat', '-'],
    stdin=subprocess.PIPE,
)
proc.communicate('stdin: to stdin\n'.encode('utf-8'))
```

<output class="xin-term">
write:
stdin: to stdin
</output>

### 与进程双向通信

为了设置 Popen 实例同时进行读写，请结合之前使用过的技术。

```python
# subprocess_popen2.py
import subprocess

print('popen2:')

proc = subprocess.Popen(
    ['cat', '-'],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
)
msg = 'through stdin to stdout'.encode('utf-8')
stdout_value = proc.communicate(msg)[0].decode('utf-8')
print('pass through:', repr(stdout_value))
```

这样设置使用就有点像 `popen2()` 了。

<output class="xin-term">
popen2:
pass through: 'through stdin to stdout'
</output>

### 捕获错误输出

同时查看 `stdout` 和 `stderr` 输出流也是可能的，就像 `popen3()`。

```python
# subprocess_popen3.py
import subprocess

print('popen3:')
proc = subprocess.Popen(
    'cat -; echo "to stderr" 1>&2',
    shell=True,
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
)
msg = 'through stdin to stdout'.encode('utf-8')
stdout_value, stderr_value = proc.communicate(msg)
print('pass through:', repr(stdout_value.decode('utf-8')))
print('stderr      :', repr(stderr_value.decode('utf-8')))
```

从 `stderr` 中读取错误输出类似于 `stdout`。传入 `PIPE` 告诉 `Popen` 附加到通道，并且使用 `communicate()` 在返回之前读取所有数据。

<output class="xin-term">
popen3:
pass through: 'through stdin to stdout'
stderr      : 'to stderr\n'
</output>

### 合并常规和错误输出

为了将进程的错误输出导向标准输出渠道，设置 `stderr` 为 `STDOUT` 而不是 `PIPE`。

```python
# subprocess_popen4.py
import subprocess

print('popen4:')
proc = subprocess.Popen(
    'cat -; echo "to stderr" 1>&2',
    shell=True,
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
)
msg = 'through stdin to stdout\n'.encode('utf-8')
stdout_value, stderr_value = proc.communicate(msg)
print('combined output:', repr(stdout_value.decode('utf-8')))
print('stderr value   :', repr(stderr_value))
```

这种合并输出的方式类似于 `popen4()` 的工作方式。

<output class="xin-term">
popen4:
combined output: 'through stdin to stdout\nto stderr\n'
stderr value   : None
</output>


`subprocess` 模块允许你生成新的进程，连接它们的输入、输出、错误管道，并且获取它们的返回码。推荐的调用子进程的方式是在任何它支持的用例中使用 `run()` 函数。对于更进阶的用例，也可以使用底层的 `Popen` 接口。

## 连接管道的段

多个命令可以被连接到一个 管道 中，类似于 Unix shell 的工作方式，实现这种操作，可以通过创建分隔的 `Popen` 实例并将他们的输入输出链在一起。一个 `Popen` 实例的 `stdout` 属性被用作下一个的 `stdin` 参数，而不是之前的常量 `PIPE`。要获取整个执行的输出，可以从最后一个 `Popen` 实例的 `stdout` 流读取。

```python
# subprocess_pipes.py
import subprocess

cat = subprocess.Popen(
    ['cat', 'index.rst'],
    stdout=subprocess.PIPE,
)

grep = subprocess.Popen(
    ['grep', '.. literalinclude::'],
    stdin=cat.stdout,
    stdout=subprocess.PIPE,
)

cut = subprocess.Popen(
    ['cut', '-f', '3', '-d:'],
    stdin=grep.stdout,
    stdout=subprocess.PIPE,
)

end_of_pipe = cut.stdout

print('Included files:')
for line in end_of_pipe:
    print(line.decode('utf-8').strip())
```

这个例子同下面的命令行操作：

```shell
$ cat index.rst | grep ".. literalinclude" | cut -f 3 -d:
```

这个部分首先管道读取 `reStructuredText` 源文件，然后找到所有包含其他文件的行，最后打印被包含的文件名称。

## 同另一个命令交互

所有前面的例子都假定了一个有限的交互，`communicate()` 方法读取所有输出并等待子进程在返回之前退出。在程序运行时也可以逐步写入和读取 `Popen` 实例使用的单个管道句柄。从标准输入中读取并希望如标准输出的简单回声程序说明了这种技术。

脚本 `repeater.py` 被用作下一个例子的子进程。它从 `stdin` 读取并且写入到 `stdout` ，一次一行，直到再没有输入。当开始和停止的时候，它也往 `stderr` 写入了一条消息，展示子进程的声明周期。

```python
# repeater.py
import sys

sys.stderr.write('repeater.py: starting\n')
sys.stderr.flush()

while True:
    next_line = sys.stdin.readline()
    sys.stderr.flush()
    if not next_line:
        break
    sys.stdout.write(next_line)
    sys.stdout.flush()

sys.stderr.write('repeater.py: exiting\n')
sys.stderr.flush()
```

下一个例子中以不同的方式使用 `Popen` 实例的 `stdin` 和 `stdout` 文件句柄。在第一个例子中，五个数字被依次写入到进程的 `stdin`，每次写入后，紧接着会读出输入并打印出来了。第二个例子中相同的五个数字被写入，但是输出通过 `communicate()` 依次行读取了。

```python
# interaction.py
import io
import subprocess

print('One line at a time:')
proc = subprocess.Popen(
    'python3 repeater.py',
    shell=True,
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
)
stdin = io.TextIOWrapper(
    proc.stdin,
    encoding='utf-8',
    line_buffering=True,  # send data on newline
)
stdout = io.TextIOWrapper(
    proc.stdout,
    encoding='utf-8',
)
for i in range(5):
    line = '{}\n'.format(i)
    stdin.write(line)
    output = stdout.readline()
    print(output.rstrip())
remainder = proc.communicate()[0].decode('utf-8')
print(remainder)

print()
print('All output at once:')
proc = subprocess.Popen(
    'python3 repeater.py',
    shell=True,
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
)
stdin = io.TextIOWrapper(
    proc.stdin,
    encoding='utf-8',
)
for i in range(5):
    line = '{}\n'.format(i)
    stdin.write(line)
stdin.flush()

output = proc.communicate()[0].decode('utf-8')
print(output)
```

每个循环中， `"repeater.py: exiting"` 行在输出的不同点出现。

```shell
$ python3 -u interaction.py

One line at a time:
repeater.py: starting
0
1
2
3
4
repeater.py: exiting

All output at once:
repeater.py: starting
repeater.py: exiting
0
1
2
3
4
```

## 进程间的信号

`os` 模块的进程管理示例包括使了用 `os.fork()` 和 `os.kill()` 进程之间的信号演示。由于每个 `Popen` 实例都提供了一个 `pid` 属性和子进程 `id`，所以可以对子进程执行类似的操作。下一个例子合并了两个脚本，子进程设置了一个 `USR` 信号处理器。

```python
# signal_child.py
import os
import signal
import time
import sys

pid = os.getpid()
received = False

def signal_usr1(signum, frame):
    "Callback invoked when a signal is received"
    global received
    received = True
    print('CHILD {:>6}: Received USR1'.format(pid))
    sys.stdout.flush()

print('CHILD {:>6}: Setting up signal handler'.format(pid))
sys.stdout.flush()
signal.signal(signal.SIGUSR1, signal_usr1)
print('CHILD {:>6}: Pausing to wait for signal'.format(pid))
sys.stdout.flush()
time.sleep(3)

if not received:
    print('CHILD {:>6}: Never received signal'.format(pid))
```

这个脚本被当做父进程运行，它启动了 `signal_child.py`，然后发送了 `USR1` 信号。

```python
# signal_parent.py
import os
import signal
import subprocess
import time
import sys

proc = subprocess.Popen(['python3', 'signal_child.py'])
print('PARENT      : Pausing before sending signal...')
sys.stdout.flush()
time.sleep(1)
print('PARENT      : Signaling child')
sys.stdout.flush()
os.kill(proc.pid, signal.SIGUSR1)
```

输出是：

```shell
$ python3 signal_parent.py

PARENT      : Pausing before sending signal...
CHILD  26976: Setting up signal handler
CHILD  26976: Pausing to wait for signal
PARENT      : Signaling child
CHILD  26976: Received USR1
```

## 进程 组 / 会话

如果由 `Popen` 创建的进程产生子进程，那么子进程将不会收到任何发送给父进程的任何信号。这意味着当对 `Popen` 使用 `shell` 参数时，很难通过发送 `SIGINT` 和 `SIGTERM` 来使 `shell` 中启动的命令终止。

```python
# subprocess_signal_parent_shell.py
import os
import signal
import subprocess
import tempfile
import time
import sys

script = '''#!/bin/sh
echo "Shell script in process $$"
set -x
python3 signal_child.py
'''
script_file = tempfile.NamedTemporaryFile('wt')
script_file.write(script)
script_file.flush()

proc = subprocess.Popen(['sh', script_file.name])
print('PARENT      : Pausing before signaling {}...'.format(
    proc.pid))
sys.stdout.flush()
time.sleep(1)
print('PARENT      : Signaling child {}'.format(proc.pid))
sys.stdout.flush()
os.kill(proc.pid, signal.SIGUSR1)
time.sleep(3)
```

用于发送信号的 `pid` 与等待信号的运行 `shell` 脚本的子进程 `id` 不同，因为这个例子中有三个独立的进程在交互：

1. 主程序 `subprocess_signal_parent_shell.py`
2. 主程序创建的运行脚本的 `shell` 进程。
3. 程序 `signal_child.py`

```python
$ python3 subprocess_signal_parent_shell.py

PARENT      : Pausing before signaling 26984...
Shell script in process 26984
+ python3 signal_child.py
CHILD  26985: Setting up signal handler
CHILD  26985: Pausing to wait for signal
PARENT      : Signaling child 26984
CHILD  26985: Never received signal
```

要在不知道进程 `id` 的情况下向后代进程发送信号，请使用进程组关联这些子进程，以便可以一起发送信号。进程组使用 `os.setpgrp()` 创建，它将进程组 `id` 设置为当前进程 `id`。所有子进程都从父进程继承他们的进程组，因为它只应在由 `Popen` 及其后代创建的 `shell` 中设置，所以不应在创建 `Popen` 的相同进程中调用 `os.setpgrp()`。而是，应在作为 `Popen` 的 `preexec_fn` 参数设置的函数中调用，它会在新进程的 `fork` 之后运行，在用 `exec` 运行 `shell` 之前。为了给进程组发送信号，应该使用 `os.killpg()` 并使用 `Popen` 实例的进程 `id`。

```python
# subprocess_signal_setpgrp.py
import os
import signal
import subprocess
import tempfile
import time
import sys

def show_setting_prgrp():
    print('Calling os.setpgrp() from {}'.format(os.getpid()))
    os.setpgrp()
    print('Process group is now {}'.format(os.getpgrp()))
    sys.stdout.flush()

script = '''#!/bin/sh
echo "Shell script in process $$"
set -x
python3 signal_child.py
'''
script_file = tempfile.NamedTemporaryFile('wt')
script_file.write(script)
script_file.flush()

proc = subprocess.Popen(
    ['sh', script_file.name],
    preexec_fn=show_setting_prgrp,
)
print('PARENT      : Pausing before signaling {}...'.format(
    proc.pid))
sys.stdout.flush()
time.sleep(1)
print('PARENT      : Signaling process group {}'.format(
    proc.pid))
sys.stdout.flush()
os.killpg(proc.pid, signal.SIGUSR1)
time.sleep(3)
```

整个运行流程如下：

1. 父进程实例化 `Popen`；
2. `Popen` 实例 `fork` 新进程；
3. 新进程运行 `os.setpgrp()`；
4. 新进程运行 `exec()` 启动 shell；
5. shell 运行脚本；
6. shell 脚本再次 `fork`，然后启动 Python 解释器；
7。 Python 运行 `signal_child.py`.
8. 父进程发送信号非进程组，使用 `Popen` 实例的进程 `id`；
9. shell and Python 程序收到信号；
10. shell 忽略掉了信号。
11. 运行 `signal_child.py` 的 Python 程序 调用了信号处理器。


```shell
$ python3 subprocess_signal_setpgrp.py

Calling os.setpgrp() from 75636
Process group is now 75636
PARENT      : Pausing before signaling 75636...
Shell script in process 75636
+ python3 signal_child.py
CHILD  75637: Setting up signal handler
CHILD  75637: Pausing to wait for signal
PARENT      : Signaling process group 75636
CHILD  75637: Received USR1
```
