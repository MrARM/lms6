# LMS6-Radiosonde OFW modification

This repo serves as some research to modify the hex dump of the official firmware of the LMS6.

**DISCLAIMER:** It's not known if hex dumps from other radiosondes will run fine on your radiosonde. I still haven't fully understood all of the differences between the firmware, BUT likely the differences are calibration data and serial number information.

## Dumping/Programming Tips
To connect to the LMS6, I recommend using [Reid's Interface Board(https://github.com/Reid-n0rc/LMS-6_Interface_Board), you only need to add the ST7 Port and the card edge connector.
You'll need to use an Rlink programmer with RFlasher7. Select **ST72324J6** for the chip. If you encounter error 11 when writing, make sure you're using a power supply and erase the chip before writing.

To convert the hexdump to a binary to use many tools, run the following commands
* Hex->Bin - `objcopy -I ihex -O binary lms6_xxxxxxx.hex lms6_xxxxxxx.bin`
* Bin->Hex - `objcopy -I binary -O ihex lms6_xxxxxxx.bin lms6_xxxxxxx.hex`

## Repo structure
* `dumps/` - ROM dumps from LMS6 boards, file names should be named `lms6_<SERIAL>.hex/bin`. Firmware is v1.45 unless otherwise noted.

## Modification list
These are known modifications you can make to the hexdump.

For ease of use with programs like hex editors, all modifications will be written from the perspective of my hex editor. You could just modify the hex file, but it's just easier this way.

### Change the serial number
You can change the serial number to any 7 digit number

* First, you'll need to get the 3 hex bytes which we'll change in the dump, I'd recommend using [RapidTables' Decimal to Hexidecimal Converter](https://www.rapidtables.com/convert/number/decimal-to-hex.html). Type in a 7 digit number and you'll get back a hex number to modify. For example, `1234567` is `12D687`.

* In the file, go to `0xe003`, you'll replace the 3 bits, you'll see it surrounded by `00`, don't replace those.

If done correctly, you're radiosonde will now have a different serial number. I would not change this number when you aren't testing, you don't want collisions.

