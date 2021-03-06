---
title: "为 Linux ext2fs 文件系统记录日志"
date: 2022-05-23T20:25:45+08:00
draft: false
tags: ["论文", "操作系统", "文件系统"]
math: true
summary: "本文描述了为 Linux ext2fs 文件系统设计和实现事务性元数据日志的工作进展。我们回顾了崩溃后恢复文件系统的问题，并描述了一种设计，旨在通过在文件系统中添加事务日志来提高 ext2fs 的崩溃恢复速度和可靠性。


文件系统是任何现代操作系统的核心部分，人们期望它既快速又极其可靠。然而，由于硬件、软件或电源故障，问题仍然存在，机器可能会意外停机。


在一次意外重启之后，系统可能需要一些时间才能将其文件系统恢复到一致的状态。随着磁盘容量的增加，这个时间可能会成为一个严重的问题，系统会花费超过一个小时来扫描、检查和修复磁盘，在此期间你只能等待。尽管磁盘驱动器的速度每年都在提高，但与它们在容量上的巨大增长相比，这种速度的增长并不明显。不幸的是，在使用传统的文件系统检查技术时，磁盘容量每增加一倍，恢复时间就会增加一倍。


在对系统可用性要求很高的情况下，可能并不允许跳过检查以节省时间，因此需要一种机制，以避免每次机器重启要经历长时间的恢复阶段。"
---

_Stephen C. Tweedie_

_sct@dcs.ed.ac.uk_

## 摘要 Abstract

This paper describes a work-in-progress to design and implement a transactional metadata journal for the Linux ext2fs filesystem. We review the problem of recovering filesystems after a crash, and describe a design intended to increase ext2fs's speed and reliability of crash recovery by adding a transactional journal to the filesystem.

本文描述了为 Linux ext2fs 文件系统设计和实现事务性元数据日志的工作进展。我们回顾了崩溃后恢复文件系统的问题，并描述了一种设计，旨在通过在文件系统中添加事务日志来提高 ext2fs 的崩溃恢复速度和可靠性。

## 简介 Introduction

Filesystems are central parts of any modern operating system, and are expected to be both fast and exceedingly reliable. However, problems still occur, and machines can go down unexpectedly, due to hardware, software or power failures.

文件系统是任何现代操作系统的核心部分，人们期望它既快速又极其可靠。然而，由于硬件、软件或电源故障，问题仍然存在，机器可能会意外停机。

After an unexpected reboot, it may take some time for a system to recover its filesystems to a consistent state. As disk sizes grow, this time can become a serious problem, leaving a system offline for an hour or more as the disk is scanned, checked and repaired. Although disk drives are becoming faster each year, this speed increase is modest compared with their enormous increase in capacity. Unfortunately, every doubling of disk capacity leads to a doubling of recovery time when using traditional filesystem checking techniques.

在一次意外重启之后，系统可能需要一些时间才能将其文件系统恢复到一致的状态。随着磁盘容量的增加，这个时间可能会成为一个严重的问题，系统会花费超过一个小时来扫描、检查和修复磁盘，在此期间你只能等待。尽管磁盘驱动器的速度每年都在提高，但与它们在容量上的巨大增长相比，这种速度的增长并不明显。不幸的是，在使用传统的文件系统检查技术时，磁盘容量每增加一倍，恢复时间就会增加一倍。

Where system availability is important, this may be time which cannot be spared, so a mechanism is required which will avoid the need for an expensive recovery stage every time a machine reboots.

在对系统可用性要求很高的情况下，可能并不允许跳过检查以节省时间，因此需要一种机制，以避免每次机器重启要经历长时间的恢复阶段。

## 文件系统中有什么？ What's in a filesystem?

What functionality do we require of any filesystem? There are obvious requirements which are dictated by the operating system which the filesystem is serving. The way that the filesystem appears to applications is one−operating systems typically require that filenames adhere to certain conventions and that files possess certain attributes which are interpreted in a specific way.

我们对文件系统有什么要求呢？有一些明显的要求，是由文件系统所服务的操作系统决定的。文件系统出现在应用中的方式是：操作系统通常要求文件名符合某些惯例，文件拥有某些属性，并以特定的方式进行解释。

However, there are many internal aspects of a file system which are not so constrained, and which a filesystem implementor can design with a certain amount of freedom. The layout of data on disk (or alternatively, perhaps, its network protocol, if the filesystem is not local), details of internal caching, and the algorithms used to schedule disk IO−these are all things which can be changed without neces sarily violating the specification of the filesystem's application interface.

然而，文件系统的许多内部方面并没有受到这样的约束，文件系统实现者可以在一定程度上自由地进行设计。磁盘上数据的布局（或者，如果不是本地文件系统，也许是它的网络协议）、内部缓存的细节，以及用于调度磁盘 IO 的算法——这些都是可以更改的，而不必违反文件系统应用程序接口的规范。

There are a number of reasons why we might choose one design over another. Compatibility with older filesystems might be an issue: for example, Linux provides a UMSDOS filesystem which implements the semantics of a POSIX filesystem on top of the standard MSDOS on-disk file structure.

我们选择文件系统设计方案的因素有很多。与旧文件系统的兼容性可能是一个问题：例如，Linux 提供了一个 UMSDOS 文件系统，它在标准 MSDOS 磁盘文件结构的基础上实现了 POSIX 文件系统的语义。

When trying to address the problem of long filesystem recovery times on Linux, we kept a number of goals in mind:

在尝试解决 Linux 上文件系统恢复时间过长的问题时，我们牢记以下几个目标：

- Performance should not suffer seriously as a re sult of using the new filesystem;

  使用新的文件系统后，性能不应受到严重影响

- Compatibility with existing applications must not be broken

  不得破坏与现有应用程序的兼容性

- The reliability of the filesystem must not be compromised in any way.

  绝不能以任何方式损害文件系统的可靠性

## 文件系统可靠性 Filesystem Reliability

There are a number of issues at stake when we talk about filesystem reliability. For the purpose of this particular project, we are interested primarily in the reliability with which we can recover the contents of a crashed filesystem, and we can identify several aspects of this:

当我们谈论文件系统的可靠性时，有一些问题是很重要的。就本项目而言，我们主要关心的是恢复崩溃的文件系统内容的可靠性，我们可以确定其中的几个方面：

