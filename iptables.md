# iptables

## 总览

| 表(Table) | 链(Chain)    | 作用范围                                          |
|----------|-------------|-----------------------------------------------|
| filter   | INPUT       | 对进入本机的数据包过滤。                                  |                                       
|          | FORWARD     | 对经过本机转发的数据包过滤。                                |                   
|          | OUTPUT      | 对从本机出去的数据包过滤。                                 |                    
| nat      | PREROUTING  | 用于修改进入本机的数据包的地址（DNAT），在路由选择之前执行。              | 
|          | OUTPUT      | 对从本机产生，目的地需要NAT的数据包执行（例如，本地进程发起的）             |                 
|          | POSTROUTING | 用于修改离开本机的数据包的地址（SNAT/Masquerading），在路由选择之后执行。 | 
| mangle   | PREROUTING  | 对进入本机的数据包进行特殊的修改（例如，修改TTL字段）。                 |           
|          | INPUT       | （同filter的INPUT）                               |
|          | FORWARD     | （同filter的FORWARD）                             |
|          | OUTPUT      | （同filter的OUTPUT，但可以修改更多的报文属性）                 |        
|          | POSTROUTING | 对即将离开本机的数据包进行修改，最后一步处理。                       |                       
| raw      | PREROUTING  | 用于决定数据包是否被state-tracking机制处理，通常用于禁用NAT。       |    
|          | OUTPUT      | （同nat的OUTPUT）                                 |
| security | INPUT       | 用于实施强制访问控制网络规则（如SELinux），这通常出现在filter表之后。     |
|          | OUTPUT      | （同filter的OUTPUT）                              |
|          | FORWARD     | （同filter的FORWARD）                             |

## iptables中的表

### 1. `filter` 表

`filter` 表是 iptables 默认的表，它主要用于决定是否允许网络数据包通过。它包含以下三个链：

- `INPUT`：处理进入本机的流量。
- `FORWARD`：处理经过本机，但并不是最终目的地的流量（路由/转发）。
- `OUTPUT`：处理从本机发出的流量。

#### 使用场景：

- 阻止某个IP或一段IP地址的进入流量。
- 允许或拒绝某个服务（如SSH、HTTP）的访问。
- 设置默认拒绝策略并逐步放行必要的通信。

### 2. `nat` 表

`nat` 表主要用于网络地址转换（NAT），它主要用于修改数据包的源地址或目的地址。NAT 表中的规则只在第一次匹配时被调用，并且创建了一个连接跟踪条目。它包含以下三个链：

- `PREROUTING`：处理刚进入网络接口的流量，在路由之前决定数据包的去向。
- `OUTPUT`：处理从本机产生，目标需要NAT的流量。
- `POSTROUTING`：处理即将离开网络接口的流量，在流量发出之前修改。

#### 使用场景：

- 配置端口转发（如将对外网的80端口访问转发到内网某机器的8080端口）。
- 设置 masquerade （在使用动态IP时，让整个内网通过一个IP地址上网）。
- 修改源IP或目的IP来实现流量的重定向。

### 3. `mangle` 表

`mangle` 表用于修改数据包的某些位（例如：TTL 值），通常用于特殊的数据包修改。它包含以下五个链：

- `PREROUTING`：适用于修改即将进入的数据包。
- `INPUT`：适用于修改进入到本机的数据包。
- `FORWARD`：适用于修改经过本机（路由）的数据包。
- `OUTPUT`：适用于修改从本机发出的数据包。
- `POSTROUTING`：适用于修改即将离开本机的数据包。

#### 使用场景：

- 修改数据流的QoS标记，例如在数据包上设置DSCP字段。
- 修改TCPMSS值（TCP最大段大小）。
- 设置数据包的TTL值。

### 4. `raw` 表

`raw` 表用来决定哪些数据包应该不被state track机制跟踪。它主要用于设置一个例外列表，以便数据包不会被连接追踪系统处理，从而提高性能。它包含两个链：

- `PREROUTING`：适用于标记不应被state track的进入数据包。
- `OUTPUT`：适用于标记不应被state track的出站数据包。

#### 使用场景：

- 对某些不需要被网络状态保持系统追踪的流量进行标记，如某些类型的UDP流量。

### 5. `security` 表

`security` 表主要用于SELinux系统的强制访问控制，设置网络接入规则与安全。它包含以下三个链：

- `INPUT`：适用于控制进入到本机的数据包。
- `OUTPUT`：适用于控制从本机发出的数据包。
- `FORWARD`：适用于控制经过本机的数据包。

#### 使用场景：

- 应用基于SELinux策略的网络访问控制。
- 设置不同安全级别的数据包访问权限。

## 配置iptables

### 常用参数

### 表和链级别的参数：

