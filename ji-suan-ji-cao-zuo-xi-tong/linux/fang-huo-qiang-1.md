# 防火墙

## iptables

## firewall

**开启端口**

```text
firewall-cmd --zone=public --add-port=80/tcp --permanent

命令含义：
--zone #作用域
--add-port=80/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效
```

**重启防火墙**

```text
firewall-cmd --reload
```

**列出所有的开放端口**

```text
firewall-cmd --list-all
```

**删除端口**

```text
firewall-cmd --zone=public --remove-port=80/tcp
```

## 参考

[Centos7防火墙\(Firewall\)，开启，删除，重装加载，检查是否生效](http://www.blogjava.net/nkjava/archive/2015/07/27/426434.html)

[How can i use iptables on centos 7?](https://stackoverflow.com/questions/24756240/how-can-i-use-iptables-on-centos-7)

