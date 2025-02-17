.. SPDX-License-Identifier: GPL-2.0+ OR BSD-3-Clause
.. sectionauthor:: Vignesh Raghavendra <vigneshr@ti.com>

AM62 Platforms
===============

Introduction:
-------------
The AM62 SoC family is the follow on AM335x built on the K3 Multicore
SoC architecture platform, providing ultra-low-power modes, dual
display, multi-sensor edge compute, security and other BOM-saving
integrations.  The AM62 SoC targets a broad market to enable
applications such as Industrial HMI, PLC/CNC/Robot control, Medical
Equipment, Building Automation, Appliances and more.

Some highlights of this SoC are:

* Quad-Cortex-A53s (running up to 1.4GHz) in a single cluster.
  Pin-to-pin compatible options for single and quad core are available.
* Cortex-M4F for general-purpose or safety usage.
* Dual display support, providing 24-bit RBG parallel interface and
  OLDI/LVDS-4 Lane x2, up to 200MHz pixel clock support for 2K display
  resolution.
* Selectable GPU support, up to 8GFLOPS, providing better user experience
  in 3D graphic display case and Android.
* PRU(Programmable Realtime Unit) support for customized programmable
  interfaces/IOs.
* Integrated Giga-bit Ethernet switch supporting up to a total of two
  external ports (TSN capable).
* 9xUARTs, 5xSPI, 6xI2C, 2xUSB2, 3xCAN-FD, 3x eMMC and SD, GPMC for
  NAND/FPGA connection, OSPI memory controller, 3xMcASP for audio,
  1x CSI-RX-4L for Camera, eCAP/eQEP, ePWM, among other peripherals.
* Dedicated Centralized System Controller for Security, Power, and
  Resource Management.
* Multiple low power modes support, ex: Deep sleep, Standby, MCU-only,
  enabling battery powered system design.

More details can be found in the Technical Reference Manual:
https://www.ti.com/lit/pdf/spruiv7

Boot Flow:
----------
Below is the pictorial representation of boot flow:

.. code-block:: text

 +------------------------------------------------------------------------+
 |        TIFS            |      Main R5          |        A53            |
 +------------------------------------------------------------------------+
 |    +--------+          |                       |                       |
 |    |  Reset |          |                       |                       |
 |    +--------+          |                       |                       |
 |         :              |                       |                       |
 |    +--------+          |   +-----------+       |                       |
 |    | *ROM*  |----------|-->| Reset rls |       |                       |
 |    +--------+          |   +-----------+       |                       |
 |    |        |          |         :             |                       |
 |    |  ROM   |          |         :             |                       |
 |    |services|          |         :             |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |   |  *R5 ROM*   |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |<---------|---|Load and auth|     |                       |
 |    |        |          |   | tiboot3.bin |     |                       |
 |    +--------+          |   +-------------+     |                       |
 |    |        |<---------|---| Load sysfw  |     |                       |
 |    |        |          |   | part to TIFS|     |                       |
 |    |        |          |   | core        |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |         :             |                       |
 |    |        |          |         :             |                       |
 |    |        |          |         :             |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |   |  *R5 SPL*   |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |   |    DDR      |     |                       |
 |    |        |          |   |   config    |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |   |    Load     |     |                       |
 |    |        |          |   |  tispl.bin  |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |   |   Load R5   |     |                       |
 |    |        |          |   |   firmware  |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |<---------|---| Start A53   |     |                       |
 |    |        |          |   | and jump to |     |                       |
 |    |        |          |   | DM fw image |     |                       |
 |    |        |          |   +-------------+     |                       |
 |    |        |          |                       |     +-----------+     |
 |    |        |----------|-----------------------|---->| Reset rls |     |
 |    |        |          |                       |     +-----------+     |
 |    |  TIFS  |          |                       |          :            |
 |    |Services|          |                       |     +-------------+   |
 |    |        |<---------|-----------------------|---->|*TF-A/OP-TEE*|   |
 |    |        |          |                       |     +-------------+   |
 |    |        |          |                       |          :            |
 |    |        |          |                       |     +-----------+     |
 |    |        |<---------|-----------------------|---->| *A53 SPL* |     |
 |    |        |          |                       |     +-----------+     |
 |    |        |          |                       |     |   Load    |     |
 |    |        |          |                       |     | u-boot.img|     |
 |    |        |          |                       |     +-----------+     |
 |    |        |          |                       |          :            |
 |    |        |          |                       |     +-----------+     |
 |    |        |<---------|-----------------------|---->| *U-Boot*  |     |
 |    |        |          |                       |     +-----------+     |
 |    |        |          |                       |     |  prompt   |     |
 |    |        |----------|-----------------------|-----+-----------+-----|
 |    +--------+          |                       |                       |
 |                        |                       |                       |
 +------------------------------------------------------------------------+

- Here TIFS acts as master and provides all the critical services. R5/A53
  requests TIFS to get these services done as shown in the above diagram.

Sources:
--------
1. Trusted Firmware-A:
	Tree: https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git/
	Branch: master

2. OP-TEE:
	Tree: https://github.com/OP-TEE/optee_os.git
	Branch: master

