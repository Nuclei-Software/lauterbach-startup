# 目录结构

* onchip: 针对于on-chip调试，也就是trace数据是芯片内部总线，写入到存储中。如片内SRAM， DDR等。
* off-chip：针对于off-chip调试，也就是trace数据是通过芯片的PAD，保存在外部的trace工具中

# 公共脚本
提供了4个公共脚本，用于配置 etrace的各个组件：

* nuclei_etrace_encoder_config.cmm: 用于配置encoder
* nuclei_etrace_funnel_config.cmm: 用于配置funnel
* nuclei_etrace_axi_sink_config.cmm: 用于配置axi sink组件，实现on-chip trace
* nuclei_etrace_pib_config.cmm: 用于配置pib组件，实现off-chip trace


## nuclei_etrace_encoder_config.cmm
该脚本提供了对于encoder的配置。
外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： encoder的基地址
* 第二个参数： trace包中trace编号， 可以设置:
     * AUTO: 工具自动分配
     * 值：手动指定
* 第三个参数： 是否开启timestamp记录，可以设置：
     * ON: 开启
     * OFF: 关闭
* 第四个参数： trace数据模式，可以设置：
     * FIFO: FIFO模式，如果trace数据溢出，就覆盖最老的数据
     * STACK：STACK模式，如果trace数据溢出，就不更新trace数据
* 第五个参数： datatrace模式，可以设置：
     * ON: 开启data trace记录
     * OFF: 关闭data trace记录

如

```
do nuclei_etrace_encoder_config.cmm "0x10064000" "AUTO" "ON" "fifo" "OFF"
```

## nuclei_etrace_funnel_config.cmm

该脚本是用来配置funnel组件

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： funnel的基地址
* 第二个参数:  要追踪trace encoder的编号

如：

```
do nuclei_etrace_funnel_config.cmm "0x16040000" "0" 
```

## nuclei_etrace_axi_sink_config.cmm

该脚本是用来配置axi sink组件，实现on-chip trace。

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： axi sink的基地址
* 第二个参数:  保存trace数据的内存基地址
* 第三个参数： 保存trace数据的内存大小 

如：

```
do nuclei_etrace_axi_sink_config.cmm "0x16041000" "0xa0100000" "0x30000"
```

## nuclei_etrace_pib_config.cmm

该脚本是用来配置pib组件，实现off-chip trace。

外部使用do命令调用该脚本，然后传入如下参数：
* 第一个参数： pib sink的基地址
* 第二个参数:  pib的数据位宽，可以为：
  * 4
  * 8
  * 16

如：

```
do nuclei_etrace_pib_config.cmm "0x16042000" "4"
```


# demo
在 onchip和offchip(pib)目录下，提供了几个配置脚本，可以参考使用
