---
title: pySerial
lang: zh-CN
tags: PySerial
categories: Python
abbrlink: 2ceed41e
date: 2021-04-13 14:51:51
---

参考：[pySerial’s documentation](https://pyserial.readthedocs.io/en/latest/index.html)

该模块封装了对串行端口（serial port）的访问。它提供了在 Windows，OSX，Linux，BSD（可能是任何POSIX兼容系统）和 IronPython 上运行的 Python 的后端。名为 `serial` 的模块会自动选择适当的后端。

## `serial.tools.list_ports.comports(include_links=False)`

- `include_links`（bool）–在指向串行端口的 `/dev` 下包含符号链接。
- 返回包含 `ListPortInfo` 对象的列表。此列表的顺序没有特殊含义。另请注意，即使对于同一设备，报告的字符串在平台和操作系统之间也不同。

在 Linux，OSX 和 Windows 下，扩展信息将可用于 USB 设备（例如 `ListPortInfo.hwid` 字符串包含 `VID:PID`，`SER`（序列号），`LOCATION`（层次结构），这使它们可以通过 `grep()` 进行搜索。USB 的信息也可使用 `ListPortInfo` 的属性获取。比如：

```python
from serial.tools import list_ports
devices = list_ports.comports()
devices, devices[0].hwid
```

<output>
([&lt;serial.tools.list_ports_common.ListPortInfo at 0x2a0ca0280a0&gt;,
  &lt;serial.tools.list_ports_common.ListPortInfo at 0x2a0c8100940&gt;],
'USB VID:PID=0403:6001 SER=A107QDSTA')
</output>

如果`include_links`为`True`，则检查 `/dev` 下的所有设备是否是到已知串行端口设备的链接。这些项将在其 `hwid` 字符串中包含 LINK。这意味着同一设备列出了两次，一次以其原始名称，一次以链接名称。

## `serial.tools.list_ports.grep(regexp, include_links=False)`

- `regexp` –正则表达式（请参阅标准库 [re](https://docs.python.org/3/library/re.html#module-re)）
- 返回 `ListPortInfo` 对象的生成器，另请参见 `list_ports.comports()`。

使用正则表达式搜索端口。搜索端口`name`，`description`和`hwid`（不区分大小写）。比如：

```python
for port in list_ports.grep('USB'):
    print(port)
```

<output>
COM6 - USB Serial Port (COM6)
</output>

## `classserial.tools.list_ports.ListPortInfo`

该对象保存有关串行端口的信息。它支持索引访问以实现向后兼容性，例如在`port, desc, hwid = info`中。

<dl class="w3-pale-yellow w3-card-4 w3-padding">
 <dt class="w3-pale-green w3-card-4">device</dt>
 <dd>完整的设备名称/路径，例如 <code>/dev/ttyUSB0</code>。这也是当被索引访问时作为第一个元素返回的信息。</dd>
 <dt class="w3-pale-green w3-card-4">name</dt>
 <dd>设备名称</dd>
 <dt class="w3-pale-green w3-card-4">description</dt>
 <dd>人类可读的描述或 n/a。这也是当通过索引访问时作为第二个元素返回的信息。</dd>
 <dt class="w3-pale-green w3-card-4">hwid</dt>
 <dd>技术说明或 n/a。这也是当通过索引访问时作为第三个元素返回的信息。</dd>
 <dt class="w3-pale-green w3-card-4">vid</dt>
 <dd>USB 供应商（Vendor）ID（整数，0~65535）。</dd>
 <dt class="w3-pale-green w3-card-4">pid</dt>
 <dd>USB 产品 ID（整数，0~65535）。</dd>
 <dt class="w3-pale-green w3-card-4">serial_number</dt>
 <dd>USB 的字符串序列号。</dd>
 <dt class="w3-pale-green w3-card-4">location</dt>
 <dd>USB设备位置字符串（“<code>&lt;bus&gt;-&lt;port>[-&lt;port>]…</code>”）</dd>
 <dt class="w3-pale-green w3-card-4">manufacturer</dt>
 <dd>USB 制造商字符串，由设备报告。</dd>
 <dt class="w3-pale-green w3-card-4">product</dt>
 <dd>USB 产品字符串，由设备报告。</dd>
 <dt class="w3-pale-green w3-card-4">interface</dt>
 <dd>界面特定的描述，例如 用于复合 USB 设备。</dd>
</dl>

实现比较运算符，以便可以按`device`对 `ListPortInfo` 对象进行排序。字符串分为数字和文本组，因此顺序是“自然的”（即 `com1` < `com2` < `com10`）。