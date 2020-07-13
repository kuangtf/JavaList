# 打包指令： tar

```
[dmtsai@study ~]$ tar [-z&#124;-j&#124;-J] [cv] [-f 待创建的新文件名] filename... &lt;==打包与压缩
[dmtsai@study ~]$ tar [-z&#124;-j&#124;-J] [tv] [-f 既有的 tar文件名] &lt;==察看文件名
[dmtsai@study ~]$ tar [-z&#124;-j&#124;-J] [xv] [-f 既有的 tar文件名] [-C 目录] &lt;==解压缩
选项与参数：
-c ：创建打包文件，可搭配 -v 来察看过程中被打包的文件名（filename）
-t ：察看打包文件的内容含有哪些文件名，重点在察看“文件名”就是了；
-x ：解打包或解压缩的功能，可以搭配 -C （大写） 在特定目录解开
特别留意的是， -c, -t, -x 不可同时出现在一串命令行中。
-z ：通过 gzip 的支持进行压缩/解压缩：此时文件名最好为 *.tar.gz
-j ：通过 bzip2 的支持进行压缩/解压缩：此时文件名最好为 *.tar.bz2
-J ：通过 xz 的支持进行压缩/解压缩：此时文件名最好为 *.tar.xz
特别留意， -z, -j, -J 不可以同时出现在一串命令行中
-v ：在压缩/解压缩的过程中，将正在处理的文件名显示出来！
-f filename：-f 后面要立刻接要被处理的文件名！建议 -f 单独写一个选项啰！（比较不会忘记）
-C 目录 ：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
其他后续练习会使用到的选项介绍：
-p（小写） ：保留备份数据的原本权限与属性，常用于备份（-c）重要的配置文件
-P（大写） ：保留绝对路径，亦即允许备份数据中含有根目录存在之意；
--exclude=FILE：在压缩的过程中，不要将 FILE 打包！
```

- [ ] 最简单的使用 tar 就只要记忆下面的方式即可：

- 压　缩：tar -j<u>c</u>v -f filename.tar.bz2 要被压缩的文件或目录名称
- 查　询：tar -j<u>t</u>v -f filename.tar.bz2
- 解压缩：tar -j<u>x</u>v -f filename.tar.bz2 -C 欲解压缩的目录

> tar 并不会主动的产生创建的文件名。
>
> 1. 如果不加 [-z|-j|-J] 的话，文件名最好取为 .tar 即可。
>
> 2. 如果是 -j 选项，代表有 bzip2 的支持，文件名最好就取为 .tar.bz2 。
> 3. 如果是加上了 -z 的 gzip 的支持，那文件名最好取为 *.tar.gz 。

- 使用 tar 加入 -z, -j 或 -J 的参数备份 /etc/ 目录

