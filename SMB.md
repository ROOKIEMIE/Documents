# 服务器配置

> Linux中的权限：
>
> rwx <==> 421



## 服务器网络配置

服务器总共有四张网卡，其中一张使用NAT模式，其余分别连接VNet2、VNet3、VNet4。以下给出配置文件：

**通过使用命令```ip link```查看未配置的网卡名称**

位于```/etc/netplan/```下，**建议修改之前先进行备份!!!**

```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        ens33:
            dhcp4: true
        ens34:
            addresses:
            - 192.168.20.10/24
        ens35:
            addresses:
            - 192.168.30.10/24
        ens36:
            addresses:
            - 192.168.40.10/24
    version: 2
```

> 其余虚拟机需要使用该服务器，则可为其配置多一个张网卡用于内部互联，以下给出其配置文件

Ubuntu22.04

位于```/etc/netplan/```下，**建议修改之前先进行备份!!!**

```yaml
# Let NetworkManager manage all devices on this system
network:
  ethernets:
      ens33:
          dhcp4: true
      ens37:
          addresses:
          - 192.168.20.11/24
  version: 2
  renderer: NetworkManager
```

## SMB服务器配置

Ubuntu-Server 24.04

### 1. 更新系统软件包：

```bash
sudo apt update
sudo apt upgrade
```

### 2. 安装 Samba：

```bash
sudo apt install samba
```

### 3. 检查 Samba 服务状态：

安装完成之后，检查 Samba 服务是否正常运行：

```bash
sudo systemctl status smbd
```

> `smbd` 是核心 Samba 进程，负责提供文件共享服务。

如果 Samba 服务没有运行，可以使用以下命令启动或者重启：

```bash
sudo systemctl start smbd
sudo systemctl enable smbd
```

### 4. 配置 Samba 共享目录：

Samba 的配置文件位于 `/etc/samba/smb.conf`。我们可以通过修改此文件来设置共享目录及其权限。

1. 需要编辑 Samba 的配置文件：

   ```bash
   sudo vim /etc/samba/smb.conf
   ```

2. 在文件末尾添加一个共享的定义：

   举例，在 `/srv/samba/shared` 目录下创建一个共享：

   ```bash
   [Shared]
   # 可以自行修改该目录
   path = /srv/samba/shared
   # 设置某个用户可以访问
   # valid users = username
   browseable = yes
   writable = yes
   # 设置允许匿名访问
   # guest ok = yes
   # 建议为0666
   create mask = 0666
   ```

   - **`path`**：共享目录的位置（你可以使用 `path = /your/shared/directory` 来指定）。
   - **`browseable`**：用户能否通过网络浏览器发现并看到共享目录，值为 `yes` 表示可见。
   - **`writable`**：是否允许用户将文件写入该目录，值为 `yes` 表示开放写入权限。
   - **`guest ok`**：是否允许 `guest` 用户访问（即不使用账号密码登录的匿名访问），设为 `yes` 表示允许。
   - **`create mask`**：指定文件权限掩码，`0775` 表示新创建的文件默认权限为 666。

   你也可以根据实际需求调整这些参数。

3. 保存并退出编辑器（`Esc` 然后输入 `:wq`）。

### 5. 创建共享目录并设置权限：

1. 创建共享目录：

   ```bash
   sudo mkdir -p /srv/samba/shared
   ```

2. 将该目录的所有权更改为与 Samba 服务相关联的用户组 `sambashare`（可以根据实际需要使用其他用户组）：

   ```bash
   sudo chown -R nobody:nogroup /srv/samba/shared
   ```

3. 确保这个目录具有合适的权限，以便用户能够访问和写入文件：

   ```bash
   sudo chmod -R 0775 /srv/samba/shared
   ```

### 6.添加用户到 Samba 用户数据库：

即使用户已经存在于 Linux 系统中，仍需将其添加到 Samba 的用户数据库中，并为其设置 Samba 密码。Samba 有自己的用户和密码存储系统，这与 Linux 的系统用户密码是分开的。

使用以下命令将现有的 Linux 用户 `username` 添加到 Samba 用户数据库中：

```bash
sudo smbpasswd -a username
```

你会被要求设置该用户的 Samba 密码，与该用户的 Linux 系统密码可以不同。

输出将如下所示：

```bash
New SMB password:
Retype new SMB password:
```

输入并确认密码后，用户将添加到 Samba 数据库中。

### 7. 配置防火墙：

如果启用了防火墙，需要确保允许 Samba 流量通过，具体步骤如下：

```bash
sudo ufw allow 'Samba'
```

### 8. 重新启动 Samba 服务：

任何对 `smb.conf` 文件的更改都需要重新启动 Samba 服务以生效。

```bash
sudo systemctl restart smbd
```

### 9. 测试 Samba 共享：

在Windows的此电脑中，空白处右键，选择```添加网络位置```，如何SMB服务器地址```\\<SMB Server IP>\Shared```，输入相关的用户名和密码。

