## Documents about develop Xinu targeting Raspberry PI in Eclipse
[中文版本](https://github.com/hzhou81/xinu-documents/blob/master/ARM_CHN.md)

+ Install Git and download the newest XINU source code
<pre><code>sudo apt-get install git
cd Documents
git clone https://github.com/hzhou81/xinu.git xinu
</code></pre>

+ Modify Makefile in order to add debug information in project。Modify /home/${USER}/Documents/xinu/compile/Makefile，At line 309 and 313, add "-g" just before "-o"

+ Install ARM Cross-compiler toolchian and related packages
<pre><code>sudo apt-get install gcc-arm-none-eabi
sudo apt-get install gdb-arm-none-eabi
sudo apt-get install bison
sudo apt-get install flex
sudo apt-get install make
</code></pre>
+ Compile XINU in command line
<pre><code>cd xinu
make -C compile clean
make -C compile PLATFORM=arm-rpi COMPILER_ROOT=/usr/bin/arm-none-eabi-
</code></pre>
+ Rename "xinu.boot" to "kernel.img" in compile directory, and then with [bootcode.bin](https://github.com/hzhou81/xinu-documents/blob/master/bootcode.bin) and [start.elf](https://github.com/hzhou81/xinu-documents/blob/master/start.elf) these three files copy to a Raspberry PI certificated SD card（FAT file system) 's root directory，so now you can run XINU OS

　　![SD Card](https://github.com/hzhou81/xinu-documents/blob/master/images/sd.png) ![Copy three files to SD Card](https://github.com/hzhou81/xinu-documents/blob/master/images/xinuSD.png)

+ Install Oracle Java 8.0
<pre><code>sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
</code></pre>

+ Install the lastest Eclipse C/C++ for Linux 64bit Version(Download eclipse and tar extract it to /opt/eclipse,Launch eclipse,find "GNU ARM" in Marketplace

+ Move xinu directory to another directory, and then new a C++ Project named "xinu"(Toolchain location:/usr/bin),make it same location，finally copy previous xinu source code (including the .git directory) back

+ Add a optical CSDN Git backup
<pre><code>git remote add csdn https://code.csdn.net/hazel_81/xinu.git   
</code></pre>

+ Setting XINU project property, right click xinu project，In C/C++Build->Build Commands,set it to ${cross_make}，"Build directory" set it to${workspace_loc:/xinu/compile}
　　![set compile directory](https://github.com/hzhou81/xinu-documents/blob/master/images/setting1.png)

+ Setting XINU project property，right click xinu project，In Build settings,check the "Enable parallel build",In "WorkBench Build Behavior" modify all "all" to "PLATFORM=arm-rpi COMPILER_ROOT=/usr/bin/arm-none-eabi-"
　　![set compile directory](https://github.com/hzhou81/xinu-documents/blob/master/images/setting2.png) 

+ Setting XINU project property, right click xinu project，Set "Toolchain Path" to "/usr/bin"（Include Project，WorkSpace and Global)

+ Solve Compile Error "Symbol 'OK' could not be resolved"，Right Click xinu project properties, in "C/C++ General->Paths and Symbols",add "/xinu/include" to "GNU C" tab
　　![Set up include path](https://github.com/hzhou81/xinu-documents/blob/master/images/include.png) 
 
+ Make a wire to connect OpenJTAG(JTAG) and Raspberry Pi(GPIO), First buy a [20pin 2.0mm JTAG Wires] (https://item.taobao.com/item.htm?spm=a1z09.2.0.0.PdxKf4&id=13722984152&_u=15a2hlh92fa) And [FC-26P IDC 2.54MM](https://item.taobao.com/item.htm?spm=a1z10.3-c.w4002-5688543101.9.WDrVpv&id=10247398093) on Taobao.com,Cut off one side of the JTAG wires,and push wires on connector as follow:
　　![connect method](https://github.com/hzhou81/xinu-documents/blob/master/images/JTAG.png)
+ As wire is made, Push one side to GPIO port on Raspberry Pi，and then get the other side out of the Raspberry Pi case through the SD card slot, so it can tight the wire
　　![connect raspberry pi](https://github.com/hzhou81/xinu-documents/blob/master/images/JTAG-GPIO.JPG)

+ Install Padora Linux on Raspberry Pi, format the SD card to FAT file system, then use [Win32Imager](https://sourceforge.net/projects/win32diskimager/) to extract [Pidora ARM Linux](http://www.pidora.ca/pidora/releases/20/images/Pidora-2014-R3.zip) image to SD card，of course other linux for Raspberry Pi is also allowed.
+ Use a LED to test JTAG connect wire。Power on Raspberry Pi,Start Linux and put a LED light one leg directly on JTAG No.3 port(nTRST), put LED light other leg on JTAG No.4 port(GND),Run "console" on Raspberry Pi Linux(pay attention that "export" on line number 3 has SPACE character)
<pre><code>cd /sys/class/gpio
 sudo -s
 echo 22 > export
 cd gpio22
 echo out > direction
 echo 1 > value</code></pre>
 ![Turn on LED](https://github.com/hzhou81/xinu-documents/blob/master/images/LED.JPG)
 
 Tur off LED
 <pre><code>echo 0 > value</code></pre>
+ Download and Compile OpenOCD
<pre><code>cd /home/${USER}/Documents
git clone git://git.code.sf.net/p/openocd/code
mv code openocd
sudo apt-get install libtool autoconf libusb-dev libftdi-dev libusb-1.0.0 libusb-1.0.0-dev
cd openocd
</code></pre>
Modify /home/${USER}/Documents/openocd/src/target/arm11_dbgtap.c commet lines of code above line.620
<code><pre>if (error_count > 0) {
			LOG_ERROR("%u words out of %u not transferred",
				error_count, readiesNum);
			retval = ERROR_FAIL;
		}
</code></pre>and continue to compile code
<code><pre>
./bootstrap
./configure --enable-maintainer-mode --enable-ftdi
make
sudo make install
</code></pre>
+ Add OpenJTAG Device files
<pre><code>sudo cp /home/${USER}/Documents/openocd/contrib/99-openocd.rules /etc/udev/rules.d/
sudo vi /etc/udev/rules.d/99-openocd.rules</code></pre>
In "FT2232" section, modify "idVendor" value to 1457,modify "idProduct" value to 5118 and save
<pre><code>sudo udevadm control --reload-rules
</code></pre>
+  Setting Raspberry Pi's GPIO port to JTAG mode。Copy [JtagEnabler.cpp](https://github.com/hzhou81/xinu-documents/blob/master/JtagEnabler.cpp) to directory on Raspberry Pi，Run following command
<pre><code>sudo yum install gcc-c++
g++ -o JtagEnabler JtagEnabler.cpp
 sudo ./JtagEnabler
</code></pre>

+ Make JtagEnabler startup on boot. Running these command on Pidora OS with your raspberry pi，create a service named "jtag"，setting this service startup on boot.
<pre><code>
cd /usr/lib/systemd/system
sudo vi jtag.service

Input these lines:
[Unit]
Description=Start Jtag on startup
[Service]
Type=forking
ExecStart=/home/hzhos/Documents/JtagEnabler
[Install]
WantedBy=multi-user.target

Save and quit

sudo chmod 754 jtag.service
sudo systemctl start jtag.service
sudo systemctl status jtag.service
sudo systemctl enable jtag.service
</code></pre>

+ Start OpenOCD as daemon process(you can image this as a GDB Server)。Copy [raspberry pi setting files](https://github.com/hzhou81/xinu-documents/blob/master/raspberry_pi.cfg)  to openocd/tcl/board directory，and start OpenOCD process，Don't close this console windows after startup
<pre><code>openocd -f tcl/interface/ftdi/100ask-openjtag.cfg -f tcl/board/raspberry_pi.cfg	</code></pre>

+ Test GDB to debug raspberry pi using OpenOCD。Start another console window，input these commands
<pre><code>arm-none-eabi-gdb
target remote 127.0.0.1:3333
x/10i $pc
stepi
x/2i $pc
</code></pre>if pc register's value is the next instruction.this show up debug success,running following command telnet 127.0.0.1 4444
<pre><code>mww 0x20200028 0x10000
mww 0x2020001C 0x10000
</code></pre>If you find out a green LED light turn on on raspberry pi that shows up memory write success.

+ Setting OpenOCD path in Eclipse. In Eclipse Windows->Preference，Click Run/Debug->OpenOCD to setting up openocd path to /home/hzhos/Documents/openocd

+ Setting GDB to debug XINU in Eclipse.Right click Debug As->Debug Configurations,Create a new line named "XINU Debug"的GDB OpenOCD Debugging,In Project ,select "xinu"，In C/C++ Application,select "Search Project" as "compile/xinu.elf" as image followed:
![Setting Page 1](https://github.com/hzhou81/xinu-documents/blob/master/images/gdb_debug1.png)
In Debugger Page,Setting "Executable" to${openocd_path}/src/${openocd_executable}，Setting "Config options" to "-f ${openocd_path}/tcl/interface/ftdi/100ask-openjtag.cfg -f ${openocd_path}/tcl/board/raspberry_pi.cfg" ,Uncheck "Enable ARM semihosting" as image followed:
![Setting Page 2](https://github.com/hzhou81/xinu-documents/blob/master/images/gdb_debug2.png)

+ Setting GBD default source seach path in Eclipse。In Eclipse "Window" Menu->Preference->C/C++->Debug->Source Lookup Path，delete all the settings，add a item "FileSystem Directory" point to "/"

+ Final debug GUI
![Finished](https://github.com/hzhou81/xinu-documents/blob/master/images/debug.png)

+ Setting console input/output.Buy a [USB to TTL module](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.HCwAPF&id=521699082592&_u=75a2hlhcad6),follow method to connect to raspberry pi
![tty connect diagram](https://github.com/hzhou81/xinu-documents/blob/master/images/TTL.png)

+ Connect using minicom。First running sudo apt install minicom to install "minicom" software module。Make USB to TTL module connect to your computer，as a /dev/ttyUSB1 device，Open "minicom" setting port to "/dev/ttyUSB1"、rate to "115200" and "Hardware Flow Control" to "No"

+ Starting debug. You will see through PL011 chip, XINU output TTL signals using GPIO，and finally output message on your computer screen
![minicom debug picture](https://github.com/hzhou81/xinu-documents/blob/master/images/minicom.png)
