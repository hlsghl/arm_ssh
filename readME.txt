2》 编译:
/home/arm下新建目录sshwork，并且将源码复制到该目录下。
mkdir /home/arm/sshwork
cp zlib-1.2.3.tar.gz openssl-0.9.8d.tar.gz openssh-4.6p1.tar.gz /home/arm/sshwork
/home/arm/sshwork下新建目录lib，用来保存生成的库文件。
mkdir /home/arm/sshwork/lib
(1) 编译zlib :
tar zxvf zlib-1.2.3.tar.gz -C .
cd zlib-1.2.3/
./configure –prefix=/home/arm/sshwork/lib/zlib-1.2.3
修改Makefile :
CC=gcc 改为:
CROSS=/usr/local/arm/3.4.1/bin/arm-linux-
CC=(CROSS)gccLDSHARED=gcc改为:LDSHARED=( CROSS ) gcc
CPP= gcc - E 改为:CPP=(CROSS)gcc−EAR=arrc改为:AR=( CROSS ) ar rc
开始编译: make
make install
(2) 编译openssl:
tar zxvf openssl-0.9.8d.tar.gz
./Configure –prefix=/home/arm/sshwork/lib/openssl-0.9.8d os/compiler:arm-linux-gcc
make
make install
(3)编译openssh:
tar zxvf openssh-4.6p1.tar.gz
cd openssh-4.6p1/
./configure –host=arm-linux –with-libs –with-zlib=/home/arm/sshwork/lib/zlib-1.2.3
–with-ssl-dir=/home/arm/sshwork/lib/openssl-0.9.8d –disable-etc-default-login
CC=/usr/local/arm/3.4.1/bin/arm-linux-gcc AR=/usr/local/arm/3.4.1/bin/arm-linux-ar
make
不要make install

3》安装
确保目标板上有以下目录，如果没有，则新建：
/usr/sbin
/usr/local/bin
/usr/local/libexec
/usr/local/etc/
(1) 将 openssh-4.6p1目录下的sshd 拷贝到 目标板的/usr/sbin目录下
(2) 再copy scp sftp ssh ssh-add ssh-agent ssh-keygen ssh-keyscan 到目标板/usr/local/bin 目录下
copy sftp-server ssh-keysign到/usr/local/libexec
将sshd_config ,ssh_config拷贝到/usr/local/etc/目录下
（3）在目标板上新建sshd工作目录
mkdir -p /var/run
mkdir -p /var/empty/sshd
chmod 755 /var/empty
（4）在主机上：
ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N “”
ssh-keygen -t rsa -f ssh_host_rsa_key -N “”
ssh-keygen -t dsa -f ssh_host_dsa_key -N “”
将生成的 ssh_host_*_key这3个文件copy到目标板的 /usr/local/etc/目录下
(5) 添加用户（这一步不要做）
将主机上/etc/ 目下的 passwd, shadow, group 三个文件copy到目标板的 /etc 目录下， 同时记得将passwd的最后 /bin/bash 该为 /bin/sh
//这一步以后，开发板上的用户结构与你本机的结构就是一样的了，所以在cp之前先把你自己的root密码设好
//#passwd设置root密码，然后在把上面3个文件cp到开发板/etc下
其实可以删除不需要的一些用户。
4》测试
目标板启动sshd: #/usr/sbin/sshd
可能出现的错误：
Privilege separation user sshd does not exist
//需要在开发板的系统里adduser shhd
//或者在/etc/passwd 中添加下面这一行
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin

Permissions 0755 for ‘/usr/local/etc/ssh_host_dsa_key’ are too open.
//则把目标板/usr/local/etc/下的ssh_host_*几个文件的权限改为700。如果出现权限不够问题，用su - root进入root用户再改。
命令：#chmod 700 ssh_host_*

Permission denied (publickey,password,keyboard-interactive).
//打开开发板/usr/local/sshd_config,将PermitRootLogin yes前的注释“#”号去掉。
主机: $ ssh root@192.168.0.34（开发板的ip）//root密码就是你开发板上root的密码，如果之前root没有密码，需要重新设置，用passwd root，然后输入密码即可。
登录成功后如下图：
