# Kenwood
Files related to Kenwood radios

VERSION 20220129

## 710.py and 710.sh

Two scripts that provide CAT control for Kenwood TM-D710G or TM-V71A radios on a Raspberry Pi. It requires a serial/USB cable between the radio and the Pi.  An RT Systems programming cable will work, as will a Kenwood PG-5G cable or equivalent. 

- `710.sh` is a shell script that you use in the Terminal.
- `710.py` is a Python application that allows you to control the radio through a GUI that emulates the TM-D710G's screen. It also works with the TM-V71A.

The `710.sh` script requires `710.py` to talk to the radio. It does not start the `710.py` GUI in this case. `710.py` uses the Python serial library to communicate with the radio. 

## Most recent significant changes

### Latest Versions

1. `710.py` now has a multithreaded XML-RPC server. This allows Fldigi to communicate with `710.py` as if `710.py` were Flrig. Apps using Hamlib can also use `710.py` via hamlib's 'FLRig' setting. Details below.

1. `710.sh` can interact with `710.py` as it always has, by calling it and passing commands via `710.py -c COMMAND`. Now, it can also interact via XML-RPC. This means that `710.sh` can be used while `710.py` is running.

1. PTT works via XML-RPC calls. When Fldigi makes an RPC call to `710.py` to activate PTT, for example, `710.py` will use the Pi's GPIO pins in the Nexus DR-X to control PTT. Details below

## CAVEATS