_Preservation_: data which was stable on disk before the crash should never ever be damaged. Obviously, files which were being written out at the time of the crash cannot be guaranteed to be perfectly intact, but any files which were already safe on disk must not be touched by the recovery system.

_保存_：崩溃前磁盘上稳定的数据永远不应该被损坏。显然，我们无法保证崩溃时正在写入的文件完好无损，但恢复系统决不能触及磁盘上已经安全的任何文件。

_Predictability_: the failure modes from which we have to recover should be predictable in order for us to recover reliably.

_可预测性_：为了可靠地恢复，我们应该预测到所有可能的故障模式。

_Atomicity_: many filesystem operations require a significant number of separate IOs to complete. A good example is the renaming of a file from one directory to another. Recovery is atomic if such filesystem operations are either fully completed on disk or fully undone after recovery finishes. (For the rename example, recovery should leave either the old or the new filename committed to disk after a crash, but not both.)

_原子性_：许多文件系统操作需要大量单独的 IO 才能完成。一个很好的例子是将文件从一个目录重命名为另一个目录。如果此类文件系统操作在磁盘上完全完成或在恢复后完全撤消，则恢复是原子的。（以重命名为例，恢复应在崩溃后将旧文件名或新文件名保留在磁盘上，但不能同时保留两者。）

## 现有实现 Existing implementations

The Linux ext2fs filesystem offers preserving recovery, but it is non-atomic and unpredictable. Predictability is in fact a much more complex property than appears at first sight. In order to be able to predictably mop up after a crash, the recovery phase must be able to work out what the filesystem was trying to do at the time if it comes across an inconsistency representing an incomplete operation on the disk. In general, this requires that the filesystem must make its writes to disk in a predictable order whenever a single update operation changes multiple blocks on disk.

Linux ext2fs 文件系统提供了恢复功能，但它是非原子的且不可预测。事实上，可预测性是一个比乍看之下复杂得多的属性。为了能够确定崩溃后的清理工作，我们必须知道磁盘在遇到不一致时正在进行什么操作。通常，这要求文件系统的更新操作在更改磁盘上的多个块时，必须以可预测的顺序写入磁盘。

There are many ways of achieving this ordering be tween disk writes. The simplest is simply to wait for the first writes to complete before submitting the next ones to the device driver−the "synchronous metadata update" approach. This is the approach taken by the BSD Fast File System[1], which appeared in 4.2BSD and which has inspired many of the Unix filesystems which followed, including ext2fs.

有许多方法可以在磁盘写入之间实现这种排序。最简单的方法就是等待第一次写入完成，然后再将下一次写入提交给设备驱动程序——“同步元数据更新”方法。这是 BSD 快速文件系统[1]所采用的方法，它出现在 4.2BSD 中，并启发了许多之后的 Unix 文件系统，包括 ext2fs。

However, the big drawback of synchronous meta data update is its performance. If filesystems operation require that we wait for disk IO to complete, then we cannot batch up multiple filesystem updates into a single disk write. For example, if we create a dozen directory entries in the same directory block on disk, then synchronous updates require us to write that block back to disk a dozen separate times.

然而，同步元数据更新的最大缺点是其性能。如果文件系统操作需要等待磁盘 IO 完成，那么我们就不能将多个文件系统的更新批量化为一次磁盘写入。例如，如果我们在磁盘上的同一目录块中创建十几个目录条目，那么同步元数据更新需要我们将该块写回磁盘十几次。

There are ways around this performance problem. One way to keep the ordering of disk writes without actually waiting for the IOs to complete is to maintain an ordering between the disk buffers in memory, and to ensure that when we do eventually go to write back the data, we never write a block until all of its predecessors are safely on disk−the "deferred ordered write" technique.

有一些方法可以解决这个性能问题。一种方法是保持磁盘写入的顺序，而不需要实际等待 IO 完成，就是在内存中保持磁盘缓冲区之间的顺序，并确保当我们最终去写回数据时，在一个块的所有前置块都安全地存在于磁盘上之前，我们不会写入它——“延迟有序写入”技术。

One complication of deferred ordered writes is that it is easy to get into a situation where there are cyclic dependencies between cached buffers. For example, if we try to rename a file between two directories and at the same time rename another file from the second directory into the first, then we end up with a situation where both directory blocks depend on each other: neither can be written until the other one is on disk.

延迟有序写入的一个复杂情况是，很容易陷入缓存缓冲区之间存在循环依赖关系的情况。例如，如果我们试图把一个文件从目录 A 重命名到目录 B，同时将另一个文件从目录 B 重命名到目录 A，那么我们最终会出现两个目录块相互依赖的情况：在另一个目录出现在磁盘上之前，任何一个都不能被写入。

Ganger's "soft updates" mechanism[2] neatly side steps this problem by selectively rolling back specific updates within a buffer if those updates still have outstanding dependencies when we first try to write that buffer out to disk. The missing update will be restored later once all of its own dependencies are satisfied. This allows us to write out buffers in any order we choose when there are circular dependencies. The soft update mechanism has been adopted by FreeBSD and will be available as part of their next major kernel version.

Ganger 的“软更新”机制[2]巧妙地回避了这个问题，如果在我们第一次尝试将缓冲区写入磁盘时，这些更新仍有未完成的依赖关系，那么就会有选择地回滚缓冲区内的特定更新。一旦所有的依赖关系得到满足，丢失的更新将在稍后恢复。这允许我们在存在循环依赖的情况下，以我们选择的顺序将缓冲区写入到磁盘。软更新机制已经被 FreeBSD 采用，并将作为他们下一个主要内核版本的一部分提供。

All of these approaches share a common problem, however. Although they ensure that the state of the disk is in a predictable state all the way through the course of a filesystem operation, the recovery process still has to scan the entire disk in order to find and repair any uncompleted operations. Recovery becomes more reliable, but is not necessarily any faster.

然而，所有这些方法都有一个共同的问题。尽管它们确保了在文件系统操作过程中，磁盘的状态一直是可预测的，但恢复过程仍然需要扫描整个磁盘，以便找到并修复任何未完成的操作。这更可靠，但不一定更快。

