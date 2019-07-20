# Linux挂载新硬盘

在安装Linux作为服务器操作系统的时候，常常会遇到需要在新装的系统上挂载硬盘的情况，这篇文章就整理一下关于Linux挂载新硬盘的操作步骤。



### 小于2T硬盘挂载

根据硬盘容量大小的不同，为Linux系统挂载新硬盘需要不同的操作。首先介绍小于2T的硬盘挂载步骤。

##### 查看硬盘信息

使用`fdisk -l`命令查看当前系统硬盘信息，找到新硬盘所在目录路径，这里我们假设该路径为/dev/sda。

##### 为硬盘创建分区

使用`fdisk /dev/sda`进入fdisk的分区模式。依次键入如下按键完成分区设置：

<kbd>n</kbd> 创建一个新分区；

<kbd>p</kbd> 设置分区类型为主分区；

<kbd>1</kbd> 设置分区编号为1；

<kbd>w</kbd> 保存设置并退出。

完成之后我们可以在/dev目录下看到一个新的文件sda1（这个1即为设置中的分区编号）。

##### 格式化分区

使用`mkfs.ext4 /dev/sda1`将我们刚才创建好的分区格式化为ext4文件系统。

##### 挂载硬盘

为自己新建好的分区选择一个挂载目录。这里我们假设我们将该分区挂载至/data目录下。

首先确保挂载目标目录已存在`mkdir /data`，然后完成挂载操作`mount /dev/sda1 /data`。

可以使用`df -h`命令来查看挂载结果。

##### 设置开机自动挂载

最后我们需要为新分区设置开机自动挂载。

编辑/etc/fstab文件`vi /etc/fstab`，在该文件尾部添加`/dev/sda1 /data ext4 defaults 0 0`后保存文件即可。



### 大于2T硬盘挂载

fdisk命令无法为大于2T的硬盘进行分区，我们需要使用parted命令来完成大于2T硬盘的分区操作，其余步骤与上方所述一致。这里我们仍然假设硬盘路径为/dev/sda。

##### 使用parted进行硬盘分区设置

输入命令`parted /dev/sda`进入分区模式后，进行以下操作：

`mklabel gpt` 设置分区类型为gpt；

`mkpart extended 0% 100%` 使用整个硬盘空间；

`quit` 保存并退出。



完成分区设置后继续完成分区的格式化与挂载操作即可。



###### 参考内容链接

1. [Centos7.3 新磁盘挂载](https://www.zhudo.net/program/59.html)
2. [Linux挂载大于2T的磁盘硬盘](https://www.cnblogs.com/chenminklutz/p/7228266.html)
3. [在linux上如何挂载新增加的硬盘](https://jingyan.baidu.com/article/aa6a2c14bd8cc70d4c19c4c8.html)

