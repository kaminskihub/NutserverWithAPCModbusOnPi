# Nutserver With USB APCModbus On a PI5
Step by Step install Nut Server with APC USB Modbus Support on a Pi 5

I was truggling with this for 8 hours last week, so I decided to make a step by step guide

install clean Raspberry Pi OS 64bit on a SD card

Get ready to compile
```
sudo apt update
sudo apt upgrade

sudo apt install nut
sudo apt remove nut

```
make sure you have uncommented "deb-src" lines to match the "deb" lines in /etc/apt/sources*
```
sudo nano /etc/apt/sources.list

sudo apt update
sudo apt upgrade
sudo apt-get build-dep nut

sudo apt remove libmodbus5
sudo apt remove libmodbus-dev

sudo apt install libgpiod-dev

```


At first we need to a custom libmodbus version with USB support
We can find it here
https://github.com/networkupstools/libmodbus/tree/rtu_usb
```
cd ~
mkdir git
cd git
#git clone -b rtu_usb https://github.com/networkupstools/libmodbus.git
git clone -b rtu_usb https://github.com/EchterAgo/libmodbus.git
cd libmodbus
./autogen.sh
./configure --with-libusb --enable-static --disable-shared --prefix=/usr/local
sudo make install
```

next we need to download nut repository and compile nut
```
cd ~
cd git
git clone https://github.com/networkupstools/nut
cd nut
./autogen.sh
```

Run the next monster command:
```
./configure --prefix=/usr \
 --with-drivers=apc_modbus --with-usb --with-modbus \
 --with-modbus-includes=-I/usr/local/include/modbus \
 --with-modbus-libs="-L/usr/local/lib -lmodbus" \
 --mandir=/usr/share/man \
 --infodir=/usr/share/info --sysconfdir=/etc/nut --localstatedir=/var \
 --libexecdir=/usr/lib/nut --srcdir=. --enable-maintainer-mode \
 --disable-silent-rules --libdir=/usr/lib/`gcc -print-multiarch` \
 --with-cgi --with-dev --enable-static \
 --with-statepath=/var/run/nut --with-altpidpath=/var/run/nut \
 --with-drvpath=/lib/nut --with-cgipath=/usr/lib/cgi-bin/nut \
 --with-htmlpath=/usr/share/nut/www --with-pidpath=/var/run/nut \
 --datadir=/usr/share/nut --with-pkgconfig-dir=/usr/lib/`gcc -print-multiarch`/pkgconfig \
 --with-user=nut --with-group=nut --with-udev-dir=/lib/udev \
 --with-systemdsystemunitdir=/lib/systemd/system
```
if your log contains this
```
checking for modbus_set_response_timeout... yes
checking for modbus_new_rtu_usb... no
configure: WARNING: Both --with-modbus and --with-usb were requested, and a libmodbus was found, but it seems to not support USB. You may require a custom build per https://github.com/networkupstools/nut/wiki/APC-UPS-with-Modbus-protocol
```
something is wrong. And you will not have USB Modbus Support.
You need to see the following line:
checking for modbus_new_rtu_usb... yes

Go back to libmodbus and find the problem.

```
sudo make install
```
If there are errors you might need to run the make command a secound time.
```
sudo make install
```

# Configuration
```
sudo nano /etc/nut/nut.conf
mode=netserver
```
```
sudo nano /etc/nut/ups.conf
[apcmodbus]
  driver = apc_modbus
  port = auto
  desc = "APC Modbus"
  vendorid = "051D"
  productid = "0003"
  serial = "ASxxxxxxxxxx"
```

```
sudo nano /etc/nut/upsd.conf

LISTEN 127.0.0.1 3493
```
```
sudo systemctl restart nut-server
sudo nut-scanner
sudo systemctl 
sudo service nut-server restart
sudo service nut-client restart
sudo systemctl restart nut-monitor
sudo upsdrvctl stop
sudo upsdrvctl start
```
