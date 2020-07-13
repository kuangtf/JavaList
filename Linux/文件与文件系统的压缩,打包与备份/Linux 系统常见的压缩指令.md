# Linux 系统常见的压缩指令

## 一、gzip, zcat/zmore/zless/zgrep

- 常见的压缩文件扩展名：

```
*.Z compress 程序压缩的文件；
*.zip zip 程序压缩的文件；
*.gz gzip 程序压缩的文件；
*.bz2 bzip2 程序压缩的文件；
*.xz xz 程序压缩的文件；
*.tar tar 程序打包的数据，并没有压缩过；
*.tar.gz tar 程序打包的文件，其中并且经过 gzip 的压缩
*.tar.bz2 tar 程序打包的文件，其中并且经过 bzip2 的压缩
*.tar.xz tar 程序打包的文件，其中并且经过 xz 的压缩
```

- `gzip` 可以说是应用度最广的压缩指令，`gzip` 所创建的压缩文件为 `*.gz` 的文件名

```
[dmtsai@study ~]$ gzip [-cdtv#] 文件名
[dmtsai@study ~]$ zcat 文件名.gz
选项与参数：
-c ：将压缩的数据输出到屏幕上，可通过数据流重导向来处理；
-d ：解压缩的参数；
-t ：可以用来检验一个压缩文件的一致性～看看文件有无错误；
-v ：可以显示出原文件/压缩文件的压缩比等信息；
-# ：# 为数字的意思，代表压缩等级，-1 最快，但是压缩比最差、-9 最慢，但是压缩比最好！默认是 -6
范例一：找出 /etc 下面 （不含子目录） 容量最大的文件，并将它复制到 /tmp ，然后以 gzip 压缩
[dmtsai@study ~]$ ls -ldSr /etc/* # 忘记选项意义？请自行 man 啰！
.....（前面省略）.....
-rw-r--r--. 1 root root 25213 Jun 10 2014 /etc/dnsmasq.conf
-rw-r--r--. 1 root root 69768 May 4 17:55 /etc/ld.so.cache
-rw-r--r--. 1 root root 670293 Jun 7 2013 /etc/services
[dmtsai@study ~]$ cd /tmp
[dmtsai@study tmp]$ cp /etc/services .
[dmtsai@study tmp]$ gzip -v services
services: 79.7% -- replaced with services.gz
[dmtsai@study tmp]$ ll /etc/services /tmp/services*
-rw-r--r--. 1 root root 670293 Jun 7 2013 /etc/services
-rw-r--r--. 1 dmtsai dmtsai 136088 Jun 30 18:40 /tmp/services.gz
```

```
范例二：由于 services 是文本文件，请将范例一的压缩文件的内容读出来！
[dmtsai@study tmp]$ zcat services.gz
# 由于 services 这个原本的文件是是文本文件，因此我们可以尝试使用 zcat/zmore/zless 去读取！
# 此时屏幕上会显示 servcies.gz 解压缩之后的原始文件内容！
范例三：将范例一的文件解压缩
[dmtsai@study tmp]$ gzip -d services.gz
# 鸟哥不要使用 gunzip 这个指令，不好背！使用 gzip -d 来进行解压缩！
# 与 gzip 相反， gzip -d 会将原本的 .gz 删除，回复到原本的 services 文件。
范例四：将范例三解开的 services 用最佳的压缩比压缩，并保留原本的文件
[dmtsai@study tmp]$ gzip -9 -c services &gt; services.gz
范例五：由范例四再次创建的 services.gz 中，找出 http 这个关键字在哪几行？
[dmtsai@study tmp]$ zgrep -n 'http' services.gz
14:# http://www.iana.org/assignments/port-numbers
89:http 80/tcp www www-http # WorldWideWeb HTTP
90:http 80/udp www www-http # HyperText Transfer Protocol
.....（下面省略）.....
```

- `cat/more/less` 可以使用不同的方式来读取纯文本文件，那个 `zcat/zmore/zless` 则可以对应于`cat/more/less` 的方式来读取纯文本文件被压缩后的压缩文件。另外，如果你还想要从文字压缩文件当中找数据的话，可以通过 `egrep` 来搜寻关键字喔！而不需要将压缩文件解开才以 `grep` 进行。



