# Table of contents

- [Table of contents](#table-of-contents)
  - [Firmware](#firmware)
    - [Work flow](#work-flow)
    - [CC2538 physical address ranges](#cc2538-physical-address-ranges)
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

### Work flow

- The CC2538 always starts at 0x0000.
- If you’ve flashed an SBL there, the ROM will execute it. The SBL then jumps to the app at 0x2000, not 0x0020.
- If you use the Without-SBL image, then your application itself is placed at 0x0000. In that case, the chip boots directly into the app, skipping any bootloader logic entirely.

### CC2538 physical address ranges

| Region                                        | Address Range             | Size        | Description                                              |
| --------------------------------------------- | ------------------------- | ----------- | -------------------------------------------------------- |
| Code Flash (Main Program Memory)              | 0x00200000 -> 0x0027FFFF  | 512KB       | Non-volatile flash memory for firmware, constants, etc.  |
| SRAM (Data Memory)                            | 0x2000 0000 → 0x2000 7FFF | 32 KB       | Volatile memory for stack, heap, global/static variable  |
| Peripheral Registers                          | 0x4000 0000 → 0x400F FFFF | 1 MB window | Memory-mapped I/O (GPIO, UART, SPI, timers, etc.)        |
| Info Page (Flash Info Region)                 | 0x0027 F000 → 0x0027 FFFF | 4 KB        | TI factory info, IEEE address, lock bits, boot config.   |
| Customer Configuration Area (CCFG)            | 0x0027 FFD0 → 0x0027 FFFF | 48 B        | Stores bootloader enable flags, image validity, etc.     |
| ROM Bootloader (internal, not in flash space) | Fixed inside chip         |             | TI’s built-in serial bootloader, runs before user flash. |

```css
Address ↑ (higher)
┌──────────────────────────┐ 0x0027_FFFF  ← End of 512 KB flash
│ CCFG (boot/config)       │
│ Info Page (factory data) │
├──────────────────────────┤ 0x0027_F000
│ Application / ZNP code   │
│ (main flash area)        │
│                          │
│ Bootloader (optional)    │
├──────────────────────────┤ 0x0020_0000  ← Start of flash
│                          │
│ [Unused gap / reserved]  │
├──────────────────────────┤
│ SRAM (variables, stack)  │
│ 0x2000_0000–0x2000_7FFF  │
├──────────────────────────┤
│ Peripheral registers      │
│ 0x4000_0000–0x400F_FFFF  │
└──────────────────────────┘
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

- The IAR version that JetHome_RU is using is different from FTDI
- I can not find the versiion they are using
- ZNP.ewp decouples to IAR. If we want to build it with IAR 8.22.1, just reset it

```BASH
git checkout "Projects\zstack\ZNP\CC2538\ZNP.ewp"
```

#### Config IAR 8.22.1

- Ensure `Runtime Checking\C/C++ compiler/additional include directories` and `Runtime Checking\C/C++ compiler/defined symbols` have values after reseting .EWP otherwise these boxes will be empty

- FDTI board is using UART FC, the config must be applied

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
