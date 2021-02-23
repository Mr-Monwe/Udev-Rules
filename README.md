# Udev-Rules

## How to identify the device (Arduino)

- Unplug all other USB devices that are similar to the arduino or that are not in use.

- You can leave your Keyboard and mouse plugged in.

*Make sure you are not using a USB hub to connect the Arduino or device.*

**Run the command:**
```
ls -l /dev/tty*
```

****The result should look like this:****
``` 
crw-rw---- 1 root dialout   4, 69 Feb 19 08:07 /dev/ttyS5
crw-rw---- 1 root dialout   4, 70 Feb 19 08:07 /dev/ttyS6
crw-rw---- 1 root dialout   4, 71 Feb 19 08:07 /dev/ttyS7
crw-rw---- 1 root dialout   4, 72 Feb 19 08:07 /dev/ttyS8
crw-rw---- 1 root dialout   4, 73 Feb 19 08:07 /dev/ttyS9
crw-rw---- 1 root plugdev 188,  0 Feb 20 19:02 /dev/ttyUSB0 
```

- If you see a `/dev/ttyUSB` followed by a number or a `/dev/ttyACM` followed by a number, that is your device.
In this case my was ` /dev/ttyUSB0 `

## How to identify the Vendor & Product ID

**Run the command:**
```
lsusb
```

**The result should look like this:**
``` 
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub 
Bus 001 Device 027: ID 1a86:7523 QinHeng Electronics HL-340 USB-Serial adapter 
Bus 001 Device 003: ID 062a:5918 Creative Labs 
Bus 001 Device 006: ID 8087:0aaa Intel Corp. 
Bus 001 Device 022: ID 413c:2113 Dell Computer Corp. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
 This process is the hardest part as you have to identify which of the items on the list are your device.

* If you are using a genuine Arduino it will be listed as such but if you have a clone or cheapy version you will see a **340 series usb serial adapter(CH340, HL340)** or you may see **nothing listed after the numbers**.

* If you still cant tell then just unplug the device, run the command, replug and run again and you will identify the missing device that way. 

 * In my case it was `Bus 001 Device 027: ID 1a86:7523 QinHeng Electronics HL-340 USB-Serial adapter`

 * Now from this output we know we locate the vendor & product ID.

 * For the device listed above the vendor ID is `1a86` and the product ID is `7523`.

 * Save those two numbers somewhere.

 ## Creating the rule 

**Run the command:**
```
cd /etc/udev/rules.d && ls
```
* This will take you to the rules directory and list all the available files.

**The output should look something like this:**

```
70-snap.arduino.rules               70-snap.qalculate.rules
70-snap.flameshot.rules             70-snap.snapd.rules
70-snap.gnome-characters.rules      70-snap.spotify.rules
70-snap.gnome-logs.rules            70-snap.vlc.rules
```
*Now follow the naming convention of the files in this case(70-XXXXX.rules).*

**Run the command:**
```
sudo gedit 70-new-arduino.rules
```
* Put in your password and a new file will appear.

**In the new file place the following contents and replace the vendor and product ID's:**

```
SUBSYSTEM=="tty", GROUP="plugdev". MODE="0660"

SUBSYSTEMS=="usb", ATTRS{idProduct}=="7523", ATTRS{idVendor}=="1a86", SYMLINK+="arduino"
``` 
*You can add as many devices as you like, then save & exit the file.*

**Run the command:**
```
sudo udevadm control --reload-rules && sudo service udev restart && sudo udevadm trigger
```
*Once this is completed we have the symlink and better permissions for the arduino or device.*

***Restart** the computer and proceed to the final steps.*

**Run the command:**
```
ls -l /dev/arduino /dev/ttyUSB0
```

**The result should look like this:**
```
lrwxrwxrwx 1 root root        15 Feb 20 19:02 /dev/arduino -> bus/usb/001/027
crw-rw---- 1 root plugdev 188, 0 Feb 20 19:02 /dev/ttyUSB0
```

Finally unplug and replug the device and run the command again and the device should come up.

You can now use `dev/ttyUSB0` for whatever your want without worrying about the device flip flopping. 