## SMB客户端配置

Linux 环境：Ubuntu22.04Desktop

安装必要的依赖

### 安装 `cifs-utils` 工具包

要从 **Linux** 系统访问 **Samba** 共享目录，首先确保安装了支持 `SMB/CIFS` 的工具包 `cifs-utils`。在 Ubuntu 上可以通过下面的命令安装：

```bash
sudo apt update
sudo apt install cifs-utils
```

### 临时挂载

#### 1. 创建挂载点

本地的挂载点是你希望 Samba 共享目录在本地系统中显示的路径。首先需要创建一个空目录作为挂载点。

例如，我们在 `/mnt` 下创建一个 `shared` 目录：

```bash
sudo mkdir -p /mnt/shared
```

#### 2. 挂载 Samba 共享目录

使用 `mount.cifs` 或 `mount` 命令来挂载远程 Samba 共享目录。假设你分享的目录名是 `Shared`，服务器 IP 是 `192.168.1.100`，Samba 用户名是 `username`。

执行如下挂载命令：

```bash
sudo mount -t cifs -o username=username,password=your_password //192.168.1.100/Shared /mnt/shared
```

解释：

- `-t cifs`：挂载类型为 CIFS（这是 SMB 的文件系统类型）。
- `username=username,password=your_password`：指定 Samba 共享的登录凭据。
- `//192.168.1.100/Shared`：这是 Samba 共享的路径。
- `/mnt/shared`：这是本地系统上用于挂载的目录。

#### 3. 验证挂载

执行挂载命令后，你可以通过以下方式验证挂载是否成功：

1. 查看挂载情况：

   ```bash
   df -h
   ```

   输出中应该能够看到：

   ```
   //192.168.1.100/Shared  x.xG  x.xG  x.xG  xx%  /mnt/shared
   ```

2. 访问挂载后的共享目录：

   ```bash
   ls /mnt/shared
   ```

   你应该能够看到 Samba 服务器上共享目录中的文件。

### 永久挂载

#### 1. 创建挂载点

创建挂载点，与临时挂载的方法类似。例如：

```bash
sudo mkdir -p /mnt/shared
```

#### 2. 编辑 `/etc/fstab`

在 **/etc/fstab** 文件中添加一行，设置 Samba 共享的详细挂载信息。

编辑 `/etc/fstab` 文件：

```bash
sudo nano /etc/fstab
```

在文件末尾添加一行来指定挂载配置，例如：

```bash
//192.168.1.100/Shared /mnt/shared cifs username=username,password=your_password,uid=1000,gid=1000,iocharset=utf8 0 0
```

解释：

- `//192.168.1.100/Shared`：这是 Samba 服务器上的共享路径。
- `/mnt/shared`：这是本地用于挂载共享目录的挂载点。
- `cifs`：指定挂载类型为 CIFS，使用 SMB 文件系统。
- `username=username,password=your_password`：提供登录凭据（用户名和密码）。注意：**密码通过这种方式会暴露在 `/etc/fstab` 中**，如果安全性是一个问题，可以改进这个方式，见下一步的“安全挂载”方法。
- `uid=1000,gid=1000`：将挂载后的文件的所有者和组设置为当前用户（通常 `UID` 为 1000 是你第一个创建的用户）。
- `iocharset=utf8`：确保文件名使用 `UTF-8` 编码以防文件名中文或特殊字符出现乱码。

保存并退出（按 `Ctrl + O`，然后按 `Enter`，再按 `Ctrl + X` 退出）。

#### 3. 选择性地使用凭据文件进行更安全的挂载

为了避免将密码明文存储在 `/etc/fstab` 中，你可以使用单独的凭据文件存储用户凭证。

1. 创建一个凭据文件，如 `/etc/samba/credentials.txt`：

   ```bash
   sudo nano /etc/samba/credentials.txt
   ```

2. 将用户名和密码填入此文件：

   ```
   username=username
   password=your_password
   ```

   保存并退出。

3. 将凭据文件权限限制为只允许 root 读取：

   ```bash
   sudo chmod 600 /etc/samba/credentials.txt
   ```

4. 修改 `/etc/fstab` 中对应的挂载行，指向凭据文件：

   ```bash
   //192.168.1.100/Shared /mnt/shared cifs credentials=/etc/samba/credentials.txt,uid=1000,gid=1000,iocharset=utf8 0 0
   ```

#### 4. 挂载测试

在修改了 `/etc/fstab` 后，使用如下命令立即挂载所有在 `/etc/fstab` 中声明的文件系统，而不必重新启动：

```bash
sudo mount -a
```

如果没有看到任何错误信息，说明挂载成功。你可以再次执行 `df -h` 确认挂载是否成功，或者直接访问 `/mnt/shared` 下的文件。

### 创建软连接

通过软连接可以更方便地使用该挂载文件夹

```shell
ln -s /mnt/shared /home/username/shared
```

