































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































### 基础概念

#### 通配符

在查询时可使用通配符进行处理

\*代表任意字符(0到多个)
**？** 代表一个字符
**[ ]** 中间为字符组合，仅匹配其中任一一个字符 

#### 文件/目录权限

每个文件/目录权限分三种，创建该文件的用户，该用户所在的属组，其他人

每种用户可进行的操作有三种r为读，w为写，x为可执行

每种操作1代表有，0代表无，三种操作通常连在一起（rwx）可组成一个二进制数字代表其权限操作，直接使用十进制表示。

## 文件管理类

### 文件信息查看

#### 显示目录列表ls

ls -a 列出目录所有文件，包含以.开始的隐藏文件
ls -A 列出除.及..的其它文件
ls -r 反序排列
ls -t 以文件修改时间排序
ls -S 以文件大小排序
ls -h 以易读大小显示
ls -l 除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来

#### 显示当前工作目录pwd

pwd 用于显示当前工作目录

#### 切换目录cd

cd / 进入目录

cd ~ 进入 "home" 目录

cd - 进入上一次工作路径

cd !$ 把上个命令的参数作为cd参数使用。

.代表当前目录

..代表上一层目录

#### 创建目录mkdir

- **-m**: 对新建目录设置存取权限，也可以用 chmod 命令设置;
- **-p**: 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后，系统将自动建立好那些尚不在的目录，即一次可以建立多个目录。

#### 复制目录或文件cp

cp 【源】【目的】

将源文件复制至目标文件，或将多个源文件复制至目标目录。

注意：命令行复制，如果目标文件已经存在会提示是否覆盖，而在 shell 脚本中，如果不加 -i 参数，则不会提示，而是直接覆盖！

-i 提示

-r 复制目录及目录内所有项目 

-b 复制时如果覆盖已存在目标文件则将目标文件备份自动备份

-r 递归复制目录下的所有内容

-a 复制的文件与原文件时间一样

#### 移动/重命名目录或文件mv

mv 【源】【目的】

-i 提示

-f 若目标存在则强制覆盖

-b 若目标存在则覆盖前备份

-v 显示文件移动信息

#### 删除目录或文件rm

-i 提示

-f 强制删除文件/目录

-r/-R 递归处理，目录下的全部删除

-v 显示删除具体信息

### 文件查找

#### 普通文件查找find

用于在指定目录下查找符合匹配条件的文件

```shell
find pathname -options [-print -exec -ok ...]
```

pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
-print： find命令将匹配的文件输出到标准输出。
-exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格。
-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

参数选项（可使用! -a -o连接多个可选控制参数分别代表非、与、或）：

```shell
-name 按照文件名查找文件
-perm 按文件权限查找文件
-user 按文件属主查找文件
-group  按照文件所属的组来查找文件。
-type  查找某一类型的文件，诸如：
   b - 块设备文件
   d - 目录
   c - 字符设备文件
   l - 符号链接文件
   p - 管道文件
   f - 普通文件

-size n :[c] 查找文件长度为n块文件，带有c时表文件字节大小
-amin n   查找系统中最后N分钟访问的文件
-atime n  查找系统中最后n*24小时访问的文件
-cmin n   查找系统中最后N分钟被改变文件状态的文件
-ctime n  查找系统中最后n*24小时被改变文件状态的文件
-mmin n   查找系统中最后N分钟被改变文件数据的文件
-mtime n  查找系统中最后n*24小时被改变文件数据的文件
(用减号-来限定更改时间在距今n日以内的文件，而用加号+来限定更改时间在距今n日以前的文件。 )
-maxdepth n 最大查找目录深度
-prune 选项来指出需要忽略的目录。在使用-prune选项时要当心，因为如果你同时使用了-depth选项，那么-prune选项就会被find命令忽略
-newer 如果希望查找更改时间比某个文件新但比另一个文件旧的所有文件，可以使用-newer选项
```

**实例：**

（1）查找 48 小时内修改过的文件

```
find -atime -2
```

（2）在当前目录查找 以 .log 结尾的文件。 **.** 代表当前目录

```
find ./ -name '*.log'
```

（3）查找 /opt 目录下 权限为 777 的文件

```
find /opt -perm 777
```

（4）查找大于 1K 的文件

```
find -size +1000c
```

（5）查找等于 1000 字符的文件

```
find -size 1000c 
```

（6）查找不是以.txt结尾的文件

```
find . ! -name "*.txt"
```

!代表非

（7）查找以x开头并小于13字节的文件

```
find . -name "x*" -a -size 13
```

