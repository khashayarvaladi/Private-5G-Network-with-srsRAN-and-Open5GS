# Installing a 5G Private Network

## Install USRP UHD Driver
I use USRP B210 as software define radio

Update and install dependencies:

[USRP UHD Driver Installation Guide](https://kb.ettus.com/Building_and_Installing_the_USRP_Open-Source_Toolchain_(UHD_and_GNU_Radio)_on_Linux)

```bash
sudo apt-get update
sudo apt-get -y install autoconf automake build-essential ccache cmake cpufrequtils doxygen ethtool fort77 g++ gir1.2-gtk-3.0 git gobject-introspection gpsd gpsd-clients inetutils-tools libasound2-dev libboost-all-dev libcomedi-dev libcppunit-dev libfftw3-bin libfftw3-dev libfftw3-doc libfontconfig1-dev libgmp-dev libgps-dev libgsl-dev liblog4cpp5-dev libncurses5 libncurses5-dev libpulse-dev libqt5opengl5-dev libqwt-qt5-dev libsdl1.2-dev libtool libudev-dev libusb-1.0-0 libusb-1.0-0-dev libusb-dev libxi-dev libxrender-dev libzmq3-dev libzmq5 ncurses-bin python3-cheetah python3-click python3-click-plugins python3-click-threading python3-dev python3-docutils python3-gi python3-gi-cairo python3-gps python3-lxml python3-mako python3-numpy python3-opengl python3-pyqt5 python3-requests python3-scipy python3-setuptools python3-six python3-sphinx python3-yaml python3-zmq python3-ruamel.yaml swig wget

# Clone and build UHD driver
git clone https://github.com/EttusResearch/uhd
cd uhd/host
mkdir build && cd build
cmake ..
make
make test
sudo make install
sudo ldconfig
```

Install UHD package:

```bash
sudo apt-get install libuhd-dev uhd-host
cd /usr/lib/uhd/utils
sudo ./uhd_images_downloader.py
cd /usr/share/uhd/images
```

## Install srsRAN

[srsRAN Installation Guide](https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html#manual-installation)

```bash
cd ~
sudo apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev

git clone https://github.com/srsRAN/srsRAN_Project.git
cd srsRAN_Project
mkdir build && cd build
cmake ../
make -j $(nproc)
make test -j $(nproc)
sudo make install
```

## Install MongoDB

[Open5GS Quickstart Guide](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

```bash
sudo apt update
sudo apt install gnupg
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

## Install Open5GS

```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

Check running components:

```bash
cd /etc/open5gs
ps aux | grep open5gs
```

## Install WebUI

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

sudo apt update
sudo apt install nodejs -y
```

Check WebUI:

```
localhost:9999
Username: admin
Password: 1423
```

## Stop and Start 5G Core

Create startup and shutdown scripts:

```bash
cd /usr/bin
sudo nano stop_5gc
```

Paste the following and save:

```bash
sudo systemctl stop open5gs-smfd
sudo systemctl stop open5gs-amfd
sudo systemctl stop open5gs-upfd
sudo systemctl stop open5gs-nrfd
sudo systemctl stop open5gs-scpd
sudo systemctl stop open5gs-ausfd
sudo systemctl stop open5gs-udmd
sudo systemctl stop open5gs-pcfd
sudo systemctl stop open5gs-nssfd
sudo systemctl stop open5gs-bsfd
sudo systemctl stop open5gs-webui
sudo systemctl stop open5gs-udrd
```

Make it executable:

```bash
sudo chmod +x stop_5gc
```

Create a startup script:

```bash
sudo nano 5gc
```

Paste the following and save:

```bash
sudo systemctl restart open5gs-smfd
sudo systemctl restart open5gs-amfd
sudo systemctl restart open5gs-upfd
sudo systemctl restart open5gs-nrfd
sudo systemctl restart open5gs-scpd
sudo systemctl restart open5gs-ausfd
sudo systemctl restart open5gs-udmd
sudo systemctl restart open5gs-pcfd
sudo systemctl restart open5gs-nssfd
sudo systemctl restart open5gs-bsfd
sudo systemctl restart open5gs-webui
sudo systemctl restart open5gs-udrd
```

Make it executable:

```bash
sudo chmod +x 5gc
```

Start and stop the 5G Core:

```bash
5gc  # Start 5G Core
stop_5gc  # Stop 5G Core
```
##Sznarien for ipconfig
```bash
ifconfig
```
 Scenario 1- OGSTUN interfae is not configured:
Ensure that OGSTUN is configured
observe no interface named ogstun
1.sudo ip tuntap add name ogstun mode tun
2.sudo ip addr add 10.45..0.1/16 dev ogstun
3.sudo ip link set ogstun up
-------------------------------
Scenario 2-OGSTUN interface is with no IP Address
cd etc/open5gs/
sudo ip addr add 10.45.0.1/16 dev ogstun
------------------------------------------
Scenario 3 OGSTUN interface is configured with IP Address
```bash
ifconfig
```
-----------------------------

## Configure Open5GS
##we need sure ip forwarding for IPV4 is enable(for data service)
Set up IP forwarding:

```bash
sudo sysctl -a | grep ip_forward
if net.ipv4_forward = 0
sudo sysctl -w net.ipv4.ip_forward=1
```

Set up NAT forwarding:
##Ensure that IP Table are configured  for NAT forwarding

```bash
sudo iptables -L -n -v -t nat
```
##if ogstun 10.45.0.1/15 is not displayed :
```bash
sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```

Configure `amf.yaml`, `smf.yaml`, and `upf.yaml`:
##uncomment level logger,info and change it to debug
```bash
cd /etc/open5gs
sudo nano amf.yaml
```

Change log level and save.

```bash
sudo nano smf.yaml
```

Modify DNS settings if needed.

Start the 5G core:

```bash
5gc
```
##now in oppen5gs WEBUI we should enter our IMSI(Simcart), OPC and Subscriber Key
localhost:9999
username: admin
password :1423
add subscriber : IMSI
Key und OPC
Session Configuration:
DNN/APN:internet
Typ
e: IPV4
+
 +
PCC Rules
ims
IPv4

and save it
## copy gnb_n3.yml File to your srsran project path/config
cd ~/srsRAN_Project/configs
copy gnb_n3.yml

## Additional Steps and turn on Roaming on phone

- Set APN on UE:
  - Name: Test Network
  - APN: `apn`
  - Protocol: `IPV4`

- Start packet capture:

```bash
sudo tcpdump -i any -w 5gc.pcap
```

- Check logs:

```bash
sudo tail -f /var/log/open5gs/amf.yaml
```
##Connect USRP B210 tp PC
```bash
cd /usr/lib/uhd/utils
ls
sudo ./query_gpsdo_sensors
```
you should now see  GPS and UHD Device time are aligned

##now run gNB with this command
```bash
cd ~/srsRAN_Project/configs
sudo gnb -c gnb_n3.yml
```
## Conclusion

This guide provides a complete step-by-step process to install a private 5G network using USRP, srsRAN, MongoDB, and Open5GS. Ensure all configurations are correct before testing UE connectivity.
