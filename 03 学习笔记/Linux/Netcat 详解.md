## 介绍
netcat 简称 nc，安全界叫它瑞士军刀。ncat 也会顺便介绍，弥补了 nc 的不足，被叫做 21 世纪的瑞士军刀。nc 的基本功能如下：
-   telnet / 获取系统 banner 信息
-   传输文本信息
-   传输文件和目录
-   加密传输文件
-   端口扫描
-   远程控制 / 正方向 shell
-   流媒体服务器
-   远程克隆硬盘

## **端口扫描**
首先应该知道的一个参数是 - h，这是最基本的一个命令，nc 长跟的参数有两个，一个是 n 一个是 v，帮助文档解释如下：
![[Obsidian/附件/Netcat 详解.png]]

v 参数就是列出执行过程的详细信息，n 参数翻译过来就是只接收 ip 地址，没有 dns。之所以使用 n 参数，是因为使用命令的过程中只去传入 ip，减少了 nc 把域名解析为 ip 的过程，这样可以节省时间提高效率。
nc 用来进行端口扫描的命令是 nc -nvz ip 地址 端口号，z 参数翻译过来就是不进行 i/o，用来扫描。意思就是仅仅是去 ping 去探测目标是否开启指定端口，不进行任何的交互。
![[Obsidian/附件/Netcat 详解-1.png]]

z 参数默认扫描的是 tcp 类型，如果需要扫描 udp，则需要使用一个新参数 u。

## Telnet/Banner
telnet 使用率大不如以前了，基本被 ssh 取代了，最大的弊端在于其明文传输。nc 也是明文方式传输的数据，所以后续需要使用 nmap 下的 ncat 工具来结合一下，弥补其不足。nc 在这里可以获取服务器的 banner 信息。例如 163 的 smtp，命令格式是 nc -nv ip port，如下图：
![[Obsidian/附件/Netcat 详解-2.png]]
根据返回的信息可以知道，其使用的是 coremail 邮件系统。

## 传输文本信息
nc 可以在两台机器之间相互传递信息，首先需要有一台机器进行监听一个端口，另一台以连接的方式去连接其指定的端口，这样两台机器之间建立了通信后，相互之间可以传输信息。l 参数是监听模式的意思，p 是指定一个端口（为了区分，一个机器发送字母，一个机器发送数字），如下图：
![[Obsidian/附件/Netcat 详解-3.png]]

这种相互传输信息和渗透之间的关系是，电子取证的时候可以用。当机器被攻击后，为了不破坏现场，需要提出大量的信息和文件出来做分析，这时候可以用 nc 的这个机制，例如，需要一个命令的输出信息，首先在一台机器上监听一个端口，随后在被攻击的机器上执行相关的命令，然后以管道给 nc，指定另一台的地址和端口，这样输出结果就会到另一端，如下图：
![[Obsidian/附件/Netcat 详解-4.png]]

如果输出内容过多，则可以将内容定向输出到文件中，如下图：
![[Obsidian/附件/Netcat 详解-5.png]]


## **传输文件和目录** 
作为文件传输和目录这些功能，其实和文本信息传输类似，只不过是把文本信息换成了文件和目录。首先用一台机器监听一个端口，如果有人连接并传来信息时，则将信息使用 > 重定向到文件。另一台机器连接目标指定端口然后通过 < 输出要传送的文件即可。
![[Obsidian/附件/Netcat 详解-6.png]]

上面这个是正向传输的，同样的也有一个反向传输，这个需要理解一下，因为和经常用到的正反向 shell 原理一样。原先是我打开指定端口，等待别人连我，给我传文件。现在是我打开指定端口把文件准备好，别人连我，我传给他文件。
![[Obsidian/附件/Netcat 详解-7.png]]

这里需要理解两个重定向符号 <和>，> 是将文件进行输出。<是将文件进行输入。
对于传输目录其实和传输文本信息传输文件一样，当作文件处理即可，传输时将目录进行压缩进行传输，随后另一台机器接收后进行解压，这样就完成了目录的传输。例如使用 tar 命令，cvf 进行压缩，xvf 进行解压，c 是压缩的意思，v 是显示详细过程，f 是文件名，x 是解压。
![[Obsidian/附件/Netcat 详解-8.png]]

