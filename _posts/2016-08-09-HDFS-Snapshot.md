---
layout: post
title: HDFS Snapshot
date: 2016-08-09
categories: blog
tags:
description:
---

## 简介
HDFS中可以对目录创建Snapshot，创建之后不管后续目录发生什么变化，都可以通过snapshot找回原来的文件和目录结构。
为了启用这种功能，首先需要启用目标目录的snapshot功能，可以通过下面的命令来执行：

    hdfs dfsadmin -allowSnapshot

启用snapshot功能，该操作会向INodeDirectory中添加Snapshottable特性，并将该目录加入到SnapshottableManager中。并不会自动进行snapshot保存，还需要先创建snapshot, 通过下面的命令来执行：

    hdfs dfs -createSnapshot []

可以为相同的目录创建多个snapshot, 不同的snapshot通过名字来区分，默认是syyyyMMdd-HHmmss.SSS，例如/storage/WALs/.snapshot/s20140515-084657.639

## 特点和限制

1. Snapshots可以发生在设置成snapshottable的所有目录
2. 一个snapshottable的目录最多支持65536个快照
3. 一个目录被设置成snapshottable并且拥有快照，就不能删除和重命名该目录
4. 升级的时候为了防止.snapshot保留目录造成的冲突，需要删除或者重命名该目录。

![快照相关类图](http://obkkz880t.bkt.clouddn.com/class_pic.png)

一个Snapshot代表了一个Snapshottable。其中的DirectorySnapshottableFeature为了支持一个目录为了支持Sanshottable。DirectoryWithSnapshotFeature和FileWithSnapshotFeature分别为INodeDirectory和INodeFile支持了快照特性，利用一个链表来保存对该文件或者目录在进行快照之后执行的操作。保存的信息有目录的属性和配额、以及文件的属性和文件的长度和对应的数据块等信息。具体属性信息如下所示：
INodeAttributes
包括了文件或者目录的基本信息，有：

    private final byte[] name;
    private final long permission;（保存了user，group和 permission）
    private final AclFeature aclFeature;
    private final long modificationTime;
    private final long accessTime;
    private XAttrFeature xAttrFeature;
    INodeFileAttributes
    private final long header;（副本数、块大小，存储类型）

    INodeDirectoryAttributes
    QuotaCounts quota;（如果有）

增加快照的特性并没有改变原来的对外可见的文件和基于树状结构的文件访问路径（除去.snapshot目录），如果为该目录创建了一个Snapshottable，则在该目录下面创建.snapshot目录，用于保存现在的文件和目录做了哪些操作（可以通过这部分操作从当前的文件和目录数据和信息得到原来创建该快照的文件或者目录的状态）。

## 流程

### 删除文件或者目录
1. 文件目录树如下图所示，并且我们已经通过命令启动了a的snapshot功能，结构如下图所示：
![allowSnapshot之后](http://obkkz880t.bkt.clouddn.com/before_create.png)
图中.snapshot是虚拟节点，保存了所有的snapshot列表，其中diff中还保存当前节点下面的变化，一个snapshot对应于一个diff.要注意的是snapshot中可以被多个目录的diff引用，后续会进行说明。

2. 当我们执行createSnapshot命令时，结果如下：
![createSnapshot之后](http://obkkz880t.bkt.clouddn.com/after_create.png)

3. 当删除文件e的时候
不论是删除一个文件还是一个目录，只要是直接子节点，都会将节点添加快照特性.例如e会添加FileWithSnapshotFeature，在a的DirectoryDiff中ChildDiff中deleted列表中将会包含e,而在a的正常节点下会被删除。目录节点的处理同样。
![删除文件之后](http://obkkz880t.bkt.clouddn.com/after_delete.png)

其他修改操作，在操作之前增加一个该文件的diff，通过该方法来保存文件的信息

truncate之后的追加操作：
当文件处于快照下的时候，对该文件进行文件的截短的时候，如果截短的位置正好处于Block的边界，则将这些块直接从该文件的INodeFile中删除（该文件的FileDiff中中保留了这以部分Block信息），如果不与Block的边界对齐，则执行该块的拷贝，以保存原来快照的数据:


    boolean shouldCopyOnTruncate(INodeFile file, BlockInfoContiguous blk) {
        if(!isUpgradeFinalized()) {
            return true;
        }
        if (isRollingUpgrade()) {
            return true;
        }
        return file.isBlockInLatestSnapshot(blk);
    }
