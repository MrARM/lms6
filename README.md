# LMS6-Radiosonde OFW modification

This repo serves as some research to modify the hex dump of the official firmware of the LMS6.

**DISCLAIMER:** It's not known if hex dumps from other radiosondes will run fine on your radiosonde. I still haven't fully understood all of the differences between the firmware, BUT likely the differences are calibration data and serial number information.

## Dumping/Programming Tips
To connect to the LMS6, I recommend using [Reid's Interface Board](https://github.com/Reid-n0rc/LMS-6_Interface_Board), you only need to add the ST7 Port and the card edge connector.
You'll need to use an Rlink programmer with RFlasher7. Select **ST72324J6** for the chip. If you encounter error 11 when writing, make sure you're using a power supply and erase the chip before writing.

To convert the hexdump to a binary to use many tools, run the following commands
* Hex->Bin - `objcopy -I ihex -O binary lms6_xxxxxxx.hex lms6_xxxxxxx.bin`
* Bin->Hex - `objcopy -I binary -O ihex lms6_xxxxxxx.bin lms6_xxxxxxx.hex`

## Repo structure
* `dumps/` - ROM dumps from LMS6 boards, file names should be named `lms6_<SERIAL>.hex/bin`. Firmware is v1.45 unless otherwise noted.

## Modification list
These are known modifications you can make to the hexdump.

For ease of use with programs like hex editors, all modifications will be written from the perspective of a hex editor. You could just modify the hex file, but it's just easier this way.

### Change the serial number
You can change the serial number to any 7 digit number

* First, you'll need to get the 3 hex bytes which we'll change in the dump, I'd recommend using [RapidTables' Decimal to Hexidecimal Converter](https://www.rapidtables.com/convert/number/decimal-to-hex.html). Type in a 7 digit number and you'll get back a hex number to modify. For example, `1234567` is `12D687`.

* In the file, go to `0xe003`, you'll replace the 3 bits, you'll see it surrounded by `00`, don't replace those.

If done correctly, your radiosonde will now have a different serial number. I would not change this number when you aren't testing, you don't want collisions with other radiosondes.

### Change the TX frequency
It is possible to change the TX frequency by modifying what is sent to the cc1050(radio) register. 

I'd like to give a huge thanks to [rsavxc](https://github.com/rsaxvc) for providing a Ida db and a reference sheet on the frequency register, here's the respective links for this information.

* [Ida db](https://github.com/rsaxvc/LMS6ReverseEngineering)
* [CC1050 register calculator](https://github.com/rsaxvc/LMS6APRS/blob/master/docs/cc1050%20frequency%20calculator.ods)

Here's what I've found so far in reguards to changing the frequency.

* The dip switches correspond to a frequency register, also shown in the cc1050 calculator linked above.

* Dip 1 is located at `0x9cdb`
  ![image](https://user-images.githubusercontent.com/8205849/139383052-db421557-14a8-4b0e-9e23-3f56b16e1660.png)
  `03`, `05`, and `07` seem to separate the 3 values. There also seems to be 6 bytes between each register, which I think are related to the FSEP and REFDIV as mentioned on the calculator.
  
* Make sure you get these values right, adjust the FSEP and REFDIV to the respective values(I used 10 and 55 to get it working). Try to get your error Hz around the same ones the factory frequencies have.
![image](https://user-images.githubusercontent.com/8205849/141700639-ef9a757b-2b4a-4b09-a353-f46109d3f80b.png)

Copy the generated number under "FREQ REG Calc" and paste it into column D, and adjust it until it gives a small error, you'll take that number and convert it into hex.  

I made a modified firmware in the mods/ directory using the frequency given in the screenshot if you want to compare or even try and flash it on your own radiosonde. I've noticed some weird issues if you mess up the calculations on this number, such as the bandwidth increasing or other funky business.

TL;DR if you want a ham band frequency and don't care to try to calculate things yourself, replace `03 44 05 AF 07 FE` with `03 47 05 81 07 91` on `0x9D8E` and flip all the dip switches on.

This is what you'll find when you use those numbers, it's hovering around 422.512 MHz
![image](https://user-images.githubusercontent.com/8205849/141700884-96c0c545-e420-4205-812c-275a9549b6a1.png)