- `-t <table>`：指定要操作的表，默认是 `filter` 表。例如，`-t nat` 会对 `nat` 表进行操作。
- `链的名称`：这不是参数而是命令的一部分，如 `INPUT`, `FORWARD`, 和 `OUTPUT` 是属于 `filter` 表的默认链，而 `PREROUTING` 和 `POSTROUTING` 属于 `nat` 表。

### 规则级别的参数：

- `-A <chain>`：向指定链增加一条规则（Append）。
- `-I <chain> [rule-number]`：在指定链的指定位置插入（Insert）一条规则。
- `-D <chain> [rule-specification]`：根据规则细节从指定链删除（Delete）一条规则。
- `-F [chain]`：清空指定链中的所有规则（Flush）。如果没有指定链，那么清空所有链的规则。
- `-L [chain] [n]`：列出（List）指定链的规则。如果没有指定链，则列出所有链的规则。如果指定了 `n`，则显示规则与它的序号。
- `-N <chain>`：创建一个新的用户定义链（New chain）。
- `-X [chain]`：删除指定的用户定义链（Delete chain）。如果链未指定，则尝试删除所有非默认链。
- `-P <chain> <target>`：为指定链设置默认策略（Policy），如 `ACCEPT`、`DROP` 或者 `REJECT`。
- `-v`：详细模式，显示每条规则的更多信息。

### 规则级别的具体匹配参数：

- `-p <protocol>`：指定要匹配的协议类型（例如：`tcp`, `udp`, `icmp`）。
- `-s <source>`：指定源地址和/或网络的匹配条件。
- `-d <destination>`：指定目的地址和/或网络的匹配条件。
- `--dport <port>`：当匹配 `tcp` 或 `udp` 流量时，指定目标端口或端口范围。
- `--sport <port>`：当匹配 `tcp` 或 `udp` 流量时，指定源端口或端口范围。
- `-i <interface>`：匹配进入某个接口的流量，用于 `INPUT`, `FORWARD` 和 `PREROUTING` 链。
- `-o <interface>`：匹配离开某个接口的流量，用于 `OUTPUT`, `FORWARD` 和 `POSTROUTING` 链。
- `--state <state>`：使用连接追踪模块匹配包的状态，如 `NEW`, `ESTABLISHED`, `RELATED` 与 `INVALID`。

### 地址的通用表示方法

- **使用 `0.0.0.0/0` 或者仅使用 `-s` 或 `-d` 不跟具体地址** 来匹配所有地址。例如：

  ```bash
  iptables -A INPUT -s 0.0.0.0/0 -p tcp --dport 22 -j ACCEPT
  ```

  也可以不指定 `-s` 参数来默认匹配所有源地址。

- **使用 CIDR 表示法** 来匹配一个地址范围。例如，`192.168.1.0/24` 匹配从 `192.168.1.0` 到 `192.168.1.255` 的所有地址。

### 端口的通用表示方法

- **使用冒号 `:`** 来表示端口范围。例如，`--dport 1024:65535` 匹配所有从 1024 到 65535 的端口。

- **不指定具体端口** 来匹配所有端口。在使用 `-p` 参数指定了协议但没有进一步指定端口时，所有端口将会匹配。例如：

  ```bash
  iptables -A INPUT -p tcp -j ACCEPT
  ```

  这个规则将匹配所有的 TCP 流量，无论是什么端口。

### 特殊情况 - 多地址或端口：

- 如果需要指定多个特定的非连续地址或端口，不能直接在一条规则中使用通配符或者列表。不过，你可以使用多条规则，或者利用 `iptables` 的 `iprange` 扩展模块来匹配地址范围。

  ```bash
  iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 -j ACCEPT
  ```

- 对于端口，可以使用 `multiport` 匹配一系列特定的端口（最多可以指定 15 个）。

  ```bash
  iptables -A INPUT -p tcp -m multiport --dports 22,80,443 -j ACCEPT
  ```

## 例子

### 1. 拒绝某端口上所有tcp输出输入的流量

```shell
iptables -A INPUT -p tcp --dport <set-port> -j DROP
iptables -A OUTPUT -p tcp --sport <set-port> -m state --state ESTABLISHED -j DROP
```

### 2. 重定向某端口的tcp流量到另一端口（双向）

实现以上功能除了配置iptables还需要开启内核中的IP转发，有两种配置方式

通过以下命令临时开启 IP 转发：

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

或者，永久开启它，通过在 `/etc/sysctl.conf` 文件中设置：

```
net.ipv4.ip_forward = 1
```

并运行 `sysctl -p` 来应用更改。

```shell
iptables -t nat -A PREROUTING -p tcp --dport <src-port> -j REDIRECT --to-port <des-port>
iptables -t nat -A POSTROUTING -p tcp --sport <des-port> -j SNAT --to-source :<src-port>
```

### 待补充......
