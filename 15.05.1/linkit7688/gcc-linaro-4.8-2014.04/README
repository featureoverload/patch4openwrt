# Patches for gcc-linaro-4.8-2014.04



## gcc

### cfns.gperf:101:1: error: ‘gnu_inline’ attribute present on ‘libc_name_p’

```
In file included from /mnt/mobile/osmocom/toolchain/axilirator/src/gcc-4.5.2/gcc/cp/except.c:927:0:
cfns.gperf: At top level:
cfns.gperf:101:1: error: ‘gnu_inline’ attribute present on ‘libc_name_p’
cfns.gperf:26:14: error: but not here
Makefile:1047: recipe for target 'cp/except.o' failed
make[1]: *** [cp/except.o] Error 1
make[1]: Leaving directory '/mnt/mobile/osmocom/toolchain/axilirator/build/gcc-4.5.2/gcc'
Makefile:5024: recipe for target 'all-gcc' failed
make: *** [all-gcc] Error 2
```

**Solution:**

> Reference:
> 
> 1. ~~[Unable to build GCC due to c++11 errors](https://stackoverflow.com/questions/41204632/unable-to-build-gcc-due-to-c11-errors)~~
> 2. ~~[patch](https://gcc.gnu.org/git/?p=gcc.git;a=commitdiff;h=ec1cc0263f156f70693a62cf17b254a0029f4852)~~
> 3. ~~常见编译问题,不定期更新~~
> 4. [Build fix for GCC 5 #1](https://github.com/axilirator/gnu-arm-installer/pull/1) > [Merge pull request #1](https://github.com/axilirator/gnu-arm-installer/commit/11a9b412b32c91c05f9496ae9b9924cc52668385) > [fix-gcc5-build.patch](https://github.com/axilirator/gnu-arm-installer/blob/11a9b412b32c91c05f9496ae9b9924cc52668385/fix-gcc5-build.patch)
> 5. [patch-gcc_cp_cfns.h](https://drive.google.com/file/d/0BwWNLQDwiOxtYnJSRm1Dam9XTU0/view)

手动修改 ./build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/gcc-linaro-4.8-2014.04/gcc/cp/cfns.h 文件 – 根据 ***4. fix-gcc5-build.patch*** 修改。

暂时不知道应该将 .patch 放在哪个路径下（所以目前只能手动修改）。



