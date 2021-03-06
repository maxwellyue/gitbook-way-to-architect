# 登录或传输

登录远程主机

```bash
# 以用户xiaoming登录远程主机192.168.12.211
ssh xiaoming@192.168.12.211
# 以用户xiaoming登录远程主机192.168.12.21，并指定ssh端口5000
ssh -p 5000 xiaoming@192.168.12.211
```

远程传输文件

```bash
# 将文件/Users/yue/xxxx/dist/index.html传输到远程主机192.168.12.21
# 以用户xiaoming登录
scp -r /Users/yue/xxxx/dist/index.html xiaoming@192.168.12.21:/home/edu-web

# 将/Users/yue/xxxx/dist/static目录及该目录下的所有文件或子目录传输到远程主机192.168.12.21
# 以用户xiaoming登录，并指定免密私钥在本地的存放地址为/Users/yue/educloud_server
scp -i /Users/yue/educloud_server -r /Users/yue/xxxx/dist/static root@192.168.12.21:/home/edu-web

# 从远程主机向本地传输文件时，只需要将上面的scp后面的定位文件的字符的前后位置倒置即可，如
scp -r xiaoming@192.168.12.21:/home/remotefile.txt /Users/yue/Desktop

# scp参数解释
-r ：递归复制整个文件夹
-P ：指定端口（注意是大写的P）
-i ：指定密钥文件
```

远程执行命令

```bash
# 在主机192.168.12.21上执行命令rm -rf /home/edu-web/*
# 以用户xiaoming登录
ssh xiaoming@192.168.12.21  "rm -rf /home/edu-web/*;"

# 在主机192.168.12.21上执行一组命令
# 以用户xiaoming登录，并指定免密私钥在本地的存放地址为/Users/yue/educloud_server
ssh xiaoming@192.168.12.21 -i /Users/yue/educloud_server "cd /home/backup-edu-web;mkdir xxxx;cp -r /home/edu-web/* /home/backup-edu-web/xxxx;"
```

