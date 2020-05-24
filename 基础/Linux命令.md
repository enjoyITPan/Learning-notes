##一、特殊命令
| 命令                | 解释                                 |
| ------------------- | ------------------------------------ |
| last \| grep reboot | 查看系统最后的重启时间               |
| ls -lh              | 显示文件大小，带上单位的，默认是字节 |

##二、基础命令
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

#### less

```shell
less -m  显示类似于more命令的百分比
less -N  显示行号
/   字符串：向下搜索“字符串”的功能
?   字符串：向上搜索“字符串”的功能
n   重复前一个搜索（与 / 或 ? 有关）
N   反向重复前一个搜索（与 / 或 ? 有关）
b   向后翻一页
d   向后翻半页
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

#### lsof 

```java
lsof -i:port 查找占用port端口的进程
```

## 三、参考链接

- [linux命令在线查询](https://www.linuxcool.com/htop)

