# 认识Linux文件系统

## 一、磁盘组成与分区

- Linux 最传统的磁盘文件系统 （filesystem） 使用的是 EXT2 。
- 整颗磁盘的组成主要有：

```
* 圆形的盘片（主要记录数据的部分）；
* 机械手臂，与在机械手臂上的磁头（可读写盘片上的数据）；
* 主轴马达，可以转动盘片，让机械手臂的磁头在盘片上读写数据。
```

- 盘片上的物理组成则为（假设此磁盘为单碟片)

```
* 扇区（Sector）为最小的物理储存单位，且依据磁盘设计的不同，目前主要有 512Bytes
  与 4K 两种格式；
* 将扇区组成一个圆，那就是柱面（Cylinder）；
* 早期的分区主要以柱面为最小分区单位，现在的分区通常使用扇区为最小分区单位（每个扇区都有其号码喔，就好   像座位一样）；
* 磁盘分区表主要有两种格式，一种是限制较多的 MBR 分区表，一种是较新且限制较少的GPT 分区表。
* MBR 分区表中，第一个扇区最重要，里面有：（1）主要开机区（Master boot record,MBR）及分区表    （partition table）， 其中 MBR 占有 446 Bytes，而 partition table 则占有 64 Bytes。
* GPT 分区表除了分区数量扩充较多之外，支持的磁盘容量也可以超过 2TB。
```

- /dev/sd[a-p][1-128]：为实体磁盘的磁盘文件名；
- /dev/vd[a-d][1-128]：为虚拟磁盘的磁盘文件名。

## 二、文件系统特性

- 磁盘分区完毕后还需要进行格式化（format），之后操作系统才能够使用这个文件系统。 为什么需要进行“格式化”呢？

> 因为每种操作系统所设置的文件属性/权限并不相同， 为了存放这些文件所需的数据，因此就需要将分区进行格式化，以成为操作系统能够利用的“文件系统格式（filesystem）”。

- [ ] 文件系统通常会将文件权限（rwx）、文件属性（拥有者、群组、时间参数等）与文件实际内容分别存放在不同的区块，权限与属性放置到 inode 中，至于实际数据则放置到 data block 区块中。

- superblock：记录此 filesystem 的整体信息，包括inode/block的总量、使用量、剩余量，以及文件系统的格式与相关信息等；
- inode：记录文件的属性，一个文件占用一个inode，同时记录此文件的数据所在的 block号码；
- block：实际记录文件的内容，若文件太大时，会占用多个 block 。

> 由于每个 inode 与 block 都有编号，而每个文件都会占用一个 inode ，inode 内则有文件数据放置的 block 号码。，如果能够找到文件的 inode 的话，那么自然就会知道这个文件所放置数据的 block 号码， 当然也就能够读出该文件的实际数据了。

