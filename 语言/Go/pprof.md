pprof

go tool pprof  http://localhost:6060/debug/pprof/profile 命令模式

go tool pprof -http=:8081 http://localhost:6060/debug/pprof/profile 火焰图

- 交互模式中的各列数据意义说明如下：
  - flat：对应函数运行耗时                                                                                          
  - flat%：对应函数运行耗时总占比
  - sum%：从第一行开始到当前函数为止所有函数累积运行耗时总占比
  - cum：对应函数加上它上层的函数调用运行总耗时
  - cum%：对应函数加上它上层的函数调用运行总耗时总占比
  - 最后一列是函数名