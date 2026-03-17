以下是针对 **Windows 10 系统**，对 GitHub 项目 [ophub/amlogic-s9xxx-openwrt](https://github.com/ophub/amlogic-s9xxx-openwrt) 进行 **本地打包** 或 **在线（GitHub Actions）打包 OpenWrt 固件** 的完整教程。

> ⚠️ 注意：该项目官方推荐在 **Linux 环境**（如 Ubuntu）下本地编译。Windows 10 无法直接运行其 `remake` 脚本，但可通过 **WSL2（Windows Subsystem for Linux）** 实现本地打包。若不想配置环境，可直接使用 **GitHub Actions 在线编译**（推荐新手）。

***

## ✅ 方案一：【推荐】使用 GitHub Actions 在线打包（无需本地环境）

### 优点：

- 无需安装任何软件
- 自动完成编译、打包、上传
- 支持自定义内核、设备型号、IP 等

### 步骤：

#### 1. Fork 项目到你的 GitHub 账号

- 打开 <https://github.com/ophub/amlogic-s9xxx-openwrt>
- 点右上角 **Fork** → 创建你自己的副本（如 `yourname/amlogic-s9xxx-openwrt`）

#### 2. 启用 GitHub Actions 权限（一般都默认好了，确认一下即可）

- 进入你 Fork 后的仓库 → **Settings** → 左侧 **Actions** → **General**
- 在 **Workflow permissions** 中勾选：
  - ✅ **Read and write permissions**
  - ✅ **Allow GitHub Actions to create and approve pull requests**

> 这一步是为了让 Actions 能上传固件到 **Releases**。

#### 3. 触发在线编译

- 进入你仓库的 **Actions** 标签页
- 左侧选择 **Build OpenWrt system image**
- 点击 **Run workflow**
- 填写参数（示例）：

| 参数               | 示例值             | 说明                                                                   |
| ---------------- | --------------- | -------------------------------------------------------------------- |
| `openwrt_board`  | `s905x3`        | 你要打包的设备芯片，如 `s905x3_s905x2`                                          |
| `openwrt_kernel` | `6.6.30_6.12.y` | 内核版本（参考 [kernel releases](https://github.com/ophub/kernel/releases)） |
| `openwrt_ip`     | `192.168.10.1`  | 默认 IP（可选）                                                            |
| `auto_kernel`    | `true`          | 自动使用同系列最新内核                                                          |
| `builder_name`   | `YourName`      | 构建者签名（可选）                                                            |

> 💡 初学者建议只改 `openwrt_board`（如你的盒子是 X96 Max+，就填 `s905x3`），其他用默认。

#### 4. 等待编译完成（约 10\~30 分钟也可能几个小时）

- 编译成功后，固件会自动上传到：
  - **Actions 页面** → 对应 workflow → **Artifacts**
  - **Releases 页面**（如果启用了上传）

#### 5. 下载固件

- 文件名类似：`openwrt_s905x3_k6.6.30_2026.03.17.img.gz`
- 解压后用 **balenaEtcher** 或 **Rufus** 写入 U 盘即可刷机

***

## ✅ 方案二：Windows 10 本地打包（通过 WSL2）

> 适用于希望完全控制编译过程、频繁定制的用户。

### 第一步：启用 WSL2 并安装 Ubuntu

1. 以管理员身份打开 **PowerShell**，运行：
   ```powershell
   wsl --install -d Ubuntu-22.04
   ```
2. 重启电脑
3. 首次启动 Ubuntu 时，设置用户名和密码（记住！）

> ✅ 验证：打开“开始菜单” → 搜索 “Ubuntu”，能打开终端即成功。

***

### 第二步：在 WSL2 中配置编译环境

打开 **Ubuntu 终端**，依次执行：

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装依赖（按项目要求）
sudo apt install -y build-essential git wget curl python3 python3-pip \
    rsync cpio unzip bc qemu-utils u-boot-tools

# 克隆项目（深度克隆 1 层即可）
git clone --depth=1 https://github.com/ophub/amlogic-s9xxx-openwrt.git
cd amlogic-s9xxx-openwrt
```

***

### 第三步：下载通用 rootfs（必需）

> 该项目不从源码编译 OpenWrt，而是基于预编译的 `openwrt-armsr-armv8-generic-rootfs.tar.gz`

```bash
# 创建目录并下载 rootfs（从 Releases）
mkdir openwrt-armsr
cd openwrt-armsr

# 从官方 Releases 下载最新 rootfs（替换 URL 为最新版）
wget https://github.com/ophub/amlogic-s9xxx-openwrt/releases/latest/download/openwrt-armsr-armv8-generic-rootfs.tar.gz

cd ..
```

> 🔗 你也可以手动下载：[Releases 页面](https://github.com/ophub/amlogic-s9xxx-openwrt/releases) → 找 `openwrt-armsr-armv8-generic-rootfs.tar.gz`

***

### 第四步：执行本地打包命令

```bash
# 示例：打包 s905x3 设备，内核 6.6.30
sudo ./remake -b s905x3 -k 6.6.30

# 其他常用命令：
# sudo ./remake -b s905x3_s905x2 -k 6.6.30_6.12.y   # 多设备多内核
# sudo ./remake -b s905x3 -k 6.6.y -a true          # 自动用 6.6 最新版
```

> ⏱️ 首次运行会下载内核和 u-boot，需 5\~10 分钟（取决于网速）

***

### 第五步：获取固件

- 打包完成后，固件位于：
  ```bash
  ~/amlogic-s9xxx-openwrt/openwrt/out/
  ```
- 文件名如：`openwrt_s905x3_k6.6.30_03.17.1008.img.gz`

#### 如何传到 Windows？

- WSL2 文件路径在 Windows 中为：
  ```
  \\wsl$\Ubuntu-22.04\home\<你的用户名>\amlogic-s9xxx-openwrt\openwrt\out\
  ```
- 直接在 **文件资源管理器** 地址栏粘贴上述路径，复制 `.img.gz` 文件到桌面即可

***

## 📌 常见问题

### Q1: 能否不用 WSL2，直接在 Windows 上打包？

> ❌ 不行。`remake` 是 Bash 脚本，依赖 Linux 工具链（如 `mkfs.ext4`, `dd`, `tar` 等），Windows 原生不支持。

### Q2: 编译失败怎么办？

- 检查网络（需能访问 GitHub）
- 确保 rootfs 文件放在 `openwrt-armsr/` 目录
- 查看错误日志，常见问题是磁盘空间不足（WSL2 默认动态扩容，一般够用）

### Q3: 如何知道我的盒子对应哪个 `-b` 参数？

- 参考项目 README 中的 **Supported Devices** 表格\
  例如：X96 Max+ → SoC 是 `s905x3` → `-b s905x3`

***

## ✅ 总结建议

| 用户类型       | 推荐方案                             |
| ---------- | -------------------------------- |
| 新手 / 偶尔刷机  | ✅ **GitHub Actions 在线打包**（简单、可靠） |
| 开发者 / 频繁定制 | ✅ **WSL2 本地打包**（灵活、快速迭代）         |

> 🔗 官方文档参考：
>
> - [本地打包说明](https://github.com/ophub/amlogic-s9xxx-openwrt#local-packaging)
> - [GitHub Actions 说明](https://github.com/ophub/amlogic-s9xxx-openwrt#use-github-actions-for-compilation)

