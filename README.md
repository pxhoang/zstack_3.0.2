# Table of contents

- [Table of contents](#table-of-contents)
  - [Information](#information)
  - [Firmware patch 3.0.x](#firmware-patch-30x)
  - [Build JH_2538_2592_ZNP_UART_20211222.patch](#build-jh_2538_2592_znp_uart_20211222patch)
    - [Prerequisite](#prerequisite)
    - [Apply JH_2538_2592_ZNP_UART_20211222.patch](#apply-jh_2538_2592_znp_uart_20211222patch)
      - [Apply JH_2538_2592_ZNP_UART_20211222.patch](#apply-jh_2538_2592_znp_uart_20211222patch-1)
      - [Modify PIN mapping](#modify-pin-mapping)
      - [Reset .ewp](#reset-ewp)
      - [Config IAR 8.22.1](#config-iar-8221)
    - [CC2538ZNP Without SBL](#cc2538znp-without-sbl)
    - [CC2538ZNP with SBL](#cc2538znp-with-sbl)

## Information

1/ Zstack version

```log
Z-Stack_3.0.x: CC2538 + CC2592
Z-Stack_3.x.0: CC2652P/CC2652R/CC2652RB/CC1352P-2
Ref: https://github.com/Koenkk/Z-Stack-firmware/blob/master/coordinator/README.md
```

## Firmware patch 3.0.x

1/ Koenkk firmware_CC2531_CC2530.patch

- [firmware_CC2531_CC2530.patch](https://github.com/Koenkk/Z-Stack-firmware/blob/master/coordinator/Z-Stack_3.0.x/firmware_CC2531_CC2530.patch)

```log
CC2530, CC2530_CC2591, CC2530_CC2592, CC2531

20190523
Add CC2530 and CC2530_CC2591 firmware

20190425
Initial version.

CC2538_CC2592_MODKAMRU_V3
Available here.
```

2/ JetHome: JH_2538_2592_ZNP_UART_20211222.patch

- [JH_2538_2592_ZNP_UART_20211222.patch](https://github.com/jethome-ru/zigbee-firmware/blob/master/ti/coordinator/cc2538_cc2592/README.md)

```log
Firmware version 20211222
* Fixed work with devices Aqara E1;
* Added GREEN POWER cluster;
* CODE_REVISION_NUMBER changed to 20211222

Firmware version 20201010
* FIX #2
* NV flash incremented from 12 pages to 24 pages. (HAL_NV_PAGE_CNT)
* Work with devices which are not support APS encryption (zgApsAllowR19Sec = TRUE)
* Changed FLASH and NV memory boundaries
* Disabled TCLK (requestNewTrustCenterLinkKey = FALSE)
* CODE_REVISION_NUMBER changed to 20201010

Firmware version 20200729
* Enabled Serial Bootloader (SBL). Use low logic level on PA7 input to enter SBL
* Added UART mode firmware
* CODE_REVISION_NUMBER changed to 20200729

Firmware version 20200427
* Based on MODKAM_V3 firmware. Differences from MODKAM_V3:
* Decremented number of direct children from 100 to 80: (NWK_MAX_DEVICE_LIST=80)
* CODE_REVISION_NUMBER changed to 20200427

Firmware version 20200327
* Original firmware MODKAM_V3

REF: https://github.com/jethome-ru/zigbee-firmware/blob/master/ti/coordinator/cc2538_cc2592/README.md
```

## Build JH_2538_2592_ZNP_UART_20211222.patch

### Prerequisite

- [Z-STACK-3.0.2 — Z-STACK 3.0.2](https://www.ti.com/tool/Z-STACK)
- [IAR Embedded Workbench for ARM 8.22.1](C:\Users\hoang.pham\Documents\pxhoang\work\ftdi\doc\zigbee\Legacy\transfer\software)

### Apply JH_2538_2592_ZNP_UART_20211222.patch

#### Apply JH_2538_2592_ZNP_UART_20211222.patch

```shell
git apply --directory="Z-Stack 3.0.2" .\patch\JH_2538_2592_ZNP_UART_20211222.patch
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
