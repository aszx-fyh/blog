## 常用命令

-   查看端口

> netstat -anp |grep 端口号

-   查看软件安装路径

> whereis nginx

-   查询运行文件所在路径

> which nginx

### sudo xx

> Linux sudo 命令以系统管理者的身份执行指令，也就是说，经由 sudo 所执行的指令就好像是 root 亲自执行。

### yum

> yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。

### [apt](https://blog.csdn.net/liudsl/article/details/79200134)

> apt（Advanced Packaging Tool）是一个在 Debian 和 Ubuntu 中的 Shell 前端软件包管理器。

### ubuntu 安装 nodejs

> curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
> sudo apt install -y nodejs

### ~和/的区别

> ”/“是根目录，”~“是家目录。Linux 存储是以挂载的方式，相当于是树状的，源头就是”/“，也就是根目录。而每个用户都有”家“目录，也就是用户的个人目录，比如 root 用户的”家“目录就是/root,普通用户 a 的家目录就是/home/a.可以看到

-   locate 查找文件位置

> locate nginx.conf

-   ps(process status) 用于显示当前进程的状态，类似于 windows 的任务管理器。
    -   A 列出所有的进程
    -   w 显示加宽可以显示较多的资讯
    -   au 显示较详细的资讯
    -   aux 显示所有包含其他使用者的进程

### vim

> 命令行模式下输入（n 为指定的行号）：
> （1）ngg / nG

> （2）:n

> （3）vim +n filename（注意这里要输入 + 号）

### /var 目录

> /var 目录主要针对常态性变动的文件，包括缓存(cache)、登录档(log file)以及某些软件运作所产生的文件
