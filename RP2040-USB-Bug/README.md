## Description of the Problem
This is a brief note to document an issue I have experienced with some 3D printer related controller boards that utilize the RP2040 chip from the Raspberry Pi Foundation. The issue manifests when one of these RP2040-based boards is plugged in via USB to a Raspberry Pi. The issue will occur when you reboot the Raspberry Pi without power cycling the RP2040 device. When the Raspberry Pi finishes booting, the RP2040 device will not be recognized. It is not clear to me if this issue impacts all RP2040, or it is only certain steppings of the chip.

I have experienced this problem with these RP2040 based solutions:
* The LDO Picobilical board. I have only one of these to test, but the problem consistently repros with this board.
* Seeed XIAO RP2040 boards - I made my own board that used one of these to allow connecting an ADXL345 accelerometer to the Raspberry Pi via a USB cable. Notably, I switched to another XIAO board, and the problem did not recur. This is what makes me suspect that the issue is only with certain RP2040 steppings.

## Reproducing the Problem with the LDO Picobilical
Go through the steps below to reproduce the problem when connected to the LDO board.
1. Go through the instructions from LDO to flash Klipper firmware onto the RP2040
2. Starting with the power to the printer (and the Raspberry Pi, aka Rpi) completely off, power on the printer.
3. After the printer boots, ssh into the Raspberry Pi
4. Install the `uhubctl` utility if it's not already there:
```
   sudo apt install uhubctl
```
5. Verify that you can see the Picobilical RP2040 device by running: `lsusb` . ![lsusb output](images/lsusb.png)
6. Run `sudo uhubctl` to see the current USB devices and to which hub and ports they are connected. You should see the RP2040 device there too. ![uhubctl output](images/uhubctl.png)
7. Reboot the Raspberry Pi without shutting off the power to the printer:
```
   sudo reboot
```
8. After the Raspberry Pi reboots, SSH back into it and run `lsusb` again. If you see the Picobilical RP2040 in the list, your Picobilical is not impacted by this bug. For me, however, the RP2040 no longer shows up. **Klipper fails to start because it can't find the Picobilical MCU.** 
   
   ![lsusb output after reboot](images/lsusb-after-reboot.png) 
   
9.  And here is an example of the error you will see from Klipper after rebooting the Raspberry Pi without power cycling the printer:

     ![Klipper error](images/klipper-error.png)

## Workaround for USB Bus Powered Devices
For RP2040 devices that are powered directly by USB, it is possible to work around the issue by using uhubctl to power cycle the RP2040. **Unfortunately, this won't work with the Picobilical, because its RP2040 gets its power indirectly from the printer's 24V PSU.**

To use the work around for other devices, you would note the hub and port as shown with a red underline in the `uhubctl` results shown above. Then run this command:
```
sudo uhubctl -a cycle -d 3 -w 1000 -R -l <hub> -p <port>
```
Replacing `<hub>` and `<port>` with values shown above, this would be:
```
sudo uhubctl -a cycle -d 3 -w 1000 -R -l 1-1 -p 2
```

## A More Universal Fix? ##

There is a [post](https://forums.raspberrypi.com/viewtopic.php?t=331479) about a workaround for this issue on the Raspberry Pi forums. They specifically mention a feature in the Pico SDK that can be enabled with a compiler definition. However, I have not seen any mention of this on the Klipper Discord or in their Github repo. 

