蓝鲸GSE-Data编译说明
====================

# 依赖

## 硬件要求

|   类型   |   要求   |
| -------- | -------- |
| CPU架构  | x86_64   |
| 内存     | 4GB      |
| 硬盘     | 10GB     |

## 操作系统要求

服务运行环境为Linux，可根据实际生产环境需要选择指定发行版本进行编译和运行,
需要基础的make和gcc/gcc-c++（4.7以上支持C++11）命令支持进行代码编译。

## 软件要求

以下为主要的依赖软件版本信息:

|  软件名称              |     版本     |            备注               |
| -----------------------| ------------ | ----------------------------- |
| libunwind              |   v1.5.0     |             -                 |
| gperftools             |   v2.9.1     |             -                 |
| librdkafka             |   v1.2.2     |             -                 |
| pulsar                 |   v2.7.3     |             -                 |

`注意`：相关依赖组件可通过`make deps`命令进行自动安装

## 安装gse-main组件

基于Git或直接下载获取源码执行编译:

```shell
> cd gse-main
> make deps && make && make install
```

# 编译

基于Git或直接下载获取指定分支版本源码, 按照如下操作步骤进行编译,

```shell
> cd gse-data
> make deps && make
```

编译成功之后会在项目根目录下生产`build`目录，目录中为编译成功的二进制以及其他一并打包的工具脚本和文档。
基于编译后的打包文件，按照[安装](install.md)文档进行部署安装即可。
