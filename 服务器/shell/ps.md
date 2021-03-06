# 1. ps
## 1.1. ps是什么
ps命令能够给出当前系统中进程的快照。它能捕获系统在某一事件的进程状态。
如果你想不断更新查看的这个状态，可以使用top命令。

## 1.2. 常用命令
### 1.2.1. 不加参数执行ps命令
这是一个基本的 ps 使用。在控制台中执行这个命令并查看结果：
```shell
14:11:42 Brook3@centos7 /workspace/demo/Brook3.api.php$ ps
  PID TTY          TIME CMD
10967 pts/0    00:00:00 bash
25217 pts/0    00:00:00 ps
```
结果默认会显示4列信息：
* PID: 运行着的命令(CMD)的进程编号
* TTY: 命令所运行的位置（终端）
* TIME: 运行着的该命令所占用的CPU处理时间
* CMD: 该进程所运行的命令
这些信息在显示时未排序。

### 1.2.2. 显示所有当前进程
使用 -a 参数。-a 代表 all。同时加上x参数会显示没有控制终端的进程。
```shell
ps -ax
```
这个命令的结果或许会很长。为了便于查看，可以结合less命令和管道来使用。
```shell
ps -ax | less
```

### 1.2.3. 根据用户过滤进程
在需要查看特定用户进程的情况下，我们可以使用 -u 参数。比如我们要查看用户'Brook3'的进程，可以通过下面的命令：
```shell
ps -u Brook3
```

### 1.2.4. 通过cpu和内存使用来过滤进程
也许你希望把结果按照 CPU 或者内存用量来筛选，这样你就找到哪个进程占用了你的资源。要做到这一点，我们可以使用 aux 参数，来显示全面的信息：
```shell
ps -aux | less
```
默认的结果集是未排好序的。可以通过 --sort命令来排序。
根据 CPU 使用来升序排序：
```shell
ps -aux --sort -pcpu | less
```
根据 内存使用 来升序排序：
```shell
ps -aux --sort -pmem | less
```
我们也可以将它们合并到一个命令，并通过管道显示前10个结果：
```shell
ps -aux --sort -pcpu,+pmem | head -n 10
```

### 1.2.5. 通过进程名和PID过滤
使用 -C 参数，后面跟你要找的进程的名字。比如想显示一个名为getty的进程的信息，就可以使用下面的命令：
```shell
ps -C getty
```
如果想要看到更多的细节，我们可以使用-f参数来查看格式化的信息列表：
```shell
ps -f -C getty
```

### 1.2.6. 根据线程来过滤进程
如果我们想知道特定进程的线程，可以使用-L 参数，后面加上特定的PID。
```shell
ps -L 1213
```

### 1.2.7. 树形显示进程
有时候我们希望以树形结构显示进程，可以使用 -axjf 参数。
```shell
ps -axjf
```
或者可以使用另一个命令。
```shell
pstree
```

### 1.2.8. 显示安全信息
如果想要查看现在有谁登入了你的服务器。可以使用ps命令加上相关参数：
```shell
ps -eo pid,user,args
```
参数 -e 显示所有进程信息，-o 参数控制输出。Pid,User 和 Args参数显示PID，运行应用的用户和该应用。
能够与-e 参数 一起使用的关键字是args, cmd, comm, command, fname, ucmd, ucomm, lstart, bsdstart 和 start。

### 1.2.9. 格式化输出root用户（真实的或有效的UID）创建的进程
系统管理员想要查看由root用户运行的进程和这个进程的其他相关信息时，可以通过下面的命令：
```shell
ps -U root -u root u
```
-U 参数按真实用户ID(RUID)筛选进程，它会从用户列表中选择真实用户名或 ID。真实用户即实际创建该进程的用户。
-u 参数用来筛选有效用户ID（EUID）。
最后的u参数用来决定以针对用户的格式输出，由User, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME 和 COMMAND这几列组成。

### 1.2.10. 使用PS实时监控进程状态
ps 命令会显示你系统当前的进程状态，但是这个结果是静态的。
当有一种情况，我们需要像上面第四点中提到的通过CPU和内存的使用率来筛选进程，并且我们希望结果能够每秒刷新一次。为此，我们可以将ps命令和watch命令结合起来。
```shell
watch -n 1 'ps -aux --sort -pmem, -pcpu'
```

## 1.3. 差异
### 1.3.1. ps aux 和ps -ef
两者的输出结果差别不大，但展示风格不同。
aux是BSD风格，-ef是System V风格。这是次要的区别，一个影响使用的区别是aux会截断command列，而-ef不会。当结合grep时这种区别会影响到结果。