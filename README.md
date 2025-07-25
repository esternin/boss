# <img src="https://github.com/esternin/boss/blob/main/boss.svg" alt="[boss logo]" width="64" height="64"> Brock Optical Spectroscopy Software 

<img src="https://github.com/esternin/boss/blob/main/Screenshot-01.png" alt="[boss screenshot]" width="100%">

A typical optical spectrometer consists of both mechanical and optical parts that are quite robust.  It is not unusual to find multi-decade-old spectrometers still in daily use. The one component that ages and usually causes the spectrometer to be replaced is the detector, as the light sensor technology has been developing at great speed ever since the high-resolution digital cameras came into being in the 1990s, and it shows no sign of slowing down.

This project is dedicated to creating a reasonably universal software that allows one to upgrade the older spectrometer by mating it with a new high-resolution digital detector. Not a "webcam"-level detector (for those simple spectrometers a basic web-cam spectroscopy interface is available at https://www.physics.brocku.ca/Labs/WebCamSpectroscopy/), but one of high-end astrophotography cameras which offer high quantum efficiency of their sensors and typically have built-in Peltier cooling of the sensor backing, to reduce the dark currents and their associated image noise.

<details>
  <summary><b>Installing boss and prerequisites</b></summary>

**boss** software is written in tcl/tk which makes it platform independent, but it only gets tested under Linux. Xubuntu or Debian+XFCE are the primary distributions in use at the <a href="https://brocku.ca/physics" target="_blank">Department of Physics at Brock University</a>. For fast i/o and large image processing, critical parts are written in C using **critcl** (https://andreas-kupries.github.io/critcl/), and a pre-compiled low-level C library provided by the manufacturer ZWO (https://www.zwoastro.com/software/).

To install boss with its prerequisite pieces on a typical (x)ubuntu/debian installation, try:
```
sudo apt-get install build-essential tcl-dev tk-dev blt-dev critcl bwidget tklib libtk-img
sudo install -m 644 99-zwo.rules /etc/udev/rules.d
sudo ./install.sh
```
and be sure to download and untar into `/opt/ASI_linux_mac_SDK_V1.38` (or similar, but then adjust the pointers in the first line of the main file) the libraries provided by ZWO.
</details>

<details>
  <summary><b>Handling GPIB instruments</b></summary>

In simple spectrometers the slit sizes and grating angles are controlled by hand. More sophisticated devices may have GPIB or serial-line controls. Since the protocols vary, each spectrometer may need to be treated differently.  For GPIB-controlled devices, **linux-gpib** (https://github.com/coolshou/linux-gpib or https://gpib-tcl.sourceforge.net/GPIB-Tcl.html) is used to provide the kernel modules (it is rumoured to be included in the mainline Linux kernel soon) and **tcl-gpib** (https://github.com/slazav/tcl-gpib) which provides a tcl/tk interface to that library. These must also be installed. Below is the log of what I did, YMMV.

If GPIB is not required, use a `NOGPIB` flag at run-time, as in `boss NOGPIB` which will hide the GPIB tab from view.  The GPIB controls will no longer be visible, and if you then execute a macro containing GPIB commands, errors will show up. 
```
sudo apt-get install dwarves
cp /sys/kernel/btf/vmlinux /usr/lib/modules/`uname -r`/build/

mkdir ~/GPIB
cd ~/GPIB
git clone https://github.com/coolshou/linux-gpib.git

cd linux-gpib/linux-gpib-kernel/
make clean
make
sudo make install

cd ../linux-gpib-user/
./bootstrap 
./configure --prefix=/usr --sysconfdir=/etc
echo <<EOT > include/gpib/version_subst
s/^/#define GPIB_MAJOR_VERSION /
s/\./\n#define GPIB_MINOR_VERSION /
s/\./\n#define GPIB_MICRO_VERSION /
s/\-.*$//
EOT
make clean
make
sed -i -e 's/0660/0666/' usb/98-gpib-generic.rules
sudo make install

cd ~/GPIB
git clone https://github.com/slazav/tcl-gpib.git
cd tcl-gpib/tcl-gpib
make clean
make
make install tcldatadir=/usr/local/share/tcltk tcllibdir=/usr/local/lib/tcltk

cd ~/GPIB
sudo apt-get install fxload
wget --content-disposition --no-check-certificate http://linux-gpib.sourceforge.net/firmware/gpib_firmware-2008-08-10.tar.gz
tar -xvf gpib_firmware-2008-08-10.tar.gz 
cd gpib_firmware-2008-08-10/
sudo cp -r * /usr/share/usb
cd ..
rm -rf gpib_firmware-2008-08-10
```  
</details>

<details>
  <summary><b>Running boss</b></summary>

`install.sh` places the appropriate launcher and icon files where your window manager looks for such files, so you should be able to find <b>boss</b> in the applications' menus right away, no need to reboot or restart anything.

Running from a terminal, you may simply type `boss` (`install.sh` places it in `/usr/local/bin/boss`). If things do not work, try getting some additional insight by using the `DEBUG` run-time flag:
```
boss DEBUG
```
</details>

<details>
  <summary><b>boss command macros</b></summary>

Explore the nifty macro facility that allows you to call any of the tcl subroutines, so an automated multi-file data acquisition is possible. We use it to divide long data acquisition runs into several pieces, for safety, and to help identifying cosmic rays' artifacts:
```
set expTime 1000
set expUnit s
getSnapshot
saveFile 01.raw
set ROI "0 1800 4144 1850"
adjustROI
saveFile 01.dat
getSnapshot
saveFile 02.raw
adjustROI
saveFile 02.dat
(etc)
```
</details>

<details>
  <summary><b>Hardware requirements</b></summary>
  
This software is being tested on two astrophotography cameras from ZWO (https://zwoastro.com):
  - ZWO ASI294MM Pro, and
  - ZWO ASI2600MM Pro

The Pro designation indicates built-in active cooling, and MM type employs monochrome sensors of the highest sensitivity and dynamic range (the corresponding type MC cameras are the colour versions of the same, and typically have lower dynamic range but three colour channels).

A note of caution: use high-quality USB3 cables and adequate power supplies. A poor cable may work for one camera, and fail in unexpected ways for a more demanding one. If you are experiencing failed exposures, the first thing to do is to try a better USB cable.
</details>
