# 构建

brpc prefers static linkages of deps, so that they don't have to be installed on every machine running the app.

brpc依赖下面的包：

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

If you need to enable cpu/heap profilers in examples:
```shell
sudo apt-get install -y libgoogle-perftools-dev
```

If you need to run tests, install and compile libgtest-dev (which is not compiled yet):
```shell
sudo apt-get install -y cmake libgtest-dev && cd /usr/src/gtest && sudo cmake . && sudo make && sudo mv libgtest* /usr/lib/ && cd -
```
The directory of gtest source code may be changed, try `/usr/src/googletest/googletest` if `/usr/src/gtest` is not there.

### 使用config_brpc.sh编译brpc
git克隆brpc，进入到项目目录，然后运行
```shell
$ sh config_brpc.sh --headers=/usr/include --libs=/usr/lib
$ make
```
修改编译器为clang，添加选项 `--cxx=clang++ --cc=clang`。

不链接调试符号，添加选项 `--nodebugsymbols` 然后编译将会得到更清亮的二进制文件。

使用glog版的brpc，添加选项 `--with-glog`.

要启用 [thrift support](../en/thrift.md)，首先安装thrift并且添加选项 `--with-thrift`。

**运行样例**

```shell
$ cd example/echo_c++
$ make
$ ./echo_server &
$ ./echo_client
```

Examples link brpc statically, if you need to link the shared version, `make clean` and `LINK_SO=1 make`

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
To help VSCode or Emacs(LSP) to understand code correctly, add `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` to generate `compile_commands.json`

To change compiler to clang, overwrite environment variable `CC` and `CXX` to `clang` and `clang++` respectively.

To not link debugging symbols, remove `build/CMakeCache.txt` and cmake with `-DWITH_DEBUG_SYMBOLS=OFF`

To use brpc with glog, cmake with `-DWITH_GLOG=ON`.

To enable [thrift support](../en/thrift.md), install thrift first and cmake with `-DWITH_THRIFT=ON`.

**用cmake运行样例**

```shell
$ cd example/echo_c++
$ cmake -B build && cmake --build build -j4
$ ./echo_server &
$ ./echo_client
```
Examples link brpc statically, if you need to link the shared version, remove `CMakeCache.txt` and cmake with `-DLINK_SO=ON`

**用cmake运行测试**

```shell
$ mkdir build && cd build && cmake -DBUILD_UNIT_TESTS=ON .. && make && make test
```

## Fedora/CentOS

### 依赖准备

Install common deps:
```shell
brew install openssl git gnu-getopt coreutils
```

Install [gflags](https://github.com/gflags/gflags), [protobuf](https://github.com/google/protobuf), [leveldb](https://github.com/google/leveldb):
```shell
brew install gflags protobuf leveldb
```

If you need to enable cpu/heap profilers in examples:
```shell
brew install gperftools
```

If you need to run tests, download and compile googletest (which is not compiled yet):
```shell
git clone https://github.com/google/googletest -b release-1.10.0 && cd googletest/googletest && mkdir build && cd build && cmake -DCMAKE_CXX_FLAGS="-std=c++11" .. && make
```
After the compilation, copy include/ and lib/ into /usr/local/include and /usr/local/lib respectively to expose gtest to all apps

### 使用config_brpc.sh编译brpc
git clone brpc, cd into the repo and run
```shell
$ sh config_brpc.sh --headers=/usr/local/include --libs=/usr/local/lib --cc=clang --cxx=clang++
$ make
```
To not link debugging symbols, add `--nodebugsymbols` and compiled binaries will be much smaller.

To use brpc with glog, add `--with-glog`.

To enable [thrift support](../en/thrift.md), install thrift first and add `--with-thrift`.

**运行样例**

```shell
$ cd example/echo_c++
$ make
$ ./echo_server &
$ ./echo_client
```

Examples link brpc statically, if you need to link the shared version, `make clean` and `LINK_SO=1 make`

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

no known issues.

## glibc: 2.12-2.25

no known issues.

## protobuf: 2.4+

Be compatible with pb 3.x and pb 2.x with the same file:
Don't use new types in proto3 and start the proto file with `syntax="proto2";`
[tools/add_syntax_equal_proto2_to_all.sh](https://github.com/brpc/brpc/blob/master/tools/add_syntax_equal_proto2_to_all.sh)can add `syntax="proto2"` to all proto files without it.

Arena in pb 3.x is not supported yet.

## gflags: 2.0-2.2.1

no known issues.

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

no known issues.

# Track instances

We provide a program to help you to track and monitor all brpc instances. Just run [trackme_server](https://github.com/brpc/brpc/tree/master/tools/trackme_server/) somewhere and launch need-to-be-tracked instances with -trackme_server=SERVER. The trackme_server will receive pings from instances periodically and print logs when it does. You can aggregate instance addresses from the log and call builtin services of the instances for further information.