It is, however, possible to make filesystem recovery fast without sacrificing reliability and predictability. This is typically done by filesystems which guarantee atomic completion of filesystem updates (a single filesystem update is usually referred to as a transaction in such systems). The basic principle behind atomic updates is that the filesystem can write an entire batch of new data to disk, but that those updates do not take effect until a final, commit update is made on the disk. If the commit involves a write of a single block to the disk, then a crash can only result in two cases: either the commit record has been written to disk, in which case all of the committed filesystem operations can be assumed to be complete and consistent on disk; or the commit record is missing, in which case we have to ignore any of the other writes which occurred due to partial, uncommitted updates still outstanding at the time of the crash. This naturally requires a filesystem update to keep both the old and new contents of the updated data on disk somewhere, right up until the time of the commit.

然而，有可能在不牺牲可靠性和可预测性的情况下，使文件系统的恢复速度加快。这通常是由保证更新原子性的文件系统来完成的（在这种系统中，单一的文件系统更新通常被称为事务）。原子更新的基本原则是，文件系统可以将整批新数据写入磁盘，但这些更新在磁盘上进行最后的提交更新之前是不生效的。如果提交涉及到向磁盘写入单个块，那么崩溃只能导致两种情况：要么提交记录已经写入磁盘，在这种情况下，可以认为所有提交的文件系统操作在磁盘上是完整和一致的；要么提交记录丢失，在这种情况下，我们必须忽略由于部分未提交的更新而发生的任何其他写入操作，这些更新在崩溃时仍然未完成。这自然需要更新文件系统，以将更新数据的新旧内容保留在磁盘上的某个位置，直到提交。

There are a number of ways of achieving this. In some cases, filesystems keep the new copies of the updated data in different locations from the old copies, and eventually reuse the old space once the up dates are committed to disk. Network Appliance's WAFL filesystem[6] works this way, maintaining a tree of filesystem data which can be updated atomically simply by copying tree nodes to new locations and then updating a single disk block at the root of the tree.

有许多方法可以实现这一点。在某些情况下，文件系统将更新数据的新副本保留在与旧副本不同的位置，一旦更新数据被提交到磁盘，就重新使用旧空间。Network Appliance 的 WAFL 文件系统[6]就是这样工作的，它维护一个文件系统数据树，只需将树节点复制到新的位置，然后更新树根上的单个磁盘块，即可自动更新这些数据。

Log-Structured Filesystems achieve the same end by writing all filesystem data−both file contents and metadata−to the disk in a continuous stream (the "log"). Finding the location of a piece of data using such a scheme can be more complex than in a traditional filesystem, but logs have the big advantage that it is relatively easy to place marks in the log to indicate that all data up to a certain point is commit ted and consistent on disk. Writing to such a filesystem is also particularly fast, since the nature of the log makes most writes occur in a continuous stream with no disk seeks. A number of filesystems have been written based on this design, including the Sprite LFS[3] and the Berkeley LFS[4]. There is also a prototype LFS implementation on Linux[5].

日志结构文件系统以连续的数据流的形式（"日志"）将所有文件系统数据——包括文件内容和元数据——写入磁盘来达到同样的目的。使用这种方案查找一段数据的位置可能比在传统的文件系统中更复杂，但日志有一个很大的优势，即在日志中放置标记相对容易，这样一个标记的作用是表示在标记点之前的所有数据都在磁盘上提交且一致。写入这样的文件系统也特别快，因为日志的性质使得大多数写入都是在连续流中进行的，没有磁盘寻道。许多文件系统都是基于这种设计而编写的，包括 Sprite LFS[3]和 Berkeley LFS[4]。在 Linux 上也有一个 LFS 的原型实现[5]。

Finally, there is a class of atomically-updated file systems in which the old and new versions of incomplete updates are preserved by writing the new versions to a separate location on disk until such time as the update has be committed. After commit, the filesystem is free to write the new versions of the updated disk blocks back to their home locations on disk.

最后，还有一类原子更新的文件系统，是将新版本写入磁盘上的单独位置，直到提交更新。提交后，文件系统将更新的磁盘块的新版本写回其在磁盘上的原始位置。

This is the way in which "journaling" (sometimes referred to as "log enhanced") filesystems work. When metadata on the disk is updated, the updates are recorded in a separate area of the disk reserved for use as a journal. Filesystem transactions which complete have a commit record added to the journal, and only after the commit is safely on disk may the filesystem write the metadata back to its original location. Transactions are atomic because we can always either undo a transaction (throw away the new data in the journal) or redo it (copy the journal copy back to the original copy) after a crash, ac cording to whether or not the journal contains a commit record for the transaction. Many modern filesystems have adopted variations on this design.

这就是“日志”（有时被称为“日志增强”）文件系统的工作方式。当磁盘上的元数据被更新时，更新被记录在磁盘的一个单独区域中，该区域保留作为日志使用。文件系统事务完成后，会有一个提交记录添加到日志中，只有在提交安全地写入到磁盘上之后，文件系统才可以将元数据写回它的原始位置。事务是原子性的，因为我们总是可以在崩溃后撤消一个事务（丢弃日志中的新数据）或重做（将日志副本复制回原始副本），这取决于日志是否包含该事务的提交记录。许多现代文件系统都采用了这种设计的变体。

## 为 Linux 设计新的文件系统 Designing a new filesystem for Linux

The primary motivation behind this new filesystem design for Linux was to eliminate enormously long filesystem recovery times after a crash. For this reason, we chose a filesystem journaling scheme as the basis for the work. Journaling achieves fast filesystem recovery because at all times we know that all data which is potentially inconsistent on disk must be recorded also in the journal. As a result, filesystem recovery can be achieved by scanning the journal and copying back all committed data into the main filesystem area. This is fast because the journal is typically very much smaller than the full file system. It need only be large enough to record a few seconds-worth of uncommitted updates.

这个新的 Linux 文件系统设计背后的主要动机是为了缩短文件系统崩溃后漫长的恢复时间。为此，我们选择使用日志。日志实现了文件系统的快速恢复，因为我们始终知道，所有可能在磁盘上不一致的数据也必须记录在日志中。因此，文件系统的恢复可以通过扫描日志并将所有提交的数据复制回主文件系统区域来实现。这很快，因为日志通常比完整的文件系统小得多。它只需要大到足以记录几秒钟的未提交的更新。

The choice of journaling has another important ad vantage. A journaling filesystem differs from a tra ditional filesystem in that it keeps transient data in a new location, independent of the permanent data and metadata on disk. Because of this, such a file system does not dictate that the permanent data has to be stored in any particular way. In particular, it is quite possible for the ext2fs filesystem's on-disk structure to be used in the new filesystem, and for the existing ext2fs code to be used as the basis for the journaling version.