3. U-Boot:
	Tree: https://source.denx.de/u-boot/u-boot
	Branch: master

4. TI Linux Firmware:
	Tree: git://git.ti.com/processor-firmware/ti-linux-firmware.git
	Branch: ti-linux-firmware

Build procedure:
----------------
1. Trusted Firmware-A:

.. code-block:: bash

 $ make CROSS_COMPILE=aarch64-none-linux-gnu- ARCH=aarch64 PLAT=k3 \
        TARGET_BOARD=lite SPD=opteed

2. OP-TEE:

.. code-block:: bash

 $ make PLATFORM=k3 CFG_ARM64_core=y CROSS_COMPILE=arm-none-linux-gnueabihf- \
        CROSS_COMPILE64=aarch64-none-linux-gnu-

3. U-Boot:

* 3.1 R5:

.. code-block:: bash

 $ make ARCH=arm am62x_evm_r5_defconfig
 $ make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- \
        BINMAN_INDIRS=<path/to/ti-linux-firmware>

* 3.2 A53:

.. code-block:: bash

 $ make ARCH=arm am62x_evm_a53_defconfig
 $ make ARCH=arm CROSS_COMPILE=aarch64-none-linux-gnu- \
        BL31=<path/to/trusted-firmware-a/dir>/build/k3/lite/release/bl31.bin \
        TEE=<path/to/optee_os/dir>/out/arm-plat-k3/core/tee-raw.bin \
        BINMAN_INDIRS=<path/to/ti-linux-firmware>

Target Images
--------------
Copy the below images to an SD card and boot:

 - GP

        * tiboot3-am62x-gp-evm.bin from step 3.1
        * tispl.bin_unsigned, u-boot.img_unsigned from step 3.2

 - HS-FS

        * tiboot3-am62x-hs-fs-evm.bin from step 3.1
        * tispl.bin, u-boot.img from step 3.2

 - HS-SE

        * tiboot3-am62x-hs-evm.bin from step 3.1
        * tispl.bin, u-boot.img from step 3.2

Image formats:
--------------

- tiboot3.bin:

.. code-block:: text

                +-----------------------+
                |        X.509          |
                |      Certificate      |
                | +-------------------+ |
                | |                   | |
                | |        R5         | |
                | |   u-boot-spl.bin  | |
                | |                   | |
                | +-------------------+ |
                | |                   | |
                | |TIFS with board cfg| |
                | |                   | |
                | +-------------------+ |
                | |                   | |
                | |                   | |
                | |     FIT header    | |
                | | +---------------+ | |
                | | |               | | |
                | | |   DTB 1...N   | | |
                | | +---------------+ | |
                | +-------------------+ |
                +-----------------------+

- tispl.bin

.. code-block:: text

                +-----------------------+
                |                       |
                |       FIT HEADER      |
                | +-------------------+ |
                | |                   | |
                | |     A53 TF-A      | |
                | +-------------------+ |
                | |                   | |
                | |     A53 OP-TEE    | |
                | +-------------------+ |
                | |                   | |
                | |      R5 DM FW     | |
                | +-------------------+ |
                | |                   | |
                | |      A53 SPL      | |
                | +-------------------+ |
                | |                   | |
                | |   SPL DTB 1...N   | |
                | +-------------------+ |
                +-----------------------+

A53 SPL DDR Memory Layout
-------------------------

This provides an overview memory usage in A53 SPL stage.

.. list-table::
   :widths: 16 16 16
   :header-rows: 1

   * - Region
     - Start Address
     - End Address

   * - EMPTY
     - 0x80000000
     - 0x80080000

   * - TEXT BASE
     - 0x80080000
     - 0x800d8000

   * - EMPTY
     - 0x800d8000
     - 0x80200000

   * - BMP IMAGE
     - 0x80200000
     - 0x80b77660

   * - STACK
     - 0x80b77660
     - 0x80b77e60

   * - GD
     - 0x80b77e60
     - 0x80b78000

   * - MALLOC
     - 0x80b78000
     - 0x80b80000

   * - EMPTY
     - 0x80b80000
     - 0x80c80000

   * - BSS
     - 0x80c80000
     - 0x80d00000

   * - BLOBS
     - 0x80d00000
     - 0x80d00400

   * - EMPTY
     - 0x80d00400
     - 0x81000000

Switch Setting for Boot Mode
----------------------------

Boot Mode pins provide means to select the boot mode and options before the
device is powered up. After every POR, they are the main source to populate
the Boot Parameter Tables.

The following table shows some common boot modes used on AM62 platform. More
details can be found in the Technical Reference Manual:
https://www.ti.com/lit/pdf/spruiv7 under the `Boot Mode Pins` section.

*Boot Modes*

============ ============= =============
Switch Label SW2: 12345678 SW3: 12345678
============ ============= =============
SD           01000000      11000010
OSPI         00000000      11001110
EMMC         00000000      11010010
UART         00000000      11011100
USB DFU      00000000      11001010
============ ============= =============

For SW2 and SW1, the switch state in the "ON" position = 1.
