[#]: collector: (lujun9972)
[#]: translator: (zgj1024)
[#]: reviewer: ( )
[#]: publisher: ( )
[#]: url: ( )
[#]: subject: (How to set ulimit and file descriptors limit on Linux Servers)
[#]: via: (https://www.linuxtechi.com/set-ulimit-file-descriptors-limit-linux-servers/)
[#]: author: (Shashidhar Soppin https://www.linuxtechi.com/author/shashidhar/)

如何在 Linux 服务器上设置 ulimit 和文件描述符数限制
======
 **简介**：如今每天（遇到的）像是打开文件数这类的挑战在任何生产环境已经变得司空见惯了。因为许多基于 Java 和 Apache 的应用程序的正在安装和配置，可能会导致打开过多的文件、文件描述符等。如果（文件描述符等）超过默认设置限制，就可能会面临访问控制问题和打开文件的挑战。许多生产环境因此而陷入停滞状态。

<https://www.linuxtechi.com/wp-content/uploads/2019/06/ulimit-number-openfiles-linux-server.jpg>

幸运的是，在任何基于 Linux 的服务器上，都有 “**ulimit**”命令，通过它可以查看/设置/获取文件打开的状态/配置详情的数量。此命令配备了许多选项，通过此组合可以设置打开文件的数量。下面是分步命令，并用示例详细说明。


### 查看任何 Linux 系统中当前打开文件数的限制

要在任何 Linux 服务器上获得打开文件数限制，请执行以下命令，

```
[root@linuxtechi ~]# cat /proc/sys/fs/file-max
146013
```

上面的数字表明用户可以在每个用户登录会话中打开 ‘146013’ 个文件。

```
[root@linuxtechi ~]# cat /proc/sys/fs/file-max
149219
[root@linuxtechi ~]# cat /proc/sys/fs/file-max
73906
```

这清楚地表明，各个 Linux 操作系统具有不同数量的打开文件。这基于各自系统中运行的依赖关系和应用程序。

### ulimit 命令 :

顾名思义，ulimit(用户限制)用于显示和设置登录用户的资源限制。当我们使用 -a 选项运行 ulimit 命令时，它将打印登录用户的所有资源限制。现在让我们在Ubuntu/Debian 和 CentOS 系统上运行 “**ulimit -a**”，


**Ubuntu / Debian 系统** ,

```
root@linuxtechi ~}$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 5731
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 5731
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

**CentOS 系统**

```
root@linuxtechi ~}$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 5901
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 5901
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

正如我们可以在这里看到的，不同的操作系统有不同的限制设置。所有这些限制都可以使用 “ulimit” 命令进行配置/更改。

要显示单个资源限制可以在 ulimit 命令中传递特定的参数，下面列出了一些参数：

  * ulimit -n –> It will display number of open files limit
  * ulimit -c –> It display the size of core file
  * umilit -u –> It will display the maximum user process limit for the logged in user.
  * ulimit -f –> It will display the maximum file size that the user can have.
  * umilit -m –> It will display the maximum memory size for logged in user.
  * ulimit -v –> It will display the maximum memory size limit


使用以下命令检查登录用户打开文件数量的硬限制和软限制

```
root@linuxtechi ~}$ ulimit -Hn
1048576
root@linuxtechi ~}$ ulimit -Sn
1024
```

### 如何修复达到最大文件数限制的问题 ?

Let’s assume our Linux server has reached the limit of maximum number of open files and want to extend that limit system wide, for example we want to set 100000 as limit of number of open files.

让我们假设我们的 Linux 服务器已经达到了打开文件的最大数量限制，并希望将该限制扩展到整个系统，例如，我们希望将 100000 设置为打开文件数量的限制。

```
root@linuxtechi~]# sysctl -w fs.file-max=100000
fs.file-max = 100000
```

上述更改只会在下次重启之前有效，因此要使这些更改在重启后仍存在，请编辑文件 **/etc/sysctl.conf** 并添加相同的参数，

```
root@linuxtechi~]# vi /etc/sysctl.conf
fs.file-max = 100000
```

保存并推出文件

运行下面命令，使上述更改立即生效，而无需注销和重新启动。

```
root@linuxtechi~]# sysctl -p
```


现在验证新的更改是否生效。

```
root@linuxtechi~]# cat /proc/sys/fs/file-max
100000
```


使用以下命令找出当前正在使用的文件描述符数量：

```
[root@linuxtechi ~]# more /proc/sys/fs/file-nr
1216    0       100000
```

注意：-命令 “**sysctl-p**” 用于在不重新启动和注销的情况下提交更改。
### 通过 limit.conf 文件设置用户级资源限制

“**/etc/sysctl.conf**” 文件用于设置系统范围的资源限制，但如果要为 Oracle、MariaDB 和 Apache 等特定用户设置资源限制，则可以通过 “**/etc/security/limits.conf**” 文件去实现。

示例 Limit.conf 如下所示，

```
root@linuxtechi~]# cat /proc/sys/fs/file-max
```

![Limits-conf-linux-part1][1]

![Limits-conf-linux-part2][2]

假设我们要为 linuxtechi 用户设置打开文件数量的硬限制和软限制，而对于 oracle 用户设置打开进程数量的硬限制和软限制，编辑文件 “/etc/security/limits.conf” 并添加以下行

```
# hard limit for max opened files for linuxtechi user
linuxtechi       hard    nofile          4096
# soft limit for max opened files for linuxtechi user
linuxtechi       soft    nofile          1024

# hard limit for max number of process for oracle user
oracle           hard    nproc          8096
# soft limit for max number of process for oracle user
oracle           soft    nproc          4096
```

保存并退出文件

**注意：** 如果你想对一个组而不是用户进行资源限制，那么也可以通过limit.conf文件，输入 **@<Group_name>** 代替用户名，其余项都是相同的，示例如下，

```
# hard limit for max opened files for sysadmin group
@sysadmin        hard         nofile            4096
# soft limit for max opened files for sysadmin group
@sysadmin        soft         nofile            1024
```

验证新的更改是否生效

```
~]# su - linuxtechi
~]$ ulimit -n -H
4096
~]$ ulimit -n -S
1024

~]# su - oracle
~]$ ulimit -H -u
8096
~]$ ulimit -S -u
4096
```

注：其他主要使用的命令是 “[ **lsof**][3]” 用于找出“当前打开了多少个文件”，这命令对管理员非常有帮助。

**结尾：**

正如在介绍部分提到的，“ulimit” 命令非常强大，可以帮助用户配置并确保应用程序安装更加流畅，没有任何瓶颈。此命令有助于修复基于 Linux 的服务器中的（打开）大量文件的限制。 

--------------------------------------------------------------------------------

via: https://www.linuxtechi.com/set-ulimit-file-descriptors-limit-linux-servers/

作者：[Shashidhar Soppin][a]
选题：[lujun9972][b]
译者：[zgj1024](https://github.com/zgj1024)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]: https://www.linuxtechi.com/author/shashidhar/
[b]: https://github.com/lujun9972
[1]: https://www.linuxtechi.com/wp-content/uploads/2019/06/Limits-conf-linux-part1-1024x677.jpg
[2]: https://www.linuxtechi.com/wp-content/uploads/2019/06/Limits-conf-linux-part2-1024x443.jpg
[3]: https://www.linuxtechi.com/lsof-command-examples-linux-geeks/
