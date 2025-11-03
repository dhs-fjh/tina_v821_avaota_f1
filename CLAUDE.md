# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是全志科技 V821 芯片的 Tina SDK，是一个完整的嵌入式 Linux 开发环境，包含：
- **异构多核架构**: RISC-V (C906/E907) + ARM 架构支持
- **Linux 内核**: 5.4-ansc 版本
- **RTOS**: FreeRTOS 运行在 E907 小核上
- **用户态系统**: 基于 OpenWrt
- **多媒体框架**: eyesee-mpp 和 libcedarc

## 构建系统架构

### 初始化与配置流程

```bash
# 1. 加载环境变量
source build/envsetup.sh

# 2. 选择板级配置（会设置 LICHEE_* 环境变量）
lunch
# 可用板级: perf2, perf2_fastboot, perf2_fastboot_dual, ver1, ipc, avaota_f1 等

# 3. 编译（调用 build/mkcommon.sh）
./build.sh [target]
```

### 关键环境变量

构建系统依赖以下环境变量（由 `lunch` 命令从 BoardConfig.mk 中读取）：
- `LICHEE_CHIP`: 芯片型号（sun300iw1p1）
- `LICHEE_ARCH`: CPU 架构（riscv32）
- `LICHEE_KERN_VER`: 内核版本（5.4-ansc）
- `LICHEE_BRANDY_VER`: 启动引导版本（2.0）
- `LICHEE_RTOS_PROJECT_NAME`: RTOS 项目名称（如 v821_e907_perf2）
- `LICHEE_ROOTFS`: 根文件系统类型

### 板级配置文件

板级配置存放在 `device/config/chips/v821/configs/<board>/`:
- `BoardConfig.mk`: 核心配置变量
- `sys_config.fex`: 硬件配置（GPIO、时钟、电源管理等）
- `sys_partition*.fex`: 分区表配置

## 常用构建命令

### 完整编译
```bash
./build.sh              # 编译所有组件（bootloader + kernel + rootfs + rtos）
```

### 分模块编译
```bash
./build.sh bootloader   # 编译 U-Boot (brandy/brandy-2.0/u-boot-2018/)
./build.sh kernel       # 编译 Linux 内核
./build.sh modules      # 编译内核模块
./build.sh dtb          # 编译设备树
./build.sh rtos         # 编译 RTOS (rtos/v821_e907/)
./build.sh rootfs       # 编译根文件系统
./build.sh dsp          # 编译 DSP 固件（如果有）
```

### 打包固件
```bash
./build.sh pack         # 打包生成最终固件到 out/ 目录
./build.sh pack_debug   # 打包调试版本
```

### 配置相关
```bash
./build.sh menuconfig           # 配置 Linux 内核
./build.sh saveconfig           # 保存内核配置
./build.sh uboot_menuconfig     # 配置 U-Boot
./build.sh openwrt_menuconfig   # 配置 OpenWrt 软件包
```

### 清理
```bash
./build.sh clean        # 清理编译产物
```

## 代码架构说明

### 三层启动架构

1. **BROM (Boot ROM)**: 芯片固化的启动代码
2. **SPL/Boot0** (`brandy/brandy-2.0/spl/`): 第一阶段引导加载器
   - 初始化 DDR、时钟、PMIC
   - 加载 U-Boot
3. **U-Boot** (`brandy/brandy-2.0/u-boot-2018/`): 第二阶段引导加载器
   - 加载 Linux 内核和 RTOS 镜像
   - 提供启动菜单和环境变量管理

### Linux 内核与驱动

- **内核源码**: `kernel/linux-5.4-ansc/`
- **BSP 驱动**: `bsp/drivers/` - 全志定制的外设驱动
  - `bsp/drivers/video/`: 显示驱动（DE、LCD、HDMI）
  - `bsp/drivers/vin/`: 视频输入驱动（摄像头）
  - `bsp/drivers/ve/`: 硬件视频编解码驱动
  - `bsp/drivers/g2d/`: 2D 图形加速驱动
- **设备树**: `bsp/configs/` - 板级设备树配置 (.dtsi)
- **内核配置**: `bsp/configs/<board>/bsp_defconfig`

**注意**: BSP 驱动通过符号链接注入到内核源码树，实际编译时会链接到 `kernel/linux-5.4-ansc/drivers/` 下

### RTOS 架构

RTOS 运行在 RISC-V E907 小核上，负责实时任务和快速启动：

- **源码**: `rtos/lichee/rtos/` - FreeRTOS 核心
- **组件**: `rtos/lichee/rtos-components/` - AW 提供的组件（AWPlayer、AudioSystem）
- **HAL**: `rtos/lichee/rtos-hal/` - 硬件抽象层
- **板级配置**: `rtos/v821_e907/<board>/` - 板级配置和启动脚本
- **编译产物**: `out/<platform>/rtos/` - 生成 rtos.fex 固件

**RTOS 与 Linux 通信**: 通过 msgbox、RPMsg 或共享内存机制

