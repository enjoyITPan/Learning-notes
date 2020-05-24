Linux是一个多用户多任务的操作系统，可以在Linux中为不同的用户设置不同的运行环境，具体做法是设置不同用户的环境变量。

#### Linux环境变量分类

一、按照生命周期来分，Linux环境变量可以分为两类：
 1、永久的：需要用户修改相关的配置文件，变量永久生效。
 2、临时的：用户利用export命令，在当前shell进程及其子进程中生效，关闭Shell终端失效。

二、按照作用域来分，Linux环境变量可以分为：
 1、系统环境变量：系统环境变量对该系统中所有用户都有效。
 2、用户环境变量：顾名思义，这种类型的环境变量只对特定的用户有效。

## profile、bash_profile、bashrc文件的作用与区别

一、在`/etc/profile`文件中添加变量 **对所有用户生效（永久的）**

修改文件后要想马上生效还要运行`source /etc/profile`不然只能在下次重进此用户时生效。

二、在用户目录下的~/.bash_profile文件中增加变量 **【对单一用户生效（永久的）】**

三、直接运行export命令定义变量 **【只对当前shell（BASH）有效（临时的）】**

`profile`、`bash_profile`、`bashrc`三个文件在Linux或类Unix系统（如：Mac）系统中经常会用到，在本篇文章中我们将介绍这三个文件的作用。

1. `profile`文件
   - [1.1 `profile`文件的作用](https://itbilu.com/linux/management/NyI9cjipl.html#profile-function)
   - [1.2 在`profile`中添加环境变量](https://itbilu.com/linux/management/NyI9cjipl.html#profile-env)
2. [`bashrc`文件](https://itbilu.com/linux/management/NyI9cjipl.html#bashrc)
3. [`bash_profile`文件](https://itbilu.com/linux/management/NyI9cjipl.html#bash_profile)

### 1. `profile`文件

#### 1.1 `profile`文件的作用

`profile`（`/etc/profile`），用于设置系统级的环境变量和启动程序，在这个文件下配置会对**所有用户**生效。当用户登录（`login`）时，文件会被执行，并从`/etc/profile.d`目录的配置文件中查找`shell`设置。

#### 1.2 在`profile`中添加环境变量

一般不建议在`/etc/profile`文件中添加环境变量，因为在这个文件中添加的设置会对所有用户起作用。当需要添加时，我们可以按以方式添加：

如，添加一个`HOST`值为`itbilu.com`的环境变量：

```
export HOST=itbilu.com
```

添加时，可以在行尾使用`;`号，也可以不使用。一个变量名可以对应多个变量值，多个变量值使用`:`分隔。

添加环境变量后，需要重新登录才能生效，也可以使用`source`命令强制立即生效：

```
source /etc/profile
```

查看是否生效可以使用`echo`命令：

```
$ echo $HOST
itbilu.com
```

### 2. `bashrc`文件（类似的还有.zshrc）

－这个文件用于配置函数或别名。`bashrc`文件有两种级别：系统级的位于`/etc/bashrc`、用户级的`~/.bashrc`，两者分别会对所有用户和当前用户生效。

`bashrc`文件只会对指定的`shell`类型起作用，`bashrc`只会被`bash shell`调用。

### 3. `bash_profile`文件

`bash_profile`只有单一用户有效，文件存储位于`~/.bash_profile`，该文件是一个用户级的设置，可以理解为某一个用户的`profile`目录下。这个文件同样也可以用于配置环境变量和启动程序，但只针对单个用户有效。

和`profile`文件类似，`bash_profile`也会在用户登录（`login`）时生效，也可以用于设置环境变理。但与`profile`不同，`bash_profile`只会对当前用户生效。



## 参考

https://itbilu.com/linux/management/NyI9cjipl.html

https://www.jianshu.com/p/ac2bc0ad3d74