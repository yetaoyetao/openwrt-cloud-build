# OpenWrt 云编译（GitHub Actions）

零本地环境：所有编译都在 GitHub 的服务器上跑，你只需要浏览器 + 一个免费 GitHub 账号。

**产出**：针对指定设备、包含指定软件包和自定义文件的 OpenWrt 固件（.bin + sha256sums 校验文件）。

**耗时**：约 5~15 分钟。  
**成本**：公开仓库完全免费。

---

## 第一次使用（5 步）

### 第 1 步：把本文件夹上传到 GitHub

1. 打开 [github.com](https://github.com)，注册/登录（免费）。
2. 右上角 `+` → **New repository**。名字随意（比如 `openwrt-cloud-build`），可见性选 **Public**，点 **Create repository**。
3. 进入空仓库页面，点 **uploading an existing file**，把本文件夹里**所有内容**拖进去（包括 `README.md`、`files/`、以及 `.github/`）。
   - ⚠️ **`.github` 是隐藏目录**，最容易漏。上传完后在仓库文件列表里确认能看到 `.github/workflows/build-openwrt.yml`。
   - 如果没传上去：点 **Add file → Create new file**，在文件名框里**直接输入** `.github/workflows/build-openwrt.yml`（输入 `/` 会自动建目录），把本地这个文件的全部内容粘进去，Commit。
4. 点 **Commit changes**。完成。

> 会用命令行的替代方案（可选）：
>
> ```bash
> git init && git add -A && git commit -m "init"
> git branch -M main
> git remote add origin https://github.com/<你的用户名>/openwrt-cloud-build.git
> git push -u origin main
> ```

### 第 2 步：触发编译

1. 仓库页面顶部点 **Actions** 标签（第一次可能需要点一下 "I understand my workflows, go ahead and enable them"）。
2. 左侧选 **Build OpenWrt Firmware**。
3. 点右侧 **Run workflow** 按钮，弹窗里确认参数（默认值已配好小米 R3G），点绿色 **Run workflow**。

### 第 3 步：等约 5~15 分钟

刷新页面，看到运行记录变成 **绿色 ✓** 即完成。红色 ✗ 就是失败——点进去看红色步骤的日志。

### 第 4 步：下载固件

点进那次运行记录 → 页面**最底部** **Artifacts** → 下载 `firmware-xxx.zip`，解压就是固件。

### 第 5 步：刷机

产物里 `*squashfs-sysupgrade.bin` 是升级镜像。首次从 Padavan / 小米原厂刷到 OpenWrt 的方式和普通升级不一样（R3G 需要 mtd 写 `kernel1` / `rootfs0`，官方安装说明：<https://openwrt.org/toh/xiaomi/mir3g>）——需要的话把设备当前状态告诉我，单独给你写步骤。

---

## 参数怎么填

| 参数         | 说明          | 默认（小米 R3G）                                                              |
| ---------- | ----------- | ----------------------------------------------------------------------- |
| `release`  | OpenWrt 版本号 | `24.10.7`                                                               |
| `target`   | 平台架构        | `ramips/mt7621`                                                         |
| `profile`  | 设备型号        | `xiaomi_mi-router-3g`                                                   |
| `packages` | 附加软件包（空格分隔） | `luci-ssl luci-i18n-base-zh-cn ca-bundle ca-certificates wget-ssl curl` |

**给别的设备编译时**，这两个值这样查：

1. 打开 <https://firmware-selector.openwrt.org>；
2. 页面上选好你要的设备；
3. 看浏览器地址栏，形如 `?target=ramips%2Fmt7621&id=xiaomi_mi-router-3g`；
   - `target` = `ramips/mt7621`（把 `%2F` 换成 `/`）
   - `profile` = `xiaomi_mi-router-3g`（`id=` 后面的值）

**加软件包**：把包名空格分隔填进 `packages` 参数（页面触发时填，或改 `.github/workflows/build-openwrt.yml` 里的 default 长期生效）。设备默认包含的包系统会自动保留，这里填的是追加项。

**每次修改文件后**，要重新跑一次 Run workflow 才会生效。

---

## 预置自己的文件（files/ 目录）

`files/` 里的内容会被**原样拷进固件**，按目录结构对应：

- `files/etc/uci-defaults/99-xmut-defaults` → 固件里 `/etc/uci-defaults/99-xmut-defaults`，**首次开机自动执行一次**（已内置：主机名、时区、NTP 国内源；WiFi 和 LAN IP 示例都写在注释里，按需取消注释）。
- 想预置 any 文件（脚本、配置、证书）：新建 `files/etc/xxx/yyy` 这样的路径即可。

---

## 常见问题

| 问题                  | 处理                                                               |
| ------------------- | ---------------------------------------------------------------- |
| 编译失败（红 ✗）           | 点进运行记录看哪个步骤红了。最常见：版本号写错（去 firmware-selector 确认）、包名不存在（少打一个字都会失败） |
| 找不到下载按钮             | Artifacts 在运行记录页面**最底部**；公开仓库产物保留 30 天                           |
| 想换个设备编译             | 改 `target` / `profile` 两个参数重新跑，其它不用动                             |
| 想编译改过源码的固件（加驱动、改内核） | 这个仓库走的是官方 ImageBuilder 快速打包，不支持改源码；需要的话另配一套全量源码编译工作流             |
| 免费额度                | Public 仓库的 Actions 标准执行器完全免费；Private 仓库每月 2000 分钟，本流程单次约 10 分钟   |

---

## 目录结构

```
openwrt-cloud-build/
├── .github/workflows/build-openwrt.yml   # 云编译工作流（不用改也能用）
├── files/etc/uci-defaults/99-xmut-defaults  # 首启脚本（预置配置写这里）
└── README.md                             # 本文件
```
