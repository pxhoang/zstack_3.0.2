# Table of contents

- [Table of contents](#table-of-contents)
  - [Firmware](#firmware)
    - [Known issues](#known-issues)
    - [Physical address](#physical-address)
      - [No Bootloader](#no-bootloader)
      - [Bootloader](#bootloader)
      - [Extend 32K RAM for CC2538](#extend-32k-ram-for-cc2538)
    - [Flash address](#flash-address)
  - [Build notes](#build-notes)
    - [Zstack version](#zstack-version)
    - [Firmware patch 3.0.x](#firmware-patch-30x)
    - [IAR zstack workspaces](#iar-zstack-workspaces)
    - [CC2538ZNP-Debug](#cc2538znp-debug)
    - [CC2538ZNP-With-SBL](#cc2538znp-with-sbl)
    - [CC2538ZNP-Without-SBL](#cc2538znp-without-sbl)
  - [Build JH\_2538\_2592\_ZNP\_UART\_20211222.patch](#build-jh_2538_2592_znp_uart_20211222patch)
    - [Prerequisite](#prerequisite)
    - [JH\_2538\_2592\_ZNP\_UART\_20211222.patch](#jh_2538_2592_znp_uart_20211222patch)
      - [Apply JH\_2538\_2592\_ZNP\_UART\_20211222.patch](#apply-jh_2538_2592_znp_uart_20211222patch)
      - [Modify PIN mapping](#modify-pin-mapping)
      - [Reset .ewp](#reset-ewp)
      - [Config IAR 8.22.1](#config-iar-8221)
    - [CC2538ZNP Without SBL](#cc2538znp-without-sbl-1)
    - [CC2538ZNP with SBL](#cc2538znp-with-sbl-1)

## Firmware

### Known issues

```bash
# REF: https://e2e.ti.com/support/wireless-connectivity/zigbee-thread-group/zigbee-and-thread/f/zigbee-thread-forum/902631/faq-cc2538-z-stack-3-0-2-known-issues-and-fixes
1. Memory Leak in zcl.c
```

| No  | Title                                                                                                              | Description | IsFixed |
| --- | ------------------------------------------------------------------------------------------------------------------ | ----------- | ------- |
| 1   | Memory Leak in zcl.c                                                                                               |             | YES     |
| 2   | Network association issues after prior APS Remove or Leave Request                                                 |             | YES     |
| 3   | Soft Reset/Freezing/Stack Overflow due to BDB_REPORTING on CC2530/CC2531                                           |             |         |
| 4   | OTA Upgrade from ZHA 1.2 to Z3.0 results in broken device                                                          |             | YES     |
| 5   | IEEE Address of CC2538ZNP is reported incorrectly                                                                  |             |         |
| 6   | Modifying CC2538 Linker to utilize all 32k RAM causes lockup                                                       |             | TODO    |
| 7   | ZED does not receive Broadcast sent to MAC dst 0xFFFF from parent                                                  |             |         |
| 8   | NWK/APS packet retries/failure when ZED polls during commissioning (Transport Key, Simple Descriptor Request, etc) |             |         |
| 9   | ZED does not go back to low power state after losing parent and attempting recovery                                |             |         |
| 10  | ZCL ID mixup for Chatting + Voice Over Zigbee                                                                      |             |         |
| 11  | CC2530/1 ZNP device sporadically returns ZMemError                                                                 |             |         |
| 12  | ZC sends ZEDs Leave Reqs after MAC Association                                                                     |             |         |
| 13  | APS ACK transID parameter error if using zcl_TransID                                                               |             |         |
| 14  | BDB reporting doesn't support read/write attribute data callback functions                                         |             |         |
| 15  | Errors due to BDB header build types                                                                               |             |         |
| 16  | Low Power Mode not Entered upon ZED Rejoin                                                                         |             |         |
| 17  | Low Power Mode not Entered upon ZED Factory New Before Commissioning                                               |             |         |
| 18  | ZED Poll Rate Set Incorrectly if Reset from a Leave Request with Rejoin Enabled                                    |             |         |
| 19  | Memory Leak in the F&B Function of BDB                                                                             |             |         |
| 20  | Large number of Address Manager Entries Results in Long ZC Startup Times                                           |             |         |
| 21  | Inclusion of HAL_PA_LNA_CC2592 did not Propagate to ZNP Project                                                    |             |         |
| 22  | Multiple attribute reporting issue                                                                                 |             |         |
| 23  | ZNP initializes GP proxy table with the wrong item ID                                                              |             |         |
| 24  | Frame counter does not increment on device reset                                                                   |             |         |

### Physical address

#### No Bootloader

```scss
          DEFAULT TI LAYOUT                                   MODKAMRU LAYOUT
┌───────────────────────────────────────────────┐    ┌───────────────────────────────────────────────┐
│               ON-CHIP FLASH 512 KB            │    │               ON-CHIP FLASH 512 KB            │
│                                               │    │                                               │
│ [Factory / Security / Keys]                   │    │ [Factory / Security / Keys]                   │
│  FLASH_LCK   0x0027FFE0–0x0027FFFF (32 B)     │    │  FLASH_LCK   0x0027FFE0–0x0027FFFF (32 B)     │
│  FLASH_CCA   0x0027FFD4–0x0027FFDF (12 B)     │    │  FLASH_CCA   0x0027FFD4–0x0027FFDF (12 B)     │
│  CP_IEEE     0x0027FFCC–0x0027FFD3 (8 B)      │    │  CP_IEEE     0x0027FFCC–0x0027FFD3 (8 B)      │
│  CP_DEPK     0x0027FFB4–0x0027FFCB (24 B)     │    │  CP_DEPK     0x0027FFB4–0x0027FFCB (24 B)     │
│  CP_CAPK     0x0027FF9C–0x0027FFB3 (24 B)     │    │  CP_CAPK     0x0027FF9C–0x0027FFB3 (24 B)     │
│  CP_IMPC     0x0027FF6C–0x0027FF9B (48 B)     │    │  CP_IMPC     0x0027FF6C–0x0027FF9B (48 B)     │
│  CP_DEPK_283 0x0027FF48–0x0027FF6B (36 B)     │    │  CP_DEPK_283 0x0027FF48–0x0027FF6B (36 B)     │
│  CP_CAPK_283 0x0027FF20–0x0027FF47 (40 B)     │    │  CP_CAPK_283 0x0027FF20–0x0027FF47 (40 B)     │
│  CP_IMPC_283 0x0027FED4–0x0027FF1F (76 B)     │    │  CP_IMPC_283 0x0027FED4–0x0027FF1F (76 B)     │
│  Factory Params 0x0027F800–0x0027FED3 (3.8 KB)│    │  Factory Params 0x0027F800–0x0027FED3 (3.8 KB)│
│───────────────────────────────────────────────│    │───────────────────────────────────────────────│
│ [Z-Stack NV Memory]                           │    │ [Z-Stack NV Memory]                           │
│  NV_MEM 0x0027C800–0x0027F7FF (12 KB, 6 pages)│    │  NV_MEM 0x00273800–0x0027F7FF (48 KB, 24 pages)│
│  ▸ Network tables / bindings / keys           │    │  ▸ Expanded persistent storage                │
│───────────────────────────────────────────────│    │───────────────────────────────────────────────│
│ [Application + Z-Stack Firmware]              │    │ [Application + Z-Stack Firmware]              │
│  FLASH 0x00200000–0x0027C7FF (503 808 B)      │    │  FLASH 0x00200000–0x002737FF (462 336 B)      │
│  ▸ Startup / ISR vectors / HAL / OSAL         │    │  ▸ Startup / ISR vectors / HAL / OSAL         │
│  ▸ Z-Stack core (NWK/APS/ZDO)                 │    │  ▸ Z-Stack core (NWK/APS/ZDO)                 │
│  ▸ Application tasks / profiles               │    │  ▸ Application tasks / profiles               │
│  ▸ Const data, libraries                      │    │  ▸ Const data, libraries                      │
└───────────────────────────────────────────────┘    └───────────────────────────────────────────────┘
│               ON-CHIP SRAM 32 KB              │    │               ON-CHIP SRAM 32 KB              │
│───────────────────────────────────────────────│    │───────────────────────────────────────────────│
│ Used RAM 16 KB 0x20004000–0x20007FFF          │    │ Full RAM 32 KB 0x20000000–0x20007FFF          │
│  ▸ Stack / heap / Z-Stack buffers             │    │  ▸ Stack / heap / Z-Stack buffers             │
│───────────────────────────────────────────────│    │───────────────────────────────────────────────│
│ Reserved 16 KB 0x20000000–0x20003FFF          │    │ — (no reserved area, fully usable)            │
└───────────────────────────────────────────────┘    └───────────────────────────────────────────────┘
```

#### Bootloader

```text
┌───────────────────────────────────────────────────────┐
│                  ON-CHIP FLASH 512 KB                 │
│                                                       │
│ [Factory / Security / Keys]                           │
│  FLASH_LCK     0x0027FFE0–0x0027FFFF   (32 B)         │
│  FLASH_CCA     0x0027FFD4–0x0027FFDF   (12 B)         │
│  CP_IEEE       0x0027FFCC–0x0027FFD3   (8 B)          │
│  CP_DEPK       0x0027FFB4–0x0027FFCB   (24 B)         │
│  CP_CAPK       0x0027FF9C–0x0027FFB3   (24 B)         │
│  CP_IMPC       0x0027FF6C–0x0027FF9B   (48 B)         │
│  CP_DEPK_283   0x0027FF48–0x0027FF6B   (36 B)         │
│  CP_CAPK_283   0x0027FF20–0x0027FF47   (40 B)         │
│  CP_IMPC_283   0x0027FED4–0x0027FF1F   (76 B)         │
│  Factory Params 0x0027F800–0x0027FED3  (3.8 KB)       │
│───────────────────────────────────────────────────────│
│ [Z-Stack NV Memory]                                   │
│  NV_MEM 0x0027C800–0x0027F7FF  (12 KB, 6 pages)       │
│  ▸ Persistent network state, bindings, keys           │
│───────────────────────────────────────────────────────│
│ [Application + Bootloader Firmware]                   │
│  FLASH 0x00200134–0x0027AFFF  (≈503 KB)               │
│  ▸ Z-Stack core (NWK, APS, ZDO)                       │
│  ▸ Application tasks, profiles, clusters              │
│  ▸ System ISR handlers, HAL, and startup code         │
│  ▸ Embedded single bootloader routines                │
│───────────────────────────────────────────────────────│
│ [Bootloader Header / Vector Table]                    │
│  SBL_CHECKSUM 0x0020011C–0x00200133 (28 B)            │
│  INTVEC       0x00200000–0x0020011B (284 B)           │
│  ▸ Interrupt vector table and checksum for integrity  │
└───────────────────────────────────────────────────────┘
│                    ON-CHIP SRAM 32 KB                 │
│───────────────────────────────────────────────────────│
│ Used RAM  0x20004000–0x20007FFF (16 KB)               │
│  ▸ Stack, heap, RTOS variables, Z-Stack buffers       │
│ Reserved  0x20000000–0x20003FFF (16 KB)               │
│  ▸ Reserved for ROM routines / future expansion       │
└───────────────────────────────────────────────────────┘
```

#### Extend 32K RAM for CC2538

- FTDI CC2538 uses 32KB RAM, but Z-stack 3.0.2 is not enabled it by default.
- Coordinator is NOT PM2 device (device always powered). They stay in PM0 or PM1 to route messages continuously.
- Therefore, you can safely use all 32 KB of RAM.

1/ 32K memory layout

```text
         DEFAULT TI (SINGLE BOOTLOADER)                   EXTENDED (24 NV PAGES + FULL 32 KB SRAM)
┌────────────────────────────────────────────┐        ┌────────────────────────────────────────────┐
│              ON-CHIP FLASH 512 KB          │        │              ON-CHIP FLASH 512 KB          │
│                                            │        │                                            │
│ [Factory / Security / Keys]                │        │ [Factory / Security / Keys]                │
│  FLASH_LCK     0x0027FFE0–0x0027FFFF (32B) │        │  FLASH_LCK     0x0027FFE0–0x0027FFFF (32B) │
│  FLASH_CCA     0x0027FFD4–0x0027FFDF (12B) │        │  FLASH_CCA     0x0027FFD4–0x0027FFDF (12B) │
│  CP_IEEE       0x0027FFCC–0x0027FFD3 (8B)  │        │  CP_IEEE       0x0027FFCC–0x0027FFD3 (8B)  │
│  CP_xxx Keys/Certs 0x0027FED4–0x0027FFCB   │        │  CP_xxx Keys/Certs 0x0027FED4–0x0027FFCB   │
│  Factory Params 0x0027F800–0x0027FED3 (3.8K)│       │  Factory Params 0x0027F800–0x0027FED3 (3.8K)│
│────────────────────────────────────────────│        │────────────────────────────────────────────│
│ [Z-Stack NV Memory]                        │        │ [Z-Stack NV Memory — EXPANDED]             │
│  NV_MEM 0x0027C800–0x0027F7FF (12 KB)      │        │  NV_MEM 0x00273800–0x0027F7FF (48 KB)      │
│  ▸ 6 pages × 2 KB                          │        │  ▸ 24 pages × 2 KB                         │
│────────────────────────────────────────────│        │────────────────────────────────────────────│
│ [Application + Bootloader Firmware]        │        │ [Application + Bootloader Firmware]        │
│  FLASH 0x00200134–0x0027AFFF (≈502 KB)     │        │  FLASH 0x00200134–0x002737FF (≈462 KB)     │
│  ▸ Z-Stack core, app code, boot routines   │        │  ▸ Same firmware, reduced code space       │
│────────────────────────────────────────────│        │────────────────────────────────────────────│
│ [Bootloader Header / Vectors]              │        │ [Bootloader Header / Vectors]              │
│  SBL_CHECKSUM 0x0020011C–0x00200133 (28B)  │        │  SBL_CHECKSUM 0x0020011C–0x00200133 (28B)  │
│  INTVEC       0x00200000–0x0020011B (284B) │        │  INTVEC       0x00200000–0x0020011B (284B) │
└────────────────────────────────────────────┘        └────────────────────────────────────────────┘
│               ON-CHIP SRAM 32 KB           │        │               ON-CHIP SRAM 32 KB           │
│────────────────────────────────────────────│        │────────────────────────────────────────────│
│ Used RAM 16 KB 0x20004000–0x20007FFF       │        │ Full RAM 32 KB 0x20000000–0x20007FFF       │
│  ▸ Stack / Heap / Buffers (half used)      │        │  ▸ Stack / Heap / Buffers (fully enabled)  │
│ Reserved 16 KB 0x20000000–0x20003FFF       │        │  — (no reserved region)                    │
└────────────────────────────────────────────┘        └────────────────────────────────────────────┘
```

2/ Modifying CC2538 Linker to utilize all 32k RAM

- Some CC2538 parts have 32k of RAM, but the first 16k is disabled by default in our linker file since only the second 16k is retained in all power modes. The first 16k of RAM loses its contents when the device enters PM2.
- If a device which never goes into low power mode, it is fine to reclaim this RAM for application usage.

```diff
# Z-Stack 3.0.2\Projects\zstack\Tools\CC2538DB\CC2538.icf
# Z-Stack 3.0.2\Projects\zstack\Tools\CC2538DB\CC2538-sb.icf
@@
-define region SRAM = mem:[from 0x20004000 to 0x20007FFF];
+define region SRAM = mem:[from 0x20000000 to 0x20007FFF];     // use full 32 KB RAM
```

Potential issue:

- However, in some applications, increasing the RAM to this value will cause Z-Stack to lock up.
- Move the run-time stack to the end of RAM. Find the code below in your linker file and make the following change:

```bash
//
// Indicate that the noinit values should be left alone.  This includes the
// stack, which if initialized will destroy the return address from the
// initialization code, causing the processor to branch to zero and fault.
//
do not initialize { section .noinit };
place at end of SRAM { section .noinit }; // ++++++++++ ADD THIS LINE ++++++++++
```

### Flash address

Flash firmware means uploading flash image starting from 0x0020_0000. Assume,

- CC2538ZNP_Bootloader.bin: BL (20KB) `zstack_3.0.2\Z-Stack 3.0.2\Projects\zstack\Utilities\BootLoad\CC2538_UART\Boot.eww`
- CC2538ZNP_With_SBL.bin: APP_SBL that compatibles with an bootloader. (492KB) `zstack_3.0.2\Z-Stack 3.0.2\Projects\zstack\ZNP\CC2538` - Workspace CC2538ZNP_With_SBL
- CC2538ZNP_Without_SBL.bin: APP_NOBL that can work without bootloader (492KB) `zstack_3.0.2\Z-Stack 3.0.2\Projects\zstack\ZNP\CC2538` - Workspace CC2538ZNP_Without_SBL

1/ Combined Firmware (COMBINE_FIRMARE)

- Combine BL + APP_SBL (CC2538ZNP_With_SBL.bin + CC2538ZNP_Bootloader.bin) (512KB)
- Can be programmed using the SmartRF Programmer tool from TI, start address is 0x00200000

2/ APP_SBL

- Can be loaded to a module flashed with `COMBINE_FIRMARE` using the UART
- Using to update with zigbee plugin

2/ APP_NOBL (492KB)

- DON'T use this

3/ Modkamru patch

- Changes in Modkamru is only applied for CC2538.icf, It means the workspace to build must be `CC2538ZNP_Without_SBL`
- Therefore, there is no bootloader on CC2538. We can not use OTA update (APP_SBL)
- Modkamru enabled BOOTLOADER_BACKDOOR_ENABLE(`startup_ewarm.c`). By this changes, When you hold a specific GPIO pin (usually PA0) during reset, the CC2538 will enter the ROM bootloader mode. Then use TI’s SmartRF Flash Programmer 2 or cc2538-bsl.py over UART to reflash firmware.

```diff
diff --git a/Projects/zstack/Tools/CC2538DB/CC2538.icf b/Projects/zstack/Tools/CC2538DB/CC2538.icf
index d4c95cc..b56a2ff 100644
--- a/Projects/zstack/Tools/CC2538DB/CC2538.icf
+++ b/Projects/zstack/Tools/CC2538DB/CC2538.icf
-define region FLASH = mem:[from 0x00200000 to 0x0027C7FF];
+define region FLASH = mem:[from 0x00200000 to 0x002737FF];

-define region NV_MEM = mem:[from 0x0027C800 to 0x0027F7FF];
+define region NV_MEM = mem:[from 0x00273800 to 0x0027F7FF];

-define region SRAM = mem:[from 0x20004000 to 0x20007FFF];
+define region SRAM = mem:[from 0x20000000 to 0x20007FFF];
```

## Build notes

### Zstack version

```log
Z-Stack_3.0.x: CC2538 + CC2592
Z-Stack_3.x.0: CC2652P/CC2652R/CC2652RB/CC1352P-2
Ref: https://github.com/Koenkk/Z-Stack-firmware/blob/master/coordinator/README.md
```

### Firmware patch 3.0.x

1/ Koenkk firmware_CC2531_CC2530.patch

- [firmware_CC2531_CC2530.patch](https://github.com/Koenkk/Z-Stack-firmware/blob/master/coordinator/Z-Stack_3.0.x/firmware_CC2531_CC2530.patch)

2/ JetHome: JH_2538_2592_ZNP_UART_20211222.patch

- [JH_2538_2592_ZNP_UART_20211222.patch](https://github.com/jethome-ru/zigbee-firmware/blob/master/ti/coordinator/cc2538_cc2592/README.md)

### IAR zstack workspaces

- "With-SBL" = app is compatible with an SBL, not contains the SBL.

### CC2538ZNP-Debug

```bash
# options/output converter/generate additional output/raw binary
$PROJ_DIR$\dev\ZNP.bin

# options/Runtime checking/C/C++ compiler/preprocessor
xHAL_UART_USB
HAL_UART=TRUE
xZNP_ALT
```

### CC2538ZNP-With-SBL

- “With-SBL” = app is compatible with an SBL, not contains the SBL.

```BASH
# options/ Runtime checking/ output converter/generate additional output/raw binary
$PROJ_DIR$\dev\CC2538ZNP-with-SBL.bin

# options/ Runtime checking/ output converter/generate additional output/build actions
# edit znp.js
--start znp.js %2
++node znp.js %2

# options/ Runtime checking/C/C++ compiler/preprocessor
xHAL_UART_USB
HAL_UART=TRUE
xZNP_ALT
```

### CC2538ZNP-Without-SBL

```BASH
# options/ Runtime checking/ output converter/generate additional output/raw binary
$PROJ_DIR$\dev\CC2538ZNP-without-SBL.bin

# options/ Runtime checking/ output converter/generate additional output/build actions
# edit znp.js
--start znp.js %2
++node znp.js %2

# options/Runtime checking/C/C++ compiler/preprocessor
xHAL_UART_USB
HAL_UART=TRUE
xZNP_ALT
```

## Build JH_2538_2592_ZNP_UART_20211222.patch

### Prerequisite

- [Z-STACK-3.0.2 — Z-STACK 3.0.2](https://www.ti.com/tool/Z-STACK)
- [IAR Embedded Workbench for ARM 8.22.1](C:\Users\hoang.pham\Documents\pxhoang\work\ftdi\doc\zigbee\Legacy\transfer\software)
- NodeJS

### JH_2538_2592_ZNP_UART_20211222.patch

#### Apply JH_2538_2592_ZNP_UART_20211222.patch

```BASH
git apply --directory="Z-Stack 3.0.2" .\patch\JH_2538_2592_ZNP_UART_20211222.patch --ignore-space-change
```

#### Modify PIN mapping

#### Reset .ewp

- The IAR version that JetHome_RU is using to build firmware is different from ours

```BASH
git checkout "Projects\zstack\ZNP\CC2538\ZNP.ewp"
```

#### Config IAR 8.22.1

- Ensure `Runtime Checking\C/C++ compiler/additional include directories` and `Runtime Checking\C/C++ compiler/defined symbols` are not empty
- FDTI board is using UART FC, these must be applied

```bash
xHAL_UART_USB
HAL_UART=TRUE
xZNP_ALT
```

The final defined symbols must be as below

```bash
xHAL_UART_USB
xUSB_SETUP_MAX_NUMBER_OF_INTERFACES=5
xHAL_SPI=TRUE
xHAL_UART=TRUE
BDB_FINDING_BINDING_CAPABILITY_ENABLED=0
DISABLE_GREENPOWER_BASIC_PROXY
TC_LINKKEY_JOIN
ewarm
CC2538_USE_ALTERNATE_INTERRUPT_MAP=1
CC2538ZNP
ZNP_ALT
xPOWER_SAVING
FEATURE_SYSTEM_STATS
FEATURE_RESET_MACRO
ZDNWKMGR_MIN_TRANSMISSIONS=0
MT_UART_DEFAULT_OVERFLOW=FALSE
ASSERT_RESET
MAKE_CRC_SHDW
SBL_CLIENT
ZCL_READ
ZCL_DISCOVER
ZCL_WRITE
ZCL_BASIC
```

### CC2538ZNP Without SBL

- `Runtime Checking\Linker` for CC2538ZNP WithoutOnBoard.c SBL must be `C:\Users\hoang.pham\Documents\pxhoang\git\zstack_3.2.0\Projects\zstack\Tools\CC2538DB\CC2538.icf`
- Ideally, IAR should update it automatically when changing mode, but it does not, It causes build errors. In that case, update it manually

### CC2538ZNP with SBL

- `Runtime Checking\Linker` for CC2538ZNP Without SBL must be `C:\Users\hoang.pham\Documents\pxhoang\git\zstack_3.2.0\Projects\zstack\Tools\CC2538DB\CC2538-sb.icf`
- Ideally, IAR should update it automatically when changing mode, but it does not, It causes build errors. In that case, update it manually
