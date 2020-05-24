## 零、Linux文件目录

| 文件目录 | 解释                                                         |
| :------- | :----------------------------------------------------------- |
| /etc     | 这个目录包含所有系统层面的配置文件。它也包含一系列 的 shell 脚本，在系统启动时，这些脚本会开启每个系统服务。这个目录中的任何文件应该是可读的文本文件。 有趣的文件:虽然/etc 目录中的任何文件都有趣，但这里只 列出了一些我一直喜欢的文件:<br/>/etc/crontab，定义自动运行的任务。 <br/>/etc/fstab，包含存储设备的列表，以及与他们相关的挂载点。<br/>/etc/passwd，包含用户帐号列表 |
| /opt     | 这个/opt 目录被用来安装“可选的”软件。这个主要用来 存储可能安装在系统中的商业软件产品。 |
| /usr     | 在 Linux 系统中，/usr 目录可能是最大的一个。它包含普 通用户所需要的所有程序和文件。 |
| /usr/bin | /usr/bin 目录包含系统安装的可执行程序。通常，这个目录 会包含许多程序。 |
| /usr/lib | 包含由/usr/bin 目录中的程序所用的共享库。                    |



## 一、特殊命令

| 命令                | 解释                   |
| ------------------- | ---------------------- |
| last \| grep reboot | 查看系统最后的重启时间 |



## 二、基础命令

#### awk

```
awk -F'[分隔符]' '{print $1}'

//case1 
grep -w getUserAuthTemplate service-access.log | awk -F '\\001' '{if ($6 != "null") print $6,$7,$9}'
```

#### sed

```
删除纯空行和由空格组成的空行 ： sed '/^[  ]*$/d' file 
```

#### uniq

```
忽略相同行使用-u选项或者uniq，因为只判断相邻行:
1、sort -u sort.txt 
2、uniq sort.txt 
```

#### tail

```
-n 显示最后n行
-f 循环读取
```

#### grep

```shell
grep -A 5 foo file 显示foo及后5行
grep -B 5 foo file 显示foo及前5行
grep -C 5 foo file 显示file文件里匹配foo字串那行以及上下5行
```

## 三、网络相关

#### lsof

```shell
lsof -i:port 查找占用port端口的进程(因为在 linux 上一切皆文件，TCP socket 连接也是一个 fd。因此使用 lsof 也可以)
```

#### nc

```
nc -v ip port 查看端口是否打开
```

#### telnet

```
telnet ip port 查看端口是否打开
```

#### netcat

```
sudo netstat -ltpn | grep :port 查看端口被谁占用了
```

## 四、文件查看移动

#### ls（list directory contents）

```shell
ls -lh                                                   
total 0
drwx------@  5 yunshu  staff   160B  7 25 23:50 Applications
```

| drwx------@  | 权限                                                  |
| ------------ | ----------------------------------------------------- |
| 5            | 链接数（连接数-2 == 文件夹直接包含的文件数+文件夹数） |
| yunshu       | 文件持有者                                            |
| staff        | 文件所属用户组的名字                                  |
| 160B         | 大小                                                  |
| 7 25 23:50   | 最后修改时间                                          |
| Applications | 文件名                                                |

备注：

- ls -l 可以显示文件的大小，但是无法显示文件夹的实际大小

  

#### du （display disk usage statistics）

```shell
du -hd 1 : 显示文件和文件夹的实际大小
```

#### less

```shell
less -m  显示类似于more命令的百分比
less -N  显示行号
/   字符串：向下搜索“字符串”的功能
?   字符串：向上搜索“字符串”的功能
n   (向下)重复前一个搜索（与 / 或 ? 有关）
N   (向上)反向重复前一个搜索（与 / 或 ? 有关）
b   向上翻页(backward)
d or space  向下翻页(forward)
G 移动到最后一行
g 移动到开头
```

#### cp —复制文件和目录

```shell
cp –r test/ newtest : 使用指令 cp 将当前目录 test/ 下的所有文件复制到新目录 newtest 下,注意：用户使用该指令复制目录时，必须使用参数 -r 或者 -R 。
```



#### mv —移动/重命名文件和目录

```shell
1、目标目录与原目录一致，指定了新文件名，效果就是仅仅重命名。
2、目标目录与原目录不一致，没有指定新文件名，效果就是仅仅移动。
```



#### mkdir —创建目录

```shell

```



#### rm —删除文件和目录

```shell
-r 递归删除
-f 忽略错误
```



#### ln —创建硬链接和符号链接

```shell
ln -s xx/xx/a xx/xx/b 给a创建一个软链接放到b位置
```

## 五、文件统计

#### 重定向

```
1、">"：将输出重新输入到指定的地方，会覆盖原始内容
2、">>"：追加
```

## 六、端口

### linux 下查看进程占用端口：

（1）查看程序对应的进程号： ps -ef | grep 进程名字

（2）查看进程号所占用的端口号： netstat -nltp | grep  进程号

（3） lsof -nP | grep LISTEN | grep 进程号

    ubuntu :查看进程占用端口号：netstat -anp | grep pid

### linux 下查看端口号所使用的进程号：

（1）使用 lsof 命令：lsof -i:端口号 

## 参考链接

- [linux命令在线查询](https://www.linuxcool.com/htop)

## 快捷键

- ctrl + w 按照单词删除