使用日志还有一个重要的优势。日志文件系统与传统文件系统的不同之处在于，它将临时数据保存在一个新的位置，与磁盘上的永久数据和元数据无关。正因为如此，这样的文件系统并没有规定永久数据必须以何种特定方式存储。特别是，ext2fs 文件系统的磁盘结构完全可以用于新的文件系统，而现有的 ext2fs 代码也可以作为日志版本的基础。

As a result, we are not designing a new filesystem for Linux. Rather, we are adding a new feature−transactional filesystem journaling−to the existing ext2fs.

因此，我们不是在为 Linux 设计一个新的文件系统。相反，我们正在为现有的 ext2fs 增加一个新的功能——事务性文件系统日志记录。

## 事务剖析 Anatomy of a transaction

A central concept when considering a journaled file system is the transaction, corresponding to a single update of the filesystem. Exactly one transaction results from any single filesystem request made by an application, and contains all of the changed meta data resulting from that request. For example, a write to a file will result in an update to the modification timestamp in the file's inode on disk, and may also update the length information and the block mapping information if the file is extended by the write. Quota information, free disk space and used block bitmaps will all have to be updated if new blocks are allocated to the file, and all this must be recorded in the transaction.

日志文件系统的一个核心概念是事务，一个事务对应于文件系统的一次更新。由应用程序提出的任何单一文件系统请求都会产生一个事务，并包含由该请求产生的所有变化的元数据。例如，对一个文件的写入将导致对磁盘上文件节点中的修改时间戳的更新，并且如果文件通过写入进行扩展，还可能更新长度信息和块映射信息。如果给文件分配了新的块，则配额信息、可用磁盘空间和已用块位图都必须更新，所有这些都必须记录在事务中。

There is another hidden operation in a transaction which we have to be aware about. Transactions also involve reading the existing contents of the filesystem, and that imposes an ordering between transactions. A transaction which modifies a block on disk cannot commit after a transaction which reads that new data and then updates the disk based on what it read. The dependency exists even if the two transactions do not ever try to write back the same blocks−imagine one transaction deleting a filename from one block in a directory and another transac tion inserting the same filename into a different block. The two operations may not overlap in the blocks which they write, but the second operation is only valid after the first one succeeds (violating this would result in duplicate directory entries).

在事务中还有一个隐藏的操作，我们必须要注意。事务还涉及到读取文件系统的现有内容，这就要求在事务之间进行排序。一个修改了磁盘上的块的事务不能在一个读取了新数据，然后根据它所读取的内容更新磁盘的事务之后提交。即使这两个事务没有尝试写回相同的块，这种依赖性也是存在的——想象一下，一个事务从目录中的一个块中删除一个文件名，而另一个事务将相同的文件名插入到一个不同的块中。这两个操作在写入的区块中可能不会重叠，但第二个操作只有在第一个操作成功后才有效（违反这一点将导致重复的目录条目）。

Finally, there is one ordering requirement which goes beyond ordering between metadata updates. Before we can commit a transaction which allocates new blocks to a file, we have to make absolutely sure that all of the data blocks being created by the transaction have in fact been written to disk (we term these data blocks dependent data). Missing out this requirement would not actually damage the in tegrity of the filesystem's metadata, but it could po tentially lead to a new file still containing a previous file contents after crash recovery, which is a security risk as well as being a consistency problem.

最后，还有一个排序要求，它的优先级高于元数据更新之间的排序。在提交将新块分配给文件的事务之前，我们必须绝对确保所有由事务创建的数据块事实上已经被写入磁盘（我们称这些数据块为依赖数据）。遗漏这一要求实际上不会损害文件系统元数据的完整性，但可能会导致新文件在崩溃恢复后仍包含以前的文件内容，这既是一个安全风险，也是一个一致性问题。

## 合并事务 Merging transactions

Much of the terminology and technology used in a journaled filesystem comes from the database world, where journaling is a standard mechanism for ensuring atomic commits of complex transac the traditional database transaction and a filesystem, and some of these allow us to simplify things enormously.

日志式文件系统中使用的许多术语和技术都来自于数据库领域，在那里，日志是确保复杂事务（传统数据库事务和文件系统）原子提交的标准机制，其中一些允许我们极大地简化事情。

Two of the biggest differences are that filesystems have no transaction abort, and all filesystem transactions are relatively short-lived. Whereas in a data base we sometimes want to abort a transaction half way through, discarding any changes we have made so far, the same is not true in ext2fs−by the time we start making any changes to the filesystem, we have already checked that the change can be completed legally. Aborting a transaction before we have started writing changes (for example, a create file operation might abort if it finds an existing file of the same name) poses no problem since we can in that case simply commit the transaction with no changes and achieve the same effect.

两者最大的区别是，文件系统没有事务中止，而且所有文件系统的事务都是相对短暂的。在数据库中，我们有时想在事务进行到一半的时候中止事务，放弃到目前为止所做的任何修改，而在 ext2fs 中则不然——当我们开始对文件系统做任何修改的时候，我们已经检查过该修改是否可以合法地完成。在开始写入更改之前中止事务（例如，一个创建文件的操作如果发现现有的同名文件就会中止）不会造成任何问题，因为在这种情况下，我们可以简单地提交没有任何更改的事务，并达到同样的效果。

The second difference−the short life term of filesystem transactions−is important since it means that we can simplify the dependencies between transactions enormously. If we have to cater for some very long term transactions, then we need to allow transactions to commit independently in any order as long as they do not conflict with each other, as otherwise a single stalled transaction could hold up the entire system. If all transactions are sufficiently quick, however, then we can require that transactions com mit to disk in strict sequential order without signifi cantly hurting performance.

第二个区别——文件系统事务的寿命很短——很重要，因为它意味着我们可以极大地简化事务之间的依赖关系。如果我们必须满足一些非常长期的事务，那么我们需要允许事务以任何顺序独立提交，只要它们不互相冲突，否则一个停滞不前的事务可能会阻碍整个系统的运行。然而，如果所有的事务都足够快，那么我们就可以要求事务以严格的顺序提交到磁盘，而不会显著影响性能。

With this observation, we can make a simplification to the transaction model which can reduce the com plexity of the implementation substantially while at the same time increasing performance. Rather than create a separate transaction for each filesystem up date, we simply create a new transaction every so often, and allow all filesystem service calls to add their updates to that single system-wide compund transaction.

