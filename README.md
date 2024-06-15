# munifying on Windows 11 (the easy way, not the PP way)

Many users have complained about the difficulty of finding RF addresses as well as downgrading the dongle to a lower vulnerable firmware revision if needed.  The following is the easiest way to find paired RF addresses for a Unifying dongle and perform downgrade operations through Windows.

# Requirements
- Relatively up to date Windows 11 (possibly 10 but untested) installation
- Newest usbipd-win installer from here: https://github.com/dorssel/usbipd-win/releases (tested with 4.0.0)
- Ubuntu is not installed in WSL previously (or has been removed and unregistered)

# Summary
Essentially you will do the following:
- Install Ubuntu Linux via Powershell
- Install usbipd-win
- Grab and compile munifying
- Find RF addresses and downgrade FW if needed

# Steps - Part 1 - Windows Setup

1) In Windows, install the usbipd-win msi mentioned in Requirements and reboot.
2) Once rebooted, in Windows, run the following command in Powershell: ```wsl --install```
3) Follow the prompts and create a username and password.  Do not forget them.
4) If you'd like to confirm that Ubuntu is utilizing WSL2, in a different Powershell window run: ```wsl -l -v```
6) You should now be at the Ubuntu terminal with ```username@pcname:~$``` displayed.

# Steps - Part 2 - Ubuntu Setup
1) Run ```sudo apt-get update -y``` and enter your password
2) Run ```sudo apt-get install git golang -y```
3) Clone the munifying repository: ```git clone https://github.com/RoganDawes/munifying.git```
4) Move inside the new directory: ```cd munifying/```
5) Install the munifying pre-requisities: ```./install_libusb.sh```
6) Compile munifying: ```go build```
7) Make the munifying binary executable: ```chmod +x munifying```
8) Test if munifying runs via ```./munifying info``` - you should see ```FATA[0000] No known dongle found```

# Steps - Part 3 - Attaching the dongle via Windows
1) Plug in your dongle if you haven't already
2) Open a new Powershell window and find the BUSID of your dongle: ```usbipd list```
3) Attach it to Ubuntu with ```usbipd attach --wsl --busid=<BUSID>```

# Steps - Part 4 - Dongle Information within Ubuntu
1) Run ```sudo ./munifying info``` to get information on your dongle - you should see a "Dongle Info" area and (possibly) some Device Info for a keyboard/mouse.  If you are shown a paired keyboard or mouse, you can utilize this RF address for your attacks if you are on a vulnerable FW.


# Flashing Process

# Steps - Part 1 - Flash Vulnerable Firmware - Ubuntu
1) Grab the vulnerable firmware of your choice
2) Begin to flash the dongle with ```sudo ./munifying flash -f firmware.hex``` NOTE: THIS WILL FAIL AFTER PUTTING THE DEVICE INTO BOOTLOADER MODE AS THE VID:PID WILL BE CHANGED!
3) IN WINDOWS: re-attach it with ```usbipd attach --wsl --busid=<BUSID>```  Use the previous BUSID.
4) Re-run ```sudo ./munifying flash -f firmware.hex``` - You should see a long scrolling list of Flash write succeeded.  The dongle will be placed back into runtime mode.
5) IN WINDOWS: re-attach it with ```usbipd attach --wsl --busid=<BUSID>```  Use the previous BUSID.
6) Run ```sudo ./munifying info``` to check your firmware version.  Some dongles will not be able to downgrade if they have a secured bootloader.

# Detach the dongle from Ubuntu
1) ```usbipd detach --all```

# Troubleshooting
- If your .vdhx fails to attach, WSL will need to be cleaned up with ```wsl --unregister ubuntu``` and then ```wsl --install``` again.
