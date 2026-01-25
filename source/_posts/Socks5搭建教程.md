---
title: "Socks5搭建教程"
date: 2026-01-25 12:00:00
---

## 安装Dante-Server：

### Deb系：

```
apt install dante-server -y
```

## 添加代理用户名：

```
useradd 要设置的用户名
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

internal: 网卡名 port = 1080

external: 网卡名

method: username

user.notprivileged: nobody

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}

client pass {
    from: ::/0 to: ::/0
}

pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    protocol: tcp udp
}

pass {
    from: ::/0 to: ::/0
    protocol: tcp udp
}
```

#### 配置项说明：

```
external：网卡名，使用ip a查询（大多为eth、ens开头）
```

```
port = 1080：Socks5运行的端口
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
