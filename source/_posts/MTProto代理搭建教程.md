---
title: "MTProto代理搭建教程"
date: 2025-11-30 12:00:00
---

## 安装依赖：

### Deb系：

```
apt install git curl build-essential libssl-dev zlib1g-dev
```

### RPM系：

```
yum install openssl-devel zlib-devel && yum groupinstall "Development Tools"
```
<!--more-->
## 对源码进行构建：

### 克隆仓库：

```
git clone https://github.com/TelegramMessenger/MTProxy
```
### 进入目录：

```
cd MTProxy
```

### 开始构建：

```
make clean && make
```

### 进入构建输出文件目录：

```
cd objs/bin
```

此目录下的`mtproto-proxy`为后续运行的主程序

## 运行：

### 获取配置文件：

```
curl -s https://core.telegram.org/getProxySecret -o proxy-secret && curl -s https://core.telegram.org/getProxyConfig -o proxy-multi.conf
```

### 生成密钥：

```
openssl rand -hex 16
```

### 临时运行：

```
./mtproto-proxy -u nobody -p 随便一个不常用的内网端口 -H 外部访问端口 -S 生成的密钥 --aes-pwd proxy-secret proxy-multi.conf -M 5
```

## 配置 TLS：

* 此时你根据上述临时运行的是不带有`TLS`功能的

### 修改启动命令：

```
./mtproto-proxy -u nobody -p 随便一个不常用的内网端口 -H 外部访问端口 -S 生成的密钥 --aes-pwd proxy-secret proxy-multi.conf -M 5 -D "伪装域名"
```

以上为支持`TLS`功能的临时启动命令

### 连接使用的`Secret`修改为如下字符串

```
ee + secret + domain的十六进制
```

#### 注意：这个`Secret`只是客户端这样填写，服务端还是保持临时运行时设置的`Secret`

#### 伪装的站点最好支持`TLS1.3`


## 进程守护：

新建`/etc/systemd/system/mtproto.service`文件，写入以下内容：
```
[Unit]
Description=MTProxy
After=network.target

[Service]
Type=simple
WorkingDirectory=所在路径
ExecStart=/所在路径/支持TLS功能的临时启动命令去掉./
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 运行进程守护并添加开机自启：

```
systemctl daemon-reload && systemctl enable MTProxy.service $$ systemctl restart MTProxy.service
```

### 查看运行状态：

```
systemctl status MTProxy.service
```

### 重新启动服务：

```
systemctl restart MTProxy.service
```

## 其它问题：

Telegram官方的`MTProto`程序似乎有些问题，在主程序运行时的`PID ＞ 65535`时可能会出现问题，如果运行出现报错可以重启一下试试