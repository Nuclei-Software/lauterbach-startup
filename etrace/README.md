# Etrace介绍 

Etrace功能，可以将CPU的执行信息，转换成对应的etrace包，后续通过解析etrace包，从而反解析获取CPU的执行信息。

Etrace包含两种使用方式：

* onchip: 针对于on-chip调试，也就是trace数据是芯片内部总线，写入到存储中。如片内SRAM， DDR等。
* off-chip：针对于off-chip调试，也就是trace数据是通过芯片的PAD，保存在外部的trace工具中

# ETrace配置流程

ETRACE配置流程如下所示：
* 配置encoder
* 配置encoder解析器
* 配置funnel
* 如果使用on-chip trace，配置axi sink
* 如果使用off-chip trace，配置pib sink

## 配置encoder

提供了 **nuclei_etrace_encoder_config.cmm** 脚本, 用于配置encoder组件.

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： encoder地址参数 
  * 如果是单个encoder：那么为 SB:encoder地址
  * 如果是多个encoder：那么为 SB:encoder0地址  SB:encoder1地址  SB:encoder(n)地址
* 第二个参数： 是否开启timestamp记录，可以设置：
     * ON: 开启
     * OFF: 关闭
* 第三个参数： datatrace模式，可以设置：
     * ON: 开启data trace记录
     * OFF: 关闭data trace记录


例如:

一个encoder
```
do nuclei_etrace_encoder_config.cmm "SB:0x10064000" "ON" "OFF"
```

三个encoder
```
do nuclei_etrace_encoder_config.cmm "SB:0x10064000 SB:0x10065000 SB:0x10066000" "ON" "OFF"
```

## 配置encoder解析器

提供了 **nuclei_etrace_analyze_config.cmm** 脚本, 用于配置encoder解析器

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： encoder的基地址

脚本内部，会读取encoder的硬件信息，然后自动调用 RVETRACE 的命令进行设置

如

```
do nuclei_etrace_encoder_config.cmm "0x10064000" 
```


## 配置funnel

提供了 **nuclei_etrace_funnel_config.cmm** 脚本，用于配置funnel

funnel组件，最多可以接收16个encoder的trace数据，然后合并成一组trace数据输出。

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： funnel的基地址
* 第二个参数:  要追踪trace encoder的信息
  * 如果只有一个trace encoder，需要写 "RVETRACE encoder编号"
    * RVETRACE后面的数字，表示 encoder对应的funnel的硬件映射编号
  * 如果是多个trace encoder，需要写  "RVETRACE.0 encoder0编号 RVETRACE.1 encoder1编号"
     * 其中 RVETRACE. 后的数字，表示encoder的编号, 从0开始
     * 后面的数字，表示encoder硬件和funnel的硬件映射编号

例如：

funnel连接单个encoder, 并且encoder连接在funnel的端口0

```
do nuclei_etrace_funnel_config.cmm "0x16040000" "RVETRACE 0" 
```

funnel连接单个encoder, 并且encoder0连接在funnel的端口1, encoder1连接在funnel的端口2

```
do nuclei_etrace_funnel_config.cmm "0x16040000" "RVETRACE.0 1 RVETRACE.1 2" 
```


## 配置AXI SINK  

提供了 **nuclei_etrace_axi_sink_config.cmm** 脚本, 用于配置axi sink组件，实现on-chip trace

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： axi sink的基地址
* 第二个参数:  保存trace数据的内存基地址
* 第三个参数： 保存trace数据的内存大小, 以byte为单位 
* 第四个参数： trace数据模式，可以设置：
     * FIFO: FIFO模式，如果trace数据溢出，就覆盖最老的数据
     * STACK：STACK模式，如果trace数据溢出，就不更新trace数据

例如：

```
do nuclei_etrace_axi_sink_config.cmm "0x16041000" "0xa0100000" "0x30000" "fifo"
```

## 配置PIB 

提供了 **nuclei_etrace_pib_config.cmm** 脚本， 用于配置pib组件，实现off-chip trace

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： pib sink的基地址
* 第二个参数:  pib的数据位宽，可以为：
  * 4
  * 8
  * 16
* 第三个参数： trace数据模式，可以设置：
     * FIFO: FIFO模式，如果trace数据溢出，就覆盖最老的数据
     * STACK：STACK模式，如果trace数据溢出，就不更新trace数据

例如：

```
do nuclei_etrace_pib_config.cmm "0x16042000" "4" "stack"
```


# Etrace Demo

在 onchip和offchip(pib)目录下，提供了几个配置脚本，可以参考使用

需要注意:
* 脚本中的硬件基地址，自己的硬件情况，修改基地址。
* 脚本中的以下参数，需要进行修改
  * elf路径: 对应参数 &elf_file_path
  * lauterbach-startup仓库路径: 对应参数 &nuclei_home