![image](https://github.com/ktf-cool/JavaList/blob/master/images/%E6%95%B0%E6%8D%AE%E5%AD%98%E5%8F%96%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

> 这种数据存取的方法我们称为索引式文件系统（indexed allocation）。

![image](https://github.com/ktf-cool/JavaList/blob/master/images/FAT%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%95%B0%E6%8D%AE%E5%AD%98%E5%8F%96%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

> U盘使用的文件系统一般为 FAT 格式。FAT 这种格式的文件系统并没有 inode 存在，所以 FAT 没有办法将这个文件的所有 block 在一开始就读取出来。每个 block 号码都记录在前一个 block 当中， 他的读取方式有点像上面这样。但这个文件系统没有办法一口气就知道四个 block 的号码，他得要一个一个的将 block 读出后，才会知道下一个block 在何处。



## 三、Linux 的 EXT2 文件系统（inode）

- 标准的Linux 文件系统 Ext2 就是使用这种 inode 为基础的文件系统。

- 文件系统一开始就将 inode 与 block 规划好了，除非重新格式化（或者利用 resize2fs 等指令变更文件系统大小），否则 inode 与 block 固定后就不再变动。

- Ext2 文件系统在格式化的时候基本上是区分为多个区块群组 （block group） 的，每个区块群组都有独立的 inode/block/superblock 系统。

- Ext2 格式化后有点像下面这样：

![image](https://github.com/ktf-cool/JavaList/blob/master/images/ext2%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

> 在整体的规划当中，文件系统最前面有一个开机扇区（boot sector），这个开机扇区可以安装开机管理程序，能够将不同的开机管理程序安装到个别的文件系统最前端，而不用覆盖整颗磁盘唯一的 MBR， 这样也才能够制作出多重开机的环境。

- data block （数据区块）

data block 是用来放置文件内容数据地方，在 Ext2 文件系统中所支持的 block 大小有 1K, 2K及 4K 三种而已。在格式化时 block 的大小就固定了，且每个 block 都有编号，以方便 inode的记录。由于block 大小的差异，会导致该文件系统能够支持的最大磁盘容量与最大单一文件大小并不相同。

| Block 大小         | 1KB  | 2KB   | 4KB  |
| ------------------ | ---- | ----- | ---- |
| 最大单一文件限制   | 16GB | 256GB | 2TB  |
| 最大文件系统总容量 | 2TB  | 8TB   | 16TB |

- Ext2 文件系统的 block 的基本限制：

>原则上，block 的大小与数量在格式化完就不能够再改变了（除非重新格式化）；
>
>每个 block 内最多只能够放置一个文件的数据；
>
>承上，如果文件大于 block 的大小，则一个文件会占用多个 block 数量；
>
>承上，若文件小于 block ，则该 block 的剩余容量就不能够再被使用了（磁盘空间会浪费）。



- inode table （inode 表格）

1. inode 记录的文件数据至少有下面这些：

```
* 该文件的存取模式（read/write/excute）；
* 该文件的拥有者与群组（owner/group）；
* 该文件的容量；
* 该文件创建或状态改变的时间（ctime）；
* 最近一次的读取时间（atime）；
* 最近修改的时间（mtime）；
* 定义文件特性的旗标（flag），如 SetUID...；
* 该文件真正内容的指向 （pointer）；
```

2. inode的特色：

```
* 每个 inode 大小均固定为 128 Bytes （新的 ext4 与 xfs 可设置到 256 Bytes）；
* 每个文件都仅会占用一个 inode 而已；
* 承上，因此文件系统能够创建的文件数量与 inode 的数量有关；
* 系统读取文件时需要先找到 inode，并分析 inode 所记录的权限与使用者是否符合，若符
  合才能够开始实际读取 block 的内容。
```



- Superblock （超级区块）

Superblock 是记录整个 filesystem 相关信息的地方：

```
* block 与 inode 的总量；
* 未使用与已使用的 inode / block 数量；
* block 与 inode 的大小 （block 为 1, 2, 4K，inode 为 128Bytes 或 256Bytes）；
* filesystem 的挂载时间、最近一次写入数据的时间、最近一次检验磁盘 （fsck） 的时间
  等文件系统的相关信息；
* 一个 valid bit 数值，若此文件系统已被挂载，则 valid bit 为 0 ，若未被挂载，则 valid bit
  为 1 。
```

```
[root@study ~]# dumpe2fs [-bh] 设备文件名
选项与参数：
-b ：列出保留为坏轨的部分（一般用不到吧！？）
-h ：仅列出 superblock 的数据，不会列出其他的区段内容！
范例：鸟哥的一块 1GB ext4 文件系统内容
[root@study ~]# blkid &lt;==这个指令可以叫出目前系统有被格式化的设备
/dev/vda1: LABEL="myboot" UUID="ce4dbf1b-2b3d-4973-8234-73768e8fd659" TYPE="xfs"
/dev/vda2: LABEL="myroot" UUID="21ad8b9a-aaad-443c-b732-4e2522e95e23" TYPE="xfs"
/dev/vda3: UUID="12y99K-bv2A-y7RY-jhEW-rIWf-PcH5-SaiApN" TYPE="LVM2_member"
/dev/vda5: UUID="e20d65d9-20d4-472f-9f91-cdcfb30219d6" TYPE="ext4" &lt;==看到 ext4 了！
[root@study ~]# dumpe2fs /dev/vda5
dumpe2fs 1.42.9 （28-Dec-2013）
Filesystem volume name: &lt;none&gt; # 文件系统的名称（不一定会有）
Last mounted on: &lt;not available&gt; # 上一次挂载的目录位置
Filesystem UUID: e20d65d9-20d4-472f-9f91-cdcfb30219d6
Filesystem magic number: 0xEF53 # 上方的 UUID 为 Linux 对设备的定义码
Filesystem revision #: 1 （dynamic） # 下方的 features 为文件系统的特征数据
Filesystem features: has_journal ext_attr resize_inode dir_index filetype extent 64bit
flex_bg sparse_super large_file huge_file uninit_bg dir_nlink extra_isize
Filesystem flags: signed_directory_hash
Default mount options: user_xattr acl # 默认在挂载时会主动加上的挂载参数
Filesystem state: clean # 这块文件系统的状态为何，clean 是没问题
Errors behavior: Continue
Filesystem OS type: Linux
Inode count: 65536 # inode 的总数
Block count: 262144 # block 的总数
Reserved block count: 13107 # 保留的 block 总数
Free blocks: 249189 # 还有多少的 block 可用数量
Free inodes: 65525 # 还有多少的 inode 可用数量
First block: 0
Block size: 4096 # 单个 block 的容量大小
Fragment size: 4096
Group descriptor size: 64
....（中间省略）....
Inode size: 256 # inode 的容量大小！已经是 256 了喔！
....（中间省略）....
Journal inode: 8
Default directory hash: half_md4
Directory Hash Seed: 3c2568b4-1a7e-44cf-95a2-c8867fb19fbc
Journal backup: inode blocks
Journal features: （none）
Journal size: 32M # Journal 日志式数据的可供纪录总容量
Journal length: 8192
Journal sequence: 0x00000001
Journal start: 0
Group 0: （Blocks 0-32767） # 第一块 block group 位置
Checksum 0x13be, unused inodes 8181
Primary superblock at 0, Group descriptors at 1-1 # 主要 superblock 的所在喔！
Reserved GDT blocks at 2-128
Block bitmap at 129 （+129）, Inode bitmap at 145 （+145）
Inode table at 161-672 （+161） # inode table 的所在喔！
28521 free blocks, 8181 free inodes, 2 directories, 8181 unused inodes
Free blocks: 142-144, 153-160, 4258-32767 # 下面两行说明剩余的容量有多少
Free inodes: 12-8192
Group 1: （Blocks 32768-65535） [INODE_UNINIT] # 后续为更多其他的 block group 喔！
....（下面省略）....
# 由于数据量非常的庞大，因此鸟哥将一些信息省略输出了！上表与你的屏幕会有点差异。
# 前半部在秀出 supberblock 的内容，包括标头名称（Label）以及inode/block的相关信息
# 后面则是每个 block group 的个别信息了！您可以看到各区段数据所在的号码！
# 也就是说，基本上所有的数据还是与 block 的号码有关就是了！很重要！
```



## 四、与目录树的关系

- 目录

>当我们在 Linux 下的文件系统创建一个目录时，文件系统会分配一个 inode 与至少一块 block给该目录。其中，inode 记录该目录的相关权限与属性，并可记录分配到的那块 block 号码；而 block 则是记录在这个目录下的文件名与该文件名占用的 inode 号码数据。

- 文件

> 当我们在 Linux 下的 ext2 创建一个一般文件时， ext2 会分配一个 inode 与相对于该文件大小的 block 数量给该文件。

- 目录树读取

> 由于目录树是由根目录开始读起，因此系统通过挂载的信息可以找到挂载点的 inode 号码，此时就能够得到根目录的 inode 内容，并依据该 inode 读取根目录的 block 内的文件名数据，再一层一层的往下读到正确的文件名。



## 五、Linux 文件系统的运行

- Linux 使用的方式是通过一个称为非同步处理（asynchronously） 的方式。

> 当系统载入一个文件到内存后，如果该文件没有被更动过，则在内存区段的文件数据会被设置为干（clean）的。 但如果内存中的文件数据被更改过了（例如你用 nano 去编辑过这个文件），此时该内存中的数据会被设置为脏的 （Dirty）。此时所有的动作都还在内存中执行，并没有写入到磁盘中！ 系统会不定时的将内存中设置为“Dirty”的数据写回磁盘，以保持磁盘与内存数据的一致性。

- Linux 系统上面文件系统与内存有非常大的关系:

>1. 系统会将常用的文件数据放置到内存的缓冲区，以加速文件系统的读/写；
>2. 承上，因此 Linux 的实体内存最后都会被用光！这是正常的情况！可加速系统性能；
>3. 你可以手动使用 sync 来强迫内存中设置为 Dirty 的文件回写到磁盘中；
>4. 若正常关机时，关机指令会主动调用 sync 来将内存的数据回写入磁盘内；
>5. 但若不正常关机（如跳电、死机或其他不明原因），由于数据尚未回写到磁盘内， 因此重新开机后可能会花很多时间在进行磁盘检验，甚至可能导致文件系统的损毁（非磁盘损毁）。



## 六、XFS 文件系统简介

- 为啥CentOS 要舍弃对 Linux 支持度最完整的 EXT 家族而改用 XFS 呢？

> EXT 家族当前较伤脑筋的地方：支持度最广，但格式化超慢！

- XFS 文件系统的配置

> 基本上 xfs 就是一个日志式文件系统,就是被开发来用于大容量磁盘以及高性能文件系统之用， 因此，相当适
> 合现在的系统环境。

- xfs 文件系统在数据的分佈上，主要规划为三个部份，一个数据区 （data section）、一个文件系统活动登录区 （log section）以及一个实时运行区 （realtime section）。

> 1. 数据区 （data section）
>
> xfs 的这个数据区的储存区群组 （allocation groups, AG），将它想成是 ext家族的 block 群组 （blockgroups） ！
>
> 2. 文件系统活动登录区 （log section）
>
> 用来纪录文件系统的变化，其实有点像是日志区啦！文件的变化会在这里纪录下来，直到该变化完整的写入到数据区后， 该笔纪录才会被终结。
>
> 3. 实时运行区 （realtime section）
>
> 当有文件要被创建时，xfs 会在这个区段里面找一个到数个的 extent 区块，将文件放置在这个区块内，等到分配完毕后，再写入到 data section 的 inode 与 block 去！
