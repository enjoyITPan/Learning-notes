## 一、基础命令

### 容器生命周期管理

1. run

   | 语法：docker run [options] image [command] [arg...]          |
   | ------------------------------------------------------------ |
   | OPTIONS说明：<br />-d：后台运行容器，并且返回容器id；<br />-p：指定端口映射，格式为：**主机(宿主)端口:容器端口；**<br />--name="nginx-lb": 为容器指定一个名称； |
   | 实例：<br />docker run -d --name=nginx -p 80:80 nginx:latest |

2. start/stop/restart

3. rm

4. exec

