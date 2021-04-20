---
weight: 1
title: "使用 Docker 部署 V2Ray"
---

## 部署环境

仅用于参考，并非硬性要求，需按照实际情况自行更改。

1. 服务器为腾讯云轻量应用服务器（香港或新加坡）；
2. 服务器操作系统为 Ubuntu 20.04 LTS；
3. 系统默认用户 ubuntu，非默认用户 zhangsan；
4. 在海外服务商购买的域名（例如 GoDaddy、Google Domain），测试用例为 example.com；

## 部署步骤

### 配置服务器

1. 卸载腾讯云监控组件（非腾讯云轻量应用服务器请略过）。

```bash
sudo /usr/local/qcloud/stargate/admin/uninstall.sh
sudo /usr/local/qcloud/YunJing/uninst.sh
sudo /usr/local/qcloud/monitor/barad/admin/uninstall.sh
```

2. 创建新的用户 zhangsan 并修改密码。

```bash
sudo useradd -s /bin/bash -d /home/zhangsan -m zhangsan
sudo passwd zhangsan
```

3. 配置 zhangsan 允许使用 sudo 命令。

```bash
sudo visudo
```

在文件最末尾找到以下内容：

```
#includedir /etc/sudoers.d
lighthouse ALL=(ALL) NOPASSWD: ALL
ubuntu  ALL=(ALL:ALL) NOPASSWD: ALL
```

将以上内容修改为以下内容：

```
#includedir /etc/sudoers.d
zhangsan ALL=(ALL) NOPASSWD: ALL
```

修改后 Ctrl + O 写入，按 Enter 确定, 最后 Ctrl + X 退出。

4. 修改镜像源。

```bash
sudo nano /etc/apt/sources.list
```

把 `sources.list` 内所有内容删除后，粘贴以下内容并保存。

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

更新并升级所有软件包。

```bash
sudo apt update
sudo apt upgrade -y
```

由于软件包比较多，需等待一段时间，期间会多次出现大致以下格式的选项，全部 Y 然后 Enter。

```
Configuration file '/etc/cloud/cloud.cfg'
 ==> Modified (by you or by a script) since installation.
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** cloud.cfg (Y/I/N/O/D/Z) [default=N] ? 
```

5. 删除默认用户。

重启服务器以便断开所有 seesion。

```
sudo reboot
```

重新连接到服务器后，删除默认用户和腾讯云轻量应用服务器默认用户。

```
sudo userdel -r ubuntu
sudo userdel -r lighthouse
```

### 申请 SSL 证书

1. 安装 snapd 和 certbot。

```bash
sudo apt install -y snapd
sudo snap install --classic certbot
```

2. 使用 DNS 验证域名的方式申请证书。

```bash
sudo certbot certonly -d *.example.com --manual --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory
```

输入邮箱以便接收紧急更新以及安全通知邮件，输入后按 Enter 键。

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator manual, Installer None
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): 
```

输入 Y 按 Enter 键，同意相关服务协议。

```
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
(Y)es/(N)o: 
```

输入 Y 按 Enter 键，同意接收新闻邮件。

```
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
(Y)es/(N)o: 
```

**最关键的一步**，创建 DNS 记录类型为 TXT，名称为 `_acme-challenge`。值为所要求的字符串，参考：

```
4HXdvgYOlOoiWc3ktriAxrnX1fLvljl1QpzfCWeANoI
```

```
Please deploy a DNS TXT record under the name
_acme-challenge.example.com with the following value:

4HXdvgYOlOoiWc3ktriAxrnX1fLvljl1QpzfCWeANoI

Before continuing, verify the record is deployed.
Press Enter to Continue
```

**等待几分钟 DNS 刷新**，按 Enter 键后为以下内容以为申请成功。

```
Waiting for verification...
Cleaning up challenges
Subscribe to the EFF mailing list (email: example@email.com).

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your certificate will expire on 2021-07-19. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

### 安装 Docker

1. 安装所需依赖。

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

2. 添加 Docker GPG 公钥。

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

3. 更新软件包并安装 Docker CE。

```bash
sudo apt-get update
sudo apt-get install -y docker-ce
```

4. 将当前用户添加到 `docker` 用户组，需重新连接 ssh 才会生效。

```bash
sudo usermod -aG docker $USER
exit
```

### 启用 BBR 拥塞控制算法

```bash
sudo echo "net.core.default_qdisc=fq" >>/etc/sysctl.conf
sudo echo "net.ipv4.tcp_congestion_control=bbr" >>/etc/sysctl.conf
sudo sysctl -p
sudo sysctl net.ipv4.tcp_available_congestion_control
```

### 编辑 v2ray 服务端配置

1. 创建目录以及 `config.json` 文件。

```bash
sudo mkdir -p /opt/v2fly-core
sudo nano /opt/v2fly-core/config.json
```

把以下配置粘贴到 `config.json` 文件内（自行修改 UUID）。

```json
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "3DD78345-27F8-4B14-8C6E-7856A6C4FB8D",
            "alterId": 0
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls",
        "tlsSettings": {
          "certificates": [
            {
              "certificateFile": "/etc/letsencrypt/live/example.com/fullchain.pem",
              "keyFile": "/etc/letsencrypt/live/example.com/privkey.pem"
            }
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

拉取 `v2fly/v2fly-core` 镜像并启动容器（需要自行修改挂载的 SSL 证书路径）。

```bash
docker run --name v2fly-core \
    --restart always \
    -p 443:443/tcp \
    -v /opt/v2fly-core/config.json:/etc/v2ray/config.json:ro \
    -v /etc/letsencrypt/live/example.com:/etc/letsencrypt/live/example.com:ro \
    -v /etc/letsencrypt/archive/example.com:/etc/letsencrypt/archive/example.com:ro \
    -dit v2fly/v2fly-core:latest
```

### 添加 A 记录

在域名的 DNS 记录中添加 A 记录，名称为 `proxy` 或自定义其他名称。DNS 记录内容为服务器 IP 地址。添加完成后，`proxy.example.com` 就是 V2Ray 服务器地址。

### 测试代理

将配置填入任意代理客户端并连接到代理服务器，使用浏览器打开[Google 学术](https://scholar.google.com)，无论是正常访问还是 403 错误（Google 学术屏蔽了部分 IP 段）都说明部署成功。
