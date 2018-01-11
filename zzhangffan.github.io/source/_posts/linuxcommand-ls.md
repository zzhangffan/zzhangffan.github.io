---
title: Linux命令系列-ls
date: 2018-01-11 10:19:41
categoreies:
tags: Linux
---
> 此为Linux命令使用系列记录，博主使用到了哪些命令就写了哪些，命令的参数也记录不全，用到的就写了，需要完整学习，请参考[Linux命令大全(手册)_Linux常用命令行实例详解_Linux命令学习手册](http://man.linuxde.net/)

## **ls命令**
ls命令用来显示目标列表
## **语法**
> ls  (选项)  (参数)

## **参数**
目录：指定要显示列表的目录，也可以是具体的文件。

## **示例**
* ___列出当前目录下非隐藏文件及目录___
> [root@hostname ~]#  ls
&emsp;

&emsp;&emsp;<span style="color: red;">PS:我的root用户的home目录下没有非隐藏文件及目录，看看你的执行结果吧</span>

* ___列出当前目录下所有文件及目录___
> [root@hostname ~]#  ls  -a
.   .bash_history  .bash_profile  .cache  .npm  .pydistutils.cfg  .tcshrc
..  .bash_logout   .bashrc        .cshrc  .pip  .ssh

&emsp;&emsp;<span style="color: red;">PS:Linux中以.开头的为隐藏文件</span>

* ___以长格式列出当前目录下所有文件及目录___
> [root@hostname ~]# ls -a -l
total 60
dr-xr-x---.   6 root root  4096 Jan  1 09:57 .
dr-xr-xr-x.  18 root root  4096 Dec 31 14:45 ..
-rw-------    1 root root  1403 Jan 11 09:39 .bash_history
-rw-r--r--.   1 root root    18 Dec 29  2013 .bash_logout
-rw-r--r--.   1 root root   176 Dec 29  2013 .bash_profile
-rw-r--r--.   1 root root   176 Dec 29  2013 .bashrc
drwx------    3 root root  4096 Oct 15 23:24 .cache
-rw-r--r--.   1 root root   100 Dec 29  2013 .cshrc
drwxr-xr-x  237 base base 12288 Jan  1 09:58 .npm
drwxr-xr-x    2 root root  4096 Oct 15 23:24 .pip
-rw-r--r--    1 root root    64 Oct 15 23:24 .pydistutils.cfg
drwx------    2 root root  4096 Dec 31 14:45 .ssh
-rw-r--r--.   1 root root   129 Dec 29  2013 .tcshrc

&emsp;&emsp;<span style="color: red;">PS：[命令结果解析](/post_sources/linuxcommands/ls-result.xml)</span>

* ___按最近一次修改的时间排序显示___
> [root@hostname ~]#  ls  -t
.bash_history  .     ..    .pydistutils.cfg  .bash_logout   .bashrc  .tcshrc
.npm           .ssh  .pip  .cache            .bash_profile  .cshrc

* ___指定目录显示___
> [root@hostname tmp]#  ls  test
son  test.txt

* ___遍历目录显示___
> [root@hostname tmp]#  ls  test -R
>test:
>son  test.txt
>&emsp;
>test/son:
>sontest.txt

## **选项** <span style="color: red;">（网上复制的，个人觉得死记没用，放在最后）</span>
> -a：显示所有档案及目录（ls内定将档案名或目录名称为“.”的视为影藏，不会列出）；
-A：显示除影藏文件“.”和“..”以外的所有文件列表；
-C：多列显示输出结果。这是默认选项；
-l：与“-C”选项功能相反，所有输出信息用单列格式输出，不输出为多列；
-F：在每个输出项后追加文件的类型标识符，具体含义：“*”表示具有可执行权限的普通文件，“/”表示目录，“@”表示符号链接，“|”表示命令管道FIFO，“=”表示sockets套接字。当文件为普通文件时，不输出任何标识符；
-b：将文件中的不可输出的字符以反斜线“”加字符编码的方式输出；
-c：与“-lt”选项连用时，按照文件状态时间排序输出目录内容，排序的依据是文件的索引节点中的ctime字段。与“-l”选项连用时，则排序的一句是文件的状态改变时间；
-d：仅显示目录名，而不显示目录下的内容列表。显示符号链接文件本身，而不显示其所指向的目录列表；
-f：此参数的效果和同时指定“aU”参数相同，并关闭“lst”参数的效果；
-i：显示文件索引节点号（inode）。一个索引节点代表一个文件；
--file-type：与“-F”选项的功能相同，但是不显示“*”；
-k：以KB（千字节）为单位显示文件大小；
-l：以长格式显示目录下的内容列表。输出的信息从左到右依次包括文件名，文件类型、权限模式、硬连接数、所有者、组、文件大小和文件的最后修改时间等；
-m：用“,”号区隔每个文件和目录的名称；
-n：以用户识别码和群组识别码替代其名称；
-r：以文件名反序排列并输出目录内容列表；
-s：显示文件和目录的大小，以区块为单位；
-t：用文件和目录的更改时间排序；
-L：如果遇到性质为符号链接的文件或目录，直接列出该链接所指向的原始文件或目录；
-R：递归处理，将指定目录下的所有文件及子目录一并处理；
--full-time：列出完整的日期与时间；
--color[=WHEN]：使用不同的颜色高亮显示不同类型的。