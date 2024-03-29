+++
title = "FreeBSD: Updating Gettext Breaks Vim and Svn"
date = "2012-07-27T02:12:00+08:00"
type = "post"
tags = ["FreeBSD"]
+++

在一台FreeBSD 8.0-RELEASE上把`git`跟`gitolite`裝起來後，發現`vim`跟`svn`壞掉，
```bash
$ vim
/libexec/ld-elf.so.1: Shared object "libintl.so.8" not found, required by "vim"
$ svn
/libexec/ld-elf.so.1: Shared object "libintl.so.8" not found, required by "svn"
```

google後發現，因為`gettext`升級後，把`/usr/local/lib/libintl.so.8`升到`
/usr/local/lib/libintl.so.9`，而vim跟svn會使用到`libintl.so.8`，所以出了問題
```bash
$ ldconfig -r | grep "libintl"
112:-lintl.9 => /usr/local/lib/libintl.so.9
```

解決的方法就是把有影響到的ports重裝一次，讓他們depend到正確的library，但要如何
找出受影響的ports呢？
```bash
$ pkg_info -Rx gettext
Information for gettext-0.18.1.1:

Required by:
gtk-1.2.10_21
libgcrypt-1.4.4
libgpg-error-1.7
libxslt-1.1.26
neon28-0.28.6
py26-subversion-1.6.9
pysvn-1.7.2
subversion-1.6.9
gmake-3.82
python26-2.6.8_3
git-1.7.11.3
gitolite-3.03
p5-Locale-gettext-1.05_3
help2man-1.40.10
vim-7.3.556_1
```

<strike>p.s. 並不是所有的ports都depend到libintl，所以可以先測試一下binary是否可以正常執行</strike>

*Updated: 2012/10/11*

最近有人把一些機器上的python從2.6升到2.7，愈到了類似的狀況，
libpython2.6 => libpython2.7，沒想到這位仁兄竟然直接建soft link指過去...!@#$%&(，
看到都快吐血了。突然想到我有寫過這篇，於是問了朋友，想知道有沒有更確實的方式可
以找到哪些ports會被depend，一開始想到用`ldd`
```bash
$ ldd `which vim`
/usr/local/bin/vim:
    ...
    libpython2.6.so => /usr/local/lib/libpython2.6.so (0x800f9d000)
    ...
```

但這樣頗麻煩的，還要一個一個找出binary或library的位置，在朋友的指點下，翻了翻
`/usr/ports/UPDATING`，發現gettext版本升級時，UPDATING中就有提出處理的方法...XD
```bash
$ less /usr/ports/UPDATING
...
20100530:
AFFECTS: users of devel/gettext (i.e.: YOU)
AUTHOR: ade@FreeBSD.org

Another version of gettext (0.18), and another shared library version
bump (from intl.8 to intl.9), so:

All ports that have an identifiable known direct dependency on gettext
have had their PORTREVISIONs bumped.  If after upgrading:

    # portupgrade -rf gettext
    # portmaster -w -r gettext
    ...
```

用portmaster把depend gettext的ports全部重編一次就ok了，不過這樣會不會又升爛其他
東西阿...XD