- Kenwood does not provide a CAT command to change the frequency band on a given side of the radio. You can only change the frequency band on the radio itself. This omission causes the following behaviors:

	1. When in VFO mode, you cannot select a frequency outside the frequency band set for that side. 

	1. If you modify the frequency, tone type, tone frequency, step, reverse or modulation using the GUI (`710.py`), the change will take place __*in the current mode for that side (VFO, CALL or MR)*__. If the change to one of these parameters is requested when in MR mode, __*that memory location will be modified*__! A warning message will appear in the message queue window telling the user that the memory location has been modified. 
	
	1. If you use the shell script (`710.sh`) to change the frequency, the script will first change the mode to VFO (if it's not already in that mode) and attempt to set the desired frequency. If the desired frequency is not in the currently set frequency band for that side, the frequency will not be changed. 

	The workaround for the inability to use CAT to change the frequency band is to set your commonly used frequencies in memory locations and use the scripts to put the radio in MR mode and set a certain memory channel. One easy way to program the radio's memories is to use `chirp`, which is available via the the Nexus Updater. `chirp` uses the same serial/USB cable as the `710` scripts. 
	
	Here are some tips for using `chirp`:
	
	1. Close the `710` script while running `chirp`. Only one program can access the serial port at a time.
	1. For the TM-D710G: Select __TM-D710G__ and not TM-D710G_CloneMode. Select __TM-V71__ for that radio.
	1. On the TM-D710G, all changes you make in `chirp` are immediately sent to the radio. There is no "update" or "send to radio" action to take.

- Unlike the radio's display, the GUI display will not display the TX frequency while PTT is active.

## Installation
Pick either Easy or Manual Installation.

### Easy Installation (for Nexus DR-X users)
- Make sure your Pi is connected to the Internet.
- Click __Raspberry > Hamradio > Update Pi and Ham Apps__.
- Check __710__, click __OK__.

### Manual Installation
- Make sure your Pi is connected to the Internet.
- Open a Terminal and run these commands:

		cd ~
		rm -rf kenwood/
		git clone https://github.com/AG7GN/kenwood
		sudo cp kenwood/710.sh /usr/local/bin/
		sudo cp kenwood/*.py /usr/local/bin/
		sudo cp kenwood/*.png /usr/share/pixmaps/

## Operating `710.py`

- Open a terminal and run:

		710.py 
		
	- By default, `710.py` will attempt to use `/dev/ttyUSB0` at 57600 baud to communicate with the radio. 
	- You can specify a different serial port or speed on the command line. Run `710.py -h` for instructions.  Running it with `-h` will display ports that it has identified as serial ports.
	
- For example, to use port `/dev/ttyUSB1` @ 19200 baud, run it like this:
	
		710.py -p /dev/ttyUSB1 -b 19200

	The baud rate must match the radio's __PC Port Baudrate__ (menu __920__) in the 710 and the equivalent in the 71A.
	
	The GUI features tool tips, which describe the different elements on the screen as you move your mouse over them.
	
	If you want the GUI to use a smaller desktop footprint, add the `--small` argument to `710.py`.

- Running `710.py -h` on one of my Pis for example shows:

		pi@nexuspi4b-ag7gn:~ $ 710.py -h
		usage: 710.py [-h] [-v]
				[-p {/dev/gps1,/dev/ttyUSB1,/dev/serial0,/dev/serial1,/dev/ttyS0,/dev/ttyAMA0,/dev/serial/by-id/usb-Prolific_Technology_Inc._USB-Serial_Controller_D-if00-port0}]
				[-b {300,1200,2400,4800,9600,19200,38400,57600}] 
				[-s] 
				[-l x:y] 
				[-x [1024-65535]] 
				[-r {none,left,right}] 
				[-c COMMAND]

		CAT control for Kenwood TM-D710G/TM-V71A

		optional arguments:
		  -h, --help      show this help message and exit
		  -v, --version   show program's version number and exit
		  -p {/dev/gps1,/dev/ttyUSB1,/dev/serial0,/dev/serial1,/dev/ttyS0,/dev/ttyAMA0,/dev/serial/by-id/usb-Prolific_Technology_Inc._USB-Serial_Controller_D-if00-port0}, --port {/dev/gps1,/dev/ttyUSB1,/dev/serial0,/dev/serial1,/dev/ttyS0,/dev/ttyAMA0,/dev/serial/by-id/usb-Prolific_Technology_Inc._USB-Serial_Controller_D-if00-port0}
					Serial port connected to radio (default: /dev/gps1)
		  -b {300,1200,2400,4800,9600,19200,38400,57600}, --baudrate {300,1200,2400,4800,9600,19200,38400,57600}
					Serial port speed (must match radio) (default: 57600)
		  -s, --small     Smaller GUI window (default: False)
		  -l x:y, --location x:y
					x:y: Initial x and y position (in pixels) of upper left corner of GUI. (default: None)
		  -x [1024-65535], --xmlport [1024-65535]
					TCP port on which to listen for XML-RPC rig control calls from clients such as Fldigi or Hamlib (default: 12345)
		  -r {none,left,right}, --rig {none,left,right}
					Nexus DR-X Users: Select left or right radio if you want to control GPIO PTT via an XML-RPC 'rig.set_ptt' call. This will map to GPIO
					pin 12 for the left radio and pin 23 for the right. 'none' means that 'rig.set_ptt' calls will be ignored. In any case, PTT
					activation via CAT command is never used. (default: none)
		  -c COMMAND, --command COMMAND
					CAT command to send to radio (no GUI) (default: None)


### Make a __Hamradio__ menu selection for `710.py`

Installing these scripts as per the directions above does not automatically create a Hamradio menu item because it is not possible to know in advance your system's serial port that will be used to communicate with the radio. You can make your own Hamradio menu item as described here. Some sleuthing might be needed to identify your radio's serial port.

Here's one way to do it:

1. Unplug your USB-serial cable from your Pi.
1. Open a Terminal on your Pi and run this command:

		dmesg -w -H 
		
	You'll see lots of output and then it'll pause and wait for some event to happen (like plugging in a USB device).
1. Plug your USB-serial cable into your Pi (the radio does not have to be on).
1. You should see some `dmesg` output appear in your Terminal. It'll look something like this:

		[Jan28 15:00] usb 1-1.4: new full-speed USB device number 4 using xhci_hcd
		[  +0.134526] usb 1-1.4: New USB device found, idVendor=067b, idProduct=2303, bcdDevice= 4.00
		[  +0.000009] usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=0
		[  +0.000006] usb 1-1.4: Product: USB-Serial Controller D
		[  +0.000006] usb 1-1.4: Manufacturer: Prolific Technology Inc. 
		[  +0.002693] pl2303 1-1.4:1.0: pl2303 converter detected
		[  +0.009797] usb 1-1.4: pl2303 converter now attached to ttyUSB1

1. In this example, that last line tells you that the USB-Serial cable you plugged in is "`...now attached to ttyUSB1`". So, the serial port in this example is `/dev/ttyUSB1`. Yours may be different. With that information, you can now make a `desktop` file. Note that this example desktop file will launch `710.py` in "small" mode so it doesn't occupy so much screen real estate.

1. Create your `desktop` file
	- Using your favorite text editor, create a file called `$HOME/.local/share/applications/kenwoodtm.desktop`. Here's one way to do that:
	
		- Click __Raspberry > Run__
		- In the __Run__ window, enter:
		
				mousepad $HOME/.local/share/applications/kenwoodtm.desktop
			
			That will open a text editor similar to Notepad on Windows.

	- Enter this text in the file:

			[Desktop Entry]
			Name=TM-D710G/TM-V71A Controller
			Comment=Kenwood TM-D710G/TM-V71A Controller
			Exec=sh -c "710.py -p /dev/ttyUSB1 --small >/dev/null 2>&1"
			Icon=hamradio.png
			StartupNotify=true
			Terminal=false
			Type=Application
			Categories=HamRadio
			Keywords=Ham Radio;Rig Control

	- Change the `Exec=` line to add/remove/modify arguments for your particular serial port/speed. Omit the `--small` if you want to run the GUI in regular size. 
	
		If you want GPIO PTT control via XML-RPC, also specify `-r left` or `-r right` for the left or right radios respectively. This is ONLY needed if you want to control PTT via XML-RPC calls!

	- Change the `Name=` line to suit. This is the menu item name.

	- Save the file and close your editor. The new menu item should appear at the bottom of your __Raspberry > Hamradio__ menu.

### Using `710.py` with Fldigi

`710.py` runs an XML-RPC server on port `12345` by default. You can change the port using the `-x <port-number>` argument when you launch `710.py`. Port `12345` is the default XML-RPC port for Fldigi (Left Radio).

1. Launch `710.py` either from the command line or via your new menu item.

1. In Fldigi, select __Configure > Rig Control > flrig__. Assuming you're using the default port of `12345`, configure these settings:

	![Flrig Settings](img/fldigi_flrig.png)
	
	If you used the `-x <port-number>` option to change the port, you must use that same port in the configuration above.
	
	Most Nexus DR-X uses will want to leave the __Flrig PTT keys modem__ unchecked because Fldigi GPIO PTT is already configured (under __Rig Control > GPIO__).
	
1. Click the __Reconnect__ button to tell Fldigi to set up a new XML-RPC connection to `710.py`.

1.	Click __Save__, then __Close__.

1. You should now be able to change the frequency from Fldigi's frequency field, from `710.py` by clicking on the frequency or by adjusting the frequency on the radio itself.

	__NOTE:__ Setting the frequency in Fldigi will only change the frequency on the side of the radio set for Digital.

### Using `710.py` with Hamlib

Hamlib can send commands and receive state information from Flrig. `710.py` can be used as a substitute for Flrig.

Nexus DR-X users can use the __Hamlib Rig Control GUI__ to manage Hamlib.

- Start the `710.py` script. 
	- Be sure to use the default port of `12345` (in other words, don't specify a different port with the `-x` option). Hamlib will only work over port `12345`. 
	- If you want Hamlib to control PTT, then run `710.py` with `-r left` or `-r right` to control the left or right radio. This is needed for apps that cannot control GPIO PTT directly and must rely on Hamlib for PTT.
- Run __Raspberry > Hamradio > Hamlib Rig Control GUI__ 
- Enter `flrig` in the __Rig search string__ field, then click __Find__. 
- Check __Flrig__ in the list.
- Set the __Serial Port__ and __Speed__ fields to __Not Applicatble__.
- Click __Save Changes & [Re]start rigctld__.
- You should now see that rigctld is running as shown in green:

	![hamlib flrig](img/hamlib_flrig.png)

- Applications that use Hamlib for rig control and PTT should now work with the Kenwood radio.

__NOTE:__ If you close `710.py` while `rigctld` is running, you'll have to restart `rigctld` again once `710.py` is running.

## Operating `710.sh`
- Open a terminal and run:
  
		710.sh -h
	and follow the instructions.  

`710.sh` will attempt to communicate with the radio in one of 2 ways:

1.	__If `710.py` IS ALREADY running:__ `710.sh` will use XML-RPC to send the command to `710.py`, which will in turn send the command to the radio. This is the slower of the 2 ways to communicate with the radio, but has the advantage that `710.py` can be running when you send your commands.

2. __If `710.py` IS NOT running:__ `710.sh` will attempt to start `710.py` in non-GUI "one shot" mode. No GUI will open, but `710.py` will send the command passed to it by `710.sh`, return the results to `710.sh`, and then exit. This is the way that `710.sh` operated in previous versions and will return data faster than the first method.

If you use the second method, the script will look for USB-serial cables (represented as files) in `/dev/serial/by-id` unless you specify the serial port with `-p`.  If any of the devices listed have filenames that contain any of these strings, then the script will automatically select and use that cable to communicate with the radio:

		USB-Serial

		RT_Systems

		usb-FTDI

If more than one cable matches, it'll use the last matching file name alphabetically.

To view the list of files that represent the USB-serial cables, open a terminal and run this command:

	ls -al /dev/serial/by-id
	

### Notes

You can optionally supply the serial port used to connect to your radio using the `-p PORT` argument.  For example:

	710.sh -p /dev/ttyUSB0 set timeout 3

Alternatively, you optionally supply a string to grep (search) for in `/dev/serial/by-id` to determine the serial port used to connect to your radio using the `-s PORTSTRING` argument.  For example:

	710.sh -s RT_Systems get info

If a port is supplied using `-p PORT`, it will take precedence over a string supplied by `-s PORTSTRING`."

If you connect more than one serial cable and the string description for those cables contain a match of any of the strings listed above you __MUST__ use either the `-p` or `-s` options to tell the cables apart apart.

In the following example, 2 USB-serial cables are attached to the Pi:

	pi@hampi-ag7gn:~ $ ls -l /dev/serial/by-id
	total 0
	lrwxrwxrwx 1 root root 13 Jan 24 09:13 usb-Prolific_Technology_Inc._USB-Serial_Controller_D-if00-port0 -> ../../ttyUSB0
	lrwxrwxrwx 1 root root 13 Jan 24 10:52 usb-RT_Systems_K5G_Radio_Ca͢le_RT1RVT5Y-if00-port0 -> ../../ttyUSB1

In this example, both cables will match the default search string.  So, you must specify either the `-p` or `-s` options:

Continuing with this example, to use the cable with `RT_Systems` in the name, run:

	710.sh -s RT_Systems get info
	
To use the cable with `USB-Serial` in the name:

	710.sh -s USB-serial get info
	
When you use the `-s` option, make sure you use a search string that's unique to the cable you want to use.