### OpenWrt 用户态系统

- **OpenWrt 源码**: `openwrt/openwrt/` - OpenWrt 主框架
- **软件包**: `openwrt/package/` - 用户态应用和库
  - `allwinner/`: 全志定制软件包（MPP、tina_multimedia）
  - `libs/`: 第三方库
  - `utils/`: 工具软件
- **板级配置**: `openwrt/target/v821/v821-<board>/` - 文件系统配置、启动脚本
- **defconfig**: `openwrt/target/v821/v821-<board>/defconfig` - 软件包选择

### 多媒体平台 (MPP)

- **eyesee-mpp**: `platform/allwinner/eyesee-mpp/`
  - 中间件层，提供高级多媒体 API
  - 支持录像、拍照、AI 推理等场景
  - Sample 代码位于 `eyesee-mpp/middleware/sun8iw21/sample/`

- **tina_multimedia**: `platform/allwinner/multimedia/tina_multimedia/`
  - `libcedarc_mpp/`: 编解码库（H.264/H.265/JPEG）
  - `libcedarx/`: 多媒体框架

**关键概念**:
- **VIPP (Video Input Post Processor)**: 视频输入通道
- **VENC (Video Encoder)**: 视频编码器
- **ISE/ISP**: 图像信号处理

### 工具链

- **内核编译**: `prebuilt/gcc-linaro-*/` 或 `brandy/brandy-2.0/tools/toolchain/`
- **用户态编译**: OpenWrt 自带工具链（编译时自动下载/构建）
- **RTOS 编译**: `rtos/tools/` 下的 RISC-V 工具链

## 开发工作流

### 修改 Linux 驱动

1. 驱动代码在 `bsp/drivers/<subsystem>/`
2. 修改完成后运行 `./build.sh kernel` 或 `./build.sh modules`
3. 编译产物: `out/<platform>/kernel/`
4. 可以单独替换 `.ko` 模块到设备进行测试

### 修改设备树

1. 编辑 `bsp/configs/<board>/*.dtsi`
2. 运行 `./build.sh dtb`
3. 设备树会编译到内核镜像中

### 添加 OpenWrt 软件包

1. 在 `openwrt/package/` 下创建软件包目录
2. 编写 `Makefile`（遵循 OpenWrt 软件包规范）
3. 运行 `./build.sh openwrt_menuconfig` 选中软件包
4. 运行 `./build.sh rootfs` 重新编译根文件系统

### 修改 RTOS

1. RTOS 代码在 `rtos/lichee/rtos/` 和 `rtos/v821_e907/<board>/`
2. 运行 `./build.sh rtos` 编译
3. 需要重新打包固件: `./build.sh pack`

### 调试技巧

- **串口调试**: 默认串口在 UART0，波特率 115200
- **内核日志**: `dmesg` 或 `/proc/kmsg`
- **RTOS 日志**: 通过 UART 或 `/dev/rpbuf_ctrl*` 查看
- **查看分区**: `cat /proc/partitions`
- **启动时间分析**: `cat /sys/kernel/debug/boottime`

## 快速启动 (fastboot) 方案

快速启动配置（perf2_fastboot、perf2_fastboot_dual）的优化点：

1. **内核精简**: 移除不必要的驱动和服务
2. **RTOS 主导启动**: RTOS 先启动摄像头和显示，Linux 后台加载
3. **并行启动**: 使用 systemd 或 procd 并行启动服务
4. **延迟初始化**: 非关键服务延迟到启动后初始化

相关配置在 `device/config/chips/v821/configs/perf2_fastboot/` 和对应的 RTOS 配置中

## 输出产物说明

编译完成后，固件位于 `out/<platform>/`:

```
out/
├── v821_linux_<board>_<uart>_<flash>.img  # 最终打包固件
├── kernel/
│   ├── vmlinux              # 内核 ELF 文件
│   ├── zImage               # 压缩内核镜像
│   └── sun300iw1p1.dtb      # 设备树二进制
├── rtos/
│   └── rtos.fex             # RTOS 固件
├── uboot/
│   └── u-boot.fex           # U-Boot 固件
└── rootfs/
    └── rootfs.ext4          # 根文件系统镜像
```

## 用户偏好设置

- **语言**: 中文优先
- **代码风格**: 遵循内核代码风格（Linux Kernel Coding Style）
- **注释**: 中英文混合，关键部分使用中文注释

## 注意事项

1. **不要修改**: `kernel/linux-5.4-ansc/` 下的原生内核代码，驱动修改应在 `bsp/drivers/` 中进行
2. **工具链路径**: 避免硬编码工具链路径，使用环境变量
3. **多板级支持**: 代码修改应考虑多板级兼容性
4. **RTOS 与 Linux 同步**: 涉及硬件资源分配时，需要同时修改 RTOS 和 Linux 配置
5. **分区表**: 修改 `sys_partition.fex` 后需要重新烧录完整固件
