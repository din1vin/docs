# 每天一个linux命令---ps

ps(process status),可以查看系统上的进程

| 参数        | 描述                                       |
| ----------- | ------------------------------------------ |
| -A          | 显示所有进程                               |
| -N          | 显示与指定参数不符的所有进程               |
| -a          | 显示除控制进程和无终端进程外的所有进程     |
| -d          | 显示出控制进程外的所有进程                 |
| -e          | 显示所有进程                               |
| -C cmdlist  | 显示包含在cmdlist列表中的所有进程          |
| -G grplist  | 显示组ID在grplist列表中的进程              |
| -U userlist | 显示属主的用户ID在userlist列表中的进程     |
| -u userlist | 显示有效用户ID在userlist列表中的进程       |
| -g grplist  | 显示会话或组ID在grplist中的进程            |
| -p pidlist  | 显示PID在pidlist列表中的进程               |
| -s sesslist | 显示会话ID在sesslist列表中的进程           |
| -t ttylist  | 显示终端ID在ttylist中的进程                |
| -f          | 显示完整格式的输出                         |
| -F          | 相对-f,显示更多的参数                      |
| -o  format  | 仅显示由format指定的列                     |
| -O format   | 显示默认的输出列以及format列表指定的特定列 |
| -M          | 显示进程的安全信息                         |
| -c          | 显示进程的额外调度器信息                   |
| -j          | 显示任务信息                               |
| -l          | 显示长列表                                 |
| -y          | 不要显示进程标记(表名进程flag的标记)       |
| -Z          | 显示安全标签信息                           |
| -H          | 用层级格式来显示进程                       |
| -n namelist | 定义了WCHAN列显示的值                      |
| -w          | 采用宽输出模式,不限宽度显示                |
| -L          | 显示进程中的进程                           |
| -V          | 显示ps命令的版本号                         |



**使用ps命令的关键不在于记住所有可用的参数,而是记住最有用的那些参数.**

举个栗子:

* 查看系统上运行的所有进程

  ```shell
  ps -ef
  # 查看所有java进程
  ps -ef |grep java
  ```

  

* 显示所有进程更详细的信息,包括占用CPU、内存

  ```shell
  ps -aux 
  # 根据cpu排序
  ps -aux --sort -pcpu
  # 根据占用内存排序
  ps -aux --sort -pmem
  ```

  