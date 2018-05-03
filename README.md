# Build whole programs into LLVM IR

Here is some notes for how to build the projects into a single LLVM IR bitcode file. Generally, get a integrate LLVM IR bitcode has several purposes such as whole program analysis or optimization. Chromium official homepage detailed records how to checkout and build the chromium project. To avoid unnecessary compatibility problems, here we prefer Ubuntu 16.04 (64 bit) to build the newest chromium.

build LLVM with gold plugins
--

LLVM official organization recommand users to use gold plugins to build a whole project into LLVM bitcodes. There is also some useful opensource tools to achieve this goal, e.g. wllvm (a refinement python script can be found on github). The different between gold plugins and wllvm is the former performs a real linking process with LTO (link time optimization) and the latter simply use llvm-link to connect all intermediate bitcodes in series. No matter which method be chosed, the difference in the bitcodes they produce is very small in practice.

### download and build binutils.
```sh
# some necessary pre-requisite
sudo apt install bison flex libncurses5-dev texinfo
# get the trunk version of binutils
git clone --depth 1 git://sourceware.org/git/binutils-gdb.git binutils
# create a build directory
mkdir binutils_build && mkdir binutils_install && cd binutils_build
# configuration
../binutils/configure --disable-werror --prefix=/path/to/binutils_install
# build/compile
make install
```
### build LLVM with binutils header.
```sh
# some necessary pre-requisite
sudo apt install subversion cmake zlib1g zlib1g-dev
# get the trunk version of LLVM and clang
cd /where/you/want/llvm/to/live
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
cd llvm/tools
svn co http://llvm.org/svn/llvm-project/cfe/trunk clang
cd /where/you/want/to/build/llvm
# create build and install directories
mkdir llvm_build && mkdir llvm_install && cd llvm_build
# configuration with binutils header
cmake /where/you/want/llvm/to/live -DLLVM_BINUTILS_INCDIR=/path/to/binutils/include -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=/path/to/llvm_install ../llvm
# build/compile
make -j8
```
the following cmake flags is opional :
```sh
-DLLVM_ENABLE_ASSERTIONS=On
-DLLVM_ENABLE_RTTI=On
```

### add newest binutils and newest LLVM to envirnoment variables.
```sh
vim ~/.bashrc
# use (ESC + a) to insert sentences to current file in vi/vim
export PATH=/path/to/binutils_install/bin:$PATH # add this line to bashrc
export PATH=/path/to/llvm/build/bin:$PATH       # add this line to bashrc
# use (ESC + :wq) to quit vi/vim
```
then type the command to shell
```sh
source ~/.bashrc
```
to evaluate the envirnoment variables, reopen shell is also workful.

### test (optional)

we use sed as a benchmark to test whether the gold plugins work correctly in LLVM.
```sh
# check the envirnoment variables are correct.
clang --version
binutils --version
# switch to workspace where to build sed.
cd /path/to/workspace
# create a build directory.
mkdir sed_build && cd sed_build
# configuration
/path/to/sed/source/configure CC=clang LDFLAGS='-flto -fuse-ld=gold -Wl,-plugin-opt=save-temps'
# build/compile
make
```
If everything is ok, sed.0.0.preopt.bc can be found under the build directory. (sepecifically in /path/to/sed_build/sed)

