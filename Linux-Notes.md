---
title: Linux Notes
date: 2023-11-28 19:49:18
tags: 笔记
---



# 系统操作

1. ## 显示后台指令

​	1.提供所有进程信息

```shell
ps aux
```

2. ## 提供父进程信息

```shell
ps lax
```

## 	2. 终止某项进程

```shell
kill pid
```

* kill的意思实际上不是“杀死” ， kill 只是像程序发送一个信号

3. ## 查看硬件信息

* 查看内存占用和 swap 分区占用

```bash
free -h
```

* ## 查看CPU使用情况

```
top
```

# 文件操作

## vim 编辑器

![img](https://www.runoob.com/wp-content/uploads/2015/10/vi-vim-cheat-sheet-sch.gif)



## more

- 空白键 (space)：代表向下翻一页；
- Enter     ：代表向下翻『一行』；
- /字串     ：代表在这个显示的内容当中，向下搜寻『字串』这个关键字；
- :f      ：立刻显示出档名以及目前显示的行数；
- q       ：代表立刻离开 more ，不再显示该文件内容。
- b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。

## less

- 空白键  ：向下翻动一页；
- [pagedown]：向下翻动一页；
- [pageup] ：向上翻动一页；
- **/字串   ：向下搜寻『字串』的功能；**
- **?字串   ：向上搜寻『字串』的功能；**
- **n     ：重复前一个搜寻 (与 / 或 ? 有关！)**
- **N     ：反向的重复前一个搜寻 (与 / 或 ? 有关！)**
- q     ：离开 less 这个程序；

## head

​	读取前几行文本

```shell
	head -n 20 text.txt
```

## tail

​	与head相反

```shell
	tail -n 20 text.txt
```

## 压缩文件操作

* GZIP

## 	1.使用gzip进行解压缩

​	压缩：

```bash
gzip linuxFiles.tar
```

​	

​	解压缩： 

```
gunzip linuxFiles.tar.gz
gzip -d linuxFiles.tar.gz
```

2.文件打包tar

​	1.打包文件

```shell
tar -cvf linuxFiles.tar home/
```

​	2.解开打包后的文件

```shell
tar -xvf linuxFiles.tar
```

	3. tar -w 选项，每次将选择单个文件抽出或加入

```shell
tar -xvwf linuxFiles.tar //
```

4. tar -z 自动调用gzip进行解压缩

```shell
tar -czvf linuxFiles.tar.gz home/
等价于
tar -cvf linuxFiles.tar home/
gzip linuxFiles.tar
```

```shell
tar -xzvf linuxFiles.tar.gz home/
等价于
gzip -d linuxFiles.tar.gz
tar -xvf linuxFiles.tar
```

# 用户权限

![img](https://www.runoob.com/wp-content/uploads/2014/06/file-llls22.jpg)