## 二、bzip2, bzcat/bzmore/bzless/bzgrep

- `bzip2` 则是为了取代`gzip` 并提供更佳的压缩比而来的。

```
[dmtsai@study ~]$ bzip2 [-cdkzv#] 文件名
[dmtsai@study ~]$ bzcat 文件名.bz2
选项与参数：
-c ：将压缩的过程产生的数据输出到屏幕上！
-d ：解压缩的参数
-k ：保留原始文件，而不会删除原始的文件喔！
-z ：压缩的参数 （默认值，可以不加）
-v ：可以显示出原文件/压缩文件的压缩比等信息；
-# ：与 gzip 同样的，都是在计算压缩比的参数， -9 最佳， -1 最快！
范例一：将刚刚 gzip 范例留下来的 /tmp/services 以 bzip2 压缩
[dmtsai@study tmp]$ bzip2 -v services
services: 5.409:1, 1.479 bits/Byte, 81.51% saved, 670293 in, 123932 out.
[dmtsai@study tmp]$ ls -l services*
-rw-r--r--. 1 dmtsai dmtsai 123932 Jun 30 18:40 services.bz2
-rw-rw-r--. 1 dmtsai dmtsai 135489 Jun 30 18:46 services.gz
# 此时 services 会变成 services.bz2 之外，你也可以发现 bzip2 的压缩比要较 gzip 好喔！！
# 压缩率由 gzip 的 79% 提升到 bzip2 的 81% 哩！
范例二：将范例一的文件内容读出来！
[dmtsai@study tmp]$ bzcat services.bz2
范例三：将范例一的文件解压缩
[dmtsai@study tmp]$ bzip2 -d services.bz2
范例四：将范例三解开的 services 用最佳的压缩比压缩，并保留原本的文件
[dmtsai@study tmp]$ bzip2 -9 -c services &gt; services.bz2
```

## 三、xz, xzcat/xzmore/xzless/xzgrep

- `bzip2` 已经具有很棒的压缩比，不过显然某些自由软件开发者还不满足，因此后来还推出了 `xz` 这个压缩比更高的软件！

```
[dmtsai@study ~]$ xz [-dtlkc#] 文件名
[dmtsai@study ~]$ xcat 文件名.xz
选项与参数：
-d ：就是解压缩啊！
-t ：测试压缩文件的完整性，看有没有错误
-l ：列出压缩文件的相关信息
-k ：保留原本的文件不删除～
-c ：同样的，就是将数据由屏幕上输出的意思！
-# ：同样的，也有较佳的压缩比的意思！
范例一：将刚刚由 bzip2 所遗留下来的 /tmp/services 通过 xz 来压缩！
[dmtsai@study tmp]$ xz -v services
services （1/1）
100 % 97.3 KiB / 654.6 KiB = 0.149
[dmtsai@study tmp]$ ls -l services*
-rw-rw-r--. 1 dmtsai dmtsai 123932 Jun 30 19:09 services.bz2
-rw-rw-r--. 1 dmtsai dmtsai 135489 Jun 30 18:46 services.gz
-rw-r--r--. 1 dmtsai dmtsai 99608 Jun 30 18:40 services.xz
# 各位观众！看到没有啊！！容量又进一步下降的更多耶！好棒的压缩比！
范例二：列出这个压缩文件的信息，然后读出这个压缩文件的内容
[dmtsai@study tmp]$ xz -l services.xz
Strms Blocks Compressed Uncompressed Ratio Check Filename
1 1 97.3 KiB 654.6 KiB 0.149 CRC64 services.xz
# 竟然可以列出这个文件的压缩前后的容量，真是太人性化了！这样观察就方便多了！
[dmtsai@study tmp]$ xzcat services.xz
范例三：将他解压缩吧！
[dmtsai@study tmp]$ xz -d services.xz
范例四：保留原文件的文件名，并且创建压缩文件！
[dmtsai@study tmp]$ xz -k services
```

> `xz`运算时间的比 `gzip` 久很多。