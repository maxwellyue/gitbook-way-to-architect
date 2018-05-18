# 常用命令

```bash
安装：yum -y install nginx
启动：service nginx start
停止：service nginx stop
重启：service nginx restart
查看版本号：nginx -v

配置文件一般在etc/nginx/nginx.conf
检查配置文件有没有错误：
nginx -t 或者 nginx -t -c /etc/nginx/nginx.conf (-c 用来指定配置文件路径)

修改了配置后，重新加载：nginx -s reload (不必重启nginx，即可生效)
```



