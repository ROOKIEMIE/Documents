# WireGuard

## 服务器配置

### 创建公私密钥对

```shell
wg genkey | tee <private key file name>.key | wg pubkey > <public key file name>.key
```

### 服务器侧的WireGuard配置

以Ubuntu发行版为例

WireGuard配置一般需要放置于目录``` /etc/wireguard```下

```shell
sudo vim /etc/wireguard/wg0.conf
```

下面给出服务器侧的配置示例，可根据具体情况进行自定义修改。

```ini
[Interface]
# 服务器在VPN中的IP
Address = 10.0.0.1/24
SaveConfig = true
# 服务器私钥，通过[cat 服务器私钥文件]获得
PrivateKey = <server_private.key>
# 默认端口
ListenPort = 51820

# 以下为该VPN中的客户端节点配置，以两个节点为例。可以以该配置为基础增加更多的节点配置
[Peer]
# 第一个客户端公钥
PublicKey = <client1_public_key>
# 第一个客户端VPNIP
AllowedIPs = 10.0.0.2/32

[Peer]
# 第二个客户端公钥
PublicKey = <client2_public_key>
# 第二个客户端VPNIP
AllowedIPs = 10.0.0.3/32
```

### 服务器侧的一些准备工作

需要确保开启ipv4端口转发

```shell
sudo vim /etc/sysctl.conf

net.ipv4.ip_forward = 1

esc
:wq

# 应用更改
sudo sysctl -p

# 若有防火墙则需要提前放行
sudo ufw allow 51820/udp
sudo ufw enable
sudo ufw status
```

### 启动WireGuard

```shell
sudo wg-quick up wg0
# 可选：将WireGuard设置为服务器服务
sudo systemctl enable wg-quick@wg0
```

### 添加客户端节点(动态)

```shell
sudo wg set wg0 peer <CLIENT_PUBKEY> allowed-ips <For exmaple:10.0.0.2/32>
```

### 保存运行配置

```shell
sudo wg showconf wg0 > /etc/wireguard/wg0.conf
```

## 客户端配置

一般情况下，客户端都是通过导入配置文件进行使用，下面给出客户端的配置文件示例，可根据实际情况进行自定义配置后使用。

```ini
[Interface]
# 客户端的私钥
PrivateKey = <client1_private_key>
# 客户端在VPN中的IP
Address = 10.0.0.2/24

[Peer]
# 服务端的WireGuard公钥
PublicKey = <server_public_key>
# 服务端的公网IP
Endpoint = <your-server-public-ip>:51820
# 允许访问整个VPN网段
AllowedIPs = 10.0.0.0/24
```

