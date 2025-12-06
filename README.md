# ShadowShock-Tutorial
## 目的
1. 这是一个帮助大家从 0 开始构建 VPN 的教程，总共分三步：(1) 购买一个简单的海外服务器 (2) 在服务器上配置 shadowshock (3) 在你自己的电脑上配置 shadowshock 
2. 之前我有参与贡献一个类似的 [教程](https://github.com/zhaoweih/Shadowsocks-Tutorial?tab=readme-ov-file) 那个教程已经写的很完善
3. 但是上述教程在第 (2) 在服务器上配置 shadowshock 那一步总是会出现各种问题，所以我单独构建了这个教程作为一个补丁，帮助大家解决如何在已经完成 (1) 购买一个简单的海外服务器的前提下配置 shadowshock 的问题

## 步骤和代码
你现在应该已经完成了 [教程](https://github.com/zhaoweih/Shadowsocks-Tutorial?tab=readme-ov-file) 中的前 8 个步骤

### 在服务器安装 SS

#### 1. 更新系统 
```bash
sudo dnf update -y
```

#### 2. 安装编译依赖和工具

启用 CRB 仓库，这对 Centos 9 系统至关重要
```bash
sudo dnf config-manager --set-enabled crb
```
安装依赖
```bash
sudo dnf install -y autoconf automake libtool gcc git make \
  c-ares-devel libev-devel libsodium-devel openssl-devel \
  pcre-devel mbedtls-devel asciidoc xmlto
```

#### 3. 下载 Shadowsocks-libev 源码（必须带子模块, 命令里面必须用 recursive）

```bash
git clone --recursive https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
```
#### 4. 编译 Shadowsocks-libev

```bash
./autogen.sh
./configure
make
sudo make install
```
验证安装路径

```bash
which ss-server
```
应该输出：
```bash
/usr/local/bin/ss-server
```

#### 5. 创建配置文件

```bash
sudo mkdir -p /etc/shadowsocks-libev
sudo tee /etc/shadowsocks-libev/config.json << EOF
{
    "server": "0.0.0.0",
    "server_port": 11597,          
    "password": "xxxxxxxxx",  
    "timeout": 300,
    "method": "aes-256-gcm",
    "fast_open": true,
    "mode": "tcp_and_udp"
}
EOF
```
- `server_port`: 你想要指定的端口，随便一个在 0-65535 之间的数字 作为 server 对外提供服务的端口
- `password`: 指定一个连接这个服务器的密码
- `method`: 加密方式，保持默认即可


#### 6. 创建 systemd 服务文件


```bash
sudo tee /etc/systemd/system/shadowsocks-libev.service << EOF
[Unit]
Description=Shadowsocks-libev Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks-libev/config.json
Restart=on-failure
LimitNOFILE=51200

[Install]
WantedBy=multi-user.target
EOF
```

#### 7. 启动服务并设置开机自启

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now shadowsocks-libev
```

查看状态：

```bash
systemctl status shadowsocks-libev
```

应该显示：

```bash
Active: active (running)
```

可以点击键盘 `q` 来退出显示面板然后进行下一步


#### 8. 开放防火墙端口（若 firewalld 开启）
```bash
sudo firewall-cmd --permanent --add-port=11597/tcp
sudo firewall-cmd --permanent --add-port=11597/udp
sudo firewall-cmd --reload
```

- 这里的 `port` 数字需要和你之前设置的端口数字一致

#### 9. 运行和维护命令
重启服务：

```bash
sudo systemctl restart shadowsocks-libev
```
停止服务：

```bash
sudo systemctl stop shadowsocks-libev
```
查看日志：

```bash
journalctl -u shadowsocks-libev -f
```

## 结语

完成这一步之后，你又可以返回 [教程](https://github.com/zhaoweih/Shadowsocks-Tutorial?tab=readme-ov-file) 查看后续的步骤，例如下载客户端并配置连接