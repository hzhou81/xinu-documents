## Xinu 在树莓派上编译调试的具体文档
+ 安装Git并且下载最新的XINU代码
<pre><code>sudo apt-get install git
cd Documents
git clone https://github.com/hzhou81/xinu.git
</code></pre>
+ 安装ARM交叉编译环境
<pre><code>sudo apt-get install gcc-arm-none-eabi
sudo apt-get install gcc
sudo apt-get install bison
sudo apt-get install flex
</code></pre>
+ 命令行手动编译XINU
<pre><code>cd xinu
make -C compile clean
make -C compile PLATFORM=arm-rpi COMPILER_ROOT=/usr/bin/arm-none-eabi-
</code></pre>
+ 把compile目录下的xinu.boot重命名为kernel.img再和[bootcode.bin](https://github.com/hzhou81/xinu-documents/blob/master/bootcode.bin)以及[start.elf](https://github.com/hzhou81/xinu-documents/blob/master/start.elf)拷贝到一张树莓派兼容的SD卡(FAT格式)根目录下，就可以运行出XINU操作系统
![SD卡](https://github.com/hzhou81/xinu-documents/blob/master/images/sd.png) ![拷贝到SD卡的三个文件](https://github.com/hzhou81/xinu-documents/blob/master/images/xinuSD.png)
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
下载eclipse并解压到/opt/eclipse
启动eclipse,在Marketplace中搜索"GNU ARM"并安装(安装过程需要启动VPN)
把xinu这个项目移到其它文件夹，然后新建C++ Project名为xinu(Toolchain位置为/usr/bin),放在原来同样的位置，然后再把xinu的源代码(包含.git)拷贝回来
git remote add csdn https://code.csdn.net/hazel_81/xinu.git   //orgin是github的地址，添加一个csdn的备份地址
右击xinu项目，在C/C++Build中Build Commands是${cross_make}，Build directory是${workspace_loc:/xinu/compile/},Build settings中Enable parallel build前打勾,WorkBench Build Behavior中把all都去掉改为PLATFORM=arm-rpi COMPILER_ROOT=/usr/bin/arm-none-eabi-,Toolchain Path都设置成/usr/bin（包括Project，WorkSpace和Global都用这个设置)

sudo apt-get install minicom
sudo minicom -s 端口设置为/dev/ttyUSB0 Hardware Flow Control设置为NO
cd /home/hzhos/Documents
git clone git://git.code.sf.net/p/openocd/code
mv code openocd
sudo apt-get install libtool autoconf libusb-dev libftdi-dev libusb-1.0.0 libusb-1.0.0-dev
cd openocd
./bootstrap
./configure --enable-maintainer-mode --enable-ftdi
make
sudo make install
sudo cp /home/hzhos/Documents/openocd/contrib/99-openocd.rules /etc/udev/rules.d/
sudo vi /etc/udev/rules.d/99-openocd.rules 修改其中FT2232的字段把idVendor修改为1457,把idProduct修改为5118并保存
sudo udevadm control --reload-rules
用杜邦线按下表进行连接
JTAG    	RPi GPIO
VREF*(1)	Pin 1
nTRST(3)	Pin 15
GND(4)		Pin 9
TDI(5) 		Pin 7
TMS(7)		Pin 13
TCK(9)		Pin 22
RTCK(11)	Pin 16
TDO(13)		Pin 18
openocd -f tcl/interface/ftdi/100ask-openjtag.cfg -f tcl/board/raspberry_pi.cfg	
sudo apt-get install arm-none-eabi-gdb
在eclipse的Windows->Preference菜单中设置OCD的目录为/home/hzhos/Documents/code
在xinu项目的debug中新增一个Debug Configuration，C++ Application选择compile/xinu.elf,Executable设置为${openocd_path}/src/${openocd_executable}，Configure Options设置为-f ${openocd_path}/tcl/interface/ftdi/100ask-openjtag.cfg -f ${openocd_path}/tcl/board/bcmrpi2.cfg,把Enable ARM semihosting的勾去掉，
	




gdb
target remote localhost:3333
