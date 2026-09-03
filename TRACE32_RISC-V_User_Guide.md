# Nuclei RISC-V 处理器 TRACE32 调试指南

- [1. 概述](#1-概述)
- [2. 基本调试使用](#2-基本调试使用)
  - [2.1 TRACE32 基本设置](#21-trace32-基本设置)
  - [2.2 TRACE32 DM 配置](#22-trace32-dm-配置)
    - [2.2.1 隐式配置 JTAG-DTM](#221-隐式配置-jtag-dtm)
    - [2.2.2 显式配置 JTAG-DTM](#222-显式配置-jtag-dtm)
  - [2.3 单核调试](#23-单核调试)
  - [2.4 多核 SMP 调试](#24-多核-smp-调试)
  - [2.5 多核 AMP 调试](#25-多核-amp-调试)
  - [2.6 自定义指令与 CSR 解析](#26-自定义指令与-csr-解析)
- [3. 基本调试命令](#3-基本调试命令)
  - [3.1 寄存器读写](#31-寄存器读写)
  - [3.2 Memory 读写](#32-memory-读写)
  - [3.3 CSR 读写](#33-csr-读写)
  - [3.4 单步执行](#34-单步执行)
  - [3.5 断点设置](#35-断点设置)
  - [3.6 Watchpoint (数据观察点)](#36-watchpoint-数据观察点)
  - [3.7 MMU 页表查询](#37-mmu-页表查询)
  - [3.8 程序加载与符号下载](#38-程序加载与符号下载)
- [4. 操作系统调试](#4-操作系统调试)
- [5. Flash 烧录](#5-flash-烧录)
- [6. Cross Trigger 调试功能](#6-cross-trigger-调试功能)
- [7. Authentic 鉴权调试功能](#7-authentic-鉴权调试功能)
- [8. Trace 功能](#8-trace-功能)
- [9. 相关资源与参考文档](#9-相关资源与参考文档)

---

## 1. 概述

Lauterbach TRACE32 调试器全面支持 RISC-V 架构，基于 RISC-V Debug Specification 提供完整的调试与跟踪解决方案。核心功能包括：

- **JTAG 调试**：通过 JTAG-DTM 访问 Debug Module (DM)，支持单核、多核 SMP/AMP 调试。
- **OS 相关调试**：原生支持裸机、FreeRTOS（单核/多核）以及 Linux 内核与任务感知调试。
- **E-Trace 数据跟踪**：支持数据读写跟踪、地址/值匹配及跳转优化，适用于系统性能与流程分析。

> **适用范围**：本文面向 **Nuclei RISC-V 处理器** 的 TRACE32 调试。文中涉及的验证与测试均在 **Nuclei EvalSoC** 评估平台（FPGA bitstream）上完成；EvalSoC 内嵌的 CPU 可为 Nuclei RISC-V 200 - 1000 系列的全系 RV32/RV64 处理器，不同测试场景会使用对应的 FPGA bitstream，CMM 脚本文件名中的 `riscv32`/`riscv64` 与目标处理器的指令集架构一一对应。如需要相关 bitstream 文件，请联系芯来技术支持（AE）获取。

---

## 2. 基本调试使用

### 2.1 TRACE32 基本设置

初次使用时，可通过 TRACE32 启动配置管理器（T32Start）建立调试环境配置：

1. **Add Configuration**：打开 TRACE32 软件，在 Configuration Tree 区域右键点击，选择 `Add -> Configuration`，按 `F2` 重命名配置（例如 `nuclei_config`）。
   ![Add Configuration](./pic/config1.jpg)

2. **Add Podbus Device Chain**：
   ![Add Device Chain](./pic/config2.jpg)

3. **Add Power Device**：根据当前使用的 TRACE32 硬件探头类型选择对应设备。
   ![Add Power Device](./pic/config3.jpg)

4. **Choose Connection Type**：根据主机与调试器的连接方式选择（如使用 USB 数据线，选择 `USB`）。
   ![Choose Connection Type](./pic/config4.jpg)

5. **Add PowerView Instance**：
   ![Add PowerView Instance](./pic/config5.jpg)
   右键点击实例，将 Target 设置为 `RISC-V`：
   ![Add PowerView Instance-riscv](./pic/config6.jpg)

6. **Start 启动**：点击右侧 `Save` 保存配置，再点击 `Start` 即可进入 TRACE32 PowerView 主界面（未连接目标板 JTAG 时右下角显示 PowerDown 状态为正常现象）。
   ![Main Interface](./pic/main_interface.jpg)

7. **Run Script**：在主界面选择并执行对应的 CMM 启动脚本。
   ![Run Script](./pic/run_script.jpg)

> 详细配置说明可参考 TRACE32 安装目录下的 `pdf/app_t32start.pdf`。

---

### 2.2 TRACE32 DM 配置

调试器需通过 Debug Module Interface (DMI) 访问 DM 寄存器。外部 JTAG 接口访问 RISC-V 调试模块（DM）简图如下：

![DM Config](./pic/dm_config.jpg)

#### 2.2.1 隐式配置 JTAG-DTM

若目标系统中仅包含单个 JTAG-DTM 与单个 DM，TRACE32 能够自动识别 JTAG 链参数，无需额外繁琐配置。若使用 2-pin cJTAG 接口，需额外启用 cJTAG 对应选项。

典型 RV64 单核 CMM 配置脚本（若为 cJTAG 需取消注释对应两行）：

```cmm
RESet                           ; Reset debugger configuration

; SYStem.CONFIG.DebugPortType CJTAG
; SYStem.CONFIG.CJTAGFLAGS NOTCA NOKEEPER NOHARDESC CMDRTI SKIPDUMMYSP SKIPMASK 0xF CPKTSEL 1

SYStem.JtagClock 10MHz          ; Set JTAG clock frequency
SYStem.Option.RESetmode NDMRST  ; Select the reset method
SYStem.CPU RV64                 ; Select the SoC/CPU/core
SYStem.Up                       ; Resets SoC and enters debug mode

List
```

#### 2.2.2 显式配置 JTAG-DTM

对于复杂的 JTAG 菊花链（Daisy Chain）系统，需使用 `RVDMIAP` 与 `COREDEBUG` 命令手动声明 JTAG 链参数：

```cmm
RESet

SYStem.JtagClock 10MHz
SYStem.Option.RESetmode NDMRST
SYStem.CPU RV64

SYStem.CONFIG RVDMIAP1.DebugSource DebugPort    ; External debug port
SYStem.CONFIG RVDMIAP1.IRLENGTH 5.              ; Configure JTAG daisy chain
SYStem.CONFIG RVDMIAP1.IRPRE 0.
SYStem.CONFIG RVDMIAP1.IRPOST 0.
SYStem.CONFIG RVDMIAP1.DRPRE 0.
SYStem.CONFIG RVDMIAP1.DRPOST 0.
SYStem.CONFIG COREDEBUG.Base DMI:0x0            ; RISC-V DM Base

SYStem.Up                                       ; Resets SoC and enters debug mode
```

使用 `SYStem.DETECT.DaisyChain` 命令可自动探测当前扫描链分布。下图是在 EvalSoC 平台（双核 AMP 配置的 bitstream）上，单 JTAG 端口、两 TAP 菊花链相连时的探测情况：

![jtag_detect](./pic/jtag_detect.jpg)

完整的AMP双核连接脚本可参考本仓库中（[`nuclei_riscv32_jtagchain_dmi.cmm`](nuclei_riscv32_jtagchain_dmi.cmm)），此脚本因为bitstream原因，没有使用system.up， 而是用system.attach，system.up会复位bitstream，system.attach不会复位bitstream，使用时请根据具体情况选择。

---

### 2.3 单核调试

使用本仓库提供的单核 CMM 脚本即可连接目标 CPU。例如使用 [`nuclei_riscv64.cmm`](nuclei_riscv64.cmm) 连接 RV64 单核 SoC：

![rv64_1core_connect](./pic/1core_connect.jpg)

### 2.4 多核 SMP 调试

使用本仓库对应的 SMP 脚本（如 [`nuclei_riscv64_smp2.cmm`](nuclei_riscv64_smp2.cmm) 或 `nuclei_riscv32_smp<x>.cmm`）连接多核 SoC。连接成功后，主界面底部状态栏会显示当前逻辑核心编号（如 `0` 代表 core 0），右键可快捷切换核心上下文；单核连接时无此指示：

![rv64_smp2_connect](./pic/smp2_connect.jpg)

### 2.5 多核 AMP 调试

对于非对称多核系统，可使用链式 DMI 脚本（例如 [`nuclei_riscv32_jtagchain_dmi.cmm`](nuclei_riscv32_jtagchain_dmi.cmm)）连接各独立核心进行调试：

![rv32_amp_connect](./pic/amp_connect.jpg)

### 2.6 自定义指令与 CSR 解析

若应用程序使用了芯来（Nuclei）自定义扩展指令或自定义 CSR，需加载对应的解析插件，以便 TRACE32 将机器码与地址解析为可读的助记符和寄存器名。

- **加载自定义指令解析库**（使用本仓库提供的 [`nuclei_custom_inst_parser.dll`](nuclei_custom_inst_parser.dll)）：
  ```cmm
  APU.LOAD nuclei_custom_inst_parser.dll
  ```

- **加载自定义 CSR 解析文件**（使用本仓库提供的 [`nuclei_custom_csr_parser.per`](nuclei_custom_csr_parser.per)）：
  ```cmm
  PER nuclei_custom_csr_parser.per
  ```

---

## 3. 基本调试命令

除了使用工具栏上的图形按钮（Step、Step.Over、Go、Break、Memory、Register、Watch 等）外，所有操作均可在底部命令行直接输入。TRACE32 命令不区分大小写。

![main_debug](./pic/main_debug.jpg)

### 3.1 寄存器读写

| 操作 | 命令示例 | 说明 |
| :--- | :--- | :--- |
| 打开寄存器窗口 | `Register` | 显示当前核心的所有通用寄存器 |
| 读取/打印寄存器 | `PRINT Register(PC)` | 打印指定寄存器值（如 PC） |
| 修改寄存器值 | `Register.Set PC 0xfdfc9000` | 将 PC 修改为指定地址 |

> **提示**：读取寄存器使用 `PRINT Register(<reg>)` 或快捷函数 `r(<reg>)`，TRACE32 没有 `r.get` 命令。

### 3.2 Memory 读写

TRACE32 使用 `Data.Dump` 和 `Data.Set` 访问内存，配合访问类（Access Class）前缀支持不同的访问模式：

| 操作 | 命令示例 | 说明 |
| :--- | :--- | :--- |
| 内存查看 | `Data.Dump 0x80000000` | 查看指定逻辑/虚拟地址内容 |
| 物理地址查看 | `Data.Dump A:0x80000000` | 绕过 MMU/缓存，直接通过物理地址访问 |
| 运行态实时查看 | `Data.Dump E:0x80000000` | CPU 运行（不停机）状态下访问内存内容 |
| 内存写入 (Byte/Long) | `Data.Set D:0x80000000 %Long 0x12345678` | 在指定地址写入 32-bit 数据 |

### 3.3 CSR 读写

访问 RISC-V 控制与状态寄存器（CSR）需在地址前添加 `CSR:` 前缀，当前需使用十六进制地址访问：

```cmm
; RV64 读取 CSR (例如 mstatus 地址为 0x300)
PRINT Data.Quad(CSR:0x300)

; RV32 读取 CSR
PRINT Data.Long(CSR:0x300)

; RV64 向 CSR 0x300 写入 val 值（示例：mstatus）
Data.Set CSR:0x300 %LE %Quad val

; 批量查看一段连续 CSR 内容
Data.Dump CSR:0x300
```

### 3.4 单步执行

| 操作 | 命令 | 说明 |
| :--- | :--- | :--- |
| 源码/汇编单步 | `Step` | HLL 模式下单步一行 C 代码；汇编模式下执行一条指令（类似 GDB `s`/`si`） |
| 纯汇编单步 | `Step.asm` | 强制执行一条底层汇编指令（类似 GDB `si`） |
| 单步跳过 | `Step.Over` | 步过函数调用（类似 GDB `n`） |
| 执行至返回 | `Step.Out` | 运行直至当前函数返回（类似 GDB `finish`） |

### 3.5 断点设置

```cmm
; 设置软件断点
Break.Set 0x0C008000

; 设置芯片硬件断点 (Onchip Breakpoint)
Break.Set 0x0C008000 /Onchip

; 删除指定地址断点
Break.Delete 0x0C008000

; 临时禁用断点
Break.Disable 0x0C008000

; 删除所有断点
Break.Delete /All
```

![break_set](./pic/break_set.jpg)

### 3.6 Watchpoint (数据观察点)

基于 `Break.Set` 命令配合访问属性与条件触发：

```cmm
; 1. 写访问触发硬件观察点
Break.Set 0xFD76B9E8 /Write /Onchip

; 2. 变量符号观察点（需先加载 elf 符号表）
Break.Set global_counter /Write

; 3. 条件断点：当执行至 0x2228 且 R7 寄存器值大于 5 时中断
Break.Set 0x2228 /Program /CONDition Register(R7)>5

; 4. 条件断点：当执行至 0x2228 且内存 0x1234 处的值为 0x55 时中断
Break.Set 0x2228 /Program /CONDition Data.Word(D:0x1234)==0x55
```

![watch_point](./pic/watch_point.jpg)

### 3.7 MMU 页表查询

对于运行带 MMU 系统的平台（如 Linux），可直接查询虚拟地址到物理地址的映射：

```cmm
MMU.List.PageTable
```

### 3.8 程序加载与符号下载

```cmm
; 1. 下载二进制文件到指定内存地址
Data.LOAD.Binary burn_test.bin 0x20000000

; 2. 下载 ELF 文件（代码 + 符号表）
Data.LOAD.Elf application.elf

; 3. 仅加载 ELF 符号表（不下载代码，适用于 Flash/ROM 已固化程序）
Data.LOAD.Elf application.elf /NoCODE
```

---

## 4. 操作系统调试

以 Linux Kernel 调试为例：

1. **CMM 连接脚本中启用 MMU 空间转换**：
   ```cmm
   TRANSlation.CONFIG.MMUSPACES ON
   TRANSlation.TableWalk ON
   TRANSlation.ON
   ```

2. **内核编译选项要求**：
   - 必须开启调试信息：`CONFIG_DEBUG_INFO=y`
   - 建议开启符号表支持：`CONFIG_KALLSYMS=y`

3. **加载内核符号文件**：
   ```cmm
   Data.LOAD.Elf vmlinux /NoCODE
   ```

4. **加载 Linux Awareness 插件**：
   ```cmm
   TASK.CONFIG ~~/demo/riscv/kernel/linux/awareness/linux.t32
   ```

5. **查看 Linux 任务与系统状态**：
   通过顶部菜单的 Linux 标签页，可直接查看 Processes、Modules、Mounts、cgroups 等系统运行时状态。
   ![debug_linux](./pic/debug_linux.jpg)

> 详细 RTOS 调试指南（如 FreeRTOS、RT-Thread 等）请参阅 TRACE32 官方安装目录下的 `pdf/rtos_<os>.pdf`。

---

## 5. Flash 烧录

本仓库提供了针对芯来评估平台（EvalSoC）的 Flash 烧录 CMM 脚本及算法目录（[`nuclei_riscv64_flash.cmm`](nuclei_riscv64_flash.cmm) 用于 RV64，[`nuclei_riscv32_flash.cmm`](nuclei_riscv32_flash.cmm) 用于 RV32，配套烧录算法在 [`flash/`](flash/) 目录下）。

烧录流程简述：
1. 准备待烧录的二进制固件（示例中使用的测试文件为 [`burn_test.bin`](burn_test.bin)）。
2. 在 TRACE32 中运行对应架构的 flash 脚本（例如 `nuclei_riscv64_flash.cmm`），脚本会加载 [`flash/`](flash/) 目录下的烧录算法模板、配置 Flash 地址空间、擦除并写入目标固件，最后做回读比对校验。

下图为将 `burn_test.bin` 重命名为 `freeloader.bin` 后进行烧录的界面示例：

![flash_burn](./pic/flash_burn.jpg)

---

## 6. Cross Trigger 调试功能

Cross Trigger（交叉触发）用于将某个 CPU 核心/硬件线程的调试事件（如触发断点、进入 Debug 模式）同步广播给其他核心，实现多核之间的联动暂停（Halt）与恢复运行（Resume）。

TRACE32 原生支持 RISC-V 交叉触发特性，只需在 JTAG-DTM 配置阶段完整声明各 Core 的调试端口映射关系。

交叉触发验证基于 EvalSoC 平台（双核配置的 bitstream）进行：
1. 运行本仓库中的 [`nuclei_riscv32_jtagchain_dmi.cmm`](nuclei_riscv32_jtagchain_dmi.cmm) 连接芯片各核心；
2. 为不同核心分别加载测试程序：
   - Core 0 在 ILM 上运行 HelloWorld：
     ![cross_trigger1](./pic/cross_trigger1.jpg)
   - Core 1 在 SRAM 上运行 HelloWorld：
     ![cross_trigger2](./pic/cross_trigger2.jpg)
3. 在任意核心点击运行或暂停，其他核心将同步运行或暂停，实现多核硬件联动。

---

## 7. Authentic 鉴权调试功能

安全调试机制通过 Debug Module 的 `authdata` 寄存器验证访问密码。Authentic 模块与 Nuclei CPU 的交互结构如下图所示：

![authentic](./pic/authentic.jpg)

若鉴权未通过，Debug Module 将锁定调试接口，拒绝读取内部核心状态。在 CMM 脚本中完成鉴权配置的示例如下（假设鉴权密码为 `0x3c3c3c3c`，通过逻辑 Core 0 向地址 `0xC0` 的 authdata 寄存器写入）：

```cmm
Core.select 0.
Data.Set EDMI:0xC0 %LE %Long 0x3c3c3c3c
```

> **提示**：用于验证 Authentic 鉴权功能的 EvalSoC FPGA bitstream（已预置鉴权逻辑）芯来已现成提供。由于文件体积较大，默认未放置在 Git 仓库中；如需进行安全鉴权评估与测试，可联系芯来技术支持（AE）获取。

### 鉴权流程演示

1. 使用 [`nuclei_riscv32_jtagchain_dmi.cmm`](nuclei_riscv32_jtagchain_dmi.cmm) 连接 CPU；
2. 读取 DM 的 `dmstatus` 寄存器状态：
   ![authentic1](./pic/authentic1.jpg)
   若 `dmstatus` 为 `0x4003A3`，其中 bit 7 (`authenticated`) 为 1，说明当前处于已认证状态。
3. 写入密码测试：
   - 写入正确密码（`0x3c3c3c3c`），调试器正常通信：
     ```cmm
     Core.select 0.
     Data.Set EDMI:0xC0 %LE %Long 0x3c3c3c3c
     Core.select 1.
     Data.Set EDMI2:0xC0 %LE %Long 0x3c3c3c3c
     ```
   - 若写入错误密码（如 `0x3c3c3c3d`），调试器将报错并拒绝连接：
     ![authentic2](./pic/authentic2.jpg)

---

## 8. Trace 功能

*(待补充完善 / TBD)*

---

## 9. 相关资源与参考文档

- **开源脚本仓库**：[Nuclei Lauterbach Startup Scripts (GitHub)](https://github.com/Nuclei-Software/lauterbach-startup)
- **TRACE32 官方手册参考**：
  - `pdf/debugger_riscv.pdf`：*TRACE32 RISC-V Debugger User's Guide*
  - `pdf/app_t32start.pdf`：*TRACE32 Start Configuration Tool*
  - `pdf/general_ref_b.pdf`：*TRACE32 Breakpoint & Watchpoint Commands*
  - `pdf/rtos_linux.pdf`：*TRACE32 Linux Awareness Guide*

