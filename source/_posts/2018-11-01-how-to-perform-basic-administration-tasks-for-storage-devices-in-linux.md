---
title: 如何在 Linux 中执行存储设备的基本管理任务
categories:
  - linux-basics
tags:
  - storage
date: 2018-11-01 14:51:24
---

**前言**

有许多工具可用于管理 Linux 中的存储。但是，只有少数被用于日常维护和管理。在本指南中，我们将介绍一些用于管理挂载点、存储设备和文件系统时最常用的实用程序。

<!-- more -->

### 其它资源

本指南不会介绍如何准备存储设备，以便在 Linux 系统上初次使用。如果你尚未设置存储，[如何在 Linux 中分区与格式化](/2018/07/31/how-to-partition-and-format-storage-devices-in-linux)这篇指南将帮助你准备原始存储设备。

讨论存储时用到一些术语，有关这些术语的更多信息，请查看我们关于[存储术语](/2018/10/17/an-introduction-to-storage-terminology-and-concepts-in-linux)的文章。

### 使用 df 查看存储容量和使用情况

通常，你希望了解系统存储的最重要信息，是已连接存储设备的容量和当前利用率。

要检查总共有多少存储空间，并查看驱动器的当前利用率，需使用 **df** 实用程序。默认情况下，它以 1K 块输出测量值，这通常不太有用。添加 `-h` 标志，以输出人类可读的（human-readable）单位：

```
$ df -h
```

```
Output
Filesystem      Size  Used Avail Use% Mounted on
udev            238M     0  238M   0% /dev
tmpfs            49M  624K   49M   2% /run
/dev/vda1        20G  1.1G   18G   6% /
tmpfs           245M     0  245M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           245M     0  245M   0% /sys/fs/cgroup
tmpfs            49M     0   49M   0% /run/user/1000
/dev/sda1        99G   60M   94G   1% /mnt/data
```

如你所见，挂载到 `/` 的 `/dev/vda1` 分区是 6% 已满并且还有 18G 可用空间，而挂载到 `/mnt/data` 的 `/dev/sda1` 分区是空的并且有 94G 可用空间。其它条目使用 `tmpfs` 或 `devtmpfs` 文件系统，这是易失性存储器（volatile memory），就像它是永久存储一样被使用。我们可以输入以下内容，以排除这些条目：

```
$ df -h -x tmpfs -x devtmpfs
```

```
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  1.1G   18G   6% /
/dev/sda1        99G   60M   94G   1% /mnt/data
```

通过删除一些伪设备和特殊设备，此输出可以更集中地显示当前磁盘利用率。

### 使用 lsblk 查看有关块设备的信息

**块设备**是一个通用术语，它是一种存储设备，该存储设备以特定大小的块来读取或写入。该术语适用于几乎所有类型的非易失性存储，包括硬盘驱动器（HDD）、固态驱动器（SSD）、闪存等。块设备是文件系统被写入的物理设备。反过来说，文件系统决定了数据和文件的存储方式。

`lsblk` 实用程序可用于轻松显示有关块设备的信息。该实用程序的特定功能取决于安装的版本，但通常，`lsblk` 命令可用于显示有关驱动器本身的信息，以及分区信息和已被写入其中的文件系统。

没有任何参数，`lsblk` 将显示设备名称、主要和次要编号（Linux 内核用于跟踪驱动程序和设备）、驱动器是否可移动、其大小、是否以只读方式挂载、其类型（磁盘或分区）及其挂载点。有些系统需要 `sudo` 才能正确显示，所以我们将键入以下内容：

```
$ sudo lsblk
```

```
Output
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  100G  0 disk 
vda    253:0    0   20G  0 disk 
└─vda1 253:1    0   20G  0 part /
```

在显示的输出中，最重要的部分通常是名称（它指的是 `/dev` 下的设备名称）、大小、类型和挂载点。在这里，我们可以看到我们有一个磁盘（`/dev/vda`），磁盘中仅有的一个分区（`/dev/vda1`）被用作 `/` 分区，另一个磁盘（`/dev/sda`）未被分区。

要获取与磁盘和分区管理更相关的信息，可以在某些版本上传递 `--fs` 标志：

```
$ sudo lsblk --fs
```

```
Output
NAME   FSTYPE LABEL  UUID                                 MOUNTPOINT
sda                                                       
vda                                                       
└─vda1 ext4   DOROOT c154916c-06ea-4268-819d-c0e36750c1cd /
```

如果 `--fs` 标志对于你的版本不可用，则可以使用 `-o` 标志手动复制输出，以请求特定的输出。你可以使用 `-o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT` 来获取相同的信息。

要获取有关磁盘拓扑的信息，请键入：

```
$ sudo lsblk -t
```

```
Output
NAME   ALIGNMENT MIN-IO OPT-IO PHY-SEC LOG-SEC ROTA SCHED    RQ-SIZE  RA WSAME
sda            0    512      0     512     512    1 deadline     128 128    2G
vda            0    512      0     512     512    1              128 128    0B
└─vda1         0    512      0     512     512    1              128 128    0B
```

