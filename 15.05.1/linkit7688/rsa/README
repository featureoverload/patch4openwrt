# Patches for rsa



## rsa-sign.c:279:21: error

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



