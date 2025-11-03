# Tina V821 SDK 工程

## 项目简介

这是全志科技 V821 芯片的 SDK 开发包，支持多种板级配置（IPC、PERF2、VER1等），集成了 Linux 内核、U-Boot、OpenWrt、RTOS 以及多媒体处理平台（MPP）。

## 核心架构

```
tina-v821-release/
├── brandy/              # 启动引导程序
│   └── brandy-2.0/
│       ├── spl/         # BOOT 启动引导
│       ├── tools/       # 编译工具链
│       └── u-boot-2018/ # U-Boot 源码
│
├── kernel/              # Linux 内核
│   └── linux-5.4-ansc/  # 内核原生源码（5.4版本）
│
├── bsp/                 # BSP 驱动源码
│   ├── configs/         # 内核设备树配置 (dtsi)
│   ├── drivers/         # Linux 设备外设驱动
│   └── include/         # Linux 头文件
│
├── openwrt/             # OpenWrt 用户态系统
│   ├── openwrt/         # OpenWrt 原生源码
│   ├── package/         # 用户态软件包
│   └── target/v821/     # 板级文件系统配置
│
├── rtos/                # RISC-V MCU 的 RTOS
│   ├── board/           # RTOS 板级配置
│   ├── lichee/          # RTOS 源码与组件
│   └── v821_e907/       # 各板级配置（ipc/perf2/ver1等）
│
├── platform/            # 应用与多媒体平台
│   └── allwinner/
│       ├── eyesee-mpp/  # 多媒体处理平台
│       └── multimedia/  # 编解码库
│
├── device/              # 板级配置
│   └── config/chips/V821/configs/
│       ├── default/              # 公共配置
│       ├── ipc/                  # IPC 最小板级配置
│       ├── perf2/                # PERF2 开发板配置
│       ├── perf2_fastboot/       # PERF2 快起配置
│       ├── perf2_fastboot_dual/  # PERF2 双目快起配置
│       └── ver1/                 # VER1 带屏配置
│
├── build/               # SDK 构建系统与打包脚本
├── out/                 # 编译输出目录
│
└── 工具链/
    ├── prebuilt/        # 编译工具与工具链
    ├── hostbuilt/       # 编译使用的工具
    ├── kernelbuilt/     # 内核编译工具链（riscv32）
    └── rootfsbuilt/     # 文件系统编译工具链
```

## 板级配置说明

| 配置名称 | 描述 | 适用场景 |
|---------|------|---------|
| **ipc** | 最小板级配置 | 原型机开发 |
| **perf2** | 标准开发板配置 | 常规开发调试 |
| **perf2_fastboot** | 快速启动配置 | 需要快速启动的场景 |
| **perf2_fastboot_dual** | 双目快速启动配置 | 双摄像头快速启动场景 |
| **ver1** | 带屏幕配置 | 需要显示屏的应用 |

## 快速开始

### 环境要求

- Linux 开发环境（推荐 Ubuntu 18.04/20.04）
- 编译工具链已包含在 SDK 中

### 编译步骤

```bash
# 1. 选择板级配置
source build/envsetup.sh
lunch   # 选择对应的板级配置

# 2. 编译整个 SDK
./build.sh

# 3. 编译产物
# 固件输出在 out/ 目录下
# 例如: out/v821_linux_perf2_uart0_nor.img
```

### 单独编译模块

```bash
# 编译内核
./build.sh kernel

# 编译 U-Boot
./build.sh uboot

# 编译 RTOS
./build.sh rtos

# 编译 OpenWrt 用户态
./build.sh openwrt
```

## 主要开发目录

### Linux 驱动开发
- **驱动源码**: [bsp/drivers/](bsp/drivers/)
- **设备树配置**: [bsp/configs/](bsp/configs/)
- **内核源码**: [kernel/linux-5.4-ansc/](kernel/linux-5.4-ansc/)

### 用户态应用开发
- **应用软件包**: [openwrt/package/](openwrt/package/)
- **文件系统配置**: [openwrt/target/v821/](openwrt/target/v821/)

### 多媒体应用开发
- **MPP 平台**: [platform/allwinner/eyesee-mpp/](platform/allwinner/eyesee-mpp/)
- **编解码库**: [platform/allwinner/multimedia/tina_multimedia/libcedarc_mpp/](platform/allwinner/multimedia/tina_multimedia/libcedarc_mpp/)

### RTOS 开发
- **RTOS 源码**: [rtos/lichee/rtos/](rtos/lichee/rtos/)
- **RTOS 组件**: [rtos/lichee/rtos-components/](rtos/lichee/rtos-components/)
- **板级配置**: [rtos/v821_e907/](rtos/v821_e907/)

## 技术支持

- SDK 版本: V821
- Linux 内核版本: 5.4
- U-Boot 版本: 2018
- 芯片厂商: 全志科技（Allwinner）

## 注意事项

1. 首次编译需要下载依赖包，确保网络连接正常
2. 编译工具链位于 `prebuilt/`、`kernelbuilt/`、`rootfsbuilt/` 目录
3. 切换板级配置后需要重新编译
4. 修改设备树后需要重新编译内核

## 目录结构详细说明

- **brandy**: 启动引导相关，包含 SPL、U-Boot 和编译工具
- **kernel**: Linux 内核原生源码（基于 5.4 版本）
- **bsp**: 板级支持包，包含驱动和设备树配置
- **openwrt**: 用户态系统，基于 OpenWrt
- **rtos**: RISC-V 核心运行的实时操作系统
- **platform**: 应用层平台，包含多媒体处理框架
- **device**: 各板级的配置文件
- **build**: SDK 构建脚本和打包工具
- **out**: 编译输出目录，包含最终固件