通过这一观察，我们可以简化事务模型，从而大大降低实现的复杂性，同时提高性能。与其为每个文件系统更新都创建单独的事务，我们只需每隔一段时间创建一个新事务，并让所有文件系统服务调用将其更新添加到这个单一的系统整体事务中。

There is one great advantages of this mechanism. Because all operations within a compound transac tion will be committed to the log together, we do not have to write separate copies of any metadata blocks which are updated very frequently. In particular, this helps for operations such as creating new files, where typically every write to the file results in the file being extended, thus updating the same quota, bitmap blocks and inode blocks continuously. Any block which is updated many times during the life of a compound transaction need only be committed to disk once.

这种机制有一个很大的优点。因为复合事务中的所有操作都将一起提交到日志，所以我们不必为任何频繁更新的元数据块编写单独的副本。特别是，这有助于创建新文件等操作，通常情况下，对文件的每一次写入都会导致文件的大小增加，从而不断更新相同的配额、位图块和节点块。任何在复合事务期间被多次更新的块只需要提交到磁盘一次。

The decision about when to commit the current compound transaction and start a new one is a policy decision which should be under user control, since it involves a trade-off which affects system performance. The longer a commit waits, the more filesystem operations can be merged together in the log and so less IO operations are required in the long term. However, longer commits tie up larger amounts of memory and disk space, and leave a larger window for loss of updates if a crash occurs. They may also lead to storms of disk activity which make filesystem response times less predictable.

关于何时提交当前的复合事务并开始一个新的事务的决策应该由用户控制，因为它涉及到影响系统性能的权衡。提交等待的时间越长，日志中可以合并的文件系统操作就越多，因此从长远来看，需要的 IO 操作就越少。然而，较长的提交时间会占用大量的内存和磁盘空间，而且如果发生崩溃，会留下更大的更新丢失窗口。它们还可能导致磁盘活动风暴，使文件系统的响应时间更难预测。

## 磁盘上的表示 On-disk representation

The layout of the journaled ext2fs filesystem on disk will be entirely compatible with existing ext2fs kernels. Traditional UNIX filesystems store data on disk by associating each file with a unique num bered inode on the disk, and the ext2fs design al ready includes a number of reserved inode numbers. We use one of these reserved inodes to store the filesystem journal, and in all other respects the file system will be compatible with existing Linux kernels. The existing ext2fs design includes a set of compatibility bitmaps, in which bits can be set to indicate that the filesystem uses certain extensions. By allocating a new compatibility bit for the journaling extension, we can ensure that even though old kernels will be able to successfully mount a new, journaled ext2fs filesystem, they will not be permitted to write to the filesystem in any way.

磁盘上的日志式 ext2fs 文件系统的布局将与现有的 ext2fs 内核完全兼容。传统的 UNIX 文件系统通过将每个文件与磁盘上的一个唯一编号的 inode 相关联来存储数据，而 ext2fs 的设计也包括一些保留的 inode 编号。我们使用这些保留的 inode 之一来存储文件系统的日志，在所有其他方面，该文件系统将与现有的 Linux 内核兼容。现有的 ext2fs 设计包括一组兼容性位图，其中的位可以被设置来表示文件系统使用某些扩展。通过为日志扩展分配一个新的兼容性位，我们可以确保即使旧的内核能够成功地挂载一个新的、有日志的 ext2fs 文件系统，它们将不被允许以任何方式写入该文件系统。

## 文件系统日志的格式 Format of the filesystem journal

The journal file's job is simple: it records the new contents of filesystem metadata blocks while we are in the process of committing transactions. The only other requirement of the log is that we must be able to atomically commit the transactions it contains.

日志文件的工作很简单：在我们在提交事务的过程中，它记录文件系统元数据块的新内容。日志的唯一其他要求是，我们必须能够原子地提交它所包含的事务。

We write three different types of data blocks to the journal: metadata, descriptor and header blocks.

我们向日志写入三种不同类型的数据块：元数据、描述符和头块。

A journal metadata block contains the entire contents of a single block of filesystem metadata as up dated by a transaction. This means that however small a change we make to a filesystem metadata block, we have to write an entire journal block out to log the change. However, this turns out to be relatively cheap for two reasons:

日志中的元数据块包含由事务更新的单个文件系统元数据块的全部内容。这意味着，无论我们对文件系统元数据块所做的更改多么小，我们都必须用整个日志块来记录更改。然而，由于两个原因，这样做的代价是比较小的：

- Journal writes are quite fast anyway, since most writes to the journal are sequential, and we can easily batch the journal IOs into large clusters which can be handled efficiently by the disk controller;

  日志的写入是相当快的，因为大多数对日志的写入都是连续的，而且我们可以很容易地将日志 IO 批量化，形成大的集群，磁盘控制器可以有效地处理这些集群；

- By writing out the entire contents of the changed metadata buffer from the filesystem cache to the journal, we avoid having to do much CPU work in the journaling code.

  通过将更改后的元数据缓冲区的全部内容从文件系统缓存写入日志，我们避免了在日志代码中进行大量 CPU 工作。

The Linux kernel already provides us with a very efficient mechanism for writing out the contents of an existing block in the buffer cache to a different location on disk. Every buffer in the buffer cache is described by a structure known as a buffer_head, which includes information about which disk block the buffer's data is to be written to. If we want to write an entire buffer block to a new location with out disturbing the buffer_head, we can simply create a new, temporary buffer_head into which we copy the description from the old one, and then edit the device block number field in the temporary buffer head to point to a block within the journal file. We can then submit the temporary buffer_head directly to the device IO system and discard it once the IO is complete.

Linux 内核已经为我们提供了一个非常有效的机制，可以将缓冲区缓存中的现有块的内容写到磁盘上的不同位置。缓冲区缓存中的每一个缓冲区都由一个称为缓冲区头（buffer_head）的结构来描述，该结构包括有关要将缓冲区的数据写入哪个磁盘块的信息。如果要将整个缓冲区块写入新位置而不干扰缓冲区头，只需创建一个新的临时缓冲区头，将旧缓冲区头中的描述复制到其中，然后编辑临时缓冲区头中的设备块编号字段，指向日志文件中的一个块。然后，我们可以将临时缓冲头直接提交给设备 IO 系统，并在 IO 完成后丢弃它。

