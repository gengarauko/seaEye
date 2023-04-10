前置工作
1、关闭防火墙
2、关闭selinux


进入探针，安装ftp服务
```shell
yum install vsftpd -y
```

创建ftp用户，并配置密码
```shell
useradd -s /sbin/nologin ftpUser 
# 此处的ftp用户需要与中心中创建的用户、密码一致
echo 12345678 | passwd --stdin ftpUser
```

修改ftp配置
```shell
# Example config file /etc/vsftpd/vsftpd.conf
#
anonymous_enable=NO
#
local_enable=YES
write_enable=YES
local_umask=022
#
dirmessage_enable=YES
#
# Activate logging of uploads/downloads.
xferlog_enable=YES
#
# Make sure PORT transfer connections originate from port 20 (ftp-data).
connect_from_port_20=YES
#
xferlog_std_format=YES
#
reverse_lookup_enable=NO
chroot_local_user=YES
chroot_list_enable=YES
# (default follows)
#这个文件中需要包含之前创建的ftp用户
chroot_list_file=/etc/vsftpd/chroot_list
#
listen=NO
#
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
userlist_deny=NO # user_list文件此时为白名单，只保留之前创建的ftp用户
tcp_wrappers=YES
local_root=/mnt/40t/save # 这个路径是captor程序存储pcap包的路径
allow_writeable_chroot=YES
```

修改vim/etc/pam.d/vsftpd文件
```shell
%PAM-1.0
session optional pam_keyinit.so force revoke
auth required pam_listfile.so item=user sense=deny
file=/etc/vsftpd/ftpusers onerr=succeed
auth required pam_nologin.so
####修改成此项auth required pam_shells.so修改为->auth required pam_nologin.so 
auth include password-auth
account include password-auth
session required pam_loginuid.so
session include password-auth
```

开启ftp服务
```
systemctl start vsftpd
```


