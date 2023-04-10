
## 1. Linux 下生成密钥

ssh-keygen 的命令手册，通过`man ssh-keygen`命令：
![[Obsidian/附件/ssh免密登录.png]]
通过命令 `ssh-keygen -t rsa`
![[Obsidian/附件/ssh免密登录-1.png]]
生成之后会在用户的根目录生成一个 “.ssh”的文件夹
![[Obsidian/附件/ssh免密登录-2.png]]
进入“.ssh”会生成以下几个文件
![[Obsidian/附件/ssh免密登录-3.png]]

- **authorized_keys**：存放远程免密登录的公钥，主要通过这个文件记录多台机器的公钥
- **id_rsa**：生成的私钥文件
- **id_rsa.pub**：生成的公钥文件
- **know_hosts**：已知的主机公钥清单

PS：如果希望 ssh 公钥生效需满足至少下面两个条件：

1. .ssh 目录的权限必须是 700
2. .ssh/authorized_keys 文件权限必须是 600

## 2. 远程免密登录

原理图：
![[Obsidian/附件/ssh免密登录-4.png]]

**常用以下几种方法：**

### 2.1 通过 ssh-copy-id 的方式

命令： `ssh-copy-id -i ~/.ssh/id_rsa.put`
举例：

```bash
[root@test .ssh]# ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.91.135
root@192.168.91.135's password:
Now try logging into the machine, with "ssh '192.168.91.135'", and check in:
.ssh/authorized_keys
to make sure we haven't added extra keys that you weren't expecting.

[root@test .ssh]# ssh root@192.168.91.135
Last login: Mon Oct 10 01:25:49 2016 from 192.168.91.133

[root@localhost ~]#
```

常见错误：

```shell
ot@test ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.91.135
-bash: ssh-copy-id: command not found //提示命令不存在

解决办法：yum -y install openssh-clients
```

### 2.2 通过 scp 将内容写到对方的文件中

命令：`scp -p ~/.ssh/id_rsa.pub root@:/root/.ssh/authorized_keys`
举例：

```
[root@test .ssh]# scp -p ~/.ssh/id_rsa.pub root@192.168.91.135:/root/.ssh/authorized_keys
root@192.168.91.135's password:
id_rsa.pub 100% 408 0.4KB/s 00:00
[root@test .ssh]#
[root@test .ssh]#
[root@test .ssh]#
[root@test .ssh]# ssh root@192.168.91.135
Last login: Mon Oct 10 01:27:02 2016 from 192.168.91.133
[root@localhost ~]#
```

<font color="#ff0000">也可以分为两步操作：</font>
$ `scp ~/.ssh/id_rsa.pub root@:pub_key`
// 将文件拷贝至远程服务器
$ `cat ~/pub_key >>~/.ssh/authorized_keys`
// 将内容追加到 authorized_keys 文件中， 不过要登录远程服务器来执行这条命令

### 2.3 通过 Ansible 实现批量免密

将需要做免密操作的机器 hosts 添加到/etc/ansible/hosts 下：

```shell
[Avoid close]
192.168.91.132
192.168.91.133
192.168.91.134
```

执行命令进行免密操作
`ansible -m authorized_key -a "user=root key='{{ lookup('file','/root/.ssh/id_rsa.pub') }}'" -k`

示例：

```shell
[root@test sshpass-1.05]# ansible test -m authorized_key -a "user=root key='{{ lookup('file','/root/.ssh/id_rsa.pub') }}'" -k
　　SSH password: ----->输入密码
　　192.168.91.135 | success >> {
　　"changed": true,
　　"key": "ssh-rsa 　　 AAAAB3NzaC1yc2EAAAABIwAAAQEArZI4kxlYuw7j1nt5ueIpTPWfGBJoZ8Mb02OJHR8yGW7A3izwT3/uhkK7RkaGavBbAlprp5bxp3i0TyNxa/apBQG5NiqhYO8YCuiGYGsQAGwZCBlNLF3gq1/18B6FV5moE/8yTbFA4dBQahdtVP PejLlSAbb5ZoGK8AtLlcRq49IENoXB99tnFVn3gMM0aX24ido1ZF9RfRWzfYF7bVsLsrIiMPmVNe5KaGL9kZ0svzoZ708yjWQQCEYWp0m+sODbtGPC34HMGAHjFlsC/SJffLuT/ug/hhCJUYeExHIkJF8OyvfC6DeF7ArI6zdKER7D8M0SM　　WQmpKUltj2nltuv3w== root@localhost.localdomain",
　　"key_options": null,
　　"keyfile": "/root/.ssh/authorized_keys",
　　"manage_dir": true,
　　"path": null,
　　"state": "present",
　　"unique": false,
　　"user": "root"
　　}
　　[root@test sshpass-1.05]#
```

### 2.4 手工复制粘贴的方式

将本地 idrsa.pub 文件的内容拷贝至远程服务器的~/.ssh/authorizedkeys 文件中
