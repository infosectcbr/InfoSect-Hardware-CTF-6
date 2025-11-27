# Hardware CTF 1

This is the Hardware CTF 6.

Register your name/handle with InfoSect CTF to track your progress and compare yourself to others [http://ctf.infosectcbr.com.au:8000](http://ctf.infosectcbr.com.au:8000).

Solve each level to get the flag. Once you have the flag, enter it into the CTF to get your points.

![](https://busside.com.au/assets/img/gg/Hwctf1-1.jpg)
![](https://busside.com.au/assets/img/gg/Hwctf1-2.jpg)

## How to assemble

- 1.8"Adafruit TFT display and header pins
- Arduino Nano
- Header pin strip
- Button

Solder the screen first. Make sure not to solder into the mounting holes. Double check this more than once!

Solder the Arduino Nano next. Make sure you put it in the right way around. This part is tricky and the fit is tight. Keep going, you can do it! Feel free to use a little bit of force.

Solder the header pins and the button.

## How to install the software

Attach the badge via USB to your computer. If you are using a VM, be sure to do USB passthrough:

First, install `avrdude`:

```
$ sudo apt install avrdude
```

Grab the firmware blob from github and flash it under a Linux terminal:

```
$ git clone https://github.com/infosectcbr/InfoSect-Hardware-CTF-6
$ cd InfoSect-Hardware-CTF-6
$ sudo ./flash_hwctf6.sh /dev/ttyUSB0
```

If your badge is on /dev/ttyUSB1 or another port, use that instead.

## How to use

Press the button to skip the intro screen.

At the menu, 1 short button presson to go to the next menu item. 1 (slightly) longer button press to select.

## Level -

Look at the firmware blob. Can you find an embedded flag?

## Level 0 - ASM Me

If you can assemble and flash the badge, this will be enough to get the flag!

## Level 1 - UART

You will need to interface via UART. The pinout and line settings are given. Use a serial-uart bridge. We recommend and have tested with [https://www.jaycar.com.au/arduino-compatible-usb-to-serial-adaptor-module/p/XC4464](https://www.jaycar.com.au/arduino-compatible-usb-to-serial-adaptor-module/p/XC4464).

To use the usb to serial adapter, connect TXD/TX, RXD/RX, and GND/Ground.

Open up minicom in a Linux terminal. Assuming the USB serial adapter is on /dev/ttyUSB0 use the following. If the adapter is on /dev/ttyUSB1 or elsewhere, use that instead.

```
$ sudo apt-get install minicom
$ sudo minicom -D /dev/ttyUSB0

Type ctrl-a, then 'z' on it's own. This will enter you into the menu system.

Type 'o' which will enter you into the configuration. Scroll down to the serial port setup and hit enter.

Type 'f' to turn off software flow control. If you don't do this, you will not be able to interface correctly.

Change the baud rate to the appropriate setting.

Exit out of the menu system.

Do you get readable text from your UART connection?

## Level 2 - Bin2Dec

The goal of this challenge is to convert binary numbers into decimal numbers. The Hardware CTF does not echo back what we type, but it will receive and process it.

You will use pwntools to programtically interface with the Hardware CTF. Pwntools is mostly used in exploit development, but we will use to interface over the serial interface.

You might need to install it in Linux

```
$ sudo apt-get install python3-pip
$ sudo pip3 install pwntools
```

Firstly, try interacting with the Hardware CTF using minicom. After you see how the challenge works, it's time to move onto programatically interfacing with it.

To make pwntools use the serial interface, we need to use the serialtube.

```
#!/usr/bin/python3

from pwn import *

io = serialtube('/dev/ttyUSB0', baudrate=9600, convert_newlines = False)
```

We can also send and receive lines on the serial interface using the sendline and recvline APIs.

```
#!/usr/bin/python3

from pwn import *

io = serialtube('/dev/ttyUSB0', baudrate=9600, convert_newlines = False)
io.sendline("")
line = io.readline()
print(line.decode("utf-8"))
```

Now we can interface with the Hardware CTF to inspect the things it wants us to do:

```
#!/usr/bin/python3

from pwn import *

io = serialtube('/dev/ttyUSB0', baudrate=9600, convert_newlines = False)
io.sendline("")
while True:
    line = io.readline()
    print(line.decode("utf-8"))
    if line.find(b"CONVERT") >= 0:
	solution = b"XXX"
        io.sendline(solution)
```

Complete the missing code to solve the challenge.

## Level 3 - Full Adder

This challenge is similar to the last one, except you have to solve simple arithmetic problems. You will need to do this programatically using Python.

## Level 4 - Password

In this challenge, you will need to byte at a time decrypt a password.

The challenge asks you for a password in the format of `LENGTH:PASSWORD`. The `LENGTH` is the length of `PASSWORD`. For example, `4:pass` or `3:abc`.

The password authentication has a bug in its implementation. It only checks the length of the password you supply. If it is correct, it tells you it matches. If the entire password is correct, it will subsequently tell you that you've logged in.

If you enumerate the characters `[a-z][A-Z][0-9]_{}`, brute forcing 1 character at a time, you can reveal the entire password.

HINT: The password is in the `flag{XXX}` format.
