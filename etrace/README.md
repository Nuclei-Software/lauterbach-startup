# ETrace 介绍

ETrace 功能可以将 CPU 的执行信息转换成对应的 ETrace 包，后续通过解析 ETrace 包，反解析获取 CPU 的执行信息。

ETrace 包含两种使用方式：

* **on-chip**：trace 数据走芯片内部总线，写入存储器（如片内 SRAM、DDR 等）。
* **off-chip**：trace 数据通过芯片 PAD 输出，保存在外部的 trace 工具中。

## ETrace 配置流程

ETrace 配置流程如下：

* 配置 encoder
* 配置 encoder 解析器
* 配置 funnel
* 如果使用 on-chip trace，配置 axi sink
* 如果使用 off-chip trace，配置 pib

## 配置 encoder

提供了 **nuclei_etrace_encoder_config.cmm** 脚本，用于配置 encoder 组件。

外部使用 `do` 命令调用该脚本，然后传入如下参数：

| 参数 | 说明 |
|---|---|
| 第一个参数 | encoder 地址参数 |
| 第二个参数 | 是否开启 timestamp 记录 |
| 第三个参数 | datatrace 模式 |

第一个参数的取值：

* 如果是单个 encoder：`SB:encoder地址`
* 如果是多个 encoder：`SB:encoder0地址 SB:encoder1地址 SB:encoder(n)地址`

第二个参数的取值：

* `ON`：开启
* `OFF`：关闭

第三个参数的取值：

* `ON`：开启 data trace 记录
* `OFF`：关闭 data trace 记录

例如：

一个 encoder

```
do nuclei_etrace_encoder_config.cmm "SB:0x10064000" "ON" "OFF"
```

三个 encoder

```
do nuclei_etrace_encoder_config.cmm "SB:0x10064000 SB:0x10065000 SB:0x10066000" "ON" "OFF"
```

## 配置 encoder 解析器

提供了 **nuclei_etrace_analyze_config.cmm** 脚本，用于配置 encoder 解析器。

外部使用 `do` 命令调用该脚本，然后传入如下参数：

| 参数 | 说明 |
|---|---|
| 第一个参数 | encoder 的基地址 |

脚本内部会读取 encoder 的硬件信息，然后自动调用 `RVETRACE` 的命令进行设置。

例如：

```
do nuclei_etrace_analyze_config.cmm "0x10064000"
```

## 配置 funnel

提供了 **nuclei_etrace_funnel_config.cmm** 脚本，用于配置 funnel。

funnel 组件最多可以接收 16 个 encoder 的 trace 数据，然后合并成一组 trace 数据输出。

外部使用 `do` 命令调用该脚本，然后传入如下参数：

| 参数 | 说明 |
|---|---|
| 第一个参数 | funnel 的基地址 |
| 第二个参数 | 要追踪的 trace encoder 信息 |

第二个参数的取值：

* 如果只有一个 trace encoder，写 `"RVETRACE encoder编号"`，其中 RVETRACE 后面的数字表示 encoder 对应的 funnel 硬件映射编号。
* 如果有多个 trace encoder，写 `"RVETRACE.0 encoder0编号 RVETRACE.1 encoder1编号"`，其中 `RVETRACE.` 后的数字表示 encoder 的编号（从 0 开始），后面的数字表示 encoder 硬件和 funnel 的硬件映射编号。

例如：

funnel 连接单个 encoder，并且 encoder 连接在 funnel 的端口 0

```
do nuclei_etrace_funnel_config.cmm "0x16040000" "RVETRACE 0"
```

funnel 连接两个 encoder，encoder0 连接在 funnel 的端口 1，encoder1 连接在 funnel 的端口 2

```
do nuclei_etrace_funnel_config.cmm "0x16040000" "RVETRACE.0 1 RVETRACE.1 2"
```

## 配置 AXI SINK

提供了 **nuclei_etrace_axi_sink_config.cmm** 脚本，用于配置 axi sink 组件，实现 on-chip trace。

外部使用 `do` 命令调用该脚本，然后传入如下参数：

| 参数 | 说明 |
|---|---|
| 第一个参数 | axi sink 的基地址 |
| 第二个参数 | 保存 trace 数据的内存基地址 |
| 第三个参数 | 保存 trace 数据的内存大小，以 byte 为单位 |
| 第四个参数 | trace 数据模式 |

第四个参数的取值：

* `FIFO`：FIFO 模式，如果 trace 数据溢出，就覆盖最老的数据
* `STACK`：STACK 模式，如果 trace 数据溢出，就不更新 trace 数据

例如：

```
do nuclei_etrace_axi_sink_config.cmm "0x16041000" "0xa0100000" "0x30000" "fifo"
```

> 该脚本依赖 TRACE32 安装目录下的 `~~\demo\riscv\etc\smem-trace\smem-trace-utility.cmm`。

## 配置 PIB

提供了 **nuclei_etrace_pib_config.cmm** 脚本，用于配置 pib 组件，实现 off-chip trace。

外部使用 `do` 命令调用该脚本，然后传入如下参数：

| 参数 | 说明 |
|---|---|
| 第一个参数 | pib 的基地址 |
| 第二个参数 | pib 的数据位宽 |
| 第三个参数 | trace 数据模式 |

第二个参数的取值：`4` / `8` / `16`。

第三个参数的取值：

* `FIFO`：FIFO 模式，如果 trace 数据溢出，就覆盖最老的数据
* `STACK`：STACK 模式，如果 trace 数据溢出，就不更新 trace 数据

例如：

```
do nuclei_etrace_pib_config.cmm "0x16042000" "4" "stack"
```

## ETrace Demo

在 `onchip/` 和 `offchip(pib)/` 目录下提供了几个配置脚本，可以参考使用：

| 脚本 | 说明 |
|---|---|
| `onchip/rv32_axi_sink_btrace.cmm` | on-chip，单核，branch trace |
| `onchip/rv32_axi_sink_dtrace.cmm` | on-chip，单核，data trace |
| `onchip/rv32_smp2_axi_sink_btrace.cmm` | on-chip，双核 SMP，branch trace |
| `offchip(pib)/rv32_pib_btrace.cmm` | off-chip，单核，branch trace |
| `offchip(pib)/rv32_pib_dtrace.cmm` | off-chip，单核，data trace |

需要注意：

* 脚本中的硬件基地址需要根据实际的硬件情况修改。
* 脚本中的以下参数需要进行修改：
  * elf 路径：对应参数 `&elf_file_path`
  * lauterbach-startup 仓库路径：对应参数 `&nuclei_home`
