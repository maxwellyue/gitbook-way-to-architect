# 登录或传输

远程登录主机

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
scp /Users/yue/xxxx/dist/index.html xiaoming@192.168.12.21:/home/edu-web

# 将/Users/yue/xxxx/dist/static目录及该目录下的所有文件或子目录传输到远程主机192.168.12.21
# 以用户xiaoming登录，且指定免密私钥在本地的存放地址为/Users/yue/educloud_server
scp -i /Users/yue/educloud_server -r /Users/yue/xxxx/dist/static root@192.168.12.21:/home/edu-web
```

远程执行命令

