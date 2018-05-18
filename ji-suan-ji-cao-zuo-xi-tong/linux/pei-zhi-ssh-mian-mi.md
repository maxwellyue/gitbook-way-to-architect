# 配置ssh免密

原理：在本地生成公钥和私钥，将本地公钥添加到远程主机的受信任名单中，如下图所示：

![ssh&#x514D;&#x5BC6;&#x539F;&#x7406;&#x56FE;&#x793A;&#x610F;](../../.gitbook/assets/ping-mu-kuai-zhao-20180427-xia-wu-8.58.35.png)

假设你的本地主机为A，远程主机为B（ip为192.168.12.21），

现在想在A上以用户xiaoming的身份免密登录远程主机B，需要做的如下。

在本地机器A上执行：

```bash
# 在A上生成公钥和私钥（在目录~/.ssh下）
ssh-keygen -t rsa 
# 在B上新建.ssh目录，且修改权限为700，会让输入xiaoming的密码
ssh xiaoming@192.168.12.21 "mkdir .ssh;chmod 0700 .ssh" 
# 将A上生成的公钥传输到B上
scp ~/.ssh/id_rsa.pub xiaoming@192.168.12.21:.ssh/id_rsa.pub
```

在远程主机B上执行：

```bash
# 新建authorized_keys文件（如果已经存在这个文件, 跳过这条命令)
touch .ssh/authorized_keys 
# 将~/.ssh/authorized_keys的权限改为600,
# 该文件用于保存ssh客户端生成的公钥，可以修改服务器的ssh服务端配置文件/etc/ssh/sshd_config来指定其他文件名
chmod 600 ~/.ssh/authorized_keys  
# 将id_rsa.pub的内容追加到authorized_keys中,注意不要用 > ，否则会清空原有的内容，使其他人无法使用原有的密钥登录
cat ~/.ssh/id_rsa.pub  >> ~/.ssh/authorized_keys 
```

回到A机器:

\#  可以免密登录了

```bash
ssh xiaoming@192.168.12.21
```

假如在生成密钥对的时候指定了其他文件名（或者需要控制N台机器，此时你会生成多对密钥），则需要使用参数`-i`指定私钥文件，如：

```bash
ssh xiaoming@192.168.12.21 -i /path/to/your_id_rsa
```

传输文件时也是一样，如：

```bash
scp －i /root/.ssh/id_rsa ./xxx 192.168.12.21:/home/wwy/bak
```



## 参考

[Linux免密登录](https://blog.csdn.net/syani/article/details/52618840)





