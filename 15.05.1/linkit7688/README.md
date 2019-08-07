@[toc](Overview)

<br/>

------

# Build the firmware from a source code

因为我使用的是 Ubuntu 18 编译。所以出现了不少需要打 patch 修复编译的问题。修复的方式都在后文给出。

> Reference:
>
> [Build the firmware from a source code](https://docs.labs.mediatek.com/resource/linkit-smart-7688/zh_cn/tutorials/firmware-and-bootloader/build-the-firmware-from-source-codes)

This section describes how to build the firmware for LinkIt Smart 7688 and LinkIt Smart 7688 Duo development boards from a source code.

<br/>

## Environment

The following operations are performed under a Ubuntu LTS 14.04.3 environment. For a Windows or a Mac OS X host computer, you can install a virtual machine to have the same environment: 

- Download Ubuntu 14.04.3 LTS image from [http://www.ubuntu.com](http://www.ubuntu.com/)
- Install this image with VirtualBox ([http://virtualbox.org](http://virtualbox.org/)) on the host machine. 50GB disk space reserved for the virtual machine is recommended.


<br/>

------

## Steps

In the Ubuntu system, open the **Terminal** application and enter the following commands:

1. Install prerequisite packages to build the firmware:

   

   ```shell
   $ sudo apt-get install git g++ libncurses5-dev subversion libssl-dev gawk libxml-parser-perl unzip  
   ```

2. Download the OpenWrt CC source codes:
   ~~`$ git clone git://git.openwrt.org/15.05/openwrt.git`~~
   
   ```shell
   $ git clone https://github.com/openwrt/chaos_calmer.git linkit-7688
   $ cd linkit-7688
   linkit-7688$ git tag -l
   linkit-7688$ git checkout tags/v15.05.1 -b linkit7688
   linkit-7688$ 
   ```
   
   > Reference: [Download a specific tag with Git](https://stackoverflow.com/questions/791959/download-a-specific-tag-with-git)
   
3. Prepare the default configuration file for feeds:

	```shell
	$ ##cd openwrt
	linkit-7688$ cp feeds.conf.default feeds.conf
	```
  
4. Add the LinkIt Smart 7688 development board's feed: 

	```shell
	linkit-7688$ echo src-git linkit https://github.com/MediaTek-Labs/linkit-smart-7688-feed.git >> feeds.conf  
	```
  
5. Update the feed information for all available packages to build the firmware:

	```shell
	linkit-7688$ ./scripts/feeds update
	```
  
   > check [./scripts/feeds update log](#./scripts/feeds update log) for more information.

6. Install all packages:

   > Bug Fix:
   > 将 `clone 2>&1 | grep -- --recursive` 替换为 `version`
   > ```shell
   > $ cp include/prereq-build.mk include/prereq-build.mk.backup
   > $ vim include/prereq-build.mk
   > [...]
   > $(eval $(call SetupHostCommand,git,Please install Git (git-core) >= 1.6.5, \
   >    git version))
   > 
   > :wq
   > $ git diff
   > diff --git a/include/prereq-build.mk b/include/prereq-build.mk
   > index 211201af3d..bf9895629b 100644
   > --- a/include/prereq-build.mk
   > +++ b/include/prereq-build.mk
   > @@ -145,7 +145,7 @@ $(eval $(call SetupHostCommand,svn,Please install the Subversion client, \
   >   svn --version | grep Subversion))
   > 
   > 
   > $(eval $(call SetupHostCommand,git,Please install Git (git-core) >= 1.6.5, \
   > -       git clone 2>&1 | grep -- --recursive))
   > +       git version))
   > 
   > $(eval $(call SetupHostCommand,file,Please install the 'file' package, \
   >   file --version 2>&1 | grep file))
   > 
   > ```
   >
   > > Reference:
   > >
   > > [OpenWrt源码下载及固件编译](https://blog.csdn.net/mojie_babyno1/article/details/81135039)

   ```shell
   linkit-7688$ ./scripts/feeds install -a
   ```

   > check [./scripts/feeds install -a](#./scripts/feeds install -a) for more information.

7. Prepare the kernel configuration: 

   ```shell
   $ make menuconfig
   ```

   > 必须使用 v15.05.1 才有 MT7688（v15.05 没有 MT7688！）。

   - Select the following options:

     - Target System: Ralink RT288x/RT3xxx
     - Subtarget: MT7688 based boards
     - Target Profile: LinkIt7688

	- Save and exit (use the default configuration file without any modification)
   
	> 参见 [make menuconfig log](#make menuconfig log) for more detail.
   
8. Start the compilation process: 

	```shell
	$ make V=99
	```

   > 会下载 ***.tar.gz 到 `<path>/<to>/openwrt(or linkit7688)/dl/`

9. After the build process is finished, the resulted firmware file will be under "*bin/ramips/openwrt-ramips-mt7688-LinkIt7688-squashfs-sysupgrade.bin*". Depending on the hardware resources of the host environment, the build process may take more than 2 hours.

10. You can use this file to update the firmware through Web UI or rename it to *lks7688.img* to update through a USB drive.


<br/>

------

## Fix Compiler Fail

`make V=99` issue, use patch to resolve:

1. [glib-gdate](glib)
	
	```
	gdate.c:2497:7: error: format not a string literal, format string not checked [-Werror=format-nonliteral]
	    tmplen = strftime (tmpbuf, tmpbufsize, locale_format, &tm);
	    ^
	```
	
	**Solution:**
	
	```shell
	$ make tools/pkg-config/patches/
	$ vim tools/pkg-config/patches/001-glib-gdate-suppress-string-format-literal-warning.patch
	--- a/glib/glib/gdate.c
	+++ b/glib/glib/gdate.c
	@@ -2439,6 +2439,9 @@ win32_strftime_helper (const GDate     *d,
	  *
	  * Returns: number of characters written to the buffer, or 0 the buffer was too small
	  */
	+#pragma GCC diagnostic push
	+#pragma GCC diagnostic ignored "-Wformat-nonliteral"
	+
	 gsize     
	 g_date_strftime (gchar       *s, 
	                  gsize        slen, 
	@@ -2549,3 +2552,5 @@ g_date_strftime (gchar       *s,
	   return retval;
	 #endif
	 }
	+
	+#pragma GCC diagnostic pop
	:wq
	$ make V=99
	```

	> Reference:
	>
	> 1. https://github.com/hak5/wifipineapple-openwrt/blob/master/tools/pkg-config/patches/001-glib-gdate-suppress-string-format-literal-warning.patch
	> 2. https://blog.csdn.net/fan1234500/article/details/79892981
	> 3. https://blog.csdn.net/zmlovelx/article/details/81664043



2. [automake-1.5](automake)

	**Solution:**
	
	```shell
	$ vim tools/automake/patches/010-automake-port-to-Perl-5.22-and-later.patch
	--- a/bin/automake.in
	+++ b/bin/automake.in
	@@ -3878,7 +3878,7 @@ sub substitute_ac_subst_variables_worker
	sub substitute_ac_subst_variables
	{
	 my ($text) = @_;
	-  $text =~ s/\${([^ \t=:+{}]+)}/substitute_ac_subst_variables_worker ($1)/ge;
	+  $text =~ s/\$[{]([^ \t=:+{}]+)}/substitute_ac_subst_variables_worker ($1)/ge;
	 return $text;
	}
	
	:wq
	$
	```
	
	
	>  Reference:
	>
	> [FS#930 - branches fail to build (automake)](https://bugs.openwrt.org/index.php?do=details&task_id=930&opened=169&status[0]=)
	>
	> [[Solved\] Compile error on 15.05 release in gdate.c](https://forum.openwrt.org/t/solved-compile-error-on-15-05-release-in-gdate-c/11624)
	>
	> [linkit-smart-7688-feed > Issue #50](https://github.com/MediaTek-Labs/linkit-smart-7688-feed/issues/50)



3. [u-boot > gcc7.h](u-boot)
	
	```
	include/linux/compiler-gcc.h:114:1: fatal error: linux/compiler-gcc7.h: No such file or directory
	```

	**Solution:**
	
	```shell
	$ #mkdir tools/include/patches/
	$ vim tools/mkimage/patches/0001-Add-GCC7-supportation.patch
	--- /dev/null
	+++ b/include/linux/compiler-gcc7.h
	@@ -0,0 +1,65 @@
	+#ifndef __LINUX_COMPILER_H
	+#error "Please don't include <linux/compiler-gcc7.h> directly, include <linux/compiler.h> instead."
	+#endif
	+
	+#define __used				__attribute__((__used__))
	+#define __must_check			__attribute__((warn_unused_result))
	+#define __compiler_offsetof(a, b)	__builtin_offsetof(a, b)
	+
	+/* Mark functions as cold. gcc will assume any path leading to a call
	+   to them will be unlikely.  This means a lot of manual unlikely()s
	+   are unnecessary now for any paths leading to the usual suspects
	+   like BUG(), printk(), panic() etc. [but let's keep them for now for
	+   older compilers]
	+
	+   Early snapshots of gcc 4.3 don't support this and we can't detect this
	+   in the preprocessor, but we can live with this because they're unreleased.
	+   Maketime probing would be overkill here.
	+
	+   gcc also has a __attribute__((__hot__)) to move hot functions into
	+   a special section, but I don't see any sense in this right now in
	+   the kernel context */
	+#define __cold			__attribute__((__cold__))
	+
	+#define __UNIQUE_ID(prefix) __PASTE(__PASTE(__UNIQUE_ID_, prefix), __COUNTER__)
	+
	+#ifndef __CHECKER__
	+# define __compiletime_warning(message) __attribute__((warning(message)))
	+# define __compiletime_error(message) __attribute__((error(message)))
	+#endif /* __CHECKER__ */
	+
	+/*
	+ * Mark a position in code as unreachable.  This can be used to
	+ * suppress control flow warnings after asm blocks that transfer
	+ * control elsewhere.
	+ *
	+ * Early snapshots of gcc 4.5 don't support this and we can't detect
	+ * this in the preprocessor, but we can live with this because they're
	+ * unreleased.  Really, we need to have autoconf for the kernel.
	+ */
	+#define unreachable() __builtin_unreachable()
	+
	+/* Mark a function definition as prohibited from being cloned. */
	+#define __noclone	__attribute__((__noclone__))
	+
	+/*
	+ * Tell the optimizer that something else uses this function or variable.
	+ */
	+#define __visible __attribute__((externally_visible))
	+
	+/*
	+ * GCC 'asm goto' miscompiles certain code sequences:
	+ *
	+ *   http://gcc.gnu.org/bugzilla/show_bug.cgi?id=58670
	+ *
	+ * Work it around via a compiler barrier quirk suggested by Jakub Jelinek.
	+ *
	+ * (asm goto is automatically volatile - the naming reflects this.)
	+ */
	+#define asm_volatile_goto(x...)	do { asm goto(x); asm (""); } while (0)
	+
	+#ifdef CONFIG_ARCH_USE_BUILTIN_BSWAP
	+#define __HAVE_BUILTIN_BSWAP32__
	+#define __HAVE_BUILTIN_BSWAP64__
	+#define __HAVE_BUILTIN_BSWAP16__
	+#endif /* CONFIG_ARCH_USE_BUILTIN_BSWAP */
	
	:wq
	$
	```
	
	> Reference:
	>
	> [u-boot build fail rocko - odroidc2 #9](https://github.com/superna9999/meta-meson/issues/9)
	>
	> [Fix u-boot compile failed using GCC7 #10](https://github.com/superna9999/meta-meson/pull/10/commits/28e9c5e5e1b3f7cc4da05bdbea8ed7fd3ac5c0c4) > [0001-Add-GCC7-supportation.patch](https://github.com/nvl1109/meta-meson/blob/28e9c5e5e1b3f7cc4da05bdbea8ed7fd3ac5c0c4/recipes-bsp/u-boot/u-boot-odroidc2/0001-Add-GCC7-supportation.patch)


4. [rsa-sign.c](rsa)

	```
	build_dir/host/u-boot-2014.10/lib/rsa/rsa-sign.c:279:21: error: dereferencing pointer to incomplete type ‘RSA {aka struct rsa_st}’
	if (BN_num_bits(key->e) > 64)
	                   ^~
	scripts/Makefile.host:134: recipe for target 'tools/lib/rsa/rsa-sign.o' failed
	make[5]: *** [tools/lib/rsa/rsa-sign.o] Error 1
	Makefile:1195: recipe for target 'tools-only' failed
	make[4]: *** [tools-only] Error 2
	```

	**Solution:**

	```shell
	$ sudo apt install libssl1.0-dev
	```

	reference: [lede/openwrt does not compile with OpenSSL 1.1 #973](https://github.com/freifunk-gluon/gluon/issues/973#issuecomment-265911151)



5. [gcc-linaro-4.8-2014.04/gcc](gcc-linaro-4.8-2014.04)
	
	> Reference:
	> 1. ~~[Unable to build GCC due to c++11 errors](https://stackoverflow.com/questions/41204632/unable-to-build-gcc-due-to-c11-errors)~~
	> 2. ~~[patch](https://gcc.gnu.org/git/?p=gcc.git;a=commitdiff;h=ec1cc0263f156f70693a62cf17b254a0029f4852)~~
	> 3. ~~[常见编译问题,不定期更新](https://yjdwbj.github.io/2016/08/26/%E5%B8%B8%E8%A7%81%E7%BC%96%E8%AF%91%E9%97%AE%E9%A2%98-%E4%B8%8D%E5%AE%9A%E6%9C%9F%E6%9B%B4%E6%96%B0/)~~
	> 4. [Build fix for GCC 5 #1](https://github.com/axilirator/gnu-arm-installer/pull/1) > [Merge pull request #1](https://github.com/axilirator/gnu-arm-installer/commit/11a9b412b32c91c05f9496ae9b9924cc52668385) > [fix-gcc5-build.patch](https://github.com/axilirator/gnu-arm-installer/blob/11a9b412b32c91c05f9496ae9b9924cc52668385/fix-gcc5-build.patch)
	> 5. [patch-gcc_cp_cfns.h](https://drive.google.com/file/d/0BwWNLQDwiOxtYnJSRm1Dam9XTU0/view)
	
	手动修改 `./build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/gcc-linaro-4.8-2014.04/gcc/cp/cfns.h` 文件 -- 根据 4. fix-gcc5-build.patch 修改。
暂时不知道应该将 `.patch` 放在哪个路径下（所以目前只能手动修改）。



6. [feeds/packages/lang/node](node)
	```
	make[3]: Entering directory '/<path>/<to>/feeds/packages/lang/node'
	(cd /<path>/<to>/build_dir/host/node-v0.12.7/; if [ -x configure ]; then cp -fpR /<path>/<to>/scripts/config.{guess,sub} /<path>/<to>/build_dir/host/node-v0.12.7// &&  python ./configure  --dest-os=linux --without-snapshot --prefix=/<path>/<to>/staging_dir/host/ ; fi )
	Traceback (most recent call last):
	  File "./configure", line 992, in <module>
	    configure_node(output)
	  File "./configure", line 530, in configure_node
	    o['variables']['gcc_version'] = 10 * cc_version[0] + cc_version[1]
	IndexError: tuple index out of range
	Makefile:72: recipe for target '/<path>/<to>/build_dir/host/node-v0.12.7/.configured' failed
	make[3]: *** [/<path>/<to>/build_dir/host/node-v0.12.7/.configured] Error 1
	make[3]: Leaving directory '/<path>/<to>/feeds/packages/lang/node'
	package/Makefile:191: recipe for target 'package/feeds/packages/node/host/compile' failed
	make[2]: *** [package/feeds/packages/node/host/compile] Error 2
	make[2]: Leaving directory '/<path>/<to>/'
	package/Makefile:188: recipe for target '/<path>/<to>/staging_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/stamp/.package_compile' failed
	make[1]: *** [/<path>/<to>/staging_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/stamp/.package_compile] Error 2
	make[1]: Leaving directory '/<path>/<to>'
	/<path>/<to>/include/toplevel.mk:181: recipe for target 'world' failed
	make: *** [world] Error 2
	```

	Reference: [gcc 5.x detection in configure #25578](https://github.com/nodejs/node-v0.x-archive/issues/25578)
	**Solution:**
	```shell
	$ vim feeds/packages/lang/node/patches/0001-gcc5.x-detection-in-configure.patch
	ndex: node-v0.12.5/configure
	===================================================================
	--- node-v0.12.5.orig/configure
	+++ node-v0.12.5/configure
	@@ -479,6 +479,14 @@ def compiler_version():
	                           stdout=subprocess.PIPE)
	   version = tuple(map(int, proc.communicate()[0].split('.')))
	
	+  # gcc-5 returns "(5,)", but later cc_version requires at least 2 parameters
	+  # existing in version tuple.
	+  if len(version) < 2 :
	+    proc = subprocess.Popen(shlex.split(CC) + ['--version'],
	+                          stdout=subprocess.PIPE)
	+    minor_version = int(proc.communicate()[0].split('.')[1])
	+    version = (version[0],minor_version)
	+
	   return (version, is_clang)
	
	:wq
	$
	```


<br/>

------

## 附

### ./scripts/feeds update log



```
Updating feed 'packages' from 'https://github.com/openwrt/packages.git;for-15.05' ...
正克隆到 './feeds/packages'...
remote: Enumerating objects: 2716, done.
remote: Counting objects: 100% (2716/2716), done.
remote: Compressing objects: 100% (2289/2289), done.
remote: Total 2716 (delta 105), reused 2213 (delta 68), pack-reused 0
接收对象中: 100% (2716/2716), 1.72 MiB | 175.00 KiB/s, 完成.
处理 delta 中: 100% (105/105), 完成.
Create index file './feeds/packages.index' 
Collecting package info: done
Collecting target info: done
Updating feed 'luci' from 'https://github.com/openwrt/luci.git;for-15.05' ...
正克隆到 './feeds/luci'...
remote: Enumerating objects: 4136, done.
remote: Counting objects: 100% (4136/4136), done.
remote: Compressing objects: 100% (2225/2225), done.
remote: Total 4136 (delta 973), reused 2933 (delta 490), pack-reused 0
接收对象中: 100% (4136/4136), 3.66 MiB | 681.00 KiB/s, 完成.
处理 delta 中: 100% (973/973), 完成.
Create index file './feeds/luci.index' 
Collecting package info: done
Collecting target info: done
Updating feed 'routing' from 'https://github.com/openwrt-routing/packages.git;for-15.05' ...
正克隆到 './feeds/routing'...
remote: Enumerating objects: 379, done.
remote: Counting objects: 100% (379/379), done.
remote: Compressing objects: 100% (299/299), done.
remote: Total 379 (delta 27), reused 279 (delta 16), pack-reused 0
接收对象中: 100% (379/379), 255.30 KiB | 284.00 KiB/s, 完成.
处理 delta 中: 100% (27/27), 完成.
Create index file './feeds/routing.index' 
Collecting package info: done
Collecting target info: done
Updating feed 'telephony' from 'https://github.com/openwrt/telephony.git;for-15.05' ...
正克隆到 './feeds/telephony'...
remote: Enumerating objects: 187, done.
remote: Counting objects: 100% (187/187), done.
remote: Compressing objects: 100% (169/169), done.
remote: Total 187 (delta 18), reused 113 (delta 3), pack-reused 0
接收对象中: 100% (187/187), 143.95 KiB | 267.00 KiB/s, 完成.
处理 delta 中: 100% (18/18), 完成.
Create index file './feeds/telephony.index' 
Collecting package info: done
Collecting target info: done
Updating feed 'management' from 'https://github.com/openwrt-management/packages.git;for-15.05' ...
正克隆到 './feeds/management'...
remote: Enumerating objects: 36, done.
remote: Counting objects: 100% (36/36), done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 36 (delta 5), reused 24 (delta 1), pack-reused 0
展开对象中: 100% (36/36), 完成.
Create index file './feeds/management.index' 
Collecting package info: done
Collecting target info: done
Updating feed 'linkit' from 'https://github.com/MediaTek-Labs/linkit-smart-7688-feed.git' ...
正克隆到 './feeds/linkit'...
remote: Enumerating objects: 87, done.
remote: Counting objects: 100% (87/87), done.
remote: Compressing objects: 100% (57/57), done.
remote: Total 87 (delta 9), reused 72 (delta 8), pack-reused 0
展开对象中: 100% (87/87), 完成.
Create index file './feeds/linkit.index' 
Collecting package info: done
Collecting target info: done
```



### ./scripts/feeds install -a

```
Checking 'working-make'... ok.
Checking 'case-sensitive-fs'... ok.
Checking 'gcc'... ok.
Checking 'working-gcc'... ok.
Checking 'g++'... ok.
Checking 'working-g++'... ok.
Checking 'ncurses'... ok.
Checking 'zlib'... ok.
Checking 'libssl'... ok.
Checking 'tar'... ok.
Checking 'find'... ok.
Checking 'bash'... ok.
Checking 'patch'... ok.
Checking 'diff'... ok.
Checking 'cp'... ok.
Checking 'seq'... ok.
Checking 'awk'... ok.
Checking 'grep'... ok.
Checking 'getopt'... ok.
Checking 'stat'... ok.
Checking 'md5sum'... ok.
Checking 'unzip'... ok.
Checking 'bzip2'... ok.
Checking 'wget'... ok.
Checking 'perl'... ok.
Checking 'python'... ok.
Checking 'svn'... ok.
Checking 'git'... failed.
Checking 'file'... ok.
Checking 'openssl'... ok.
Checking 'ldconfig-stub'... ok.

Build dependency: Please install Git (git-core) >= 1.6.5

/<path>/<to>/linkit-7688/include/prereq.mk:12: recipe for target 'prereq' failed
Prerequisite check failed. Use FORCE=1 to override.
/<path>/<to>/openwrt/linkit-7688/include/toplevel.mk:140: recipe for target 'staging_dir/host/.prereq-build' failed
make[1]: *** [staging_dir/host/.prereq-build] Error 1
/<path>/<to>/openwrt/linkit-7688/include/toplevel.mk:69: recipe for target 'prepare-tmpinfo' failed
make: *** [prepare-tmpinfo] Error 2
Cannot open './tmp/.packageinfo': No such file or directory
Can't open file './tmp/.targetinfo': No such file or directory
Installing all packages from feed packages.
Installing package 'acl'
WARNING: No feed for package 'libc' found, maybe it's already part of the standard packages?
WARNING: No feed for package 'libssp' found, maybe it's already part of the standard packages?
WARNING: ...
...

############### after bug fix, run it again:   ####################
Checking 'working-make'... ok.
Checking 'case-sensitive-fs'... ok.
Checking 'gcc'... ok.
Checking 'working-gcc'... ok.
Checking 'g++'... ok.
Checking 'working-g++'... ok.
Checking 'ncurses'... ok.
Checking 'zlib'... ok.
Checking 'libssl'... ok.
Checking 'tar'... ok.
Checking 'find'... ok.
Checking 'bash'... ok.
Checking 'patch'... ok.
Checking 'diff'... ok.
Checking 'cp'... ok.
Checking 'seq'... ok.
Checking 'awk'... ok.
Checking 'grep'... ok.
Checking 'getopt'... ok.
Checking 'stat'... ok.
Checking 'md5sum'... ok.
Checking 'unzip'... ok.
Checking 'bzip2'... ok.
Checking 'wget'... ok.
Checking 'perl'... ok.
Checking 'python'... ok.
Checking 'svn'... ok.
Checking 'git'... ok.
Checking 'file'... ok.
Checking 'openssl'... ok.
Checking 'ldconfig-stub'... ok.
Collecting package info: done
Collecting target info: done
Installing all packages from feed packages.
Installing all packages from feed luci.
Installing all packages from feed routing.
Installing all packages from feed telephony.
Installing all packages from feed management.
Installing all packages from feed linkit.
```



### make menuconfig log

```
$ make menuconfig


*** End of the configuration.
*** Execute 'make' to start the build or try 'make help'.
```