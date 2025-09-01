# ZFS使用指南

ZFS官方仓库：https://github.com/openzfs/zfs

ZFS官方文档：https://openzfs.github.io/openzfs-docs/Getting%20Started/index.html

## 安装

### 包管理器安装

> Debian

```shell
sudo apt update
sudo apt install zfsutils-linux
zfs --version
```

## 环境状态

> 查看磁盘挂载

```shell
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
```

> 查看各磁盘分区情况

```shell
sudo wipefs -a /dev/nvme0n1
sudo wipefs -a /dev/nvme1n1
sudo wipefs -a /dev/nvme2n1
sudo wipefs -a /dev/nvme3n1
```

## 使用

### 创建存储池

```shell
sudo zpool create -f tank raidz2 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
```

上述命令中各参数的含义：

tank：池的名字，后续访问挂载点需要用到。

raidz2：阵列模式，以下给出更多的阵列模式。

#### RAID-Z 系列（ZFS 独有）

RAID-Z 是 ZFS 实现的数据冗余机制，避免了传统 RAID 5 中的“写孔（Write Hole）问题”，保证数据的一致性。

##### RAID-Z1

- **容错能力**：可容忍 1 块磁盘故障。
- **推荐磁盘数**：最少 3 块。
- **存储效率**：`(n - 1)/n`，例如 4 块盘中，有效数据为 3 块容量。
- **用途场景**：数据安全与存储效率的平衡选择。

示例命令：

```
zpool create mypool raidz1 /dev/sdX /dev/sdY /dev/sdZ
```

------

##### RAID-Z2

- **容错能力**：可容忍 2 块磁盘故障。
- **推荐磁盘数**：最少 4 块，推荐 5~10 块。
- **存储效率**：`(n - 2)/n`，如 6 块盘中，有效为 4 块容量。
- **用途场景**：对数据可靠性要求较高的场景，如 NAS/服务器。

示例命令：

```
zpool create mypool raidz2 /dev/sdX /dev/sdY /dev/sdZ /dev/sdW
```

------

##### RAID-Z3（高级模式）

- **容错能力**：可容忍 3 块磁盘故障。
- **推荐磁盘数**：最少 5 块，推荐 6~12 块。
- **存储效率**：`(n - 3)/n`
- **用途场景**：极高数据可靠性要求，如大数据归档、科研数据、企业数据中心。

示例命令：

```
zpool create mypool raidz3 /dev/sdX /dev/sdY /dev/sdZ /dev/sdW /dev/sdV
```

------

#### 镜像（Mirror）

ZFS 支持类似 RAID 1 的镜像模式。

##### Mirror（双盘/多盘镜像）

- **容错能力**：可容忍任意 n-1 块损坏（假设有 n 个镜像副本）
- **最小设备数**：2 个磁盘一组
- **存储效率**：50%（每个写入数据被复制到两个盘）
- **用途场景**：高读取性能、强一致性保障、小型服务器或数据库系统

示例命令（两个磁盘镜像）：

```
zpool create mypool mirror /dev/sdX /dev/sdY
```

 多个镜像组（分布式 RAID 10 类结构）：

```
zpool create mypool \
  mirror /dev/sdX /dev/sdY \
  mirror /dev/sdZ /dev/sdW
```

------

#### RAID-0（Striped）

ZFS 也支持无冗余的 **条带模式**，即 RAID 0。

- **容错能力**：无（任意一块盘损坏就会丢数据）
- **性能**：最高，写入时数据会分布在所有磁盘上并行写入
- **存储效率**：100%
- **用途场景**：不重要数据、高性能缓存、临时存储、转码中间文件等

示例命令：

```
zpool create mypool /dev/sdX /dev/sdY /dev/sdZ
```

------

#### 其他支持方式

##### 单盘池（Single Disk）

ZFS 可以在单块磁盘上创建池，但不推荐用于重要数据：

```
zpool create mypool /dev/sdX
```

可后续通过 `zpool attach` 添加镜像：

```
zpool attach mypool /dev/sdX /dev/sdY
```

------

#### 各模式对比表

| 模式    | 容错块数 | 推荐盘数  | 存储效率 | 性能      | 安全性 |
| ------- | -------- | --------- | -------- | --------- | ------ |
| RAID-Z1 | 1        | ≥3        | 中       | 中        | 一般   |
| RAID-Z2 | 2        | ≥4        | 中下     | 中        | 较高   |
| RAID-Z3 | 3        | ≥5        | 低       | 中        | 很高   |
| Mirror  | 1        | 2（每组） | 低       | 读高/写中 | 高     |
| RAID-0  | 0        | ≥2        | 高       | 很高      | 无     |