还有许多其它快捷方式，它们可用于显示有关磁盘和分区的相关特征。你可以使用 `-O` 标志输出所有可用列，也可以通过使用 `-o` 标志指定列名以自定义要显示的字段。`-h` 标志可用于列出可用列：

```
$ lsblk -h
```

```
Output
. . .

Available columns (for --output):
        NAME  device name
       KNAME  internal kernel device name

       . . .

  SUBSYSTEMS  de-duplicated chain of subsystems
         REV  device revision
      VENDOR  device vendor

For more details see lsblk(8).
```

### 使用文件系统挂载

在使用新磁盘之前，通常必须对其进行分区，使用一个文件系统对其进行格式化，然后挂载驱动器或分区。分区和格式化通常是一次性过程，因此我们不在此讨论它们。如前所述，你可以在[这篇文章](/2018/07/31/how-to-partition-and-format-storage-devices-in-linux)中找到有关如何使用 Linux 对驱动器进行分区和格式化的更多信息。

另一方面，挂载是你可能更频繁地管理的事情。挂载文件系统，就是使其在服务器所选挂载点上可用。简单地说，**挂载点**就是一个可以访问新文件系统的目录。

两个互补的命令主要用于管理挂载：`mount` 和 `umount`。`mount` 命令用于将文件系统附加到当前文件树。在 Linux 系统中，一个单独的、统一的文件层次结构用于整个系统，无论它由多少个物理设备组成。`umount` 命令（注意：这是 `umount`，而不是 `unmount`）用于卸载文件系统。此外，`findmnt` 命令非常有用，它收集有关已安装文件系统当前状态的信息。

**使用 mount 命令**

使用 `mount` 最基本方法是，传入已格式化的设备或分区，以及要连接的挂载点：

```
$ sudo mount /dev/sda1 /mnt
```

挂载点是最后一个参数，它指定新文件系统应附加到文件层次结构中的哪里，它应该几乎总是一个空目录。

通常，你需要在挂载时选择更具体的选项。尽管 `mount` 可以尝试猜测文件系统的类型，但使用 `-t` 选项来传递文件系统类型，这几乎总是个更好的主意。对于 Ext4 文件系统，这将是：

```
$ sudo mount -t ext4 /dev/sda1 /mnt
```

还有许多其它选项会影响文件系统的挂载方式。那些通用的挂载选项，可以在 `man mount` 的 **FILESYSTEM INDEPENDENT MOUNT OPTIONS** 章节找到。在相同的手册页 filesystem-dependent 选项中，文件系统也通常在 **FILESYSTEM SPECIFIC MOUNT OPTIONS** 标题下有一个章节。

用 `-o` 标志传入其它选项。例如，要使用默认选项（即 `rw,suid,dev,exec,auto,nouser,async`）挂载分区，我们可以传入 `-o defaults`。如果我们想要覆盖读写权限，并以只读方式挂载，我们可以添加 `ro` 作为后续选项，那么它将覆盖 `defaults` 选项中的 `rw`：

```
$ sudo mount -t ext4 -o defaults,ro /dev/sda1 /mnt
```

要挂载 `/etc/fstab` 文件中列出的所有文件系统，可以传递 `-a` 选项：

```
$ sudo mount -a
```

**列出文件系统挂载选项**

要显示用于一个特定挂载的挂载选项，那么就将其传递给 `findmnt` 命令。例如，如果我们使用 `findmnt` 查看上例中的只读挂载，它看起来像这样：

```
$ findmnt /mnt
```

```
Output
TARGET SOURCE    FSTYPE OPTIONS
/mnt   /dev/sda1 ext4   ro,relatime,data=ordered
```

如果你一直在尝试多个选项，并且最终发现了您喜欢的组合，那么这非常有用。你可以使用 `findmnt` 找到它正在使用的选项，以便知道什么适合添加到 `/etc/fstab` 文件，以供将来挂载。

**卸载文件系统**

`umount` 命令用于卸载给定的文件系统。再次强调，这是 `umount` 而不是 `unmount`。

该命令的一般形式只是指定当前已挂载文件系统的挂载点或设备。请确保你没有使用挂载点上的任何文件，并且你没有在挂载点内部运行任何应用程序（包括当前 shell）：

```
$ cd ~
$ sudo umount /mnt
```

对于绝大多数用户来说，除了默认的卸载行为之外，什么都不需要。

### 结语

虽然此列表并非详尽无遗，但这些实用程序应该涵盖了日常系统管理任务中所需的大部分内容。通过学习一些工具，你可以轻松处理服务器上的存储设备。

(END)

> 本文转译自互联网，[原文链接在这里](https://www.digitalocean.com/community/tutorials/how-to-perform-basic-administration-tasks-for-storage-devices-in-linux)
> 我是 zZhiw，[更多内容请关注](https://zhengzhiwei.net)

