---
layout: post
date: 2020-11-12 09:00:00 UTC
title: MacBook Air - Hacking EFI Boot Password
description: Additional notes on hardware hacking the boot firmware of an old MacBook Air.
---

# The Setup
Hooking up a bunch of wires to the motherboard of a MacBook isn't something I felt comfortable to do, especially to a machine of a friend.
The setup looks rather horrible, and I anxiously triple checked everything before applying any current, but it worked fine in the end. 
![](/res/mac-efi-hack/cover.png)

A quick summary of what this does:
* Fetch the firmware from a "bootloader" chip on the motherboard
* Manually erase the password protection with empty `FF` hex characters
* Write out the new firmware with the erased variables to the bootloader chip

# The Original Post

![](/res/mac-efi-hack/original.png)

I will not being going over the whole procedure, rather describe some of the modifications and troubles I ran into.
[https://blog.wzhang.me/2017/10/29/removing-mac-firmware-password-without-going-to-apple.html](https://blog.wzhang.me/2017/10/29/removing-mac-firmware-password-without-going-to-apple.html)

I was going to copy over some of the initial tutorial but decided against it, instead I made sure the internet archive had a copy.
Given that the above URL doesn't work anymore, you can always find it on [the internet archive](https://web.archive.org/web/20201120152234/https://blog.wzhang.me/2017/10/29/removing-mac-firmware-password-without-going-to-apple.html).


# Note 1: Enable SPI on the Raspberry Pi
The default behavior of the Raspberry Pi OS is to boot with SPI disabled.
It can easily be enabled by editing the boot config and rebooting.

```
pi@raspberrypi:~ $ sudo nano /boot/config.txt
```

Scroll down and you should see the following line commented out:
```
#dtparam=spi=on
```
Remove the `#` in front and `CTRL + X` + `Y` to exit and save.

Time to reboot!
```
pi@raspberrypi:~ $ sudo shutdown -a
```

You should now find the SPI devices under `/dev`.
```
pi@raspberrypi:~ $ cd /dev/
pi@raspberrypi:/dev $ ls | grep spi
spidev0.0 spidev0.1
```

# Note 2: Forcing a slower SPI Speed

If you use the `flashrom` command as described in the original post, then the Raspberry Pi will not detect your chip, there seems to be a bug.
```
flashrom -r read1.bin -c "MX25L3205D/MX25L3208D" -V -p linux_spi:dev=/dev/spidev0.0
```

Either it will say it has found a generic chip
```
Found Generic flash chip "unknown SPI chip (RDID)" (0 kB, SPI) on linux_spi.
```

Or it will proclaim to not have found one at all..

The solution is to lower the SPI Speed.
```
flashrom -r read1.bin -c "MX25L3205D/MX25L3208D" -V -p linux_spi:dev=/dev/spidev0.0,spispeed=512
```

# Note 3: Verify the chipset model
There are a few models out there in MacBooks, mine was using a different model than the original post and therefore I had to apply some changes.
You will have to adopt the parameters in your `flashrom` commands to accomodate for the difference.
![](/res/mac-efi-hack/MX25L3205D.png)

