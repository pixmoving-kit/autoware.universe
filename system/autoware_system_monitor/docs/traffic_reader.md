# traffic_reader

<a id="name"></a>

## 名称

traffic_reader - 按进程监控网络流量

<a id="synopsis"></a>

## 用法概要

traffic_reader [OPTION]

<a id="description"></a>

## 说明

按进程监控网络流量。<br>
程序以守护进程运行，并监听 UNIX 域套接字（默认为 "/tmp/traffic_reader"）。

**选项：**<br>
_-h, --help_<br>
&nbsp;&nbsp;&nbsp;&nbsp;显示帮助<br>
_-s, --socket PATH_<br>
&nbsp;&nbsp;&nbsp;&nbsp;UNIX 域套接字路径

**退出状态：**<br>
正常时返回 0，否则返回非零值。

<a id="notes"></a>

## 说明事项

'traffic_reader' 需要 nethogs 命令。<br>

<a id="operation-confirmed-platform"></a>

## 已验证运行的平台

- Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-40-generic x86_64)
