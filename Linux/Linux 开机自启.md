## Linux 开机自启（Systemd）

#### 一、基础配置

1. 在路径为**/etc/systemd/system/**下，创建一个文件 **frpc.service**，其中**frpc**为你想要开机自启软件的名字或者可以随便取。这是一个***.service**结尾的文件

   ```sh
   vi /etc/systemd/system/frpc.service
   ```

2. 在文件中填入以下内容

   ```sh
   [Unit]
   Description=frpc service
   After=network.target syslog.target
   Wants=network.target
   
   [Service]
   Type=simple
   #启动服务的命令（frpc的实际安装目录）
   ExecStart=/usr/local/bin/frpc -c /etc/frpc/frpc.ini
   
   [Install]
   WantedBy=multi-user.target
   ```

   

3. 启动

   ```
   systemctl enable frpc
   systemctl start frpc
   systemctl status frpc
   ```

#### 二、 启动和查询systemd的简单语法

```
Systemctl是一个systemd工具，主要负责控制systemd系统和服务管理器。
这里以一个docker.service为例。一般开机会加载的配置文件都放在/lib/systemd/system中。用户和三方软件定义的配置文件一般在/usr/lib/systemd/system中。
```

1. 启动或关闭一个服务

   ```shell
   systemctl start frpc.service
   systemctl stop frpc.service
   ```

2. 查看可使用服务

   ```shell
   systemctl list-unit-files --type=service|grep frpc
   ```

   注：

   1. 启用**enabled**为绿色。
   2. 禁用**disabled**为红色。
   3. 标记为**static**的不能直接启用，它们是其他所依赖的对象。

3. 查看服务的状态

   ```shell
   systemctl status frpc.service
   ```

4. 开启和关闭开机自启

   ```shell
   systemctl enable[disable] frpc.service
   ```

5. 查看服务是否为自动启动

   ```shell
   systemctl is-enabled frpc.service 
   ```

6. 重新加载配置文件（不停止服务）

   ```shell
   systemctl reload frpc.service
   ```

7. 重启服务

   ```shell
   systemctl restart frpc.service
   ```

#### 三、配置及语法

```shell
[Unit]
Description=OpenSSH server daemon 
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service 
Wants=sshd-keygen.service

[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

###### 1. Unit区块：启动顺序和依赖关系

- Description：应用简单描述
- After Before：定义启动关系,如果依赖的应用需要启动,那么本应用应该在他之前,还是之后.
- Wants：表示sshd.service与sshd-keygen.service之间存在"弱依赖"关系，即如果"sshd-keygen.service"启动失败或停止运行，不影响sshd.service继续执行。
- Requires：表示"强依赖"关系，即如果该服务启动失败或异常退出，那么sshd.service也必须退出。注意，Wants字段与Requires字段只涉及依赖关系，与启动顺序无关，默认情况下是同时启动的。

###### 2. Service区块：启动行为

1. EnvironmentFile字段：指定当前服务的环境参数文件。该文件内部的key=value键值对，可以用$key的形式，在当前配置文件中获取。
2. ExecStart：定义启动进程时执行的命令。上面的例子中，启动sshd，执行的命令是`/usr/sbin/sshd -D $OPTIONS`，其中的变量$OPTIONS就来自EnvironmentFile字段指定的环境参数文件。 所有的启动设置之前，都可以加上一个连词号（-），表示"抑制错误"，即发生错误的时候，不影响其他命令的执行。
    比如，`EnvironmentFile=-/etc/sysconfig/sshd`（注意等号后面的那个连词号），就表示即使/etc/sysconfig/sshd文件不存在，也不会抛出错误。

| 类似的其他命令    | 含义                   |
| ----------------- | ---------------------- |
| ExecReload字段    | 重启服务时执行的命令   |
| ExecStop字段      | 停止服务时执行的命令   |
| ExecStartPre字段  | 启动服务之前执行的命令 |
| ExecStartPost字段 | 启动服务之后执行的命令 |
| ExecStopPost字段  | 停止服务之后执行的命令 |

其中Post pre 类命令写多个不覆盖,其他会覆盖.

1. Type:字段定义启动类型。它可以设置的值如下。

| 字段值           | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| simple（默认值） | ExecStart字段启动的进程为主进程                              |
| forking          | ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程 |
| oneshot          | 类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务 |
| dbus             | 类似于simple，但会等待 D-Bus 信号后启动                      |
| notify           | 类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务 |
| idle             | 类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合。 |

1. KillMode:定义 Systemd 如何停止服务。上面这个例子中，将KillMode设为process，表示只停止主进程，不停止任何sshd 子进程，即子进程打开的 SSH session 仍然保持连接。
    这个设置不太常见，但对 sshd很重要，否则你停止服务的时候，会连自己打开的 SSH session 一起杀掉。

| KillMode字段            | 含义                                               |
| ----------------------- | -------------------------------------------------- |
| control-group（默认值） | 当前控制组里面的所有子进程，都会被杀掉             |
| process                 | 只杀主进程                                         |
| mixed                   | 主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号 |
| none                    | 没有进程会被杀掉，只是执行服务的 stop 命令。       |

1. Restart：定义了应用的重启方式。

| Restart字段  | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| no（默认值） | 退出后不会重启                                               |
| on-success   | 只有正常退出时（退出状态码为0），才会重启                    |
| on-failure   | 非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启 |
| on-abnormal  | 只有被信号终止和超时，才会重启                               |
| on-abort     | 只有在收到没有捕捉到的信号终止时，才会重启                   |
| on-watchdog  | 超时退出，才会重启                                           |
| always       | 不管是什么退出原因，总是重启                                 |

1. RemainAfterExit:启动命令退出时,是否保持服务.启动命令分为前台命令和后台命令.当启动命令为后台命令时,必须加此参数.
2. RestartSec:表示Systemd重启服务之前，需要等待的秒数。上面的例子设为等待42

###### 3. Install区块：开机启动

WantedBy字段：表示该服务所在的 Target。一般来说，常用的 Target 有两个：

1. multi-user.target，表示多用户命令行状态；
2. graphical.target，表示图形用户状态，它依赖于multi-user.target
    一般配置 multi-user.target 开机启动.