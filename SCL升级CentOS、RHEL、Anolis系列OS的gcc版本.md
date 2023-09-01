# SCL升级CentOS/RHEL/Anolis系列OS的gcc版本

`CentOS 7`、`BigCloud 7` 以及`Anolis 7`等系列`OS`的默认`gcc`版本为`4.8.5`，有时编译程序或者驱动源码需要更高版本的`gcc`，本文以升级至`8.3.1`版本为例，仅仅需要分别执行下面三条命令即可，无需手动[下载源码](http://ftp.gnu.org/gnu/gcc/)编译。

## 一、查看默认gcc版本

分别查看`CentOS 7`、`BigCloud 7 `以及`Anolis 7`等系列`OS`的默认`gcc`版本，查询结果如下：

```shell
#CentOS 7.9平台gcc默认版本
[wz@localhost ~]$ cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
[wz@localhost ~]$ 
[wz@localhost ~]$ gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 

#BigCloud 7.6平台gcc默认版本
[root@localhost ~]# cat /etc/redhat-release
BigCloud Enterprise Linux release 7.6.1905 (Core)
[root@localhost ~]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)

#Anolis 7.9平台gcc默认版本
[root@localhost ~]# cat /etc/anolis-release
Anolis OS release 7.9
[root@localhost ~]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/aarch64-redhat-linux/4.8.5/lto-wrapper
目标：aarch64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=https://bugs.openanolis.cn --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-aarch64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-aarch64-redhat-linux/cloog-install --enable-gnu-indirect-function --build=aarch64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Anolis 4.8.5-44.0.1) (GCC)
```

## 二、升级gcc版本

网上有很多博客会教我们通过`gcc` 源码进行编译，随后进行安装，从而升级到我们想要的版本，但是这种方法小编也进行了尝试，结果在我的机器上始终编译报错，而且根本不知道问题出在哪里，这种方式实施的过程中在不同的平台及机器可能遇见各种各样的问题，比较麻烦且容易出错。

因此，本文我们采用`CentOS`的一个第三方库`SCL`(软件选集)来进行`gcc`版本的升级，`SCL`可以在不覆盖原系统软件包的情况下安装新的软件包与老软件包共存并且可以使用`scl`命令切换，不过也有个缺点就是只支持64位的。

[Software collections](https://www.softwarecollections.org/)（SCLs）是一个Linux软件多版本共存的解决方案，适用于`RHEL/CentOS/Fedora`。SCL不修改已安装的软件版本，也不会与其产生冲突。`SCL`的创建就是为了给 `RHEL/CentOS` 用户提供一种以方便、安全地安装和使用应用程序和运行时环境的多个（而且可能是更新的）版本的方式，同时避免把系统搞乱。

### 1、安装scl源

首先，要解决的第一个问题就是 **yum** 源的问题。CentOS 7 最晚在**2024年6月30后**停止更新维护，在此之前在 CentOS 7 可以通过 **yum** 直接安装 **SCL** 源基本都是可以正常使用的。

```textile
[root@localhost ~]# rpm -ivh https://cbs.centos.org/kojifiles/packages/centos-release-scl-rh/2/3.el7.centos/noarch/centos-release-scl-rh-2-3.el7.centos.noarch.rpm
获取https://cbs.centos.org/kojifiles/packages/centos-release-scl-rh/2/3.el7.centos/noarch/centos-release-scl-rh-2-3.el7.centos.noarch.rpm
准备中...                          ################################# [100%]
正在升级/安装...
   1:centos-release-scl-rh-2-3.el7.cen################################# [100%]
[root@localhost ~]# rpm -ivh https://cbs.centos.org/kojifiles/packages/centos-release-scl/2/3.el7.centos/noarch/centos-release-scl-2-3.el7.centos.noarch.rpm
获取https://cbs.centos.org/kojifiles/packages/centos-release-scl/2/3.el7.centos/noarch/centos-release-scl-2-3.el7.centos.noarch.rpm
准备中...                          ################################# [100%]
正在升级/安装...
   1:centos-release-scl-2-3.el7.centos################################# [100%]
```

安装完成后，会默认在 **/etc/yum.repos.d** 下生成 2 个 repo 源文件：

```textile
[root@localhost ~]# ls -l /etc/yum.repos.d/ | grep -i scl
-rw-r--r--  1 root root  998 12月 11 2018 CentOS-SCLo-scl.repo
-rw-r--r--  1 root root  971 10月 29 2018 CentOS-SCLo-scl-rh.repo
```

- **CentOS-SCLo-scl.repo**

```textile
# CentOS-SCLo-sclo.repo
#
# Please see http://wiki.centos.org/SpecialInterestGroup/SCLo for more
# information

[centos-sclo-sclo]
name=CentOS-7 - SCLo sclo
# baseurl=http://mirror.centos.org/centos/7/sclo/$basearch/sclo/
mirrorlist=http://mirrorlist.centos.org?arch=$basearch&release=7&repo=sclo-sclo
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[centos-sclo-sclo-testing]
name=CentOS-7 - SCLo sclo Testing
baseurl=http://buildlogs.centos.org/centos/7/sclo/$basearch/sclo/
gpgcheck=0
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[centos-sclo-sclo-source]
name=CentOS-7 - SCLo sclo Sources
baseurl=http://vault.centos.org/centos/7/sclo/Source/sclo/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[centos-sclo-sclo-debuginfo]
name=CentOS-7 - SCLo sclo Debuginfo
baseurl=http://debuginfo.centos.org/centos/7/sclo/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
```

- **CentOS-SCLo-scl-rh.repo**

```textile
# CentOS-SCLo-rh.repo
#
# Please see http://wiki.centos.org/SpecialInterestGroup/SCLo for more
# information

[centos-sclo-rh]
name=CentOS-7 - SCLo rh
#baseurl=http://mirror.centos.org/centos/7/sclo/$basearch/rh/
mirrorlist=http://mirrorlist.centos.org?arch=$basearch&release=7&repo=sclo-rh
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[centos-sclo-rh-testing]
name=CentOS-7 - SCLo rh Testing
baseurl=http://buildlogs.centos.org/centos/7/sclo/$basearch/rh/
gpgcheck=0
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[centos-sclo-rh-source]
name=CentOS-7 - SCLo rh Sources
baseurl=http://vault.centos.org/centos/7/sclo/Source/rh/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[centos-sclo-rh-debuginfo]
name=CentOS-7 - SCLo rh Debuginfo
baseurl=http://debuginfo.centos.org/centos/7/sclo/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
```

随后，更新 yum 源的缓存：

```textile
[root@localhost ~]# yum clean all
[root@localhost ~]# yum makecache 
```

### 2、安装centos-release-scl与centos-release-scl-rh

```textile
[root@localhost ~]# yum install centos-release-scl centos-release-scl-rh
```

### 3、安装devtoolset工具

`devtoolset` 本身也有不同的版本，因为我们想要将`gcc` 升级到`8.3.1`，所以这里我们安装的是 `devtoolset-8-gcc`。

```textile
[root@localhost ~]# yum -y install devtoolset-8-gcc devtoolset-8-gcc-c++ devtoolset-8-binutils
```

如果想安装`7.x`版本的，就改成`devtoolset-7-gcc`，以此类推：

```
[root@localhost ~]# yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
```

全部文件都会被安装在`/opt/rh/`目录下，如下：

```textile
[root@localhost ~]# ls -l /opt/rh/
总用量 0
dr-xr-xr-x 3 root root 32 8月  31 14:23 devtoolset-7
dr-xr-xr-x 3 root root 32 8月  31 14:26 devtoolset-8
```

另外，我们可以用`SCL`来管理软件集，如安装`gcc`，首先查看可安装的版本：

```textile
[root@localhost ~]#  yum list dev*gcc
已加载插件：fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * centos-sclo-rh: mirrors.bfsu.edu.cn
 * centos-sclo-sclo: mirrors.bfsu.edu.cn
 * epel: mirrors.bfsu.edu.cn
已安装的软件包
devtoolset-7-gcc.aarch64                                                                          7.3.1-5.16.el7                                                                          @centos-sclo-rh
devtoolset-8-gcc.aarch64                                                                          8.3.1-3.2.el7                                                                           @centos-sclo-rh
可安装的软件包
devtoolset-10-gcc.aarch64                                                                         10.2.1-11.2.el7                                                                         centos-sclo-rh
devtoolset-9-gcc.aarch64                                                                          9.3.1-2.2.el7                                                                           centos-sclo-rh
```

随后，即可用选择合适的版本安装。

### 4、激活devtoolset

你可以一次安装多个版本的`devtoolset`，需要的时候用下面这条命令切换到对应的版本，激活其运行环境：

```
[root@localhost ~]# scl enable devtoolset-8 bash
```

> **注意**：使用exit 退出当前scl版本的bash环境。

大功告成，查看一下gcc版本：

```
[root@localhost ~]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/opt/rh/devtoolset-8/root/usr/libexec/gcc/aarch64-redhat-linux/8/lto-wrapper
Target: aarch64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefix=/opt/rh/devtoolset-8/root/usr --mandir=/opt/rh/devtoolset-8/root/usr/share/man --infodir=/opt/rh/devtoolset-8/root/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --with-default-libstdcxx-abi=gcc4-compatible --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-8.3.1-20190311/obj-aarch64-redhat-linux/isl-install --disable-libmpx --enable-gnu-indirect-function --build=aarch64-redhat-linux
Thread model: posix
gcc version 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC)
```

> 注意：这条激活命令只对本次会话有效，重启会话后还是会变回原来的`4.8.5`版本。`scl`命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。

由前面可知，安装的`devtoolset`是在 `/opt/rh` 目录下的，而其每个版本的目录下面都有个 `enable` 文件，该文件是一个`ASCII`文本文件，如下：

```
[root@localhost ~]# tree -L 2 /opt/rh/devtoolset-8/
/opt/rh/devtoolset-8/
├── enable
└── root
    ├── bin -> usr/bin
    ├── boot
    ├── dev
    ├── etc
    ├── home
    ├── lib -> usr/lib
    ├── lib64 -> usr/lib64
    ├── media
    ├── mnt
    ├── opt
    ├── proc
    ├── root
    ├── run
    ├── sbin -> usr/sbin
    ├── srv
    ├── sys
    ├── tmp
    ├── usr
    └── var

20 directories, 1 file
   
[root@localhost ~]# file /opt/rh/devtoolset-8/enable
/opt/rh/devtoolset-8/enable: ASCII text
```

如果需要启用某个版本，只需要执行如下命令：

```
[root@localhost devtoolset-8]# source ./enable
```

紧接着，让我们在当前会话中，执行`gcc -v`命令查看其版本，如下：

```
[root@localhost devtoolset-8]# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/opt/rh/devtoolset-8/root/usr/libexec/gcc/aarch64-redhat-linux/8/lto-wrapper
Target: aarch64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefix=/opt/rh/devtoolset-8/root/usr --mandir=/opt/rh/devtoolset-8/root/usr/share/man --infodir=/opt/rh/devtoolset-8/root/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --with-default-libstdcxx-abi=gcc4-compatible --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-8.3.1-20190311/obj-aarch64-redhat-linux/isl-install --disable-libmpx --enable-gnu-indirect-function --build=aarch64-redhat-linux
Thread model: posix
gcc version 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC)
```

根据上面结果可知，`gcc`版本为`8.3.1`， `devtoolset`已切换成功。所以要想切换到某个`X`版本，只需要执行：

```
source /opt/rh/devtoolset-X/enable
```

可以将对应版本的切换命令写个`shell`文件放在配了环境变量的目录下，需要时随时切换，或者开机自启。

当然，也可以配置`bashrc`或`/etc/profile`使环境自动生效：

```
echo "source /opt/rh/devtoolset-8/enable" >> ~/.bashrc

#或

echo "source /opt/rh/devtoolset-8/enable" >> /etc/profile
```

> **拓展**：旧的gcc是运行的 /usr/bin/gcc，所以也可以将该目录下的gcc/g++替换为刚安装的新版本gcc软连接，免得每次enable。如下：
> 
> ```
> mv /usr/bin/gcc /usr/bin/gcc-4.8.5
> 
> ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc
> 
> mv /usr/bin/g++ /usr/bin/g++-4.8.5
> 
> ln -s /opt/rh/devtoolset-8/root/bin/g++ /usr/bin/g++
> 
> gcc --version
> 
> g++ --version
> ```

### 5、卸载

可能大家用完开发工具集后就会想要删除它，删除方法很简单，输入以下命令：

```
[root@localhost ~]# yum remove devtoolset-7*
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 devtoolset-7-binutils.aarch64.0.2.28-11.el7 将被 删除
---> 软件包 devtoolset-7-gcc.aarch64.0.7.3.1-5.16.el7 将被 删除
---> 软件包 devtoolset-7-gcc-c++.aarch64.0.7.3.1-5.16.el7 将被 删除
---> 软件包 devtoolset-7-libstdc++-devel.aarch64.0.7.3.1-5.16.el7 将被 删除
---> 软件包 devtoolset-7-runtime.aarch64.0.7.1-4.el7 将被 删除
--> 解决依赖关系完成

依赖关系解决

=========================================================================================================================================================================================================
 Package                                                     架构                                   版本                                            源                                              大小
=========================================================================================================================================================================================================
正在删除:
 devtoolset-7-binutils                                       aarch64                                2.28-11.el7                                     @centos-sclo-rh                                 24 M
 devtoolset-7-gcc                                            aarch64                                7.3.1-5.16.el7                                  @centos-sclo-rh                                 48 M
 devtoolset-7-gcc-c++                                        aarch64                                7.3.1-5.16.el7                                  @centos-sclo-rh                                 20 M
 devtoolset-7-libstdc++-devel                                aarch64                                7.3.1-5.16.el7                                  @centos-sclo-rh                                 17 M
 devtoolset-7-runtime                                        aarch64                                7.1-4.el7                                       @centos-sclo-rh                                3.2 k

事务概要
=========================================================================================================================================================================================================
移除  5 软件包

安装大小：110 M
是否继续？[y/N]：Y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : devtoolset-7-gcc-c++-7.3.1-5.16.el7.aarch64                                                                                                                                          1/5
  正在删除    : devtoolset-7-libstdc++-devel-7.3.1-5.16.el7.aarch64                                                                                                                                  2/5
  正在删除    : devtoolset-7-gcc-7.3.1-5.16.el7.aarch64                                                                                                                                              3/5
  正在删除    : devtoolset-7-binutils-2.28-11.el7.aarch64                                                                                                                                            4/5
  正在删除    : devtoolset-7-runtime-7.1-4.el7.aarch64                                                                                                                                               5/5
  验证中      : devtoolset-7-libstdc++-devel-7.3.1-5.16.el7.aarch64                                                                                                                                  1/5
  验证中      : devtoolset-7-gcc-c++-7.3.1-5.16.el7.aarch64                                                                                                                                          2/5
  验证中      : devtoolset-7-gcc-7.3.1-5.16.el7.aarch64                                                                                                                                              3/5
  验证中      : devtoolset-7-runtime-7.1-4.el7.aarch64                                                                                                                                               4/5
  验证中      : devtoolset-7-binutils-2.28-11.el7.aarch64                                                                                                                                            5/5

删除:
  devtoolset-7-binutils.aarch64 0:2.28-11.el7    devtoolset-7-gcc.aarch64 0:7.3.1-5.16.el7    devtoolset-7-gcc-c++.aarch64 0:7.3.1-5.16.el7    devtoolset-7-libstdc++-devel.aarch64 0:7.3.1-5.16.el7
  devtoolset-7-runtime.aarch64 0:7.1-4.el7

完毕！
```

## 三、总结

不同平台的编译环境不一样，所以 `RedHat` 就推出了 `SCL (Software Collections)` ，它可以根据 `devtoolset` 一起配合来快速统一开发环境，不用一个个的去找各个官网再去编译源码安装。使用 `SCL` 可以暂时的改变当前用户的编译工具，例如你的系统版本 `gcc 4.8.5`，但是你可以使用 `SCL` 工具临时把你的 `gcc` 版本提升到 `8.3.1`。

其实，简单的来说，`devtoolset` 就是 `SCL` 提供的一套专门用于 `CentOS` 或 `Red Hat Enterprise Linux` 平台编译开发的一套工具集。


