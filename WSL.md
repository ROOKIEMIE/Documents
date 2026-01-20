# WSL

## 安装WSL

在WSL的Linux中查看本地Windows主机的IP：

```bash
ip route show | grep -i default | awk '{ print $3}'
```

WSL中的Linux使用本地Windows主机中安装的代理软件所提供的代理端口：

```bash
export http_proxy=http://<本地Windows主机IP>:<代理软件所提供的代理端口>
export https_proxy=$http_proxy
```

修改本地Windows防火墙规则，实现放行本地Windows主机上的某个端口至WSL中的Linux的流量

注意：**该操作是在本地Windows主机中进行！！！** 

```powershell
# 自定义端口
$selfPort = 52336

# 创建入站规则允许连接到自定义端口
New-NetFirewallRule -DisplayName "Allow Inbound TCP Port $selfPort for WSL" `
-Direction Inbound -Protocol TCP -LocalPort $selfPort -Action Allow -Profile Any

# 创建出站规则允许连接到自定义端口
New-NetFirewallRule -DisplayName "Allow Outbound TCP Port $selfPort for WSL" `
-Direction Outbound -Protocol TCP -LocalPort $selfPort -Action Allow -Profile Any
```

顺便给出移除规则的命令：

```powershell
# 自定义端口
$selfPort = 52336

# 移除入站规则
Remove-NetFirewallRule -DisplayName "Allow Inbound TCP Port $selfPort for WSL"

# 移除出站规则
Remove-NetFirewallRule -DisplayName "Allow Outbound TCP Port $selfPort for WSL"
```



# 参考博文

[Windows下安装WSL2及基础配置](https://zhuanlan.zhihu.com/p/148511634) ps：WSL安装好后，安装docker时建议直接使用docker官方提供的Docker Desktop软件

[Windows中，安装kali的WSL官方文档](https://www.kali.org/docs/wsl/win-kex/#win-kex-supports-three-modes)

[Windows下安装WSL2后无法ping同WSL内部的虚拟机，解决办法](https://zhuanlan.zhihu.com/p/365058237)

[Windos中，使用WSL时发现其内存占用过大，通过以下方法限制WSL的内存使用](https://www.shuijingwanwq.com/2022/02/23/6014/#google_vignette)

