---
layout:     post
title:      Verilator从源代码安装
subtitle:   ubuntu-20.04.1
date:       2020-01-31
author:     BY, Amadeus979
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog
---

由于相关项目无法直接用`apt-get install verilator`下载下来的verilator，故使用源码安装。

## 安装相关依赖

```shell
sudo apt-get install perl python3 make
sudo apt-get install g++  # 亦可用clang代替
sudo apt-get install libgz  # Ubuntu不需要 (会报错)
sudo apt-get install libfl2 libfl-dev zlibc zlib1g zlib1g-dev  # 只有ubuntu需要
```

下面是可选项，为了获得良好的性能，应使用以下命令安装相关的包：

```shell
sudo apt-get install ccache  # If present at build, needed for run
sudo apt-get install libgoogle-perftools-dev numactl perl-doc
```

构建Verilator需要安装这些软件包。运行Verilator时不需要：

```shell
sudo apt-get install git autoconf flex bison
```

Veriloator的开发者才会想要这些：

```shell
sudo apt-get install gdb asciidoctor graphviz cmake clang clang-format gprof lcov
cpan install Pod::Perldoc
cpan install Parallel::Forker
```

### Install SystemC

If you will be using SystemC (vs straight C++ output), download [SystemC](https://www.accellera.org/downloads/standards/systemc). Follow their installation instructions. You will need to set `SYSTEMC_INCLUDE` to point to the include directory with `systemc.h` in it, and `SYSTEMC_LIBDIR` to points to the directory with `libsystemc.a` in it. (Older installations may set `SYSTEMC` and `SYSTEMC_ARCH` instead.)

不知道干嘛的，没装。

### 安装GTKWave

[GTKwave](http://gtkwave.sourceforge.net/)看波形图的，Verilator构建的时候并不需要。

由于和verilator没有直接的依赖关系，我直接`sudo apt-get install gtkwave`

## 获取源

从资源库获取源：（只需执行一次）

```shell
git clone https://github.com/verilator/verilator   # Only first time
```

输入`checkout`并确定要使用的版本/分支：

```shell
cd verilator
git pull        # Make sure we're up-to-date
git tag         # See what versions exist
#git checkout master      # Use development branch (e.g. recent bug fix)
#git checkout stable      # Use most recent release
#git checkout v{version}  # Switch to specified release version
```

我使用了stable分支。

## 自动配置

创建配置文件：

```shell
autoconf        # 创建 ./configure script
```

## 最终安装选项

配置安装之前，必须决定最终如何安装该套件。

Verilator将把VERILATOR_ROOT，SYSTEMC_INCLUDE和SYSTEMC_LIBDIR的当前值作为默认值编译到可执行文件中，因此在配置之前它们必须正确。

以下是一些可选项：

#### 3.5.1. 1. Run-in-Place from VERILATOR_ROOT

Our personal favorite is to always run Verilator in-place from its Git directory. This allows the easiest experimentation and upgrading, and allows many versions of Verilator to co-exist on a system.

```
export VERILATOR_ROOT=`pwd`   # if your shell is bash
setenv VERILATOR_ROOT `pwd`   # if your shell is csh
./configure
# Running will use files from $VERILATOR_ROOT, so no install needed
```

Note after installing (below steps), a calling program or shell must set the environment variable `VERILATOR_ROOT` to point to this Git directory, then execute `$VERILATOR_ROOT/bin/verilator`, which will find the path to all needed files.

#### 从VERILATOR_ROOT（免安装）直接运行

我们个人最喜欢的是始终从其Git目录中就地运行Verilator。这允许最简单的实验和升级，并且允许许多版本的Verilator并存于系统上。emmmmm，其实并不总是。

```shell
export VERILATOR_ROOT=`pwd`   # if your shell is bash
# setenv VERILATOR_ROOT `pwd`   # if your shell is csh
./configure
# Running will use files from $VERILATOR_ROOT, so no install needed
```

请注意，安装后（执行后续步骤），调用程序或shell必须将环境变量VERILATOR_ROOT设置为指向此Git目录，然后执行`$VERILATOR_ROOT/bin/verilator`，这将找到所有所需文件的路径。

```shell
xry@BettyRay:~/Software/verilator$ $VERILATOR_ROOT/bin/verilator
Usage:
        verilator --help
        verilator --version
        verilator --cc [options] [source_files.v]... [opt_c_files.cpp/c/cc/a/o/so]
        verilator --sc [options] [source_files.v]... [opt_c_files.cpp/c/cc/a/o/so]
        verilator --lint-only -Wall [source_files.v]...


```

干脆直接弄到.bashrc里面：

```shell
# Verilator exports
export VERILATOR_ROOT=/home/xry/Software/verilator
```



#### 3.5.2. 2. Install into a CAD Disk

You may eventually be installing onto a project/company-wide "CAD" tools disk that may support multiple versions of every tool. Target the build to a destination directory name that includes the Verilator version name:

```shell
unset VERILATOR_ROOT      # if your shell is bash
unsetenv VERILATOR_ROOT   # if your shell is csh
# For the tarball, use the version number instead of git describe
./configure --prefix /CAD_DISK/verilator/`git describe | sed "s/verilator_//"`
```

Note after installing (below steps), if you use [modulecmd](http://modules.sourceforge.net/), you’ll want a module file like the following:

modulecmd’s verilator/version file

```shell
set install_root /CAD_DISK/verilator/{version-number-used-above}
unsetenv VERILATOR_ROOT
prepend-path PATH $install_root/bin
prepend-path MANPATH $install_root/man
prepend-path PKG_CONFIG_PATH $install_root/share/pkgconfig
```

#### 3.5.3. 3. Install into a Specific Path

You may eventually install Verilator into a specific installation prefix, as most GNU tools support:

```shell
unset VERILATOR_ROOT      # if your shell is bash
unsetenv VERILATOR_ROOT   # if your shell is csh
./configure --prefix /opt/verilator-VERSION
```

Then after installing (below steps) you will need to add `/opt/verilator-VERSION/bin` to `$PATH`.

#### 3.5.4. 4. Install System Globally

The final option is to eventually install Verilator globally, using the normal system paths:

```shell
unset VERILATOR_ROOT      # if your shell is bash
unsetenv VERILATOR_ROOT   # if your shell is csh
./configure
```

Then after installing (below) the binary directories should already be in your `$PATH`.

## 配置

上面描述了配置软件包的command。开发人员应配置为具有更完整的开发人员测试。这些测试可能需要其他软件包。不知道这个开发人员指的是不是verilator的开发人员，以防万一，我执行了。

```shell
export VERILATOR_AUTHOR_SITE=1    # Put in your .bashrc
./configure --enable-longtests  ...above options...
```

后面自检出bug了，还是不建议执行。

```shell
TESTS DONE, PASSED: Passed 1  Failed 0  Unsup 0  Time 0:07
make -C test_regress
make[1]: Entering directory '/home/xry/Software/verilator/test_regress'
/usr/bin/perl driver.pl -j 0 --quiet --rerun --vlt --vltmt --dist 
%Error: Can't use -j: Can't locate Parallel/Forker.pm in @INC (you may need to install the Parallel::Forker module) (@INC contains: . /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/x86_64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at (eval 12) line 1.
BEGIN failed--compilation aborted at (eval 12) line 1.

make[1]: *** [Makefile:51: test] Error 255
make[1]: Leaving directory '/home/xry/Software/verilator/test_regress'
make: *** [Makefile:251: test_regress] Error 2

```

看起来是因为上面的开发者依赖库我没装的原因。



## 编译

```shell
make -j
```

## 测试

通过运行自检来检查编译是否成功：

```shell
make test
```

## 安装

## 4. Running Verilator

运行Verilator，参见 [Verilator manual (HTML)](https://verilator.org/verilator_doc.html), or [Verilator manual (PDF)](https://verilator.org/verilator_doc.pdf)的示例部分。

另参阅工具包中已有的`examples/` 目录。安了了的话应该在系统指定位置，常在`/usr/local/share/verilator/examples`。

英文原文：

To run Verilator, see the example sections in the [Verilator manual (HTML)](https://verilator.org/verilator_doc.html), or [Verilator manual (PDF)](https://verilator.org/verilator_doc.pdf).

Also see the `examples/` directory that is part of the kit, and is installed (in a OS-specific place, often in e.g. `/usr/local/share/verilator/examples`).



```
cd examples/make_hello_c
make
```

**！！！！注意！！！！**如果你要使用`make install`安装，上面你不应该设置 `VERILATOR_ROOT` ，***如果在.bashrc中有一定要删掉***。它内置在可执行文件中。

原文：Note if you did a `make install` above you should not have `VERILATOR_ROOT` set in your environment; it is built into the executable.

Log：

```shell
-- Verilator hello-world simple example
-- VERILATE & BUILD --------
/home/xry/Software/verilator/bin/verilator -cc --exe --build -j top.v sim_main.cpp
make[1]: Entering directory '/home/xry/Software/verilator/examples/make_hello_c/obj_dir'
make[1]: Nothing to be done for 'default'.
make[1]: Leaving directory '/home/xry/Software/verilator/examples/make_hello_c/obj_dir'
-- RUN ---------------------
obj_dir/Vtop
Hello World!
- top.v:11: Verilog $finish
-- DONE --------------------
Note: Once this example is understood, see examples/make_tracing_c.
Note: Also see the EXAMPLE section in the verilator manpage/document.

```

## 目录结构

该软件包中的一些相关文件和目录如下：

```shell
Changes                     => Version history
README.adoc                 => This document
bin/verilator               => Compiler wrapper invoked to Verilate code
docs/                       => Additional documentation
examples/make_hello_c       => Example GNU-make simple Verilog->C++ conversion
examples/make_hello_sc      => Example GNU-make simple Verilog->SystemC conversion
examples/make_tracing_c     => Example GNU-make Verilog->C++ with tracing
examples/make_tracing_sc    => Example GNU-make Verilog->SystemC with tracing
examples/make_protect_lib   => Example using --protect-lib
examples/cmake_hello_c      => Example building make_hello_c with CMake
examples/cmake_hello_sc     => Example building make_hello_sc with CMake
examples/cmake_tracing_c    => Example building make_tracing_c with CMake
examples/cmake_tracing_sc   => Example building make_tracing_sc with CMake
examples/cmake_protect_lib  => Example building make_protect_lib with CMake
include/                    => Files that should be in your -I compiler path
include/verilated*.cpp      => Global routines to link into your simulator
include/verilated*.h        => Global headers
include/verilated.mk        => Common Makefile
src/                        => Translator source code
test_regress                => Internal tests
```

对于在验证设计后创建的文件，请参见 [Verilator manual (HTML)](https://verilator.org/verilator_doc.html)，或 [Verilator manual (PDF)](https://verilator.org/verilator_doc.pdf)。

附上编译和测试的Log：

```shell
xry@BettyRay:~/Software/verilator$ make -j
------------------------------------------------------------
making verilator in src
make -C src  
make[1]: Entering directory '/home/xry/Software/verilator/src'
make -C obj_dbg -j 1  TGT=../../bin/verilator_bin_dbg VL_DEBUG=1 -f ../Makefile_obj serial
make -C obj_dbg       TGT=../../bin/verilator_coverage_bin_dbg VL_DEBUG=1 VL_VLCOV=1 -f ../Makefile_obj serial_vlcov
make -C obj_opt -j 1  TGT=../../bin/verilator_bin -f ../Makefile_obj serial
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Nothing to be done for 'serial'.
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make -C obj_dbg       TGT=../../bin/verilator_bin_dbg VL_DEBUG=1 -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_opt'
make[2]: Nothing to be done for 'serial'.
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_opt'
make -C obj_opt       TGT=../../bin/verilator_bin -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Nothing to be done for 'serial_vlcov'.
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make -C obj_dbg       TGT=../../bin/verilator_coverage_bin_dbg VL_DEBUG=1 VL_VLCOV=1 -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_opt'
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
      Compile flags:  g++ -MMD -I. -I.. -I.. -I../../include -I../../include -ggdb -DVL_DEBUG -D_GLIBCXX_DEBUG -MP -faligned-new -Wno-unused-parameter -Wno-shadow -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -DDEFENV_SYSTEMC="" -DDEFENV_SYSTEMC_ARCH="" -DDEFENV_SYSTEMC_INCLUDE="" -DDEFENV_SYSTEMC_LIBDIR="" -DDEFENV_VERILATOR_ROOT="/home/xry/Software/verilator"
      Compile flags:  g++ -MMD -I. -I.. -I.. -I../../include -I../../include -O2 -MP -faligned-new -Wno-unused-parameter -Wno-shadow -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -DDEFENV_SYSTEMC="" -DDEFENV_SYSTEMC_ARCH="" -DDEFENV_SYSTEMC_INCLUDE="" -DDEFENV_SYSTEMC_LIBDIR="" -DDEFENV_VERILATOR_ROOT="/home/xry/Software/verilator"
      Compile flags:  g++ -MMD -I. -I.. -I.. -I../../include -I../../include -ggdb -DVL_DEBUG -D_GLIBCXX_DEBUG -MP -faligned-new -Wno-unused-parameter -Wno-shadow -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -DDEFENV_SYSTEMC="" -DDEFENV_SYSTEMC_ARCH="" -DDEFENV_SYSTEMC_INCLUDE="" -DDEFENV_SYSTEMC_LIBDIR="" -DDEFENV_VERILATOR_ROOT="/home/xry/Software/verilator"
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_opt'
make[1]: Leaving directory '/home/xry/Software/verilator/src'
Build complete!

Now type 'make test' to test.

xry@BettyRay:~/Software/verilator$ make test
------------------------------------------------------------
making verilator in src
make -C src  
make[1]: Entering directory '/home/xry/Software/verilator/src'
make -C obj_dbg -j 1  TGT=../../bin/verilator_bin_dbg VL_DEBUG=1 -f ../Makefile_obj serial
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Nothing to be done for 'serial'.
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make -C obj_dbg       TGT=../../bin/verilator_bin_dbg VL_DEBUG=1 -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
      Compile flags:  g++ -MMD -I. -I.. -I.. -I../../include -I../../include -ggdb -DVL_DEBUG -D_GLIBCXX_DEBUG -MP -faligned-new -Wno-unused-parameter -Wno-shadow -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -DDEFENV_SYSTEMC="" -DDEFENV_SYSTEMC_ARCH="" -DDEFENV_SYSTEMC_INCLUDE="" -DDEFENV_SYSTEMC_LIBDIR="" -DDEFENV_VERILATOR_ROOT="/home/xry/Software/verilator"
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make -C obj_dbg       TGT=../../bin/verilator_coverage_bin_dbg VL_DEBUG=1 VL_VLCOV=1 -f ../Makefile_obj serial_vlcov
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
make[2]: Nothing to be done for 'serial_vlcov'.
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make -C obj_dbg       TGT=../../bin/verilator_coverage_bin_dbg VL_DEBUG=1 VL_VLCOV=1 -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_dbg'
      Compile flags:  g++ -MMD -I. -I.. -I.. -I../../include -I../../include -ggdb -DVL_DEBUG -D_GLIBCXX_DEBUG -MP -faligned-new -Wno-unused-parameter -Wno-shadow -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -DDEFENV_SYSTEMC="" -DDEFENV_SYSTEMC_ARCH="" -DDEFENV_SYSTEMC_INCLUDE="" -DDEFENV_SYSTEMC_LIBDIR="" -DDEFENV_VERILATOR_ROOT="/home/xry/Software/verilator"
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_dbg'
make -C obj_opt -j 1  TGT=../../bin/verilator_bin -f ../Makefile_obj serial
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_opt'
make[2]: Nothing to be done for 'serial'.
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_opt'
make -C obj_opt       TGT=../../bin/verilator_bin -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/src/obj_opt'
      Compile flags:  g++ -MMD -I. -I.. -I.. -I../../include -I../../include -O2 -MP -faligned-new -Wno-unused-parameter -Wno-shadow -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -DDEFENV_SYSTEMC="" -DDEFENV_SYSTEMC_ARCH="" -DDEFENV_SYSTEMC_INCLUDE="" -DDEFENV_SYSTEMC_LIBDIR="" -DDEFENV_VERILATOR_ROOT="/home/xry/Software/verilator"
make[2]: Leaving directory '/home/xry/Software/verilator/src/obj_opt'
make[1]: Leaving directory '/home/xry/Software/verilator/src'
test_regress/t/t_a1_first_cc.pl
======================================================================
dist/t_a1_first_cc: ==================================================
-Skip: dist/t_a1_first_cc: scenario 'dist' not enabled for test
dist/t_a1_first_cc: -Skip: Skip: scenario 'dist' not enabled for test
==SUMMARY: Passed 0  Failed 0  Unsup 0  Time 0:00
======================================================================
vlt/t_a1_first_cc: ==================================================
	perl ../bin/verilator --debug --debugi 0 --gdbbt --no-dump-tree -V
warning: File "/home/xry/Software/verilator/test_regress/.gdbinit" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
No stack.
warning: File "/home/xry/Software/verilator/test_regress/.gdbinit" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
	add-auto-load-safe-path /home/xry/Software/verilator/test_regress/.gdbinit
line to your configuration file "/home/xry/.gdbinit".
To completely disable this security protection add
	set auto-load safe-path /
line to your configuration file "/home/xry/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
	info "(gdb)Auto-loading safe path"
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Starting Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d
Starting Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d
Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d

Copyright 2003-2021 by Wilson Snyder.  Verilator is free software; you can
redistribute it and/or modify the Verilator internals under the terms of
either the GNU Lesser General Public License Version 3 or the Perl Artistic
License Version 2.0.

See https://verilator.org for documentation

Summary of configuration:
  Compiled in defaults if not in environment:
    SYSTEMC            = 
    SYSTEMC_ARCH       = 
    SYSTEMC_INCLUDE    = 
    SYSTEMC_LIBDIR     = 
    VERILATOR_ROOT     = /home/xry/Software/verilator
    SystemC system-wide = 1

Environment:
    MAKE               = make
    PERL               = 
    SYSTEMC            = 
    SYSTEMC_ARCH       = 
    SYSTEMC_INCLUDE    = 
    SYSTEMC_LIBDIR     = 
    VERILATOR_ROOT     = /home/xry/Software/verilator
    VERILATOR_BIN      = 

Features (based on environment or compiled-in support):
    SystemC found      = 1
[Inferior 1 (process 152302) exited normally]
No stack.
	perl /home/xry/Software/verilator/bin/verilator --prefix Vt_a1_first_cc ../obj_vlt/t_a1_first_cc/Vt_a1_first_cc__main.cpp --exe --make gmake --x-assign unique -cc -Mdir obj_vlt/t_a1_first_cc -OD --debug-check --comp-limit-members 10 --debug --debugi 0 --gdbbt --no-dump-tree --trace --clk clk  -f input.vc +define+TEST_OBJ_DIR=obj_vlt/t_a1_first_cc t/t_a1_first_cc.v    > obj_vlt/t_a1_first_cc/vlt_compile.log
warning: File "/home/xry/Software/verilator/test_regress/.gdbinit" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
No stack.
warning: File "/home/xry/Software/verilator/test_regress/.gdbinit" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
	add-auto-load-safe-path /home/xry/Software/verilator/test_regress/.gdbinit
line to your configuration file "/home/xry/.gdbinit".
To completely disable this security protection add
	set auto-load safe-path /
line to your configuration file "/home/xry/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
	info "(gdb)Auto-loading safe path"
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Starting Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d
Starting Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d
[Inferior 1 (process 152322) exited normally]
No stack.
	make -C obj_vlt/t_a1_first_cc -f /home/xry/Software/verilator/test_regress/Makefile_obj --no-print-directory VM_PREFIX=Vt_a1_first_cc TEST_OBJ_DIR=obj_vlt/t_a1_first_cc CPPFLAGS_DRIVER=-DT_A1_FIRST_CC  OPT_FAST=-O0 OPT_GLOBAL=-O0 Vt_a1_first_cc    > obj_vlt/t_a1_first_cc/vlt_gcc.log
driver: Entering directory '/home/xry/Software/verilator/test_regress/obj_vlt/t_a1_first_cc'
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -DVERILATOR=1 -DVL_DEBUG=1 -DTEST_OBJ_DIR=obj_vlt/t_a1_first_cc -DVM_PREFIX=Vt_a1_first_cc -DVM_PREFIX_INCLUDE="<Vt_a1_first_cc.h>" -DT_A1_FIRST_CC   -O0 -c -o Vt_a1_first_cc__main.o ../../obj_vlt/t_a1_first_cc/Vt_a1_first_cc__main.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -DVERILATOR=1 -DVL_DEBUG=1 -DTEST_OBJ_DIR=obj_vlt/t_a1_first_cc -DVM_PREFIX=Vt_a1_first_cc -DVM_PREFIX_INCLUDE="<Vt_a1_first_cc.h>" -DT_A1_FIRST_CC   -O0 -c -o verilated.o /home/xry/Software/verilator/include/verilated.cpp
g++    Vt_a1_first_cc__main.o verilated.o verilated_vcd_c.o Vt_a1_first_cc__ALL.a      -o Vt_a1_first_cc
driver: Leaving directory '/home/xry/Software/verilator/test_regress/obj_vlt/t_a1_first_cc'
	obj_vlt/t_a1_first_cc/Vt_a1_first_cc    > obj_vlt/t_a1_first_cc/vlt_sim.log
*-* All Finished *-*
- t/t_a1_first_cc.v:17: Verilog $finish
vlt/t_a1_first_cc: Self PASSED
==SUMMARY: Passed 1  Failed 0  Unsup 0  Time 0:04
==SUMMARY: Passed 1  Failed 0  Unsup 0  Time 0:04

======================================================================
TESTS DONE, PASSED: Passed 1  Failed 0  Unsup 0  Time 0:04
test_regress/t/t_a2_first_sc.pl
======================================================================
dist/t_a2_first_sc: ==================================================
-Skip: dist/t_a2_first_sc: scenario 'dist' not enabled for test
dist/t_a2_first_sc: -Skip: Skip: scenario 'dist' not enabled for test
==SUMMARY: Passed 0  Failed 0  Unsup 0  Time 0:00
======================================================================
vlt/t_a2_first_sc: ==================================================
	perl /home/xry/Software/verilator/bin/verilator --prefix Vt_a2_first_sc ../obj_vlt/t_a2_first_sc/Vt_a2_first_sc__main.cpp --exe --make gmake --x-assign unique -cc -Mdir obj_vlt/t_a2_first_sc -OD --debug-check --comp-limit-members 10 --debug --debugi 0 --gdbbt --no-dump-tree -sc --trace --clk clk  -f input.vc +define+TEST_OBJ_DIR=obj_vlt/t_a2_first_sc t/t_a1_first_cc.v    > obj_vlt/t_a2_first_sc/vlt_compile.log
warning: File "/home/xry/Software/verilator/test_regress/.gdbinit" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
No stack.
warning: File "/home/xry/Software/verilator/test_regress/.gdbinit" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
	add-auto-load-safe-path /home/xry/Software/verilator/test_regress/.gdbinit
line to your configuration file "/home/xry/.gdbinit".
To completely disable this security protection add
	set auto-load safe-path /
line to your configuration file "/home/xry/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
	info "(gdb)Auto-loading safe path"
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Starting Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d
Starting Verilator 4.108 2021-01-10 rev v4.108-14-g2075db3d
[Inferior 1 (process 152360) exited normally]
No stack.
	make -C obj_vlt/t_a2_first_sc -f /home/xry/Software/verilator/test_regress/Makefile_obj --no-print-directory VM_PREFIX=Vt_a2_first_sc TEST_OBJ_DIR=obj_vlt/t_a2_first_sc CPPFLAGS_DRIVER=-DT_A2_FIRST_SC  OPT_FAST=-O0 OPT_GLOBAL=-O0 Vt_a2_first_sc    > obj_vlt/t_a2_first_sc/vlt_gcc.log
driver: Entering directory '/home/xry/Software/verilator/test_regress/obj_vlt/t_a2_first_sc'
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=1 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow        -std=gnu++14 -DVERILATOR=1 -DVL_DEBUG=1 -DTEST_OBJ_DIR=obj_vlt/t_a2_first_sc -DVM_PREFIX=Vt_a2_first_sc -DVM_PREFIX_INCLUDE="<Vt_a2_first_sc.h>" -DT_A2_FIRST_SC   -O0 -c -o Vt_a2_first_sc__main.o ../../obj_vlt/t_a2_first_sc/Vt_a2_first_sc__main.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=1 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow        -std=gnu++14 -DVERILATOR=1 -DVL_DEBUG=1 -DTEST_OBJ_DIR=obj_vlt/t_a2_first_sc -DVM_PREFIX=Vt_a2_first_sc -DVM_PREFIX_INCLUDE="<Vt_a2_first_sc.h>" -DT_A2_FIRST_SC   -O0 -c -o verilated.o /home/xry/Software/verilator/include/verilated.cpp
g++      Vt_a2_first_sc__main.o verilated.o verilated_vcd_c.o verilated_vcd_sc.o Vt_a2_first_sc__ALL.a     -lsystemc -o Vt_a2_first_sc
driver: Leaving directory '/home/xry/Software/verilator/test_regress/obj_vlt/t_a2_first_sc'
	obj_vlt/t_a2_first_sc/Vt_a2_first_sc    > obj_vlt/t_a2_first_sc/vlt_sim.log
*-* All Finished *-*
- t/t_a1_first_cc.v:17: Verilog $finish
vlt/t_a2_first_sc: Self PASSED
==SUMMARY: Passed 1  Failed 0  Unsup 0  Time 0:02
==SUMMARY: Passed 1  Failed 0  Unsup 0  Time 0:02

======================================================================
TESTS DONE, PASSED: Passed 1  Failed 0  Unsup 0  Time 0:02
for p in examples/make_hello_c examples/make_hello_sc  examples/cmake_hello_c examples/cmake_hello_sc examples/cmake_protect_lib examples/cmake_tracing_c examples/cmake_tracing_sc examples/make_protect_lib examples/make_tracing_c examples/make_tracing_sc examples/xml_py ; do \
  make -C $p VERILATOR_ROOT=`pwd` || exit 10; \
done
make[1]: Entering directory '/home/xry/Software/verilator/examples/make_hello_c'
-- Verilator hello-world simple example
-- VERILATE & BUILD --------
/home/xry/Software/verilator/bin/verilator -cc --exe --build -j top.v sim_main.cpp
make[2]: Entering directory '/home/xry/Software/verilator/examples/make_hello_c/obj_dir'
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o sim_main.o ../sim_main.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o verilated.o /home/xry/Software/verilator/include/verilated.cpp
/usr/bin/perl /home/xry/Software/verilator/bin/verilator_includer -DVL_INCLUDE_OPT=include Vtop.cpp Vtop__Slow.cpp Vtop__Syms.cpp > Vtop__ALL.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o Vtop__ALL.o Vtop__ALL.cpp
Archive ar -cr Vtop__ALL.a Vtop__ALL.o
g++    sim_main.o verilated.o Vtop__ALL.a      -o Vtop
make[2]: Leaving directory '/home/xry/Software/verilator/examples/make_hello_c/obj_dir'
-- RUN ---------------------
obj_dir/Vtop
Hello World!
- top.v:11: Verilog $finish
-- DONE --------------------
Note: Once this example is understood, see examples/make_tracing_c.
Note: Also see the EXAMPLE section in the verilator manpage/document.
make[1]: Leaving directory '/home/xry/Software/verilator/examples/make_hello_c'
make[1]: Entering directory '/home/xry/Software/verilator/examples/make_hello_sc'

%Skip: SYSTEMC_INCLUDE not in environment
(If you have SystemC see the README, and rebuild Verilator)

make[1]: Leaving directory '/home/xry/Software/verilator/examples/make_hello_sc'
make[1]: Entering directory '/home/xry/Software/verilator/examples/cmake_hello_c'

-- Verilator CMake hello world example

-- VERILATE ----------------
mkdir -p build && cd build && cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Performing Test _faligned_new
-- Performing Test _faligned_new - Success
-- Performing Test _fbracket_depth_4096
-- Performing Test _fbracket_depth_4096 - Failed
-- Performing Test _fcf_protection_none
-- Performing Test _fcf_protection_none - Success
-- Performing Test _mno_cet
-- Performing Test _mno_cet - Failed
-- Performing Test _Qunused_arguments
-- Performing Test _Qunused_arguments - Failed
-- Performing Test _Wno_bool_operation
-- Performing Test _Wno_bool_operation - Success
-- Performing Test _Wno_tautological_bitwise_compare
-- Performing Test _Wno_tautological_bitwise_compare - Success
-- Performing Test _Wno_parentheses_equality
-- Performing Test _Wno_parentheses_equality - Success
-- Performing Test _Wno_sign_compare
-- Performing Test _Wno_sign_compare - Success
-- Performing Test _Wno_uninitialized
-- Performing Test _Wno_uninitialized - Success
-- Performing Test _Wno_unused_but_set_variable
-- Performing Test _Wno_unused_but_set_variable - Success
-- Performing Test _Wno_unused_parameter
-- Performing Test _Wno_unused_parameter - Success
-- Performing Test _Wno_unused_variable
-- Performing Test _Wno_unused_variable - Success
-- Performing Test _Wno_shadow
-- Performing Test _Wno_shadow - Success
-- Performing Test _mt
-- Performing Test _mt - Failed
-- Performing Test _pthread
-- Performing Test _pthread - Success
-- Performing Test _lpthread
-- Performing Test _lpthread - Success
-- Performing Test _latomic
-- Performing Test _latomic - Success
-- Executing Verilator...
-- Configuring done
-- Generating done
-- Build files have been written to: /home/xry/Software/verilator/examples/cmake_hello_c/build

-- BUILD -------------------
cmake --build build
make[2]: Entering directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
make[3]: Entering directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
Scanning dependencies of target example
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
[ 12%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/examples/make_hello_c/sim_main.cpp.o
[ 25%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop.cpp.o
[ 37%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Slow.cpp.o
[ 50%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Syms.cpp.o
[ 62%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/include/verilated.cpp.o
[ 75%] Linking CXX executable example
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
[100%] Built target example
make[3]: Leaving directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'
make[2]: Leaving directory '/home/xry/Software/verilator/examples/cmake_hello_c/build'

-- RUN ---------------------
build/example
Hello World!
- ../make_hello_c/../make_hello_c/top.v:11: Verilog $finish

-- DONE --------------------

make[1]: Leaving directory '/home/xry/Software/verilator/examples/cmake_hello_c'
make[1]: Entering directory '/home/xry/Software/verilator/examples/cmake_hello_sc'
CMake Error: Problem processing arguments. Aborting.


%Skip: CMake could not find SystemC.
% Make sure that either:
% - The environment variables SYSTEMC_INCLUDE and SYSTEMC_LIBDIR are exported.
% - Or, the environment variable SYSTEMC_ROOT is exported.
% - Or, The environment variable SYSTEMC is exported.
% - Or, if the SystemC installation provides CMake support,
%   that its installation prefix is in CMAKE_PREFIX_PATH.
% Also that the C++ standard of the SystemC library is the same as this example.
% Please see the Verilator documentation's CMake section for more information.

make[1]: Leaving directory '/home/xry/Software/verilator/examples/cmake_hello_sc'
make[1]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib'

-- Verilator CMake protect_lib example

-- VERILATE ----------------
mkdir -p build && cd build && cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Performing Test _faligned_new
-- Performing Test _faligned_new - Success
-- Performing Test _fbracket_depth_4096
-- Performing Test _fbracket_depth_4096 - Failed
-- Performing Test _fcf_protection_none
-- Performing Test _fcf_protection_none - Success
-- Performing Test _mno_cet
-- Performing Test _mno_cet - Failed
-- Performing Test _Qunused_arguments
-- Performing Test _Qunused_arguments - Failed
-- Performing Test _Wno_bool_operation
-- Performing Test _Wno_bool_operation - Success
-- Performing Test _Wno_tautological_bitwise_compare
-- Performing Test _Wno_tautological_bitwise_compare - Success
-- Performing Test _Wno_parentheses_equality
-- Performing Test _Wno_parentheses_equality - Success
-- Performing Test _Wno_sign_compare
-- Performing Test _Wno_sign_compare - Success
-- Performing Test _Wno_uninitialized
-- Performing Test _Wno_uninitialized - Success
-- Performing Test _Wno_unused_but_set_variable
-- Performing Test _Wno_unused_but_set_variable - Success
-- Performing Test _Wno_unused_parameter
-- Performing Test _Wno_unused_parameter - Success
-- Performing Test _Wno_unused_variable
-- Performing Test _Wno_unused_variable - Success
-- Performing Test _Wno_shadow
-- Performing Test _Wno_shadow - Success
-- Performing Test _mt
-- Performing Test _mt - Failed
-- Performing Test _pthread
-- Performing Test _pthread - Success
-- Performing Test _lpthread
-- Performing Test _lpthread - Success
-- Performing Test _latomic
-- Performing Test _latomic - Success
-- Executing Verilator...
-- Executing Verilator...
-- Configuring done
-- Generating done
-- Build files have been written to: /home/xry/Software/verilator/examples/cmake_protect_lib/build

-- BUILD -------------------
cmake --build build
make[2]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
make[3]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
Scanning dependencies of target verilated_secret
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
[  5%] Building CXX object CMakeFiles/verilated_secret.dir/verilated_secret/Vsecret_impl.cpp.o
[ 11%] Building CXX object CMakeFiles/verilated_secret.dir/verilated_secret/Vsecret_impl__Slow.cpp.o
[ 16%] Building CXX object CMakeFiles/verilated_secret.dir/verilated_secret/Vsecret_impl_PS0p0a.cpp.o
[ 22%] Building CXX object CMakeFiles/verilated_secret.dir/home/xry/Software/verilator/include/verilated.cpp.o
[ 27%] Building CXX object CMakeFiles/verilated_secret.dir/verilated_secret/verilated_secret.cpp.o
[ 33%] Linking CXX static library libverilated_secret.a
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
[ 44%] Built target verilated_secret
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
Scanning dependencies of target example
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
[ 50%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/examples/make_protect_lib/sim_main.cpp.o
[ 55%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop.cpp.o
[ 61%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Slow.cpp.o
[ 66%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Dpi.cpp.o
[ 72%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Syms.cpp.o
[ 77%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/include/verilated.cpp.o
[ 83%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/include/verilated_dpi.cpp.o
[ 88%] Linking CXX executable example
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
[100%] Built target example
make[3]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'
make[2]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib/build'

-- RUN ---------------------
build/example
TOP.top.secret.secret_impl: initialized
[3] cyc=0 a=0 b=0 x=2756837218
[5] cyc=1 a=5 b=7 x=9
[7] cyc=2 a=6 b=2 x=21
[9] cyc=3 a=1 b=9 x=17
[11] cyc=4 a=1 b=9 x=9
Done
- ../make_protect_lib/top.v:31: Verilog $finish

-- DONE --------------------

make[1]: Leaving directory '/home/xry/Software/verilator/examples/cmake_protect_lib'
make[1]: Entering directory '/home/xry/Software/verilator/examples/cmake_tracing_c'

-- Verilator CMake tracing example

-- VERILATE ----------------
mkdir -p build && cd build && cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Performing Test _faligned_new
-- Performing Test _faligned_new - Success
-- Performing Test _fbracket_depth_4096
-- Performing Test _fbracket_depth_4096 - Failed
-- Performing Test _fcf_protection_none
-- Performing Test _fcf_protection_none - Success
-- Performing Test _mno_cet
-- Performing Test _mno_cet - Failed
-- Performing Test _Qunused_arguments
-- Performing Test _Qunused_arguments - Failed
-- Performing Test _Wno_bool_operation
-- Performing Test _Wno_bool_operation - Success
-- Performing Test _Wno_tautological_bitwise_compare
-- Performing Test _Wno_tautological_bitwise_compare - Success
-- Performing Test _Wno_parentheses_equality
-- Performing Test _Wno_parentheses_equality - Success
-- Performing Test _Wno_sign_compare
-- Performing Test _Wno_sign_compare - Success
-- Performing Test _Wno_uninitialized
-- Performing Test _Wno_uninitialized - Success
-- Performing Test _Wno_unused_but_set_variable
-- Performing Test _Wno_unused_but_set_variable - Success
-- Performing Test _Wno_unused_parameter
-- Performing Test _Wno_unused_parameter - Success
-- Performing Test _Wno_unused_variable
-- Performing Test _Wno_unused_variable - Success
-- Performing Test _Wno_shadow
-- Performing Test _Wno_shadow - Success
-- Performing Test _mt
-- Performing Test _mt - Failed
-- Performing Test _pthread
-- Performing Test _pthread - Success
-- Performing Test _lpthread
-- Performing Test _lpthread - Success
-- Performing Test _latomic
-- Performing Test _latomic - Success
-- Executing Verilator...
-- Configuring done
-- Generating done
-- Build files have been written to: /home/xry/Software/verilator/examples/cmake_tracing_c/build

-- BUILD -------------------
cmake --build build
make[2]: Entering directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
make[3]: Entering directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
Scanning dependencies of target example
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
make[4]: Entering directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
[  8%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/examples/make_tracing_c/sim_main.cpp.o
[ 16%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop.cpp.o
[ 25%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Slow.cpp.o
[ 33%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Trace.cpp.o
[ 41%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Syms.cpp.o
[ 50%] Building CXX object CMakeFiles/example.dir/CMakeFiles/example.dir/Vtop.dir/Vtop__Trace__Slow.cpp.o
[ 58%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/include/verilated.cpp.o
[ 66%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/include/verilated_cov.cpp.o
[ 75%] Building CXX object CMakeFiles/example.dir/home/xry/Software/verilator/include/verilated_vcd_c.cpp.o
[ 83%] Linking CXX executable example
make[4]: Leaving directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
[100%] Built target example
make[3]: Leaving directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'
make[2]: Leaving directory '/home/xry/Software/verilator/examples/cmake_tracing_c/build'

-- RUN ---------------------
build/example +trace
[1] Tracing to logs/vlt_dump.vcd...

[1] Model running...

[1] clk=1 rstl=1 iquad=1234 -> oquad=1235 owide=3_22222222_11111112
[2] clk=0 rstl=0 iquad=1246 -> oquad=0 owide=0_00000000_00000000
[3] clk=1 rstl=0 iquad=1246 -> oquad=0 owide=0_00000000_00000000
[4] clk=0 rstl=0 iquad=1258 -> oquad=0 owide=0_00000000_00000000
[5] clk=1 rstl=0 iquad=1258 -> oquad=0 owide=0_00000000_00000000
[6] clk=0 rstl=0 iquad=126a -> oquad=0 owide=0_00000000_00000000
[7] clk=1 rstl=0 iquad=126a -> oquad=0 owide=0_00000000_00000000
[8] clk=0 rstl=0 iquad=127c -> oquad=0 owide=0_00000000_00000000
[9] clk=1 rstl=0 iquad=127c -> oquad=0 owide=0_00000000_00000000
[10] clk=0 rstl=1 iquad=128e -> oquad=128f owide=3_22222222_11111112
[11] clk=1 rstl=1 iquad=128e -> oquad=128f owide=3_22222222_11111112
[12] clk=0 rstl=1 iquad=12a0 -> oquad=12a1 owide=3_22222222_11111112
[13] clk=1 rstl=1 iquad=12a0 -> oquad=12a1 owide=3_22222222_11111112
[14] clk=0 rstl=1 iquad=12b2 -> oquad=12b3 owide=3_22222222_11111112
[15] clk=1 rstl=1 iquad=12b2 -> oquad=12b3 owide=3_22222222_11111112
[16] clk=0 rstl=1 iquad=12c4 -> oquad=12c5 owide=3_22222222_11111112
*-* All Finished *-*
- ../make_tracing_c/sub.v:29: Verilog $finish
[17] clk=1 rstl=1 iquad=12c4 -> oquad=12c5 owide=3_22222222_11111112

-- COVERAGE ----------------
/home/xry/Software/verilator/bin/verilator_coverage --annotate logs/annotated logs/coverage.dat
Total coverage (2/31) 6.00%
See lines with '%00' in logs/annotated

-- DONE --------------------
To see waveforms, open vlt_dump.vcd in a waveform viewer

make[1]: Leaving directory '/home/xry/Software/verilator/examples/cmake_tracing_c'
make[1]: Entering directory '/home/xry/Software/verilator/examples/cmake_tracing_sc'
CMake Error: Problem processing arguments. Aborting.


%Skip: CMake could not find SystemC.
% Make sure that either:
% - The environment variables SYSTEMC_INCLUDE and SYSTEMC_LIBDIR are exported.
% - Or, the environment variable SYSTEMC_ROOT is exported.
% - Or, The environment variable SYSTEMC is exported.
% - Or, if the SystemC installation provides CMake support,
%   that its installation prefix is in CMAKE_PREFIX_PATH.
% Also that the C++ standard of the SystemC library is the same as this example.
% Please see the Verilator documentation's CMake section for more information.

make[1]: Leaving directory '/home/xry/Software/verilator/examples/cmake_tracing_sc'
make[1]: Entering directory '/home/xry/Software/verilator/examples/make_protect_lib'

-- Verilator --protect-lib example -_--------------------------

-- VERILATE secret module -------------------------------------
 --protect-lib will produce both a static and shared library
 In this example the static library is used, but some
 simulators may require the shared library.
---------------------------------------------------------------
/home/xry/Software/verilator/bin/verilator  -cc -Os -x-assign 0 -Wall --protect-lib verilated_secret -Mdir obj_dir_secret/ secret_impl.v

-- COMPILE protected library ----------------------------------
 This builds verilated_secret.sv, libverilated_secret.a and
 libverilated_secret.so which can be distributed apart from
 the source
---------------------------------------------------------------
make -j 4 -C obj_dir_secret -f Vsecret_impl.mk
make[2]: Entering directory '/home/xry/Software/verilator/examples/make_protect_lib/obj_dir_secret'
/usr/bin/perl /home/xry/Software/verilator/bin/verilator_includer -DVL_INCLUDE_OPT=include Vsecret_impl.cpp Vsecret_impl__Slow.cpp Vsecret_impl_PSkBhp.cpp > Vsecret_impl__ALL.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow     -fPIC  -std=gnu++14 -Os -c -o verilated.o /home/xry/Software/verilator/include/verilated.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow     -fPIC  -std=gnu++14 -Os -c -o verilated_secret.o verilated_secret.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow     -fPIC  -std=gnu++14 -Os -c -o Vsecret_impl__ALL.o Vsecret_impl__ALL.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=0 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow     -fPIC  -std=gnu++14 -Os -shared -o libverilated_secret.so Vsecret_impl__ALL.o verilated.o verilated_secret.o
Archive ar -cr libverilated_secret.a Vsecret_impl__ALL.o verilated.o verilated_secret.o
make[2]: Leaving directory '/home/xry/Software/verilator/examples/make_protect_lib/obj_dir_secret'

-- VERILATE top module ----------------------------------------
 Use the SystemVerilog wrapper (verilated_secret.sv) and the
 library (libverilated_secret.a) generated from the previous
 step
---------------------------------------------------------------
/home/xry/Software/verilator/bin/verilator  -cc -Os -x-assign 0 -Wall --trace --exe -LDFLAGS '../obj_dir_secret/libverilated_secret.a' top.v obj_dir_secret/verilated_secret.sv sim_main.cpp

-- COMPILE entire design --------------------------------------
make -j 4 -C obj_dir -f Vtop.mk
make[2]: Entering directory '/home/xry/Software/verilator/examples/make_protect_lib/obj_dir'
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o sim_main.o ../sim_main.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o verilated.o /home/xry/Software/verilator/include/verilated.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o verilated_dpi.o /home/xry/Software/verilator/include/verilated_dpi.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o verilated_vcd_c.o /home/xry/Software/verilator/include/verilated_vcd_c.cpp
/usr/bin/perl /home/xry/Software/verilator/bin/verilator_includer -DVL_INCLUDE_OPT=include Vtop.cpp Vtop__Dpi.cpp Vtop__Trace.cpp Vtop__Slow.cpp Vtop__Syms.cpp Vtop__Trace__Slow.cpp > Vtop__ALL.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=0 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -Os -c -o Vtop__ALL.o Vtop__ALL.cpp
Archive ar -cr Vtop__ALL.a Vtop__ALL.o
g++    sim_main.o verilated.o verilated_dpi.o verilated_vcd_c.o Vtop__ALL.a   ../obj_dir_secret/libverilated_secret.a    -o Vtop
make[2]: Leaving directory '/home/xry/Software/verilator/examples/make_protect_lib/obj_dir'

-- RUN --------------------------------------------------------
obj_dir/Vtop +trace
Enabling waves into logs/vlt_dump.vcd...
TOP.top.secret.secret_impl: initialized
[3] cyc=0 a=0 b=0 x=1660529494
[5] cyc=1 a=5 b=7 x=9
[7] cyc=2 a=6 b=2 x=21
[9] cyc=3 a=1 b=9 x=17
[11] cyc=4 a=1 b=9 x=9
Done
- top.v:31: Verilog $finish

-- DONE -------------------------------------------------------
To see waveforms, open logs/vlt_dump.vcd in a waveform viewer

make[1]: Leaving directory '/home/xry/Software/verilator/examples/make_protect_lib'
make[1]: Entering directory '/home/xry/Software/verilator/examples/make_tracing_c'

-- Verilator tracing example

-- VERILATE ----------------
/home/xry/Software/verilator/bin/verilator  -cc --exe -Os -x-assign 0 -Wall --trace --assert --coverage -f input.vc top.v sim_main.cpp

-- BUILD -------------------
make -j -C obj_dir -f ../Makefile_obj
make[2]: Entering directory '/home/xry/Software/verilator/examples/make_tracing_c/obj_dir'
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=1 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -MMD -MP -DVL_DEBUG=1 -Os -fstrict-aliasing -c -o sim_main.o ../sim_main.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=1 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -MMD -MP -DVL_DEBUG=1 -Os -c -o verilated.o /home/xry/Software/verilator/include/verilated.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=1 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -MMD -MP -DVL_DEBUG=1 -Os -c -o verilated_cov.o /home/xry/Software/verilator/include/verilated_cov.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=1 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -MMD -MP -DVL_DEBUG=1 -Os -c -o verilated_vcd_c.o /home/xry/Software/verilator/include/verilated_vcd_c.cpp
/usr/bin/perl /home/xry/Software/verilator/bin/verilator_includer -DVL_INCLUDE_OPT=include Vtop.cpp Vtop__Trace.cpp Vtop__Slow.cpp Vtop__Syms.cpp Vtop__Trace__Slow.cpp > Vtop__ALL.cpp
ccache g++  -I.  -MMD -I/home/xry/Software/verilator/include -I/home/xry/Software/verilator/include/vltstd -DVM_COVERAGE=1 -DVM_SC=0 -DVM_TRACE=1 -DVM_TRACE_FST=0 -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow      -std=gnu++14 -MMD -MP -DVL_DEBUG=1 -Os -fstrict-aliasing -c -o Vtop__ALL.o Vtop__ALL.cpp
Archive ar -cr Vtop__ALL.a Vtop__ALL.o
g++    sim_main.o verilated.o verilated_cov.o verilated_vcd_c.o Vtop__ALL.a      -o Vtop
make[2]: Leaving directory '/home/xry/Software/verilator/examples/make_tracing_c/obj_dir'

-- RUN ---------------------
obj_dir/Vtop +trace
[1] Tracing to logs/vlt_dump.vcd...

[1] Model running...

[1] clk=1 rstl=1 iquad=1234 -> oquad=1235 owide=3_22222222_11111112
[2] clk=0 rstl=0 iquad=1246 -> oquad=0 owide=0_00000000_00000000
[3] clk=1 rstl=0 iquad=1246 -> oquad=0 owide=0_00000000_00000000
[4] clk=0 rstl=0 iquad=1258 -> oquad=0 owide=0_00000000_00000000
[5] clk=1 rstl=0 iquad=1258 -> oquad=0 owide=0_00000000_00000000
[6] clk=0 rstl=0 iquad=126a -> oquad=0 owide=0_00000000_00000000
[7] clk=1 rstl=0 iquad=126a -> oquad=0 owide=0_00000000_00000000
[8] clk=0 rstl=0 iquad=127c -> oquad=0 owide=0_00000000_00000000
[9] clk=1 rstl=0 iquad=127c -> oquad=0 owide=0_00000000_00000000
[10] clk=0 rstl=1 iquad=128e -> oquad=128f owide=3_22222222_11111112
[11] clk=1 rstl=1 iquad=128e -> oquad=128f owide=3_22222222_11111112
[12] clk=0 rstl=1 iquad=12a0 -> oquad=12a1 owide=3_22222222_11111112
[13] clk=1 rstl=1 iquad=12a0 -> oquad=12a1 owide=3_22222222_11111112
[14] clk=0 rstl=1 iquad=12b2 -> oquad=12b3 owide=3_22222222_11111112
[15] clk=1 rstl=1 iquad=12b2 -> oquad=12b3 owide=3_22222222_11111112
[16] clk=0 rstl=1 iquad=12c4 -> oquad=12c5 owide=3_22222222_11111112
*-* All Finished *-*
- sub.v:29: Verilog $finish
[17] clk=1 rstl=1 iquad=12c4 -> oquad=12c5 owide=3_22222222_11111112

-- COVERAGE ----------------
/home/xry/Software/verilator/bin/verilator_coverage --annotate logs/annotated logs/coverage.dat
Total coverage (2/31) 6.00%
See lines with '%00' in logs/annotated

-- DONE --------------------
To see waveforms, open vlt_dump.vcd in a waveform viewer

make[1]: Leaving directory '/home/xry/Software/verilator/examples/make_tracing_c'
make[1]: Entering directory '/home/xry/Software/verilator/examples/make_tracing_sc'

%Skip: SYSTEMC_INCLUDE not in environment
(If you have SystemC see the README, and rebuild Verilator)

make[1]: Leaving directory '/home/xry/Software/verilator/examples/make_tracing_sc'
make[1]: Entering directory '/home/xry/Software/verilator/examples/xml_py'
-- vl_file_copy example
python3 vl_file_copy -odir copied top.v
NOTE: vl_file_copy is only an example starting point for writing your own tool.
-- vl_hier_graph example
python3 vl_hier_graph -o graph.dot top.v
NOTE: vl_hier_graph is only an example starting point for writing your own tool.
Manually run:  dot -Tpdf -o graph.pdf graph.dot
make[1]: Leaving directory '/home/xry/Software/verilator/examples/xml_py'
Tests passed!

Now type 'make install' to install.
Or type 'make' inside an examples subdirectory.

```

