# Patches for glib



## gdate.c:2497:7: error

```
gdate.c:2497:7: error: format not a string literal, format string not checked [-Werror=format-nonliteral]
    tmplen = strftime (tmpbuf, tmpbufsize, locale_format, &tm);
    ^
```

**Solution:**

```shell
$ cd /<path>/<to>/<openwrt-15.05.1-linkit7688>/
$ make tools/pkg-config/patches/
$ cp /<path>/<to>/patch4openwrt/15.05.1/linkit7688/glib/001-glib-gdate-suppress-string-format-literal-warning.patch \
     tools/pkg-config/patches/
```



