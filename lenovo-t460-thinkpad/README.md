# Flashing the BIOS to Disable Intel ME on Lenovo Laptop T460 (ThinkPad)

Recently, I decided to flash the BIOS of my Lenovo ThinkPad T460 with a free or open-source implementation to disable Intel's backdoor as much as possible. After some research, I found that this specific model is not supported by Libreboot.

You can check if your laptop is supported here: [Libreboot Supported Hardware](https://libreboot.org/docs/install/#thinkpads).

**Note**: There are many detailed guides available online. This document serves as a reference for anyone planning to flash their laptop's BIOS.

**Computer details**:
- **Model**: Lenovo ThinkPad T460
- **Motherboard**: Lenovo BT462 NM-A581 Rev:3
- **Chip**: Macronix MXIC MX 25L12873F

**Summary**:
- [Chip location](#chip-location)
- [Flashing Setup](#flashing-setup)
- [Flashing Process](#flashing-process)
- [BIOS Dumps](#bios-dumps)

---

## Chip Location

Fortunately, this model has the BIOS chip located on the back of the motherboard. I only had to remove the battery and all visible screws. The motherboard model is in the center, and slightly to the right, near the heatsink fan, is the BIOS chip.

![Chip location](assets/lenovo-t460-laptop-thinkpad-back_600x400.webp)

![BIOS Chip](assets/macronix-MXIC-MX-25L12873F_300x300.webp)

---

## Flashing Setup

I already had a Raspberry Pi at home, so I only needed to acquire a few extra components. You can find them at affordable prices on AliExpress with a quick search:

- **Raspberry Pi 3 Model B Rev 2** (around 45.00 USD)
- **Male-to-Female 10cm / 40-Pin Cable** (around 2.00 USD)
- **SOIC8/SOP8 Clip** (around 3.70 USD)

Here are some reference images to help you identify the components and setup:

![Male-to-Female Cable](assets/male-to-female-10-cm-40pin-cable_300x300.webp)

![SOIC8/SOP8 Clip](assets/soic8-sop8-clip_300x300.webp)

Documentation: [MX25L12873F Datasheet](assets/MX25L12873F.pdf) (Page 7)

![Pin Configuration](assets/macronix-MXIC-MX-25L12873F-pin-configuration.webp)

![Color Pin Configuration](assets/color-pin-connections_1000x600.webp)

![Raspberry Pi Pin Configuration](assets/raspberry-pi-pin-configuration_600x600.webp)

![SOIC8/SOP8 Clip Plugged In](assets/soic8-sop8-clip_plugged_600x600.webp)

---

## Flashing process

### Backup the current ROM

Back up the original ROM to a file:

```bash
$ raspberry@raspberrypi:~ $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c "MX25L12833F/MX25L12835F/MX25L12845E/MX25L12865E/MX25L12873F" -r original_dump.bin
# flashrom unknown on Linux 6.6.51+rpt-rpi-v7 (armv7l)
# flashrom is free software, get the source code at https://flashrom.org

# Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
# Found Macronix flash chip "MX25L12833F/MX25L12835F/MX25L12845E/MX25L12865E/MX25L12873F" (16384 kB, SPI) on linux_spi.
# ===
# This flash part has status UNTESTED for operations: WP
# The test status of this chip may have been updated in the latest development
# version of flashrom. If you are running the latest development version,
# please email a report to flashrom@flashrom.org if any of the above operations
# work correctly for you with this flash chip. Please include the flashrom log
# file for all operations you tested (see the man page for details), and mention
# which mainboard or programmer you tested in the subject line.
# Thanks for your help!
# Reading flash... done.
```

Do it twice:

```bash
$ raspberry@raspberrypi:~ $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c "MX25L12833F/MX25L12835F/MX25L12845E/MX25L12865E/MX25L12873F" -r original_dump_2.bin
# flashrom unknown on Linux 6.6.51+rpt-rpi-v7 (armv7l)
# flashrom is free software, get the source code at https://flashrom.org

# Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
# Found Macronix flash chip "MX25L12833F/MX25L12835F/MX25L12845E/MX25L12865E/MX25L12873F" (16384 kB, SPI) on linux_spi.
# ===
# This flash part has status UNTESTED for operations: WP
# The test status of this chip may have been updated in the latest development
# version of flashrom. If you are running the latest development version,
# please email a report to flashrom@flashrom.org if any of the above operations
# work correctly for you with this flash chip. Please include the flashrom log
# file for all operations you tested (see the man page for details), and mention
# which mainboard or programmer you tested in the subject line.
# Thanks for your help!
# Reading flash... done.
```

Compare both dumps. If the `diff` command returns no output, it means the two files are identical. This indicates that the backup process was successful, and the original ROM has been accurately copied.

```bash
$ raspberry@raspberrypi:~ $ ls
# Bookshelf  Desktop  Documents  Downloads Music  original_dump_2.bin  original_dump.bin  Pictures  Public  Templates  Videos
$ raspberry@raspberrypi:~ $ diff original_dump.bin original_dump_2.bin 
```

### Applying ME cleaner

From this point on, it is best to follow the official ME Cleaner guide for [internal flashing with OEM firmware](https://github.com/corna/me_cleaner/wiki/Internal-flashing-with-OEM-firmware).


You can use the following outputs as a reference:

```bash
$ raspberry@raspberrypi:~/coreboot/util/ifdtool $ ./ifdtool -d ~/original_dump.bin 
# Warning: No platform specified. Output may be incomplete
# File /home/raspberry/original_dump.bin is 16777216 bytes
# ICH Revision: Unknown PCH
# FLMAP0:    0x00040003
#   NR:      0
#   FRBA:    0x40
#   NC:      1
# ...
# ...
# If you are interested in this output, you can find it in ifdtool_output.txt.
```

Writing down the modified BIOS.

```bash
$ raspberry@raspberrypi:~/me_cleaner $ flashrom -p linux_spi:dev=/dev/spidev0.0,spispeed=10000 -c "MX25L12833F/MX25L12835F/MX25L12845E/MX25L12865E/MX25L12873F" -w ~/modified_image.bin
# flashrom unknown on Linux 6.6.51+rpt-rpi-v7 (armv7l)
# flashrom is free software, get the source code at https://flashrom.org
#
# Using clock_gettime for delay loops (clk_id: 1, resolution: 1ns).
# Found Macronix flash chip "MX25L12833F/MX25L12835F/MX25L12845E/MX25L12865E/MX25L12873F" (16384 kB, SPI) on linux_spi.
# ===
# This flash part has status UNTESTED for operations: WP
# The test status of this chip may have been updated in the latest development
# version of flashrom. If you are running the latest development version,
# please email a report to flashrom@flashrom.org if any of the above operations
# work correctly for you with this flash chip. Please include the flashrom log
# file for all operations you tested (see the man page for details), and mention
# which mainboard or programmer you tested in the subject line.
# Thanks for your help!
# Reading old flash chip contents... done.
# Erasing and writing flash chip... Erase/write done.
# Verifying flash... VERIFIED
```

---

## BIOS Dumps

In case you find this useful, I'll leave this here.

| File                                  | Motherboard | Chip |
| ------------------------------------- | ------------| -----|
| [original_dump.bin](assets/original_dump.bin) | Lenovo BT 462 NM-A581 Rev:3 | MX25L12873F |
| [modified_dump.bin](assets/original_dump.bin) | Lenovo BT 462 NM-A581 Rev:3 | MX25L12873F |
