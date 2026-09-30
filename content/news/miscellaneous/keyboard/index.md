+++
date = '2026-09-30T01:34:13+02:00'
draft = false
title = 'Remapping a key on a USB keyboard under Linux'
summary = 'How to customize a USB keyboard under Linux' 
+++

## Informations 

Hello ! This tutorial is taken from [here](https://www.davidrevoy.com/article989/how-to-customise-a-usb-numeric-keypad-under-gnulinux/). My customization is a little bit different but uses the same process. The purpose here is to change one keyboard key on my japanese USB keyboard and to have the whole process easily accessible in case I need to use it again. To use 'esc' I have to use the 'FN' keyboard key, which I find a little bit annoying. Therefore I decided to change this following the tutorial at the previous shown link. 

## Let's begin

###  Find the name of the keyboard

Let's first use the `lsusb` command that list the USB devices. 

```text 
vachermatteo@BrokenThinkpad:~$ lsusb
...
Bus 001 Device 008: ID 320f:5000 Evision RGB Keyboard
...
```

### Download && use the 'evtest' package 

Let's install the 'evtest' package, that shows what the USB device is sending to the Linux system, with `sudo dnf install evtest`. 

Let's run the evtest command `sudo evtest` : 

```text 
vachermatteo@BrokenThinkpad:~$ sudo evtest
No device specified, trying to scan all of /dev/input/event*
Available devices:
...
/dev/input/event4:      Evision RGB Keyboard
/dev/input/event7:      Evision RGB Keyboard
/dev/input/event8:      Evision RGB Keyboard
/dev/input/event9:      Evision RGB Keyboard Mouse
...
Select the device event number [0-20]: 4
```

Here is the USB device associated to my USB keyboard, I select the device number `4`. When I press the 'esc' key without the 'FN' key here is what happen :

```text 
Event: time 1790726525.129582, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70035
Event: time 1790726525.129582, type 1 (EV_KEY), code 41 (KEY_GRAVE), value 1
Event: time 1790726525.129582, -------------- SYN_REPORT ------------
Event: time 1790726525.272481, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70035
Event: time 1790726525.272481, type 1 (EV_KEY), code 41 (KEY_GRAVE), value 0
Event: time 1790726525.272481, -------------- SYN_REPORT ------------
```

### Customize keys 

Now let's create a new file that will indicate what this key does at `/etc/udev/hwdb.d/95-custom-reddragon-keypad.hwdb` by using an editor to modify the file `sudo vim 95-custom-reddragon-keypad.hwdb` : 

For the filename, it has to begin by a two digit number e.g. `95-`, followed by a custom name `custom-reddragon-keypad` (even if it is not a keypad), and the extension `.hwdb` which is mandatory. 

Then I continue with the content of the file : 

```text 
evdev:input:b0003v320Fp5000*
 KEYBOARD_KEY_70035=0x01 # japanese e/j => esc
```

In this file, `b0003` stands for USB device, `320F` and `5000` can be found in the ID column of the USB device line when using the `lsusb` command, but the letters have to be in capital letter and the first 4 characters have to be after a `v` and the last 4 after a `p`, then the `*` to match anything after. 

Then the content of the file itself uses KEYBOARD_KEY_<scancode>=<target_keycode> where the scancode is to be found in the MSC_SCAN value of the `evtest` command and the target_keycode [here](https://kbdlayout.info/KBDUS/scancodes) and written under the `0x` + `value` form, here `value` being `01`. 

Finally save the file and type `sudo systemd-hwdb update && sleep 2 && sudo udevadm trigger` to refresh the system (I totally don't know what this line is doing). 

### Check (just to be sure)

Then check again with `sudo evtest` again. 

Here is what I have now, so everything seems to be perfect : 

```text 
Event: time 1790730555.953743, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70035
Event: time 1790730555.953743, type 1 (EV_KEY), code 1 (KEY_ESC), value 1
Event: time 1790730555.953743, -------------- SYN_REPORT ------------
^[Event: time 1790730556.062743, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70035
Event: time 1790730556.062743, type 1 (EV_KEY), code 1 (KEY_ESC), value 0
Event: time 1790730556.062743, -------------- SYN_REPORT ------------
```

## Conclusion 

I have now an 'esc' key that works without using the 'FN' key and will be really much appreciated while using VIM. 






