---
title: 运维2017.9.4
categories: operations
toc: true
---
1. ufw
ufw enable
之后一定要打开ssh端口！！
ufw open a range of ports:
ufw allow 11200:11299/tcp

And don’t forget ufw reload

2. mosh
Two major problems：
ssh_exchange_identification: Connection closed by remote host
非默认端口需要按一下命令执行：
mosh –ssh=”ssh -p 8022” root@cds.yuqianshan.com

The locale requested by LC_CTYPE=UTF-8 isn’t available here.
需要保持server和client的LC_ALL, LANG一致
将以下代码加入mac的.zshrc中即可：
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8

（最终需要source .zshrc）

3. ssh-keygen -t rsa
http://blog.csdn.net/leexide/article/details/17252369
ssh-keygen -t rsa
scp id_rsa.pub root@192.168.1.4:~/.ssh/authorized_keys
ipv6:
scp -6 id_rsa.pub qianshan@\[2402:f000:1:5003:c1bb:6e61:d4e:acd8\]:~/.ssh/authorized_key

cat /etc/hosts
192.168.1.2 Server1
192.168.1.3 Server2
192.168.1.4 Server3

ssh Server3

4. ohmyzsh
注意passwd文件的改动，可能会导致无法登陆
先安装zsh (apt-get等)
然后wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
切换zsh: chsh -s /usr/local/bin/zsh (chsh -s which zsh)
theme: 修改.zshrc (ZSH_THEME=”bira”)
source:
http://yijiebuyi.com/blog/b9b5e1ebb719f22475c38c4819ab8151.html

5. sshd_config
https://segmentfault.com/a/1190000000481249
http://blog.csdn.net/lansesl2008/article/details/16113193

6. smbmount
https://segmentfault.com/a/1190000002548622

7. ftp
FTP Issue - Failed to retrieve directory listing
Solution:
This happens when you have firewall configured and have kept only port 20 and 21 opened. Change the FTP client to connect using Active or the PORT method and you will be able to connect to the FTP and retrieve the directory listing. Or else you can try after disabling the firewall.
Passive对应其他端口，需要打开或者关闭防火墙。

550 Failed to change directory.
