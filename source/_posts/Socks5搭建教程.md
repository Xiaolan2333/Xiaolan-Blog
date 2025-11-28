---
title: "Socks5搭建教程"
date: 2025-11-01 12:00:00
---

## 安装Dante-Server：

### Deb系：

```
apt install dante-server -y
```

## 添加代理用户名：

```
useradd -M -s /usr/sbin/nologin 要设置的用户名
```
<!--more-->
## 配置密码：

```
passwd 上面设置用户名
```

## 修改配置文件

### 写入配置文件到"/etc/danted.conf"：

```
logoutput: syslog

internal: 0.0.0.0 port = 8080

external: eth0

method: username

user.notprivileged: nobody

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect disconnect error
}

pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    protocol: tcp udp
    log: connect disconnect error
}
```

#### 配置项说明：

```
external：网卡名，使用ip a查询
```

```
port = 8080：监听的端口
```

## 运行设置：

### 设置开机自启：

```
systemctl enable danted
```

### 重新启动服务（修改配置文件后需要执行）：

```
systemctl restart danted
```
