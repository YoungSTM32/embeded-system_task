# 1.人员信息
姓名：拖拉机
学号：M2021*****
班级：硕210*班
项目名称：基于UDP的智慧农业后端演示
# 2.系统设计与实现
（1）采集终端向服务器上报温度、湿度、电机状态（用于降低温度）、开关状态（用于自动浇水）；
（2）服务器接收到数据后提取信息，将数据及其上报时间写入数据库存储历史记录，便于用于查看；
（3）服务器接收到数据后从数据库中读取用户设定的阈值，对数据进行判断，如果超过或者低于阈值即发送对应指令，打开/关闭电机，或者打开/关闭开关；
整个系统的实现架构如下：
（1）采集终端运行在嵌入式Linux开发板上（使用Qemu模拟）；
（2）采集系统服务端使用本机Ubuntu20.04；
（3）数据存储使用MySQL；
其中，所有数据均采用JSON数据格式，使用UDP传输，数据库中有两张表，一张表为history用于保存历史数据，一张表为value用于保存阈值信息。
# 3. 建立数据库及数据表
(1)建立数据库agricultural_system:
```sql
create database agricultural_system;
```
(2)建立数据表history:
```sql
create table history (id INT(10) not null,temp FLOAT,humi FLOAT,motor CHAR(3), switch CHAR(3));
```
(3)建立数据表value:
```sql
create table value (id INT(10) not null,temp_max FLOAT,humi_max FLOAT,temp_min FLOAT,humi_min FLOAT)
```
(4)提前在数据表value中设置一个阈值：
```sql
insert into value(id,temp_max,humi_max,temp_min,humi_min) values(20211230,10.0,40.0,10.0,40.0);
```
# 4. 运行服务端
服务端源码在server文件夹，需要在ubuntu20.04上运行，使用gcc编译。
进入到server文件夹，执行make命令编译：

```bash
cd server
make
```
然后运行程序（默认监听8002端口，使用UDP协议，如果有安全组需要放行该端口）:
```bash
./server
```
# 5. 运行终端
终端中只使用到了UDP Socket编程，编译(arm-linux-gcc)为ARM开发板上的程序。

进入endpoint文件夹，编译：
```bash
cd endpoint
make 
```

使用NFS复制可执行程序到开发板中,运行程序，第一个参数是server ip，第二个参数是server端口：

```bash
./endpoint [server_ip] [port]
make 
```

可以看到终端每隔5s向服务器上报一次数据，在服务端也可以看到上报的数据
也在MySQL中查看历史记录

# 6. 项目改进
# 6.1系统规模
当前服务端仅可以接收一个终端的连接，可采用改进I/O模型提高并发量。
# 6.2 通信质量
需要改进服务器程序，预防丢包等情况发生。

