# 1013D Alternative Firmware & FPGA Upgrade Guide

There are two primary open-source firmware alternatives available for the FNIRSI 1013D oscilloscope:

* [**Atlan-Peco (`Atlan4/Fnirsi1013D`)**](https://github.com/Atlan4/Fnirsi1013D) – Feature-rich alternative firmware with custom UI, DSP enhancements, and support for all hardware variants.

* [**fnirsi-1013d-mcp (`mikljohansson/fnirsi-1013d-mcp`)**](https://github.com/mikljohansson/fnirsi-1013d-mcp) – Open firmware, custom FPGA bitstream, and USB MCP server allowing AI agents to control and read measurements from the scope (requires the **Anlogic AL3A10** FPGA variant).

On Windows, the entire installation process can be performed using GUI tools—no command-line utilities are required.

## 1. Backup Your Stock SD Card

Before making any modifications, create a full block-level backup image of your stock internal MicroSD card using a disk imaging utility such as [Win32 Disk Imager](https://win32diskimager.org/).

> **Tip:** Consider using a spare MicroSD card (up to 32 GB) for custom firmware so your original factory card remains untouched.
> **Tip**: Card image is highly compressable, it is mostly empty.

## 2. Installing Atlan-Peco Firmware (From Stock FW/FPGA)

1. Download [**`loader_v1.1.3.zip`**](https://github.com/Atlan4/Fnirsi1013D/blob/main/Guide%20to%20firmware/loader_v1.1.3.zip) and the **`v0.28a` firmware binary** ([`fnirsi_1013d_scope.bin`](https://github.com/Atlan4/Fnirsi1013D/blob/main/Scope%20v0.28a/dist/Debug/GNU_ARM-Linux/fnirsi_1013d_scope.bin)) from the [Atlan4 repository](https://github.com/Atlan4/Fnirsi1013D).

2. Connect the oscilloscope to your PC in USB mode (or insert the internal MicroSD card into a card reader) and use `FNIRSI_Firmware_Loader.exe` to write `fnirsi_1013d_scope.bin` to the card.

3. The **`v0.28a` branch** (along with all `0.x.x` versions) runs with the **stock factory FPGA** and demonstrates what is possible with open-source firmware.

4. Installing this package also installs **Bootloader `v0.8`** and **Updater `v0.02q`**, which are required for future menu-based updates or migrating to `fnirsi-1013d-mcp`.

### Limitations on Stock FPGA

* **Signal Generator Menu:** Will not function (requires a custom FPGA bitstream).

* **Automatic DC Shift:** Disabled by default. Use **Auto Set** or adjust vertical offsets manually in DC mode to prevent signal clipping around **\~4.2V**.

> **Note:** Atlan's [`v1.x.x` versions](https://github.com/Atlan4/Fnirsi1013D/tree/main/Verzia%201.xxx) require a matched custom FPGA bitstream and will not operate properly on stock FPGA logic.

## 3. Migrating from Atlan-Peco to `fnirsi-1013d-mcp`

> **Hardware Requirement:** `fnirsi-1013d-mcp` requires an [oscilloscope](https://github.com/mikljohansson/fnirsi-1013d-mcp/blob/main/docs/photos/board-al3a10-programmer.jpg) equipped with the **Anlogic AL3A10** FPGA and its matching custom FPGA bitstream (`0x153F`). On stock FPGA logic, the firmware will only display a startup banner.

### Step A: Update the Scope Firmware via SD Card

1. Download the latest `fnirsi_1013d-fw-x.x.x.bin` binary from the [fnirsi-1013d-mcp Releases](https://github.com/mikljohansson/fnirsi-1013d-mcp/releases) page.

2. Rename the downloaded file to `scope vx.x.x.bin` (matching the naming format expected by the Atlan updater).

3. Copy the renamed file into the `firmware Atlan & PECO` directory on the oscilloscope's SD card.

4. Boot the oscilloscope into the **Updater** menu and flash the DSO firmware using the renamed file.

### Step B: Update the FPGA Bitstream (Anlogic AL3 Variant Only)

#### 1. Wiring the SPI Programmer

1. Open the back cover of the oscilloscope and locate the **25Q80** (actual part may vary slightly) 8-pin SPI flash chip next to the unmarked 144-pin Anlogic AL3 FPGA, along with its adjacent **6-pin (`2x3`) header footprint**.

2. Connect a 3.3V SPI programmer—such as an **FT232H / UM232H** module—to the 6-pin header. On a UM232H module, jumper `5V0` to `USB` and `VIO` to `3V3`:

| Signal | UM232H Pin | 1013D Header Pin | 
| ----- | ----- | ----- | 
| **CS** | `AD3` | `1` | 
| **MISO** | `AD2` | `2` | 
| **CLK** | `AD0` | `3` | 
| **MOSI** | `AD1` | `4` | 
| **3.3V** | *Do Not Connect* | `5` | 
| **GND** | `GND` | `6` | 

> **Important Wiring Warnings:**
>
> * **Do NOT connect Pin 5 (`3.3V`):** Despite the onboard protection diode, powering the flash chip from the programmer causes read/write failures. Let the oscilloscope's own battery power the board.
>
> * **No soldering required:** You can lightly [angle/press](solderless_flash_connection.png) male jumper pins into the unpopulated through-holes, but keep the wires as short as possible for signal integrity.
>
> * **Avoid 5V programmers:** Do not use unmodified 5V CH341A programmers, as 5V logic levels can damage the `25Q80` flash chip.

#### 2. Backing Up & Flashing with AsProgrammer

1. Launch [**AsProgrammer**](https://github.com/nofeletru/UsbAsp-flash) on your PC.

2. Power on the oscilloscope and wait for it to finish booting.

3. Plug the SPI programmer into your PC's USB port.

4. In AsProgrammer, click **Read ID**. If prompted with multiple matching chips, select the actual chip installed on the board.

5. **Back up the stock FPGA flash:** Read the flash contents twice, save both dumps, and compare them to verify a clean, identical backup. *(If read errors occur, lower the SPI clock speed to **6 MHz** or lower).* Compare to Atlan's stock [backup](https://github.com/Atlan4/Fnirsi1013D/blob/main/Verzia%201.xxx/FPGA%20for%201.xxx/backup%20fpga/fpga-1013D-1-zb25vq80.bin) too.

6. Download the matching FPGA bitstream (e.g., `fnirsi_1013d-fpga-al3-0x153f-flash1m.bin`) from the [fnirsi-1013d-mcp Releases](https://github.com/mikljohansson/fnirsi-1013d-mcp/releases) page.

7. Load the `.bin` file in AsProgrammer and flash it to the chip using **Unprotect → Erase → Program → Verify**.

8. If automatic verification was not used, read the flash contents back and verify they match the downloaded bitstream file.

#### 3. Finalize & Reassemble

1. Restart the oscilloscope and confirm it boots cleanly into the `fnirsi-1013d-mcp` firmware and prompts for the first-run calibration wizard.

2. Power down the unit, disconnect the programmer leads, and reassemble the case.
