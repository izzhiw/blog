---
title: 如何在 Linux 中分区与格式化
categories:
  - linux-basics
tags:
  - storage
date: 2018-07-31 10:43:25
---

**前言**

为 Linux 系统添置一块新的磁盘，这是件很轻松的事情。有许多工具、文件系统格式和分区方案可以满足你的特殊需求，当然这会使操作流程复杂一点。但如果你只是想尽快把磁盘用起来，操作流程也相当简单。

本指南涵盖以下操作流程：

- 识别系统中的新磁盘。
- 创建一个独立分区，这个分区跨越整个驱动器。（大多数操作系统都要求有分区，即便是操作系统只有一个文件系统。）
- 使用 Ext4 文件系统格式化分区。（Ext4 文件系统是大多数现代 Linux 发行版中的默认文件系统。）
- 挂载，并且设置系统启动时自动挂载。

<!-- more -->

### 安装工具

对驱动器进行分区，我们将使用 `parted` 实用程序。大多数情况下，程序已默认安装。

如果在 Ubuntu 或 Debian 服务器上还没有 `parted`，你可以通过键入以下命令来安装：

```
$ sudo apt-get update
$ sudo apt-get install parted
```

如果在 CentOS 或 Fedora 服务器上，你可以通过键入以下命令来安装：

```
$ sudo yum install parted
```

### 识别系统中的新磁盘

在我们设置驱动器之前，我们需要在服务器中正确识别它。

如果这是一个全新驱动器，要在服务器上找到它，最简单方法是查看分区方案，找未被分区的。 如果用 `parted` 列出磁盘的分区布局，它会提示一个错误，说哪个磁盘未被分区。这就可以帮助我们识别新磁盘：

```
$ sudo parted -l | grep Error
```

你应该看到一个 `unrecognized disk label` （不可识别的磁盘标签）的错误，以提示这个新的、未分区的磁盘：

```
Output
Error: /dev/sda: unrecognised disk label
```

你还可以使用 `lsblk` 命令查找已知容量的磁盘，这个磁盘并没有任何关联：

```
$ lsblk
```

```
Output
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk 
vda    253:0    0    20G  0 disk 
└─vda1 253:1    0    20G  0 part /
```

> <center>警告</center>
>
> 在进行操作前的每个会话中，务必用 `lsblk` 查看磁盘信息。每次系统引导，`/dev/sd*` 和 `/dev/hd*` 磁盘标识符未必一致。这就是说，假如你没有确认磁盘标识符，而进行分区或格式化，就会引发误操作。
>
> 考虑使用更持久的磁盘标识符，例如 `/dev/disk/by-uuid`、`/dev/disk/by-label` 或 `/dev/disk/by-id`。有关详细信息，请参阅 [Linux 存储术语和概念的介绍](/2018/10/17/an-introduction-to-storage-terminology-and-concepts-in-linux)。

内核为磁盘分配了名称，当你知道这个名称后，你就可以对驱动器进行分区了。

### 对新驱动器进行分区

如前言所述，我们将在本指南中说明，怎样创建一个跨越整个磁盘的单个分区。

**选择一个分区标准**

为此，我们首先需要指定要使用的分区标准。GPT 是更现代的分区标准，而 MBR 标准在各种操作系统中提供更广泛的支持。如果没有任何特殊要求，你最好在此使用 GPT。

要选择 **GPT** 标准，请这样键入以标识磁盘：

```
$ sudo parted /dev/sda mklabel gpt
```

要选择 **MBR** 标准，请这样键入以标识磁盘：

```
$ sudo parted /dev/sda mklabel msdos
```

> <center>译者注</center>
>
> 请将原命令中的 `/dev/sda` 替换为你的磁盘标识，下同，不再赘述。

**创建新分区**

选择格式后，你通过键入以下命令，创建一个跨越整个驱动器的分区：

```
$ sudo parted -a opt /dev/sda mkpart primary ext4 0% 100%
```

如果我们查看 `lsblk` 命令，我们应该看到新分区可用：

```
$ lsblk
```

```
Output
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk 
└─sda1   8:1    0   100G  0 part 
vda    253:0    0    20G  0 disk 
└─vda1 253:1    0    20G  0 part /
```

### 在新分区上创建文件系统

现在我们有了一个可用的分区，我们可以将它格式化为 Ext4 文件系统。 为此，使用 `mkfs.ext4` 实用程序进行分区。

我们可以通过设置 `-L` 参数来添加分区标签。选择一个易于识别驱动器的名称：

> <center>注意</center>
>
> 确保操作对象是**分区**而不是整个**磁盘**。在Linux中，磁盘名称像 `sda`、`sdb`、`hda` 这样。在这些磁盘名称末尾加一个数字，这才是分区名称。因此，我们要用类似 `sda1` 而**不是** `sda`。