Descriptor blocks are journal blocks which describe other journal metadata blocks. Whenever we want to write out metadata blocks to the journal, we need to record which disk blocks the metadata normally lives at, so that the recovery mechanism can copy the metadata back into the main filesystem. A descriptor block is written out before each set of meta data blocks in the journal, and contains the number of metadata blocks to be written plus their disk block numbers.

描述符块是描述其他日志元数据块的日志块。每当我们想将元数据块写入日志时，我们都需要记录元数据通常位于哪个磁盘块，以便恢复机制可以将元数据复制回主文件系统。在日志中的每一组元数据块之前，都会写入一个描述符块，该块包含要写入的元数据块的数量及其磁盘块号。

Both descriptor and metadata blocks are written se quentially to the journal, starting again from the start of the journal whenever we run off the end. At all times, we maintain the current head of the log (the block number of the last block written) and the tail (the oldest block in the log which has not been unpinned, as described below). Whenever we run out of log space−the head of the log has looped back round and caught up with the tail−we stall new log writes until the tail of the log has been cleaned up to free more space.

描述符和元数据块都是按顺序写入日志的，每当我们从日志的开头运行到结尾时，就绕回日志的开头重新开始。在任何时候，我们都会维护日志的当前头部（最后写入的块的块号）和尾部（日志中最旧的块，尚未取消固定，如下所述）。每当我们的日志空间用完时——日志的头部绕了一圈，然后赶上了尾部——我们就会暂停新的日志写入，直到日志的尾部被清理掉以释放更多的空间。

Finally, the journal file contains a number of header blocks at fixed locations. These record the current head and tail of the journal, plus a sequence number. At recovery time, the header blocks are scanned to find the block with the highest sequence number, and when we scan the log during recovery we just run through all journal blocks from the tail to the head, as recorded in that header block.

最后，日志文件在固定位置包含一些头块。这些块记录了当前日志的头部和尾部，以及序号。在恢复时，扫描头块以找到序号最高的块，当我们在恢复过程中扫描日志时，我们只需按照头块中记录的那样，从头到尾遍历所有日志块。

## 日志的提交和检查点 Committing and checkpointing the journal

At some point, either because we have waited long enough since the last commit or because we are run ning short of space in the journal, we will wish to commit our outstanding filesystem updates to the log as a new compound transaction.

在某种程度上，要么因为我们自上次提交以来已经等待了足够长的时间，要么因为我们的日志空间不足，我们希望将未完成的文件系统更新作为一个新的复合事务提交到日志中。

Once the compound transaction has been com pletely committed, we are still not finished with it. We need to keep track of the metadata buffers re corded in a transaction so that we can notice when they get written back to their main locations on disk.

在复合事务被完全提交之后，我们仍然没有完成它。我们需要跟踪事务中的元数据缓冲区，以便我们能够注意到它们何时被写回磁盘上的主要位置。

Recall that when we commit a transaction, the new updated filesystem blocks are sitting in the journal but have not yet been synced back to their perma nent home blocks on disk (we need to keep the old blocks unsynced in case we crash before committing the journal). Once the journal has been committed, the old version on the disk is no longer important and we can write back the buffers to their home lo cations at our leisure. However, until we have fin ished syncing those buffers, we cannot delete the copy of the data in the journal.

回顾一下，当我们提交一个事务时，新更新的文件系统块在日志中，但尚未同步回磁盘上的主块上（我们需要保持旧块不被同步，以防在提交日志之前崩溃）。一旦提交日志，磁盘上的旧版本就不再重要，我们可以在空闲时将缓冲区写回其主位置。但是，在完成这些缓冲区的同步之前，我们不能删除日志中的数据副本。

To completely commit and finish checkpointing a transaction, we go through the following stages:

要完全提交并完成事务检查点，我们将经历以下阶段：

1. Close the transaction. At this point we make a new transaction in which we will record any filesystem operations which begin in the future. Any existing, incomplete operations will still use the existing transaction: we cannot split a single filesystem operation over multiple transactions!

   关闭事务。此时，我们将创建一个新事务，在该事务中，我们将记录从此开始的任何文件系统操作。任何现有的、不完整的操作仍将使用现有的事务：我们不能将一个文件系统的操作分割到多个事务中！

2. Start flushing the transaction to disk. In the context of a separate log-writer kernel thread, we begin writing out to the journal all meta data buffers which have been modified by the transaction. We also have to write out any dependent data at this stage (see the section above, Anatomy of a transaction). When a buffer has been committed, mark it as pinning the transaction until it is no longer dirty (it has been written back to the main stor age by the usual writeback mechanisms).

   开始向磁盘写入事务。在一个单独的日志写入器内核线程中，我们开始将事务修改过的所有元数据缓冲区写入日志。在这个阶段，我们还必须写入所有的附属数据（请参见上面的“事务剖析”一节）。提交缓冲区后，将其标记为固定事务，直到它已通过正常的写回机制写回主存储器。

3. Wait for all outstanding filesystem operations in this transaction to complete. We can safely start writing the journal before all operations have completed, and it is faster to allow these two steps to overlap to some extent.

   等待该事务中所有未完成的文件系统操作完成。我们可以在所有操作完成之前安全地开始写日志，而且允许这两个步骤在一定程度上重叠会更快。

4. Wait for all outstanding transaction updates to be completely recorded in the journal.

   等待所有未完成的事务更新全部记录在日志中。

5. Update the journal header blocks to record the new head and tail of the log, committing the transaction to disk.

   更新日志头块以记录新的日志头和尾，将事务提交到磁盘。

6. When we wrote the transaction's updated buffers out to the journal, we marked them as pinning the transaction in the journal. These buffers become unpinned only when they have been synced to their homes on disk. Only when the transaction's last buffer becomes unpinned can we reuse the journal blocks occupied by the transaction. When this occurs, write another set of journal headers recording the new position of the tail of the journal. The space released in the journal can now be reused by a later transaction.

   当我们把事务的更新缓冲区写到日志中时，我们将它们在日志中标记为固定事务。这些缓冲区只有在与磁盘上的主缓冲区同步后才能解除固定。只有当事务的最后一个缓冲区取消固定时，我们才能重新使用该事务占用的日志块。发生这种情况时，再写一组日志头，记录日志尾部的新位置。日志中释放的空间现在可以被后来的事务重新使用。