-a 代表与

#### 程序文件查找whereis

 whereis 命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。whereis 及 locate 都是基于系统内建的数据库进行搜索，因此效率很高，而find则是遍历硬盘查找文件。 

```
-b   定位可执行文件。
-m   定位帮助文件。
-s   定位源代码文件。
-u   搜索默认路径下除可执行文件、源代码文件、帮助文件以外的其它文件。
```

**实例：**

（1）查找 mv 程序相关文件

```
whereis mv
```

#### 查找命令位置which

 which 是在 PATH 就是指定的路径中，搜索某个系统命令的位置，并返回第一个搜索结果。使用 which 命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。 

**实例：**

（1）查看 ls 命令是否存在，执行哪个

```
which ls
```

（2）查看 which

```
which which
```

（3）查看 cd

```
which cd（显示不存在，因为 cd 是内建命令，而 which 查找显示是 PATH 中的命令）
```

查看当前 PATH 配置：

```
echo $PATH
```

或使用 env 查看所有环境变量及对应值

### 文件内容查看

#### 显示文件内容cat

cat 文件名

-b 对非空输出行号

-n 输出所有行号

-s 缩减空行，如果有两个及其以上空行则缩减为一个空行

#### 逐页显示文件内容more/less

 功能类似于 cat, more 会以一页一页的显示方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示。 

+n      从笫 n 行开始显示
-n       定义屏幕大小为n行
+/pattern 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示 
-c       从顶部清屏，然后显示
-d       提示“Press space to continue，’q’ to quit（按空格键继续，按q键退出）”，禁用响铃功能
-l        忽略Ctrl+l（换页）字符
-p       通过清除窗口而不是滚屏来对文件进行换页，与-c选项相似
-s       把连续的多个空行显示为一行
-u       把文件内容中的下画线去掉

查看时的操作：

Enter    向下 n 行，需要定义。默认为 1 行
Ctrl+F   向下滚动一屏
空格键  向下滚动一屏
Ctrl+B  返回上一屏
=       输出当前行的行号
:f     输出文件名和当前行的行号
V      调用vi编辑器
!命令   调用Shell，并执行命令
q       退出more

less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。

### 文件编辑处理

#### 文件内容查找grep

  grep 命令用于查找文件里符合条件的字符串。 

grep 【参数】查找字符串 【文件/目录】

-A 显示行数

-n 显示匹配行及行号

-r/R 搜索整个目录树（递归）

如果查找多个字符串，查找字符串之间可以使用\隔开

#### 文件域排序sort

既然是排序，那么文件必须遵循某一特定的格式，文件由多个域组成，每个域按照空格进行隔开，将按照每行的域进行排序

 -f 排序时，将小写字母视为大写字母。 

 -d 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。 

 -n 依照数值的大小排序。 

 -u 意味着是唯一的(unique)，输出的结果是去完重了的。 

-k【域索引】域索引是一个数字，代表每一行的第几个域，按照该域进行排序，索引从1开始

 -m 将几个排序好的文件进行合并。 

###  文件/目录属性操作

#### 变更文件/目录权限chmod

有两种方式，十进制数字表示法直接赋予相应权限

```
chmod 777 file
```

或者按照参数使用符号：

chmod 【参数】mode 【文件/目录]

mode : 权限设定字串，格式如下 :

```
[ugoa...][[+-=][rwxX]...][,...]
```

其中：

- u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
- \+ 表示增加权限、- 表示取消权限、= 表示唯一设定权限。
- r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行。

常用参数：

-v : 显示权限变更的详细资料

-R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递回的方式逐个变更)

### 文件压缩与解压缩

#### gzip

压缩为gz格式的压缩文件，默认情况下是压缩，带参数-d则为解压缩

gzip 【参数】 文件/目录

 -d或--decompress或----uncompress 　解开压缩文件。 

 -v或--verbose 　显示指令执行过程。 

-r 压缩目录下的所有文件（递归）

#### 打包/解包tar

 tar是用来建立，还原备份文件的工具程序，它可以加入，解开备份文件内的文件。 

tar 【参数】包文件 待打包文件

 -v或--verbose 　显示指令执行过程。 

-c 建立新的备份文件，即源文件（待打包文件）仍保存

-t 列出包文件里的内容

-f 指定备份文件名，后面跟着备份文件名

-x 解包文件，后面跟着解包文件名

































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































































-z  通过gzip指令处理备份文件。 

-C 解包时解包到指定目录下，后面跟着指定目录

**实例**

压缩文件 非打包

