# 配置ssh免密

假设你的本地主机为A，远程主机为B（ip为192.168.12.21），

现在想在A上以用户xiaoming的身份免密登录远程主机B，需要做的如下。

在本地机器A上执行

```bash
# 在A上生成公钥和私钥（在目录~/.ssh下）
ssh-keygen -t rsa 
# 在B上新建.ssh目录，且修改权限为700，会让输入xiaoming的密码
ssh xiaoming@192.168.12.21 "mkdir .ssh;chmod 0700 .ssh" 
# 将A上生成的公钥传输到B上
scp ~/.ssh/id_rsa.pub xiaoming@192.168.12.21:.ssh/id_rsa.pub
```













在B上的命令:

\# touch /root/.ssh/authorized\_keys \(如果已经存在这个文件, 跳过这条\)

\# chmod 600 ~/.ssh/authorized\_keys  \(\# 注意： 必须将**~/.ssh/authorized\_keys**的权限改为**600**, 该文件用于保存**ssh**客户端生成的公钥，可以修改服务器的**ssh**服务端配置文件**/etc/ssh/sshd\_config**来指定其他文件名）

\# cat /root/.ssh/id\_rsa.pub  &gt;&gt; /root/.ssh/authorized\_keys \(将id\_rsa.pub的内容追加到 authorized\_keys 中,注意不要用 **&gt;** ，否则会清空原有的内容，使其他人无法使用原有的密钥登录\)

回到A机器:

\# ssh root@172.24.253.2 \(不需要密码, 登录成功\)

假如在生成密钥对的时候指定了其他文件名（或者需要控制N台机器，此时你会生成多对密钥），则需要使用参数-i指定私钥文件

\# ssh root@172.24.253.2 -i /path/to/your\_id\_rsa

scp也是一样，如：

scp －i /root/.ssh/id\_rsa  ./xxx 192.168.102.158:/home/wwy/bak

