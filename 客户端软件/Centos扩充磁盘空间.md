# Centos扩充磁盘空间

一般情况下使用VMWare创建Centos虚拟系统时，分配的磁盘大小不能满足需求，需要增加磁盘大小

```
第一步：
		①关闭Centos系统
​		②在VMWare上手动添加一块磁盘，磁盘类型SCSI,磁盘大小10G
第二步：
		①启动Centos系统
		②命令fdisk -l，查看是否出现刚刚加入的磁盘/dev/sdb
		③分区：
			①命令fdisk /dev/sdb
			m
			n
			接下来默认就好，回车
			w，保存
		④创建物理卷：
			pvcreate /dev/sdb1
		⑤命令pvdisplay，查看VG Name对应的组名XXX（新的物理卷接下来要加入到这个组）
		⑥添加新的物理卷到XXX卷组
			①使用命令：vgextend XXX /dev/sdb1
			②验证是否加入成功：
				使用命令pvdisplay，确定新添加的物理卷的VG Name也显示XXX
		⑦增加磁盘空间
			lvextend -L +9.99G /dev/mapper/XXX-root
			注意：
			①/dev/mapper/XXX-root 是磁盘分区的文件系统名，可以通过df -h查看
			②+9.99G，表示往文件系统增加9.99G(一定要<10G，否则不成功)
			
			这一步可能出错,按照以下排查：
			①删除无效的物理卷：
				vgreduce --removemissing --force 卷组名
			③删除无效的逻辑卷：
				lvremove --force 卷组名 逻辑卷名
		⑧重新识别分区大小
			xfs_growfs /dev/mapper/XXX-root
		⑨验证是否成功：
			df -h，观察磁盘容量
```

​		