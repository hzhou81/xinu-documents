## 在Eclipse中开发运行于树莓派的Xinu之具体文档
+ 安装Git并且下载最新的XINU代码
<pre><code>sudo apt-get install git
cd Documents
git clone https://github.com/hzhou81/xinu.git xinu
</code></pre>

+ 为项目的Makefile文件添加调试信息。修改/home/${USER}/Documents/xinu/compile/Makefile文件，在309行和313行中的-o前添加-g开关

+ 安装ARM交叉编译环境
<pre><code>sudo apt-get install gcc-arm-none-eabi
sudo apt-get install gdb-arm-none-eabi
sudo apt-get install bison
sudo apt-get install flex
sudo apt-get install make
</code></pre>
+ 命令行手动编译XINU
<pre><code>cd xinu
make -C compile clean
make -C compile PLATFORM=arm-rpi COMPILER_ROOT=/usr/bin/arm-none-eabi-
</code></pre>
+ 把compile目录下的xinu.boot重命名为kernel.img再和[bootcode.bin](https://github.com/hzhou81/xinu-documents/blob/master/bootcode.bin)以及[start.elf](https://github.com/hzhou81/xinu-documents/blob/master/start.elf)拷贝到一张树莓派兼容的SD卡(FAT格式)根目录下，就可以运行出XINU操作系统

　　![SD卡](https://github.com/hzhou81/xinu-documents/blob/master/images/sd.png) ![拷贝到SD卡的三个文件](https://github.com/hzhou81/xinu-documents/blob/master/images/xinuSD.png)

+ 安装Oracle Java 8.0
<pre><code>sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
</code></pre>

+ 安装最新版的Eclipse C/C++ for Linux 64bit版本(下载eclipse并解压到/opt/eclipse,启动eclipse,在Marketplace中搜索"GNU ARM"并安装

+ 把xinu这个项目移到其它文件夹，然后新建C++ Project名为xinu(Toolchain位置为/usr/bin),放在原来同样的位置，然后再把xinu的源代码(包含里面的.git目录)拷贝回来

+ 添加一个CSDN的Git备份(这步可选做)
<pre><code>git remote add csdn https://code.csdn.net/hazel_81/xinu.git   
</code></pre>

+ 设置XINU项目的属性，右击xinu项目，在C/C++Build中Build Commands是${cross_make}，Build directory是${workspace_loc:/xinu/compile}
　　![设置编译路径](https://github.com/hzhou81/xinu-documents/blob/master/images/setting1.png)

+ 设置XINU项目的属性，右击xinu项目，在Build settings中Enable parallel build前打勾,WorkBench Build Behavior中把all都去掉改为PLATFORM=arm-rpi COMPILER_ROOT=/usr/bin/arm-none-eabi-
　　![设置编译参数](https://github.com/hzhou81/xinu-documents/blob/master/images/setting2.png) 

+ 设置XINU项目的属性，右击xinu项目，将Toolchain Path都设置成/usr/bin（包括Project，WorkSpace和Global都用这个设置)

+ 解决出现Symbol 'OK' could not be resolved，右击xinu项目点属性，进入C/C++ General->Paths and Symbols,在GNU C中添加/xinu/include
　　![设置包含路径](https://github.com/hzhou81/xinu-documents/blob/master/images/include.png) 
 
+ 制作用于连接OpenJTAG和树莓派的JTAG转GPIO连接线,首先在淘宝购买[20pin 2.0mm间距 JTAG排线](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.PdxKf4&id=13722984152&_u=15a2hlh92fa) 和[FC-26P IDC压线头 2.54MM间距](https://item.taobao.com/item.htm?spm=a1z10.3-c.w4002-5688543101.9.WDrVpv&id=10247398093) 把JTAG排线的一头拆掉,按照下图所示把压线头安装上去
　　![连线方法](https://github.com/hzhou81/xinu-documents/blob/master/images/JTAG.png)
+ 把制作好的线，一端插入树莓派的GPIO端口，另外一端从塑料外壳的SD卡口接出来，这样可以把我的线固定的更加牢靠
　　![连接树莓派](https://github.com/hzhou81/xinu-documents/blob/master/images/JTAG-GPIO.JPG)

+ 给树莓派安装Linux操作系统,把刚才那张SD卡用FAT格式化掉，然后用[Win32Imager](https://sourceforge.net/projects/win32diskimager/) 把[Pidora ARM Linux](http://www.pidora.ca/pidora/releases/20/images/Pidora-2014-R3.zip) 解压出来的镜像写到SD卡中，当然你也可以用其它支持树莓派的Linux发行版

+ 用LED测试JTAG连接线的正确性。将树莓派上电，启动Linux，并将一个LED灯泡一端连接在JTAG的3号口(nTRST),另外一端连接在JTAG的4号口(GND),在树莓派的Linux上执行(注意:第三行export前面是有空格的)
<pre><code>cd /sys/class/gpio
 sudo -s
 echo 22 > export
 cd gpio22
 echo out > direction
 echo 1 > value</code></pre>
 ![打开LED](https://github.com/hzhou81/xinu-documents/blob/master/images/LED.JPG)
 
 关闭LED灯
 <pre><code>echo 0 > value</code></pre>
+ 下载并编译OpenOCD
<pre><code>cd /home/${USER}/Documents
git clone git://git.code.sf.net/p/openocd/code
mv code openocd
sudo apt-get install libtool autoconf libusb-dev libftdi-dev libusb-1.0.0 libusb-1.0.0-dev
cd openocd
</code></pre>
修改/home/${USER}/Documents/openocd/src/target中的arm11_dbgtap.c中注释掉620行以下这段代码
<code><pre>if (error_count > 0) {
			LOG_ERROR("%u words out of %u not transferred",
				error_count, readiesNum);
			retval = ERROR_FAIL;
		}
</code></pre>然后继续编译源代码
<code><pre>
./bootstrap
./configure --enable-maintainer-mode --enable-ftdi
make
sudo make install
</code></pre>
+ 添加OpenJTAG设备文件
<pre><code>sudo cp /home/${USER}/Documents/openocd/contrib/99-openocd.rules /etc/udev/rules.d/
sudo vi /etc/udev/rules.d/99-openocd.rules</code></pre>
修改其中FT2232的字段把idVendor修改为1457,把idProduct修改为5118并保存
<pre><code>sudo udevadm control --reload-rules
</code></pre>
+  将树莓派的GPIO端口改为JTAG模式。把[JtagEnabler.cpp](https://github.com/hzhou81/xinu-documents/blob/master/JtagEnabler.cpp) 拷贝到树莓派的某个目录，执行下列命令
<pre><code>sudo yum install gcc-c++
g++ -o JtagEnabler JtagEnabler.cpp
 sudo ./JtagEnabler
</code></pre>

+ 将上面那个JtagEnabler设为开机自启动。在树莓派的Pidora操作系统中执行以下命令，创建jtag的服务，并且设置为开机自启动并运行服务。
<pre><code>
cd /usr/lib/systemd/system
sudo vi jtag.service
输入
[Unit]
Description=Start Jtag on startup
[Service]
Type=forking
ExecStart=/home/hzhos/Documents/JtagEnabler
[Install]
WantedBy=multi-user.target
保存退出
sudo chmod 754 jtag.service
sudo systemctl start jtag.service
sudo systemctl status jtag.service
sudo systemctl enable jtag.service
</code></pre>

+ 启动OpenOCD守护进程(可以理解成是一个GDB Server)。将[树莓派配置文件](https://github.com/hzhou81/xinu-documents/blob/master/raspberry_pi.cfg) 拷贝到openocd的tcl/board目录下，然后启动OpenOCD进程，这个命令行启动好后不要关闭
<pre><code>openocd -f tcl/interface/ftdi/100ask-openjtag.cfg -f tcl/board/raspberry_pi.cfg	</code></pre>

+ 测试GDB是否可以通过OpenOCD来调试树莓派。另开一个命令行窗口，输入以下命令
<pre><code>arm-none-eabi-gdb
target remote 127.0.0.1:3333
x/10i $pc
stepi
x/2i $pc
</code></pre>如果pc寄存器所指向的命令变成下一条指令了，就说明JTAG调试成功了,下面在命令行下执行telnet 127.0.0.1 4444
<pre><code>mww 0x20200028 0x10000
mww 0x2020001C 0x10000
</code></pre>如果发现树莓派上的有个绿色的灯点亮并且熄灭了就说明写内存成功

+ Eclipse里设置OpenOCD路径。在Eclipse的Windows->Preference里，点击Run/Debug->OpenOCD中设置openocd的路径是/home/hzhos/Documents/openocd

+ Eclipse里设置GDB调试XINU。右击XINU点Debug As->Debug Configurations,新建一个名为"XINU Debug"的GDB OpenOCD Debugging的项目,Project选xinu，C/C++ Application选Search Project里面的compile/xinu.elf ,如下图
![设置页1](https://github.com/hzhou81/xinu-documents/blob/master/images/gdb_debug1.png)
在Debugger页内,Executable设为${openocd_path}/src/${openocd_executable}，Config options设为-f ${openocd_path}/tcl/interface/ftdi/100ask-openjtag.cfg -f ${openocd_path}/tcl/board/raspberry_pi.cfg ,把Enable ARM semihosting的勾去掉如下图
![设置页1](https://github.com/hzhou81/xinu-documents/blob/master/images/gdb_debug2.png)

+ Eclipse里设置GDB默认源代码搜索路径。在Eclipse中Window菜单->Preference->C/C++->Debug->Source Lookup Path中，删除原来所有的设置，新增一个"FileSystem Directory",并指向根目录/

+ 最终调试界面
![完成](https://github.com/hzhou81/xinu-documents/blob/master/images/debug.png)

+ 配置终端输入输出。购买一个[USB转TTL的模块](https://item.taobao.com/item.htm?spm=a1z09.2.0.0.HCwAPF&id=521699082592&_u=75a2hlhcad6),按照下图方式和树莓派连接
![tty连接图](https://github.com/hzhou81/xinu-documents/blob/master/images/TTL.png)

+ 使用minicom连接。首先sudo apt install minicom安装minicom组件。将USB转TTL模块连接到系统，成为一个/dev/ttyUSB1的设备，打开minicom设置端口是/dev/ttyUSB1、波特率是115200并且Hardware Flow Control等于No

+ 开始调试时候，看到Xinu通过PL011芯片，输出到GPIO上TTL信号，最终显示在终端调试电脑的屏幕上的信息
![minicom调试图](https://github.com/hzhou81/xinu-documents/blob/master/images/minicom.png)
