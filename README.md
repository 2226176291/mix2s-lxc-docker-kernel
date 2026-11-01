# LXC-Docker 内核 — Xiaomi Mi MIX 2S (polaris) for LineageOS 22.2

基于 [LineageOS 官方 sdm845 内核](https://github.com/LineageOS/android_kernel_xiaomi_sdm845)（lineage-22.2 分支，与官方构建同一源码提交）开启 **LXC / Docker 支持**的内核构建项目。

> 适用于小米 MIX 2S（polaris）刷入 LineageOS 22.2 的设备。

## 内核特性

在官方 `mi845_defconfig` 基础上追加：

- **Docker / LXC 完整内核支持**：CONFIG_DOCKER、runc cgroup 补丁、qtaguid netfilter 补丁
- **OverlayFS、Kprobes**（容器与调试依赖）
- 关闭 `ANDROID_PARANOID_NETWORK`（容器网络必需）
- **MADV_WIPEONFORK 回移植**（Linux 4.14 特性 → 4.9 内核）：修复新版本 runc 在 4.9 内核上 `arc4random data MADV_WIPEONFORK failed` 导致容器启动 abort 的问题（上游提交 df3735c5b40f / d2cd9ede6e19 / 93949bb21b52，含针对本源码的手工适配）
- 编译器与 LineageOS 官方构建一致（AOSP clang r536225），关闭 LTO 与 CC_WERROR
- ramdisk 取自 LineageOS 官方最新 boot.img，仅替换内核，兼容性最佳

## 使用方法

1. Fork 本仓库，进入 **Actions**，选择工作流「编译内核」，在 `stable/*` 或 `dev/*` 分支上 **Run workflow**（默认 `stable/polaris-rebuild` 分支已就绪）
2. 构建完成后从 **Releases** 下载产物：
   - `boot-lineage-polaris.img` — 直接 `fastboot flash boot boot-lineage-polaris.img`
   - `Anykernel3-polaris.zip` — TWRP 卡刷
3. 建议先用 `fastboot boot boot-lineage-polaris.img` 内存试跑，确认能进系统再正式刷入
4. 回退：刷回你设备对应 LineageOS 构建的官方 boot.img 即可

> ⚠️ 刷机有风险，操作前务必备份好 boot、persist、modemst1/2、fsg 等分区。

### 运行 Docker（Termux 方案）

内核侧就绪后，用户态参考 Termux root-repo 的 docker 包 + 社区启动脚本（cgroup v1 命名空间 + SELinux permissive）。Android `/data` 的 FBE fscrypt v1 与容器命名空间存在结构性限制（fscrypt issue #128），4.9 内核下建议 tmpfs 数据区。

## 分支说明

| 分支 | 内容 |
| :--- | :--- |
| `lineage-15.1` ~ `lineage-22.2` | 各版本 LineageOS 内核源码 |
| `stable/polaris-rebuild` | polaris 构建工作流（push 自动触发，产物自动发布 Release） |
| `wipeonfork-patches` | MADV_WIPEONFORK 回移植补丁（3 上游提交 + polaris 适配 fixup） |
| `backups/*`、`staging/*` | 上游历史分支，未使用 |

## 致谢

- [msxy2318](https://github.com/msxy2318) — 上游内核源仓库与构建流程基础
- [tomxi1997](https://github.com/tomxi1997) — [lxc-docker-support-for-android](https://github.com/tomxi1997/lxc-docker-support-for-android)
- [awkoo](https://github.com/awkoo) — 工作流基础
- [osm0sis](https://github.com/osm0sis) — [AnyKernel3](https://github.com/osm0sis/AnyKernel3)
- [LineageOS](https://lineageos.org) — 官方 sdm845 内核维护

## 免责声明

本项目按"现状"提供，刷写第三方内核存在变砖与数据丢失风险，请自行评估并做好备份。使用本项目产生的任何后果由使用者自行承担。
