# macOS Development Guide for Bouffalo BL602

This guide details how to set up the development environment, compile firmware, and flash it to a Pinecone BL602 board on macOS.

## 1. Prerequisites

Ensure you have Homebrew installed. Then, install the required tools:

### Install CMake

```bash
brew install cmake
```

### Install RISC-V Toolchain

We use the standard RISC-V toolchain provided by Homebrew.

```bash
brew tap riscv-software-src/riscv
brew install riscv-tools
```

## 2. SDK Setup & Patching

The standard RISC-V toolchain is slightly different from the one originally used by Bouffalo Lab. We need to apply a small patch to the SDK to make it compatible.

### Patch `start.S`

Open the file `drivers/soc/bl602/std/startup/start.S` and locate the following lines (around line 55):

```asm
    /* Intial the mtvt, MUST BE 64 bytes aligned*/
    .weak __Vectors
    la t0, __Vectors
    csrw mtvt, t0
```

Change `csrw mtvt, t0` to `csrw 0x307, t0`. The `mtvt` CSR is not recognized by name in the standard toolchain, so we use its address.

**Modified code:**

```asm
    /* Intial the mtvt, MUST BE 64 bytes aligned*/
    .weak __Vectors
    la t0, __Vectors
    csrw 0x307, t0
```

## 3. Environment Configuration

You must set the `BL_SDK_BASE` environment variable to the path of your `bouffalo_sdk` directory.

Run this in your terminal (or add it to `~/.zshrc` for persistence):

```bash
export BL_SDK_BASE=$(pwd)/bouffalo_sdk
```

_(Adjust the path if you are not in the parent directory of `bouffalo_sdk`)_

## 4. Compiling Firmware

Navigate to an example project and run `make`.

**Example: Hello World**

```bash
cd $BL_SDK_BASE/examples/helloworld
make CHIP=bl602 BOARD=bl602dk
```

If successful, the binary will be generated in `build/build_out/helloworld_bl602.bin`.

## 5. Flashing Firmware

### Enter Bootloader Mode

1.  Connect the BL602 board to your Mac via USB.
2.  Press and **hold** the **BOOT** button.
3.  Press and **release** the **RESET** button.
4.  **Release** the **BOOT** button.

### Find Serial Port

List available serial devices:

```bash
ls /dev/tty.*
```

Look for a device named like `/dev/tty.usbserial-XXXX` or `/dev/tty.usbmodemXXXX`.

### Flash Command

Run the flash command from the example directory:

```bash
make flash CHIP=bl602 BOARD=bl602dk COMX=/dev/tty.usbserial-110
```

_(Replace `/dev/tty.usbserial-110` with your actual port name)_

## 6. Monitoring Output

To see the serial output (printf debugging):

1.  Press the **RESET** button on the board (without holding BOOT) to start the application.
2.  Use `screen` to open the serial port at 115200 baud:

```bash
screen /dev/tty.usbserial-110 115200
```

_Note: The default baud rate in the SDK is 2,000,000. We have modified `bsp/board/bl602dk/board.c` to use 115200 for better compatibility._

**To exit screen:** Press `Ctrl + A`, then `K`, then `Y`.
