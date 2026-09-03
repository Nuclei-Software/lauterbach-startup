# Nuclei TRACE32 Startup Scripts

This repository provides TRACE32 (Lauterbach) startup and utility scripts for debugging
Nuclei RISC-V processors on the Nuclei EvalSoC evaluation platform.

- A step-by-step guide (in Chinese) is available at
  [TRACE32_RISC-V_User_Guide.md](TRACE32_RISC-V_User_Guide.md).
- For general TRACE32 usage, please refer to the TRACE32 manual installed with your
  debugger (see the *TRACE32 PDF documents* section in the guide).

## 1. Connection Scripts

All connection scripts assume a Nuclei RISC-V core accessed through the RISC-V Debug
Module (DM). Scripts are provided for both RV32 and RV64 cores.

### Single-core (JTAG)

- `nuclei_riscv32.cmm` / `nuclei_riscv64.cmm`: Connect to a Nuclei RV32/RV64
  single-core SoC via a standard JTAG interface. These scripts configure the
  JTAG debug port (RVDMIAP) explicitly before bringing the system up.

### Multi-core SMP

- `nuclei_riscv32_smp2.cmm` / `nuclei_riscv32_smp4.cmm` / `nuclei_riscv32_smp8.cmm`:
  Connect to a Nuclei RV32 SMP SoC with 2/4/8 cores via JTAG.
- `nuclei_riscv64_smp2.cmm` / `nuclei_riscv64_smp4.cmm` / `nuclei_riscv64_smp8.cmm`:
  Connect to a Nuclei RV64 SMP SoC with 2/4/8 cores via JTAG.

### Multi-core AMP (JTAG daisy chain)

- `nuclei_riscv32_jtagchain_dmi.cmm`: Connect to a Nuclei RV32 AMP SoC whose cores
  share one JTAG port but sit on separate TAPs (daisy-chained). It attaches to the
  already-running cores with `SYStem.Attach` instead of resetting the SoC.

### cJTAG

- `nuclei_riscv32_cjtag.cmm` / `nuclei_riscv64_cjtag.cmm`: Connect to a Nuclei
  RV32/RV64 single-core SoC via the 2-pin cJTAG interface (implicit DMI).

### Flash programming

- `nuclei_riscv32_flash.cmm` / `nuclei_riscv64_flash.cmm`: Connect to a Nuclei
  EvalSoC RV32/RV64 single-core via JTAG and program the EvalSoC XIP flash. The
  scripts use `burn_test.bin` as the file to be programmed; the flash programming
  algorithms live under [`flash/`](flash/).

> **Note:** EvalSoC is the Nuclei evaluation platform. For more details, please
> refer to the *Nuclei_Processor_Integration_Guide.pdf*.

## 2. Nuclei Custom Instructions and CSRs

Nuclei cores may use custom instructions and custom CSRs. To make TRACE32 decode
them as readable mnemonics and register names, load the corresponding parser files
in the connection script or from the TRACE32 command line:

- `apu.load nuclei_custom_inst_parser.dll`

  Loads the Nuclei custom instruction parser file `nuclei_custom_inst_parser.dll`.

- `per nuclei_custom_csr_parser.per`

  Loads the Nuclei custom CSR parser file `nuclei_custom_csr_parser.per`.