------

#### 关键概念补充

##### vdev（虚拟设备）

ZFS 构建池的基本单位是 vdev（virtual device）：

- 一个池由多个 vdev 构成。
- 每个 vdev 可以是 RAID-Z、镜像、单盘等结构。
- **池的容错能力由 vdev 决定，整个池会因某个 vdev 损坏而挂掉。**

因此，**多个 RAID-Z vdev 才能同时提升容错与性能。**

### 查看池状态

```shell
sudo zpool status -v
# 状态示例
  pool: tank
 state: DEGRADED
status: One or more devices has been taken offline by the administrator.
        Sufficient replicas exist for the pool to continue functioning in a
        degraded state.
action: Replace the device and restore the redundancy of the pool.
  scan: scrub repaired 0B in 0 days 00:10:00 with 0 errors on ...
config:

        NAME                      STATE     READ WRITE CKSUM
        tank                      DEGRADED     0     0     0
          raidz1-0                DEGRADED     0     0     0
            nvme0n1               ONLINE       0     0     0
            nvme1n1               ONLINE       0     0     0
            nvme2n1               UNAVAIL      0     0     0  (device faulted)
            nvme3n1               ONLINE       0     0     0
```

从中你能明确看到：

- RAID-Z1 容忍了 nvme2n1（盘3）损坏
- 整个池状态为 DEGRADED（降级运行），但 **数据仍然可读写**

#### 进行主动数据校验

```shell
sudo zpool scrub tank
# 再次检查池状态
sudo zpool status -v
# 输出示例
  scan: scrub repaired 0B in 0 days 00:10:00 with 0 errors on ...
```

说明目前剩下的 3 块盘包含了足够的奇偶信息，ZFS **成功校验并自动重构损坏数据**。

#### 替换损坏盘并恢复 RAID-Z1 的完整性

> 盘1：nvme0n1
>
> 盘2：nvme1n1
>
> 盘3：nvme2n1
>
> 盘4：nvme3n1

假设替换了盘3，用一块新磁盘 `/dev/nvme4n1` 替换原来的 `/dev/nvme2n1`。需要在替换操作后执行ZFS替换命令：

```shell
# 此命令告诉 ZFS：使用新磁盘重建原来那块坏盘的全部数据块。
sudo zpool replace tank /dev/nvme2n1 /dev/nvme4n1
# 或者使用盘ID执行该命令
sudo zpool replace tank <旧盘的GUID或ID> /dev/nvme4n1
# 再次查看池状态
sudo zpool status
# 输出示例
scan: resilver in progress since ...
        100G scanned out of 2.00T at 200M/s, 2h30m to go
# 重建完成后，再次检查池状态，输出示例如下：
  pool: tank
 state: ONLINE
status: All vdevs are healthy
# 表明冗余已恢复，RAID-Z1 再次具备 1 块盘容错能力。
```

#### 补充

- 校验检查应定期执行，可选择加入cron定时任务：

```shell
sudo zpool scrub tank
```

- **备份池配置信息**

```shell
sudo zpool export tank
sudo zpool import -d /dev/disk/by-id tank
```

### 数据集

#### 创建数据集

创建数据集需要依赖池的名字，ZFS 会自动挂载到 `/tank/shared`。

```shell
sudo zfs create tank/shared
```

#### 查看所有数据集

```shell
zfs list
```

#### 查看数据集属性

如，查看数据集的mountpoint和canmount属性的当前值

```shell
zfs get mountpoint,canmount tank/manager
```

### 缓存

#### 查看池缓存状态

```shell
sudo zpool get cachefile tank
# 输出示例
NAME  PROPERTY   VALUE                       SOURCE
tank  cachefile  -                           default
```

如果是 `-`（默认），说明 **ZFS 没有使用任何池配置缓存文件**，这会导致开机时系统不知道有池可导入。

#### 设置池缓存

```shell
sudo zpool import tank   # 如果未导入则导入
sudo zpool set cachefile=/etc/zfs/zpool.cache tank
sudo zpool export tank
sudo zpool import tank
```

#### 更新缓存文件

```shell
sudo update-initramfs -u
```