```
# touch a.c       
# tar -czvf test.tar.gz a.c   //压缩 a.c文件为test.tar.gz
a.c
```

列出压缩文件内容

```
# tar -tzvf test.tar.gz 
-rw-r--r-- root/root     0 2010-05-24 16:51:59 a.c
```

解压文件

```
# tar -xzvf test.tar.gz 
a.c
```

## 进程管理类

### 查看进程

#### 查看进程信息ps

 ps命令用于显示当前进程 (process) 的状态。 

ps 【参数】

- -A 列出所有的进程
- -w 显示加宽可以显示较多的资讯

使用ps显示的信息为：

- USER: 进程拥有者
- PID: pid
- %CPU: 占用的 CPU 使用率
- %MEM: 占用的内存使用率
- VSZ: 占用的虚拟内存大小
- RSS: 占用的内存大小
- TTY: 终端的次要装置号码 (minor device number of tty)
- STAT: 该进程的状态: *
  * D: 无法中断的休眠状态 (通常 IO 的进程)
  * R: 正在执行中
  * S: 静止状态
  * T: 暂停执行
  * Z: 不存在但暂时无法消除
  * W: 没有足够的内存分页可分配
  * <: 高优先序的进程
  * N: 低优先序的进程
  * L: 有内存分页分配并锁在内存内 (实时系统或捱A I/O)
- START: 进程开始时间
- TIME: 执行的时间
- COMMAND:所执行的指令

#### 查看进程树pstree

 pstree命令将所有进程以树状图显示，树状图将会以 pid (如果有指定) 或是以 init 这个基本行程为根 (root)，如果有指定使用者 id，则树状图会只显示该使用者所拥有的进程。

pstree 【参数】

#### 动态查看进程运行状况top

  top命令用于实时显示 process 的动态。 每10s更新一次，按CPU使用率排序

### 进程控制

#### 删除进程/程序kill

kill命令用于删除执行中的程序或工作。

kill可将指定的信息送至程序。预设的信息为SIGTERM(15)，可将指定程序终止。若仍无法终止该程序，可使用SIGKILL(9)信息尝试强制删除程序。程序或工作的编号可利用ps指令或jobs指令查看。

也就是说，kill实际上是向进程发送一个信号，进程根据该信号进行相应处理，默认为15，强制删除为9

kill 【参数】进程号/程序名称

-l 列出所有可发送的信号

-l【数字】后面跟着一个数字代表向进程发送的信号

**实例**

杀死进程

```
# kill 12345
```

强制杀死进程

```
# kill -KILL 123456
```

#### 后台运行命令&

在命令的后面加上&就代表后台运行命令，通常与nohup一起使用

```
command  >out.file  2>&1  & 
```

 所有的标准输出和错误输出都将被重定向到一个叫做out.file 的文件中 

 2>&1 是将标准出错重定向到标准输出，这里的标准输出已经重定向到了out.file文件，即将标准出错也输出到out.file文件中。

#### 退出时继续执行命令nohup

通常与&一起使用，将命令放在后台并且不挂起。

 当用户注销（logout）或者网络断开时，终端会收到 HUP（hangup）信号从而关闭其所有子进程。nohup 的用途就是让提交的命令忽略 hangup 信号。 

```bash
nohup command > myout.file 2>&1 &
```

 2>&1 是将标准出错重定向到标准输出，这里的标准输出已经重定向到了out.file文件，即将标准出错也输出到out.file文件中。

#### 杀死nohup进程

使用ps ux查看进程

使用kill -9 PID杀死进程

## I/O管理类

### 输入输出重定向操作符

#### 输出重定向> >>

\>与\>>都是将原本送往显示器的输出送往其它地方（通常是文件），\>会覆盖原有内容，\>>则是追加原有内容

#### 输入重定向< <<

<与<<都是将从键盘的输入送往其它地方（通常是文件），\<后面跟着文件名代表从文件中读取数据

<<的用法为<<tag，即将tag之前的内容当做输入，如：

```shell
cat <<EOF >newFile
```

输入的EOF之前的内容都输出到新文件newFile中

### 管道操作

#### 管道操作符|

|用来连接多个命令，将一条命令的输出作为另一条命令的输入

### 远程发送

使用sz与rz

- sz中的s意为send（发送），告诉客户端，我（服务器）要发送文件 send to cilent，就等同于客户端在下载。
- rz中的r意为received（接收），告诉客户端，我（服务器）要接收文件 received by cilent，就等同于客户端在上传。

 记住一点，不论是send还是received，**动作都是在服务器上发起的**。