## **加密传输文件**
[加密](https://so.csdn.net/so/search?q=%E5%8A%A0%E5%AF%86&spm=1001.2101.3001.7020)传输文件需要使用 mcrypt 库，linux 系统默认是没有安装的，需要手动安装。随后和传输文件类似，只需要在传输文件时使用 mcrypt 加密即可。
命令用到的参数有，--flush 立即冲洗输出，-F 输出数据，-b 不保留算法信息，-q 关闭一些非严重的警告，-d 解密，首先在接收端监听一个端口，等待另一台进行连接传送文件，随后在要传送的机器上把要传送的文件进行加密使用 nc 连接指定的地址和 ip，如下图：
![[Obsidian/附件/Netcat 详解-9.png]]

第一次需要输入加密的密码，然后回车再次确认密码，接收端输入相应的密码就会收到传输的文件，如下图：
![[Obsidian/附件/Netcat 详解-10.png]]

![[Obsidian/附件/Netcat 详解-11.png]]


## **流媒体服务** 
对于直播的功能就类似于[流媒体](https://so.csdn.net/so/search?q=%E6%B5%81%E5%AA%92%E4%BD%93&spm=1001.2101.3001.7020)，就是对方把视频通过流的方式进行传输，传输多少，对方就会实时的播放多少。
首先，需要在要传送文件的机器上把文件通过管道给 nc，然后监听一个端口。在接收端，连接此端口然后通过管道给 mplayer 进行实时播放，mplayer 默认 linux 不安装，需要手动安装。其中 vo 参数是选择驱动程序，cache 是每秒要接收的播放帧。
![[Obsidian/附件/Netcat 详解-12.png]]

## **远程克隆硬盘**
对于远程克隆硬盘，在远程电子取证时可以用，使用方法需要借助 dd 命令，首先通过 nc 监听一个端口，然后通过 dd 指定要克隆的分区，dd 的 of 参数相当于一个复制功能，然后再另一台机器通过 nc 连接此端口，dd 的 if 参数相当于粘贴的命令。格式如下：
```shell
nc -lp 6666 | dd of=/dev/sda
dd if=/dev/sda | nc -nv 192.168.228.128 6666 -q 1
```

## **远程控制 / 正反向 shell**
原理和传输信息传输文件一样，只不过传输得是 bash，windows 系统是 cmd，正向是目标机器主动指定 bash，然后通过别人连接自己的端口，别人连接自己后，执行的命令就是自己的机器，如下图（左边是目标服务器，右边是客户端）：
![[Obsidian/附件/Netcat 详解-13.png]]

通常情况下，一般的服务器都会有防火墙，很少会允许其他外在的机器来连接自己的某一个端口，只有某些指定端口可能会允许访问，例如 web 服务的 80 端口。这时正向的 shell 就不不起作用了，而防火墙一般都会禁止外在机器来连接自己机器的其他端口，但自己的机器访问外面的端口一般不会做限制，这时候就可以使用反向 shell，也就是攻击者指定一个端口和 bash，让目标服务器来连接自己，这样就可以写一个脚本放到目标服务器的开机启动中，只要目标服务器运行就会连接自己。当然，有些管理员安全意思比较好的话，也会限制自己的服务器访问外在的一些端口，这种情况很少见，但也有，这时可以指定常用的端口，例如服务器要使用 dns 服务的 53 端口，这时候自己就可以监听 53 让目标服务器来连接自己。

下图（左边是目标服务器，右边是客户端）。
![[Obsidian/附件/Netcat 详解-14.png]]

总结：正向 shell 是服务器开启一个端口指定 shell 让别人来连。但出于防火墙原因，一般都连不上。这时需要用反向 shell，让目标服务器指定 shell 后来连接自己。

## **ncat** 
nc 也有不足之处，首先就是明文传输，可能会被嗅探。其次对于反向 shell，如果其他人通过网络扫描发现了这个端口，也就意味着任何人都可以去监听这个端口进行连接，缺乏身份验证功能。
ncat 则弥补了这些缺点，ncat 不是 linux 系统自带的命令，而是 nmap 中的。ncat 中很多参数和 nc 一样，其中可以通过 --alow 参数来指定允许连接的机器，通过 --ssl 进行数据的加密，如下图（左边是服务器，右边是客户端）：
![[Obsidian/附件/Netcat 详解-15.png]]

通过 allow 参数即可指定允许连接的机器，这时如果其他人即使扫描到了这个端口也无法进行连接，如下图：
![[Obsidian/附件/Netcat 详解-16.png]]
客户端 ip 是 128，当服务器只允许 111 时，再连接就会提示失败。