```
[dmtsai@study ~]$ su - # 因为备份 /etc 需要 root 的权限，否则会出现一堆错误
[root@study ~]# time tar -zpcv -f /root/etc.tar.gz /etc
tar: Removing leading `/' from member names &lt;==注意这个警告讯息
/etc/
....（中间省略）....
/etc/hostname
/etc/aliases.db
real 0m0.799s # 多了 time 会显示程序运行的时间！看 real 就好了！花去了 0.799s
user 0m0.767s
sys 0m0.046s
# 由于加上 -v 这个选项，因此正在作用中的文件名就会显示在屏幕上。
# 如果你可以翻到第一页，会发现出现上面的错误讯息！下面会讲解。
# 至于 -p 的选项，重点在于“保留原本文件的权限与属性”之意。
[root@study ~]# time tar -jpcv -f /root/etc.tar.bz2 /etc
....（前面省略）....
real 0m1.913s
user 0m1.881s
sys 0m0.038s
[root@study ~]# time tar -Jpcv -f /root/etc.tar.xz /etc
....（前面省略）....
real 0m9.023s
user 0m8.984s
sys 0m0.086s
# 显示的讯息会跟上面一模一样啰！不过时间会花比较多！使用了 -J 时，会花更多时间
[root@study ~]# ll /root/etc*
-rw-r--r--. 1 root root 6721809 Jul 1 00:16 /root/etc.tar.bz2
-rw-r--r--. 1 root root 7758826 Jul 1 00:14 /root/etc.tar.gz
-rw-r--r--. 1 root root 5511500 Jul 1 00:16 /root/etc.tar.xz
[root@study ~]# du -sm /etc
28 /etc # 实际目录约占有 28MB 的意思！
```

- 查阅 tar 文件的数据内容 （可察看文件名），与备份文件名有否根目录的意义。

要察看由 tar 所创建的打包文件内部的文件名：

```
[root@study ~]# tar -jtv -f /root/etc.tar.bz2
....（前面省略）....
-rw-r--r-- root/root 131 2015-05-25 17:48 etc/locale.conf
-rw-r--r-- root/root 19 2015-05-04 17:56 etc/hostname
-rw-r--r-- root/root 12288 2015-05-04 17:59 etc/aliases.db
```

> 如果加上 -v 这个选项时，详细的文件权限/属性都会被列出来！如果只是想要知道文件名而已， 那么就将 -v 拿掉即可.

- 将备份的数据解压缩，并考虑特定目录的解压缩动作 （-C 选项的应用）

```
[root@study ~]# tar -jxv -f /root/etc.tar.bz2
[root@study ~]# ll
....（前面省略）....
drwxr-xr-x. 131 root root 8192 Jun 26 22:14 etc
....（后面省略）....
```

- 仅解开单一文件的方法:使用 -jtv 找到你要的文件名，然后将该文件名解开即可。

```
# 1\. 先找到我们要的文件名，假设解开 shadow 文件好了：
[root@study ~]# tar -jtv -f /root/etc.tar.bz2 &#124; grep 'shadow'
---------- root/root 721 2015-06-17 00:20 etc/gshadow
---------- root/root 1183 2015-06-17 00:20 etc/shadow-
---------- root/root 1210 2015-06-17 00:20 etc/shadow &lt;==这是我们要的！
---------- root/root 707 2015-06-17 00:20 etc/gshadow-
# 先搜寻重要的文件名！其中那个 grep 是“撷取”关键字的功能！我们会在第三篇说明！
# 这里您先有个概念即可！那个管线 &#124; 配合 grep 可以撷取关键字的意思！
# 2\. 将该文件解开！语法与实际作法如下：
[root@study ~]# tar -jxv -f 打包档.tar.bz2 待解开文件名
[root@study ~]# tar -jxv -f /root/etc.tar.bz2 etc/shadow
etc/shadow
[root@study ~]# ll etc
total 4
----------. 1 root root 1210 Jun 17 00:20 shadow
# 很有趣！此时只会解开一个文件而已！不过，重点是那个文件名！你要找到正确的文件名。
# 在本例中，你不能写成 /etc/shadow ！因为记录在 etc.tar.bz2 内的并没有 / 之故！
```

- 打包某目录，但不含该目录下的某些文件之作法

```
[root@study ~]# tar -jcv -f /root/system.tar.bz2 --exclude=/root/etc* \
&gt; --exclude=/root/system.tar.bz2 /etc /root
```

> exclude 就是不包含的意思。如果想要两行输入时，最后面加上反斜线 （\） 并立刻按下 [enter] ， 就能够到第二行继续输入了。

- 仅备份比某个时刻还要新的文件

一个是“ --newer ”另一个就是“ --newer-mtime ；当使用 --newer 时，表示后续的日期包含“ mtime 与 ctime ”，而 --newer-mtime 则仅是 mtime 而已！

```
# 1\. 先由 [find](../Text/index.html#find) 找出比 /etc/passwd 还要新的文件
[root@study ~]# find /etc -newer /etc/passwd
....（过程省略）....
# 此时会显示出比 /etc/passwd 这个文件的 mtime 还要新的文件名，
# 这个结果在每部主机都不相同！您先自行查阅自己的主机即可，不会跟鸟哥一样！
[root@study ~]# ll /etc/passwd
-rw-r--r--. 1 root root 2092 Jun 17 00:20 /etc/passwd
# 2\. 好了，那么使用 tar 来进行打包吧！日期为上面看到的 2015/06/17
[root@study ~]# tar -jcv -f /root/etc.newer.then.passwd.tar.bz2 \
&gt; --newer-mtime="2015/06/17" /etc/*
tar: Option --newer-mtime: Treating date `2015/06/17' as 2015-06-17 00:00:00
tar: Removing leading `/' from member names
/etc/abrt/
....（中间省略）....
/etc/alsa/
/etc/yum.repos.d/
....（中间省略）....
tar: /etc/yum.repos.d/CentOS-fasttrack.repo: file is unchanged; not dumped
# 最后行显示的是“没有被备份的”，亦即 not dumped 的意思！
# 3\. 显示出文件即可
[root@study ~]# tar -jtv -f /root/etc.newer.then.passwd.tar.bz2 &#124; grep -v '/$'
# 通过这个指令可以调用出 tar.bz2 内的结尾非 / 的文件名！就是我们要的啦！
```

- 基本名称： tarfile, tarball ？

1. 如果仅是打包而已，就是“ tar -cv -f file.tar ”而已，这个文件我们称呼为 tarfile。
2. 如果还有进行压缩的支持，例如“ tar -jcv -f file.tar.bz2 ”时，我们就称呼为 tarball （tar 球？）

> tar 除了可以将数据打包成为文件之外，还能够将文件打包到某些特别的设备去，举例来说， 磁带机 （tape）。