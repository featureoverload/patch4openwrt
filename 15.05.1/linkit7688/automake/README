# Patches for automake



## Makefile:50: recipe for target '/home/karthi/chaos_calmer/build_dir/host/automake-1.15/.configured' failed

```
(cd /home/karthi/chaos_calmer/build_dir/host/automake-1.15; AUTOM4TE=/home/karthi/chaos_calmer/staging_dir/host/bin/autom4te AUTOCONF=/home/karthi/chaos_calmer/staging_dir/host/bin/autoconf AUTOMAKE=/home/karthi/chaos_calmer/staging_dir/host/bin/automake ACLOCAL=/home/karthi/chaos_calmer/staging_dir/host/bin/aclocal AUTOHEADER=/home/karthi/chaos_calmer/staging_dir/host/bin/autoheader LIBTOOLIZE=/home/karthi/chaos_calmer/staging_dir/host/bin/libtoolize LIBTOOL=/home/karthi/chaos_calmer/staging_dir/host/bin/libtool M4=/home/karthi/chaos_calmer/staging_dir/host/bin/m4 AUTOPOINT=true STAGING_DIR="" ./bootstrap.sh)
Unescaped left brace in regex is illegal here in regex; marked by <-- HERE in m/${ <-- HERE ([^ \t=:+{}]+)}/ at ./bin/automake.tmp line 3938.
Makefile:50: recipe for target '/home/karthi/chaos_calmer/build_dir/host/automake-1.15/.configured' failed
make[3]: *** [/home/karthi/chaos_calmer/build_dir/host/automake-1.15/.configured] Error 255
make[3]: Leaving directory '/home/karthi/chaos_calmer/tools/automake'
tools/Makefile:122: recipe for target 'tools/automake/compile' failed
make[2]: *** [tools/automake/compile] Error 2
make[2]: Leaving directory '/home/karthi/chaos_calmer'
tools/Makefile:121: recipe for target '/home/karthi/chaos_calmer/staging_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/stamp/.tools_install_yynyynynynyyyyyyyyynyyyyyyyyynyyyyynnyyynnyynnnyy' failed
make[1]: *** [/home/karthi/chaos_calmer/staging_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2/stamp/.tools_install_yynyynynynyyyyyyyyynyyyyyyyyynyyyyynnyyynnyynnnyy] Error 2
make[1]: Leaving directory '/home/karthi/chaos_calmer'
/home/karthi/chaos_calmer/include/toplevel.mk:181: recipe for target 'world' failed
make: *** [world] Error 2
```

**Solution:**

```shell
$ cd /<path>/<to>/<openwrt-15.05.1-linkit7688>/
$ cp /<path>/<to>/patch4openwrt/15.05.1/linkit7688/automake/010-automake-port-to-Perl-5.22-and-later.patch \
     tools/automake/patches/
```



