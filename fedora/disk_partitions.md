# An Introduction to Disk Partitions 硬盘分区介绍

*ps: 来自 Fedora　官方文档 `Installation Guide` 附录Ｂ*

**个人翻译供学习用，版权归Fedora所有。因中国大陆的网络问题，暂无法提交到 `Fedora  Documentation` 项目。 -- ssbunny**

> This appendix is not necessarily applicable to architectures other than AMD64 and Intel 64. However, the general concepts mentioned here may apply.

> 该附录不仅适用于 AMD64 或 Intel64　系统。这里提到的一般性概念通常可以通用。

This section discusses basic disk concepts, disk repartitioning　strategies, the partition naming scheme　used by Linux systems, and related topics.

该部分讨论了磁盘的基本概念、磁盘重分区策略、Linux系统分区命名方案及相关主题。

If you are comfortable with disk partitions, you can skip ahead to　Section B.2, “Strategies for Disk　Repartitioning” for more information on the process of freeing up disk space to prepare for a Fedora　installation.

如何你对磁盘分区感兴趣，可以直接跳至第B.2小节<磁盘重新分区策略>，
以便了解更多关于Fedora安装时释放磁盘空间的过程。

## Hard Disk Basic Concepts 硬盘的基本概念

Hard disks perform a very simple function - they store data and reliably retrieve it on command.

硬盘的功能非常简单——它们被用来存储数据并可以通过命令可靠地检索到这些数据。

When discussing issues such as disk partitioning, 
it is important to have a understanding of the　underlying hardware;
however, since the　theory is very complicated and expansive,
only the basic　concepts will be explained here.
This appendix uses a set of simplified diagrams of a disk drive to help
explain what is the process and theory behind partitions.

Figure B.1, “An Unused Disk Drive”, shows a brand-new, unused disk drive.

![An Unused Disk Drive](img/disk_partition_1.png)
Figure B.1　An Unused Disk Drive
