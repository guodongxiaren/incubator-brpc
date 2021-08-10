# 构建

brpc鼓励静态链接依赖，以便于每个运行brpc服务的机器不必再安装依赖。

brpc有如下依赖：

* [gflags](https://github.com/gflags/gflags): Extensively used to define global options.
* [protobuf](https://github.com/google/protobuf): Serializations of messages, interfaces of services.
* [leveldb](https://github.com/google/leveldb): Required by [/rpcz](rpcz.md) to record RPCs for tracing.

# 支持的环境

* [Ubuntu/LinuxMint/WSL](#ubuntulinuxmintwsl)
* [Fedora/CentOS](#fedoracentos)
* [Linux with self-built deps](#linux-with-self-built-deps)
* [MacOS](#macos)

## Ubuntu/LinuxMint/WSL
### 依赖准备

安装通用依赖， [gflags](https://github.com/gflags/gflags), [protobuf](https://github.com/google/protobuf), [leveldb](https://github.com/google/leveldb):
```shell
sudo apt-get install -y git g++ make libssl-dev libgflags-dev libprotobuf-dev libprotoc-dev protobuf-compiler libleveldb-dev
```

如果你需要静态链接leveldb：
```shell
sudo apt-get install -y libsnappy-dev
```

如果你要在样例中启用cpu/heap的profiler：
```shell
sudo apt-get install -y libgoogle-perftools-dev
```

如果你要运行测试，那么要安装并编译 ligtest-dev（它没有被默认编译）：
```shell
sudo apt-get install -y cmake libgtest-dev && cd /usr/src/gtest && sudo cmake . && sudo make && sudo mv libgtest* /usr/lib/ && cd -
```
gtest源码目录可能变动，如果 `/usr/src/gtest` 不存在，请尝试 `/usr/src/googletest/googletest`。

### 使用config_brpc.sh编译brpc
git克隆brpc，进入到项目目录，然后运行
```shell
$ sh config_brpc.sh --headers=/usr/include --libs=/usr/lib
$ make
```
修改编译器为clang，添加选项 `--cxx=clang++ --cc=clang`。

不想链接调试符号，添加选项 `--nodebugsymbols` 然后编译将会得到更清亮的二进制文件。

使用glog版的brpc，添加选项 `--with-glog`.

要启用 [thrift 支持](../en/thrift.md)，首先安装thrift并且添加选项 `--with-thrift`。

**运行样例**

```shell
$ cd example/echo_c++
$ make
$ ./echo_server &
$ ./echo_client
```

上述操作会链接brpc的静态库到样例中，如果你想链接brpc的共享库，请依次执行：`make clean` 和 `LINK_SO=1 make`

**运行测试**
```shell
$ cd test
$ make
$ sh run_tests.sh
```

### 使用cmake编译brpc
```shell
cmake -B build && cmake --build build -j6
```
要帮助VSCode或Emacs(LSP)去正确地理解代码，添加 `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON`选项去生成 `compile_commands.json`。

要修改编译器为clang，请依次覆盖环境变量 `CC` 和 `CXX` 为 `clang` 和 `clang++`。

不想链接调试符号，请移除 `build/CMakeCache.txt`，然后带着 `-DWITH_DEBUG_SYMBOLS=OFF`选项执行cmake。

想要让brpc使用glog，带着 `-DWITH_GLOG=ON`选项执行cmake。

要启用 [thrift 支持](../en/thrift.md)，先安装thrift，然后带着 `-DWITH_THRIFT=ON`选项执行cmake。

**用cmake运行样例**

```shell
$ cd example/echo_c++
$ cmake -B build && cmake --build build -j4
$ ./echo_server &
$ ./echo_client
```

上述操作会链接brpc的静态库到样例中，如果你想链接brpc的共享库，请先移除`CMakeCache.txt`，然后用`-DLINK_SO=ON`选项重新执行cmake。

**运行测试**

```shell
$ mkdir build && cd build && cmake -DBUILD_UNIT_TESTS=ON .. && make && make test
```

## Fedora/CentOS

### 依赖准备

CentOS一般需要安装EPEL，否则很多包都默认不可用。
```shell
sudo yum install epel-release
```

安装通用依赖：
```shell
sudo yum install git gcc-c++ make openssl-devel
```

安装 [gflags](https://github.com/gflags/gflags), [protobuf](https://github.com/google/protobuf), [leveldb](https://github.com/google/leveldb):
```shell
sudo yum install gflags-devel protobuf-devel protobuf-compiler leveldb-devel
```

如果你要在样例中启用cpu/heap的profiler：
```shell
sudo yum install gperftools-devel
```

如果你要运行测试，那么要安装ligtest-dev:
```shell
sudo yum install gtest-devel
```

### 使用config_brpc.sh编译brpc

git clone brpc, cd into the repo and run

```shell
$ sh config_brpc.sh --headers=/usr/include --libs=/usr/lib64
$ make
```
修改编译器为clang，添加选项 `--cxx=clang++ --cc=clang`。

不想链接调试符号，添加选项 `--nodebugsymbols` 然后编译将会得到更清亮的二进制文件。

想要让brpc使用glog，添加选项：`--with-glog`。

要启用 [thrift 支持](../en/thrift.md)，先安装thrift，然后添加选项：`--with-thrift`。

**运行样例**

```shell
$ cd example/echo_c++
$ make
$ ./echo_server &
$ ./echo_client
```

上述操作会链接brpc的静态库到样例中，如果你想链接brpc的共享库，请依次执行：`make clean` 和 `LINK_SO=1 make`

**运行测试**
```shell
$ cd test
$ make
$ sh run_tests.sh
```

### 使用cmake编译brpc
参考[这里](#使用cmake编译brpc)

## 自己构建依赖的Linux

### 依赖准备

brpc默认会构建出静态库和共享库，因此它也需要依赖有静态库和共享库两个版本。

以 [gflags](https://github.com/gflags/gflags)为例，它默认不够尖共享库，你需要给 `cmake` 指定选项去改变这一行为：
```shell
$ cmake . -DBUILD_SHARED_LIBS=1 -DBUILD_STATIC_LIBS=1
$ make
```

### 编译brpc

还以gflags为例，让 `../gflags_dev` 表示gflags被克隆的位置。

git克隆brpc。进入到项目目录然后运行：

```shell
$ sh config_brpc.sh --headers="../gflags_dev /usr/include" --libs="../gflags_dev /usr/lib64"
$ make
```

这里我们给 `--headers` 和 `--libs`传递多个路径使得脚本能够在多个地方进行检索。你也可以打包所有依赖和brpc一起放到一个目录中，然后把目录传递给 --headers/--libs选项，它会递归搜索所有子目录直到找到必须的文件。

修改编译器为clang，添加选项 `--cxx=clang++ --cc=clang`。

不想链接调试符号，添加选项 `--nodebugsymbols` 然后编译将会得到更清亮的二进制文件。

使用glog版的brpc，添加选项 `--with-glog`.

要启用 [thrift 支持](../en/thrift.md)，首先安装thrift并且添加选项 `--with-thrift`。

```shell
$ ls my_dev
gflags_dev protobuf_dev leveldb_dev brpc_dev
$ cd brpc_dev
$ sh config_brpc.sh --headers=.. --libs=..
$ make
```

### 使用cmake编译brpc
参考[这里](#使用cmake编译brpc)

## MacOS

注意：在相同运行环境下，当前Mac版brpc的性能比Linux版差2.5倍。如果你的服务是性能敏感的，请不要使用MacOs作为你的生产环境。

### 依赖准备

安装通用依赖：
```shell
brew install openssl git gnu-getopt coreutils
```

安装 [gflags](https://github.com/gflags/gflags)，[protobuf](https://github.com/google/protobuf)，[ [leveldb](https://github.com/google/leveldb)：
```shell
brew install gflags protobuf leveldb
```

如果你要在样例中启用cpu/heap的profiler：
```shell
brew install gperftools
```

如果你要运行测试，那么要安装并编译 googletest（它没有被默认编译）：
```shell
git clone https://github.com/google/googletest -b release-1.10.0 && cd googletest/googletest && mkdir build && cd build && cmake -DCMAKE_CXX_FLAGS="-std=c++11" .. && make
```
在编译完成后，复制 include/ 和 lib/ 目录到 /usr/local/include 和 /usr/local/lib目录中，以便于让所有应用都能使用gtest。

### 使用config_brpc.sh编译brpc
git克隆brpc，进入到项目目录然后运行：
```shell
$ sh config_brpc.sh --headers=/usr/local/include --libs=/usr/local/lib --cc=clang --cxx=clang++
$ make
```
不想链接调试符号，添加选项 `--nodebugsymbols` 然后编译将会得到更清亮的二进制文件。

使用glog版的brpc，添加选项 `--with-glog`.

要启用 [thrift support](../en/thrift.md)，首先安装thrift并且添加选项 `--with-thrift`。

**运行样例**

```shell
$ cd example/echo_c++
$ make
$ ./echo_server &
$ ./echo_client
```
上述操作会链接brpc的静态库到样例中，如果你想链接brpc的共享库，请依次执行：`make clean` 和 `LINK_SO=1 make`

**运行测试**
```shell
$ cd test
$ make
$ sh run_tests.sh
```

### 使用cmake编译brpc
Same with [here](#compile-brpc-with-cmake)

# 支持的依赖

## GCC: 4.8-7.1

c++11 is turned on by default to remove dependencies on boost (atomic).

The over-aligned issues in GCC7 is suppressed temporarily now.

Using other versions of gcc may generate warnings, contact us to fix.

Adding `-D__const__=` to cxxflags in your makefiles is a must to avoid [errno issue in gcc4+](thread_local.md).

## Clang: 3.5-4.0

无已知问题。

## glibc: 2.12-2.25

无已知问题。

## protobuf: 2.4+

Be compatible with pb 3.x and pb 2.x with the same file:
Don't use new types in proto3 and start the proto file with `syntax="proto2";`
[tools/add_syntax_equal_proto2_to_all.sh](https://github.com/brpc/brpc/blob/master/tools/add_syntax_equal_proto2_to_all.sh)can add `syntax="proto2"` to all proto files without it.

Arena in pb 3.x is not supported yet.

## gflags: 2.0-2.2.1

无已知问题。

## openssl: 0.97-1.1

required by https.

## tcmalloc: 1.7-2.5

brpc does **not** link [tcmalloc](http://goog-perftools.sourceforge.net/doc/tcmalloc.html) by default. Users link tcmalloc on-demand.

Comparing to ptmalloc embedded in glibc, tcmalloc often improves performance. However different versions of tcmalloc may behave really differently. For example, tcmalloc 2.1 may make multi-threaded examples in brpc perform significantly worse(due to a spinlock in tcmalloc) than the one using tcmalloc 1.7 and 2.5. Even different minor versions may differ. When you program behave unexpectedly, remove tcmalloc or try another version.

Code compiled with gcc 4.8.2 and linked to a tcmalloc compiled with earlier GCC may crash or deadlock before main(), E.g:

![img](../images/tcmalloc_stuck.png)

When you meet the issue, compile tcmalloc with the same GCC.

Another common issue with tcmalloc is that it does not return memory to system as early as ptmalloc. So when there's an invalid memory access, the program may not crash directly, instead it crashes at a unrelated place, or even not crash. When you program has weird memory issues, try removing tcmalloc.

If you want to use [cpu profiler](cpu_profiler.md) or [heap profiler](heap_profiler.md), do link `libtcmalloc_and_profiler.a`. These two profilers are based on tcmalloc.[contention profiler](contention_profiler.md) does not require tcmalloc.

When you remove tcmalloc, not only remove the linkage with tcmalloc but also the macro `-DBRPC_ENABLE_CPU_PROFILER`.

## glog: 3.3+

brpc implements a default [logging utility](../../src/butil/logging.h) which conflicts with glog. To replace this with glog, add *--with-glog* to config_brpc.sh or add `-DWITH_GLOG=ON` to cmake.

## valgrind: 3.8+

brpc detects valgrind automatically (and registers stacks of bthread). Older valgrind(say 3.2) is not supported.

## thrift: 0.9.3-0.11.0

无已知问题。

# 实例追踪

我们提供了一个程序去帮助你追踪和监控所有brpc实例。 只需要现在某处运行 [trackme_server](https://github.com/brpc/brpc/tree/master/tools/trackme_server/) 然后再带着 -trackme_server=SERVER参数启动需要被追踪的实例。 The trackme_server will receive pings from instances periodically and print logs when it does. You can aggregate instance addresses from the log and call builtin services of the instances for further information.
