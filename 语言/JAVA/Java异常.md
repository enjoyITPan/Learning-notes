# java异常

![](https://gitee.com/nieyunshu/picture/raw/master/img/20220220001522.jpg)

java的异常都是继承自Throwable。Throwable有两个直接子类，Error和Exception。一般我们只需要关注Exception，Error一般是系统级的异常,如操作系统、文件损坏等异常。Exception则是程序级别的异常，是我们需要关注并且处理的。

Exception的子类又可以分为RuntimeException和其他异常。这样分是因为除了RuntimeException外，其他子类异常都是check exception(需要在方法中显示声明抛出，RuntimeException` and its subclasses are *unchecked exceptions*.)

