# Homebrew 6502
This is a single-board homebrew 6502 computer. It is intended to be usable as a standable computer with a full set of IO devices. A primary design goal is to build a period-accurate retro system with capabilities inline with systems such as the Commodore 64 and the Nintendo Entertainment System.

## Planned Specs
- Unified USB-C power and MCU firmware flashing.
- 65C02 clocked at 14 MHz.
- 1-2x 65C22 Versatile Interface Adapter for IO devices.
- 1x 65C52 Asynchronous Communications Interface Adapter for serial UART communication.
- 128 KB ROM split into 8x 16 KB banks. 2x banks mapped at a time.
- 128 KB RAM split into 8x 16 KB banks. 2x banks mapped at a time.
- microSD card via 65C22 bitbanged SPI.
- SPI Real Time Clock via 65C22.
- PS/2 keyboard input via 65C22.
- RP2040 MCU for VGA signal generation (red, green, blue, pixel clock, HSYNC, VSYNC) and ROM flashing.
- 640 x 480px VGA signal with 8-bit color depth. 8 KB of MCU-internal VRAM accessible by 65C02 system bus at a time.
- Hardware RST and NMI buttons.
- Programmable digital GPIO for expansion.

### Unknown
- Sound output? Potentially via the second core of the RP2040.

### Maybe?
- An external I2C bus on the expansion port - either bitbanged directly, or by bitbanging an SPI-based I2C controller.

### microSD
System storage will be facilitated by an onboard microSD card, which can be interacted with via SPI banging through the onboard VIAs.

Maybe: Onboard ROM will contain *Kernal*-like BIOS calls that can perform raw storage interaction.

Maybe: Onboard ROM will contain a simple FAT-32 filesystem driver.

### Video System
The RP2040 will produce the 640 x 480 video signal. Its internal RAM will contain the framebuffer. The framebuffer will present to the 65C02 in 8 KB blocks in the RAM region of the system memory map.

Rather than a full bitmapped framebuffer, the system will use a tilemap system like the NES. The 640 x 480 output will be divided into 8px x 8px tiles. Each tile can be drawn using one of 256x 4 bit per pixel bitmap images. Each tile can be colored using one of 256x 16 entry, 8 bit colored palettes.

The system will internally hold a 128 x 128 tile map which will allow for both vertical and horizontal tile map scrolling.

The system will be capable of render TBD number of sprites, which are configured to use one of the bitmap images and colored with one of the color palettes.

Besides VGA control registers and the sprite table, the total VRAM used by the system will be 44 KB. It will be accessible in 8 KB blocks, of which two can be mapped into the system memory map at a time.

- 128 x 128 tile map with 2 bytes per tile to specify the bitmap and tile, for a total of **32 KB**.
- 256x color palettes, each containing 16x 1 byte entries that represent 3 bits of red depth, 3 bits of green depth, and 2 bits of blue depth. A total of **4 KB** used.
- 256x 8x8, 4bpp bitmaps for a total of **8 KB**.

### Interrupts
Different IO devices will produce interrupts, which can collectively masked by the 65C02 "IRQ Enable" control flag.

- Configurable VSYNC and/or HSYNC scanline interrupts (2x total).
- 2x VIA timer interrupts per VIA.
- 1x ACIA serial input interrupts.
- Maybe: 1x VIA timer produced by internal shift register upon receipt of data from microSD card.
- Maybe: 1x IRQ via configurable external GPIO.

### RP2040 MCU
Despite being anachronistic, a RP2040 will be available on the system to provide video generation in a pragmatic manner. It will also allow any of thge 128 KBs of ROM to be programmed in-circuit.

The MCU firmware responsible for these capabilities will be flashable via the on-board USB-C port, which also powers the system.

Due to bandwidth restrictions of the RP2040, video generation is disabled when its firmware is being flashed or ROM is being programmed.

Maybe: The RP2040 will store copies of wozmon, BASIC, Forth, or other configurable (via firmware flashing) images in internal QSPI Flash memory. This allows for the system ROMs to be programmed without providing the images externally.

## Memory Map
The 65C02 expects RST, IRQ, and NMI interrupt vectors to be found in the last 6 bytes of the addressable range. This mandates ROM in high memory.

The 65C02 has a "zero-page" addressable region in the first 256 bytes of the addressable range for quick memory accesses. It also expects its stack to be located at `0x01xx`. This mandates RAM in low memory.

The memory map from `0x0000` to `0xffff` will be divided into 4x 16 KB pages. The low pages may contain RAM or VRAM & IO. The high pages may contain ROM.

TBD: Make ROM and RAM mappable into any of the 4x pages?