## 事务之间的冲突 Collisions between transactions

To increase performance, we do not completely sus pend filesystem updates when we are committing a transaction. Rather, we create a new compound transaction in which to record updates which arrive while we commit the old transaction.

为了提高性能，在提交事务时，我们不会完全暂停文件系统更新。相反，我们创建一个新的复合事务，在其中记录提交旧事务时发生的更新。

This leaves open the question of what to do if an up date wants access to a metadata buffer already owned by another, older transaction which is cur rently being committed. In order to commit the old transaction we need to write its buffer to the journal, but we cannot include in that write any changes which are not part of the transaction, as that would allow us to commit incomplete updates.

这就留下了一个悬而未决的问题：如果一个新事务想要访问另一个当前正在提交的旧事务已经拥有的元数据缓冲区，该怎么办？为了提交旧事务，我们需要将其缓冲区写入日志，但我们不能在写入过程中包含任何不属于事务的更改，因为这将允许我们提交不完整的更新。

If the new transaction only wants to read the buffer in question, then there is no problem: we have created a read/write dependency between the two trans actions, but since compound transactions always commit in strict sequential order we can safely ig nore the collision.

如果新事务只想读取相关的缓冲区，那么就没有问题了：我们在两个事务之间建立了一个读/写依赖关系，但是由于复合事务总是以严格的顺序提交，我们可以安全地避免冲突。

Things are more complicated if the new transaction wants to write to the buffer. We need the old copy of the buffer to commit the first transaction, but we cannot let the new transaction proceed without let ting it modify the buffer.

如果新事务想写入缓冲区，事情就比较复杂了。我们需要缓冲区的旧副本来提交第一个事务，但我们不能让新事务在不修改缓冲区的情况下继续进行。

The solution here is to make a new copy of the buffer in such cases. One copy is given to the new transaction for modification. The other is left owned by the old transaction, and will be committed to the journal as usual. This copy is simply deleted once that transaction commits. Of course, we cannot reclaim the old transaction's log space until this buffer has been safely recorded elsewhere in the filesystem, but that is taken care of automatically due to the fact that the buffer must necessarily be commit ted into the next transaction's journal records.

这里的解决方案是为缓冲区制作一个新的副本。一个副本给新事务进行修改。另一个由旧事务保留，并像往常一样提交到日志中。一旦该事务提交，这个副本就会被简单地删除。当然，在这个缓冲区被安全地记录在文件系统的其他地方之前，我们不能重新请求旧事务的日志空间，但是由于缓冲区必须被提交到下一个事务的日志记录中，这一点被自动处理了。

## 项目状态和未来工作 Project status and future work

This is still a work-in-progress. The design of the initial implementation is both stable and simple, and we do not expect any major revisions in design to be necessary in order to complete the implementation.

这仍然是一项正在进行的工作。初步实现的设计既稳定又简单，我们不希望为了完成这个实现而对设计进行任何重大的修改。

The design described above is relatively straightfor ward and will require minimal modifications to the existing ext2fs code other than the code to handle the management of the journal file, the association between buffers and transactions and the recovery of filesystems after an unclean shutdown.

上述设计相对简单，只需对现有的 ext2fs 代码进行少量修改，即可处理日志文件的管理、缓冲区和事务之间的关联以及在不干净的关闭后恢复文件系统。

Once we have a stable codebase to test, there are many possible directions in which we could extent the basic design. Of primary importance will be the tuning of the filesystem performance. This will re quire us to study the impact of arbitrary parameters in the journaling system such as commit frequencies and log sizes. It will also involve a study of bottle necks to determine if performance might be im proved through modifications to the design of the system, and several possible extensions to the de sign already suggest themselves.

一旦我们有了稳定的代码来测试，就有可以在许多可能的方向扩展基本设计。最重要的是调整文件系统性能。这将要求我们研究日志系统中各种参数的影响，如提交频率和日志大小。它还将涉及对瓶颈的研究，以确定是否可以通过修改系统的设计来提高性能，并且已经提出了几个可能的扩展设计。

One area of study may be to consider compressing the journal updates of updates. The current scheme requires us to write out entire blocks of metadata to the journal even if only a single bit in the block has been modified. We could compress such updates quite easily by recording only the changed values in the buffer rather than logging the buffer in its en tirety. However, it is not clear right now whether this would offer any major performance benefits. The current scheme requires no memory-to-memory copies for most writes, which is a big performance win in terms of CPU and bus utilisation. The IOs which result from writing out the whole buffers are cheap−the updates are contiguous and on modern disk IO systems they are transferred straight out from main memory to the disk controller without passing through the cache or CPU.

研究的一个领域可能是考虑压缩日志的更新。当前的方案要求我们将整个元数据块写入日志，即使块中只有一个位被修改。我们可以通过只在缓冲区中记录更改的值，而不是完整地记录缓冲区，来非常轻松地压缩此类更新。然而，目前尚不清楚这是否会带来任何重大性能优势。当前的方案对于大多数写操作不需要内存到内存的拷贝，这在 CPU 和总线利用率方面是一个巨大的性能优势。写出整个缓冲区的 IO 代价很小——更新是连续的，在现代磁盘 IO 系统中，它们直接从主内存传输到磁盘控制器，而不经过高速缓存或 CPU。

Another important possible area of extension is the support of fast NFS servers. The NFS design allows a client to recover gracefully if a server crashes: the client will reconnect when the server reboots. If such a crash occurs, any client data which the server has not yet written safely to disk will be lost, and so NFS requires that the server must not acknowledge completion of a client's filesystem request until that request has been committed to the server's disk.

另一个重要的可能扩展领域是对快速 NFS 服务器的支持。NFS 设计允许客户端在服务器崩溃时正常恢复：客户端将在服务器重新启动时重新连接。如果发生这种崩溃，服务器尚未安全写入磁盘的客户端数据都将丢失，因此 NFS 要求服务器在将客户端文件系统请求提交到服务器磁盘之前，不得确认该请求已完成。

This can be an awkward feature for general purpose filesystems to support. The performance of an NFS server is usually measured by the response time to client requests, and if these responses have to wait for filesystem updates to be synchronised to disk then overall performance is limited by the latency of on-disk filesystem updates. This contrasts with most other uses of a filesystem, where performance is measured in terms of the latency of in-cache up dates, not on-disk updates.