```
$ sudo mkfs.ext4 -L datapartition /dev/sda1
```

如果要在以后更改分区标签，可以使用 `e2label` 命令：

```
$ sudo e2label /dev/sda1 newlabel
```

你可以使用 `lsblk`，来查看识别分区的所有不同方法。我们想要找出分区的名称、标签和 UUID。

如果我们键入以下命令，某些版本的 `lsblk` 将显示所有这些信息：

```
$ sudo lsblk --fs
```

如果你的版本未显示所有相应字段，那么你可以手动指定输出：

```
$ sudo lsblk -o NAME,FSTYPE,LABEL,UUID,MOUNTPOINT
```

你应该看到以下信息。高亮的输出字段即是那些不同的标识方法，可用于标识新文件系统：

```
Output
NAME   FSTYPE LABEL         UUID                                 MOUNTPOINT
sda                                                              
└─sda1 ext4   datapartition 4b313333-a7b5-48c1-a957-d77d637e4fda 
vda                                                              
└─vda1 ext4   DOROOT        050e1e34-39e6-4072-a03e-ae0bf90ba13a /
```

### 挂载新文件系统

现在，我们可以挂载文件系统以供使用了。

[Filesystem Hierarchy Standard](http://refspecs.linuxfoundation.org/fhs.shtml)（文件系统层次结构标准）建议，使用 `/mnt` 或其子目录来临时挂载文件系统。 但标准中并未推荐，在何处挂载永久存储器，因此随你便。在本教程中，我们将驱动器挂载到 `/mnt/data`。

键入以下命令，创建挂载点目录：

```
$ sudo mkdir -p /mnt/data
```

**临时挂载文件系统**

键入以下命令，临时挂载文件系统：

```
$ sudo mount -o defaults /dev/sda1 /mnt/data
```

**系统引导时自动挂载文件系统**

如果你想在每次服务器引导时自动挂载文件系统，请修改 `/etc/fstab` 文件：

```
$ sudo nano /etc/fstab
```

之前，我们使用 `sudo lsblk --fs` 命令，显示出我们文件系统的三种标识符。 在文件中，我们可以使用其中任何一种。 我们使用了下面的分区标签标识符（LABEL），当然你也可以使用注释掉的其他两种标识符：

> <center>/etc/fstab</center>
>
> . . .
> \## Use one of the identifiers you found to reference the correct partition
> \# /dev/sda1 /mnt/data ext4 defaults 0 2
> \# UUID=4b313333-a7b5-48c1-a957-d77d637e4fda /mnt/data ext4 defaults 0 2
> LABEL=datapartition /mnt/data ext4 defaults 0 2

<br />

> <center>注意</center>
>
> 通过键入 `man fstab`，你能了解 `/etc/fstab` 文件中各个字段的含义。有关特殊文件系统类型的可用挂载选项的信息，请查看 `man [filesystem]`（比如 `man ext4`）。现在来说，上面的挂载代码行应该是你认识的开始。
>
> 对于 SSD，有时会附加 `discard` 选项以启用 continuous TRIM。以这种方式执行 continuous TRIM，在性能和完整性影响的方面存在争议。大多数 Linux 发行版中包含 periodic TRIM，并把它作为一个替代方案。

完成后，保存并关闭文件。

如果之前没有挂载文件系统，现在你可以键入以下命令来挂载它：

```
$ sudo mount -a
```

**测试挂载**

在挂载完毕之后，我们应该检查一下，以确保文件系统是可以访问的。

我们可以在 `df` 命令的输出中，检查磁盘是否可用：

```
$ df -h -x tmpfs -x devtmpfs
```

```
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  1.3G   18G   7% /
/dev/sda1        99G   60M   94G   1% /mnt/data
```

你还应该在 `/mnt/data` 目录中看到 `lost+found` 目录，`/mnt/data` 目录是 Ext\* 文件系统的根目录：

```
$ ls -l /mnt/data
```

```
Output
total 16
drwx------ 2 root root 16384 Jun  6 11:10 lost+found
```

通过写入测试文件，我们还可以来检查读写功能：

```
$ echo "success" | sudo tee /mnt/data/test_file
```

回读文件，确保刚才正确写入：

```
$ cat /mnt/data/test_file
```

```
Output
success
```

验证新文件系统运行正常后，你就可以删除该文件：

```
$ sudo rm /mnt/data/test_file
```

### 结语

你的新驱动器现在已经分区、格式化、挂载，可以正常使用了。你能将原始磁盘转换为 Linux 可用于存储的文件系统，以上是基本流程。另有一些更复杂的分区、格式化和挂载方法，在某些情况下可能更合适，但以上可满足基本需求。

(END)

> 本文转译自互联网，[原文链接在这里](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
> 我是 zZhiw，[更多内容请关注](https://zhengzhiwei.net)

