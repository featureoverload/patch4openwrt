# Patches for node



## configure: IndexError: tuple index out of range

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
```

**Solution:**

```shell
$ cd /<path>/<to>/<openwrt-15.05.1-linkit7688>/
$ cp /<path>/<to>/patch4openwrt/15.05.1/linkit7688/node/0001-gcc5.x-detection-in-configure.patch \
     feeds/packages/lang/node/patches/
```