对于一般的文件系统来说，这可能是一个难以支持的特性。NFS 服务器的性能通常是由客户端请求的响应时间来衡量的，如果这些响应需要等待文件系统的更新同步到磁盘上，那么整体性能就会受到磁盘上文件系统更新的延迟的限制。这与文件系统的大多数其他用途形成鲜明对比，后者的性能是以缓存中的更新延迟来衡量的，而不是磁盘上的更新。

There are filesystems which have been specifically designed to address this problem. WAFL[6] is a transactional tree-based filesystem which can write updates anywhere on the disk, but the Calaveras filesystem[7] achieves the same end through use of a journal much like the one proposed above. The difference is that Calaveras logs a separate transaction to the journal for each application filesystem request, thus completing individual updates on disk as quickly as possible. The batching of commits in the proposed ext2fs journaling sacrifices that rapid commit in favour of committing several updates at once, gaining throughput at the expense of latency (the on-disk latency is hidden from applications by the effects of the cache).

有一些文件系统是专门为解决这个问题而设计的。WAFL[6]是一个基于事务树的文件系统，它可以在磁盘的任何地方写入更新信息，但 Calaveras 文件系统[7]通过使用类似于上面提出的日志来实现相同的目的。不同的是，Calaveras 为每个应用文件系统的请求向日志记录一个单独的事务，从而尽快完成磁盘上的单个更新。建议的 ext2fs 日志中的批量提交牺牲了快速提交，而支持一次提交多个更新，以牺牲延迟为代价获得吞吐量（磁盘上的延迟被缓存的效果所掩盖）。

Two ways in which the ext2fs journaling might be made more fit for use on an NFS server may be the use of smaller transactions, and the logging of file data as well as metadata. By tuning the size of the transactions committed to the journal, we may be able to substantially improve the turnaround for committing individual updates. NFS also requires that data writes be committed to disk as quickly as possible, and there is no reason in principle why we should not extend the journal file to cover writes of normal file data.

有两种方法可以使 ext2fs 日志更适合在 NFS 服务器上使用，即使用更小的事务，以及记录文件数据和元数据。通过调整提交到日志中的事务的大小，我们可能能够大大改善提交单个更新的周转时间。NFS 还要求尽快将数据写入磁盘，原则上我们没有理由不扩展日志文件以涵盖正常文件数据的写入。

Finally, it is worth noting that there is nothing in this scheme which would prevent us from sharing a single journal file amongst several different filesystems. It would require little extra work to allow multiple filesystems to be journaled to a log on a separate disk entirely reserved for the purpose, and this might give a significant performance boost in cases where there are many journaled filesystems all experiencing heavy load. The separate journal disk would be written almost entirely sequentially, and so could sustain high throughput without hurting the bandwidth available on the main filesystem disks.

最后，值得注意的是，这个方案中没有任何东西可以阻止我们在几个不同的文件系统中共享一个日志文件。只需要做一点额外的工作，就可以让多个文件系统将日志记录在一个单独的磁盘上，而这个磁盘完全是为这个目的保留的，如果有许多日志记录的文件系统都经历了重载，这可能会带来显著的性能提升。独立的日志磁盘几乎完全是按顺序写入的，因此可以维持高吞吐量，而不影响主文件系统磁盘上的可用带宽。

## 结论 Conclusions

The filesystem design outlined in this paper should offer significant advantages over the existing ext2fs filesystem on Linux. It should offer increased avail ability and reliability by making the filesystem re cover more predictably and more quickly after a crash, and should not cause much, if any, perfor mance penalty during normal operations.

本文概述的文件系统设计应该比 Linux 上现有的 ext2fs 文件系统具有明显的优势。它应该提供更高的可用性和可靠性，使文件系统在崩溃后能够更可预测、更快地恢复，并且在正常操作期间不会造成太多（如果有的话）性能损失。

The most significant impact on day-to-day perfor mance will be that newly created files will have to be synced to disk rapidly in order to commit the creates to the journal, rather than allowing the deferred writeback of data normally supported by the kernel. This may make the journaling filesystem unsuitable for use on /tmp filesystems.

对日常性能影响最大的是，新创建的文件必须快速同步到磁盘，以便将副本提交到日志，而不是允许内核通常支持的延迟写回数据。这可能会使日志文件系统不适合在/tmp 文件系统上使用。

The design should require minimal changes to the existing ext2fs codebase: most of the functionality is provided by a new journaling mechanism which will interface to the main ext2fs code through a simple transactional buffer IO interface.

这个设计应该只需要对现有的 ext2fs 代码库做最小的改动：大部分的功能是由一个新的日志机制提供的，它将通过一个简单的事务性缓冲区 IO 接口与 ext2fs 的主代码对接。

Finally, the design presented here builds on top of the existing ext2fs on-disk filesystem layout, and so it will be possible to add a transactional journal to an existing ext2fs filesystem, taking advantage of the new features without having to reformat the file system.

最后，这里提出的设计建立在现有 ext2fs 磁盘文件系统布局的基础上，因此有可能在现有 ext2fs 文件系统中增加一个事务性日志，利用新的功能而不必重新格式化文件系统。

## 参考文献 References

[1] A fast file system for UNIX. McKusick, Joy, Leffler and Fabry. ACM Transactions on Computer Systems, vol. 2, Aug. 1984

[2] Soft Updates: A Solution to the Metadata Up date Problem in File Systems. Ganger and Patt. Technical report CSE-TR-254-95, Com puter Science and Engineering Division, Uni versity of Michigan, 1995.

[3] The design and implementation of a log structured file system. Rosenblum and Oust erhout. Proceedings of the Thirteenth ACM Symposium on Operating Systems Principles, Oct. 1991

[4] An implementation of a log-structured file system for Unix. Seltzer, Bostic, McKusick and Staelin. Proceedings of the Winter 1993 USENIX Technical Conference, Jan. 1993

[5] Linux Log-structured Filesystem Project. Deuel and Cook. <http://collective.cpoint.net/prof/lfs/>

[6] File System Design for an NFS File Server Appliance. Dave Hitz, James Lau and Michael Malcolm. <http://www.netapp.com/technology/level3/30> 02.html#preface

[7] Metadata Logging in an NFS Server. Uresh Vahalia, Cary G. Gray, Dennis Ting. Pro ceedings of the Winter 1995 USENIX Techni cal Conference, 1995: pp. 265-276
