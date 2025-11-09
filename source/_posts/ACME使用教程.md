---
title: "acme.sh使用教程"
date: 2025-08-01 12:00:00
---

# 一、输入一下命令进行安装：
```Bash
curl https://get.acme.sh | sh -s email=你的邮箱
```
命令运行完成后 acme.sh 会被安装到：
```
~/.acme.sh
```
<!--more--> 
# 二、证书签发

## 1.直接签发

只需要指定域名，并指定域名所在的网站根目录 acme.sh 会全自动的生成验证文件，并放到网站的根目录，验证完成后会聪明的删除验证文件，整个过程没有任何副作用。
```
acme.sh --issue -d mydomain.com -d www.mydomain.com --webroot /home/wwwroot/mydomain.com/
```

### (1).使用 Apache 模式

如果你用的 Apache 服务器，acme.sh 还可以智能的从 Apache 的配置中自动完成验证，你不需要指定网站根目录：
```
acme.sh --issue --apache -d example.com -d www.example.com -d cp.example.com
```
### (2).使用 Nginx 模式

如果你用的 Nginx 服务器，或者反代，acme.sh 还可以智能的从 Nginx 的配置中自动完成验证，你不需要指定网站根目录：
```
acme.sh --issue --nginx -d example.com -d www.example.com -d cp.example.com
```

* 注意，无论是 Apache 还是 Nginx 模式，acme.sh 在完成验证之后，会恢复到之前的状态，都不会私自更改程序本身的配置。好处是你不用担心配置被搞坏，也有一个缺点，你需要自己配置 SSL 项，否则只能成功生成证书，你的网站还是无法正常使用 HTTPS。

## 2.使用独立服务模式

如果 80 端口是空闲的，acme.sh 还能假装自己是一个 WebServer，临时监听 80 端口，完成验证：
```
acme.sh --issue --standalone -d example.com -d www.example.com -d cp.example.com
```

## 3.手动验证

### (1).纯手动验证

这需要你手动在域名上添加一条 TXT 解析记录，验证域名所有权。

注意，如果使用手动验证，acme.sh 将无法自动更新证书，每次都需要手动添加解析来验证域名所有权。如果有自动更新证书的需求，请使用自动验证（DNS API）。
```
acme.sh --issue --dns -d example.com -d www.example.com -d cp.example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```
然后，acme.sh 会生成相应的解析记录显示出来，你只需要在你的域名管理面板中添加这条 TXT 记录即可。

等待解析完成之后，执行以下命令重新生成证书：
```
acme.sh --renew --dns -d example.com --dns -d www.example.com --dns -d cp.example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```
注意这里现在用的是 参数 `--renew`

### (2).自动验证（DNS API）

请参考：
```
https://github.com/acmesh-official/acme.sh/wiki/dnsapi
```

# 三、复制证书

证书生成好以后，我们需要把证书复制给对应的 Apache、Nginx 或其他服务器去使用。

必须使用 命令来把证书复制到目标文件，请勿直接使用 目录下的证书文件，这里面的文件都是内部使用，而且目录结构将来可能会变化：`--install-cert`

#### Apache 示例：
```
acme.sh --install-cert -d example.com \
--cert-file      /path/to/certfile/in/apache/cert.pem  \
--key-file       /path/to/keyfile/in/apache/key.pem  \
--fullchain-file /path/to/fullchain/certfile/apache/fullchain.pem \
--reloadcmd     "service apache2 force-reload"
```
#### Nginx 示例：
```
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx reload"
```

默认情况下，证书每 60 天更新一次（可自定义）。更新证书后，Apache 或者 Nginx 服务会通过 传递的命令自动重载配置。

* 注意：证书会自动申请续签，但是如果没有正确的命令，证书可能无法被重新应用到 Apache 或者 Nginx，因为配置没有被重载。reloadcmdreloadcmd

* 注：我是直接使用 ~/acme.sh/example.com 文件夹内的文件，只需要将 fullchain.cer 在上， ca.cer 在下拼合在一个文件，就可以拼合一个有完整证书链的一个证书，私钥为 example.com.key，但是这样操作极其不规范，不建议使用.

# 四、其它

## 1.各 CA 对比

以下排序按我个人推荐程度进行排列

| CA | 时长（天） | 签发ECC证书 | 单证书最大域名个数 | 国际化域名 | 测试服务器 |
| :----:| :----: | :----: | :----: | :----: | :----: |
| ZeroSSL | 90 | 根证书为RSA | 100 | 支持 | 没有 |
| Let's Encrypt | 90 | 根证书为ECC | 100 | 支持 | 有 |
| Google | 90 | 根证书为RSA | 100 | 不支持 | 有 |
| SSL.com | 104 | 根证书为ECC | 2 | 支持 | 没有 |
| Buypass | 180 | 根证书为RSA | 5 | 支持 | 有 |

注：仅 ZeroSSL 支持使用 Http 认证方式签发IP证书

## 2.修改默认 CA

acme.sh 脚本默认 CA 服务器是 ZeroSSL ，有时可能会导致获取证书的时候一直出现：`The CA is processing your order，please just wait.`

只需要把 CA 服务器改成 Let's Encrypt 即可，虽然更改以后还是有概率出现 pending，但基本 2-3 次即可成功
```
acme.sh --set-default-ca --server letsencrypt
```
更高级的用法请参考： https://github.com/acmesh-official/acme.sh/wiki/How-to-issue-a-cert

## 3.有关ZeroSSL

如果使用ZeroSSL签发证书提示需要指定 KeyID：
```
Using CA: https://acme.zerossl.com/v2/DV90
Registering account: https://acme.zerossl.com/v2/DV90
Already register EAB.
ACCOUNT_THUMBPRINT='XXXXXXXXXXXXXXXXXXXXXXX'
Single domain='sub.mydomain.com.br'
Getting domain auth token for each domain
Create new order error. Le_OrderFinalize not found. {"type":"urn:ietf:params:acme:error:malformed","status":400,"detail":"A Key ID MUST be specified"}
```
请使用 EAB 凭据：

使用外部帐户绑定 （EAB） 凭据引导，如下所示：

从 https://app.zerossl.com/developer 生成 EAB 凭据

注册 EAB 凭据：
```
acme.sh  --register-account  --server zerossl \
        --eab-kid  xxxxxxxxxxxx  \
        --eab-hmac-key  xxxxxxxxx
```

## 4.有关Google Trust Services

### 使用ACME申请证书的服务器必须可以访问谷歌

请按照Google的指南创建 EAB 密钥和 EAB ID：

https://cloud.google.com/public-certificate-authority/docs/quickstart

注册 EAB 凭据：
```
acme.sh  --register-account  -m  myemail@example.com --server google \
    --eab-kid xxxxxxx \
    --eab-hmac-key xxxxxxx
```