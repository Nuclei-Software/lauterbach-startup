# User Guide

This User Guide primarily explains the usage related to Nuclei Cores. For general Trace32 usage, please refer to the Trace32 manual.

## 1. Connection Scripts

- `nuclei_riscv32.cmm`: Connects to a Nuclei RV32 single-core via JTAG.
- `nuclei_riscv32_smpx.cmm`: Connects to a Nuclei RV32 SMP multi-core via JTAG, where `x` is the number of cores.
- `nuclei_riscv64.cmm`: Connects to a NUCLEI RV64 single-core via JTAG.
- `nuclei_riscv64_smpx.cmm`: Connects to a Nuclei RV64 SMP multi-core via JTAG, where `x` is the number of cores.

- `nuclei_riscv32_flash.cmm`: Connects to a Nuclei Evalsoc RV32 single-core via JTAG and programs the Evalsoc XIP flash. The script uses `burn_test.bin` as the file to be programmed.
- `nuclei_riscv64_flash.cmm`: Connects to a Nuclei Evalsoc RV64 single-core via JTAG and programs the Evalsoc XIP flash. The script uses `burn_test.bin` as the file to be programmed.

- `nuclei_riscv32_cjtag.cmm`: Connects to a Nuclei RV32 single-core via cJTAG.
- `nuclei_riscv64_cjtag.cmm`: Connects to a Nuclei RV64 single-core via cJTAG.

**Note:** Evalsoc is the Nuclei evaluation platform. For more details, please refer to the *Nuclei_Processor_Integration_Guide.pdf*.

## 2. Nuclei Custom Instructions and CSRs

To load Nuclei custom instructions or CSRs, use the following commands in the connection script or the Trace32 command line:

- `apu.load nuclei_custom_inst_parser.dll`

  Loads the Nuclei custom instruction parser file `nuclei_custom_inst_parser.dll`.

- `per nuclei_custom_csr_parser.per`

  Loads the Nuclei custom CSR parser file `nuclei_custom_csr_parser.per`.
