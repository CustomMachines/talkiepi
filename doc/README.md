# TalkiePi Setup Guide

Here are the steps required to setup a TalkiePi with a Raspberry Pi Zero W and SeeedStudio ReSpeaker 2-Mic Pi HAT. These instructions assume you are using a Windows PC to configure the RPi.

![Talkiepi internals](_DSF9672.jpg "Talkiepi internals")

By default TalkiePi will run without any arguments, it will autogenerate a username and then connect to Daniel Chote's mumble server.
You can change this behavior by appending commandline arguments of `-server YOUR_SERVER_ADDRESS`, `-username YOUR_USERNAME` to the ExecStart line in `/etc/systemd/system/mumble.service` once installed.

TalkiePi will also accept arguments for `-password`, `-insecure`, `-certificate` and `-channel`, all defined in `cmd/talkiepi/main.go`, if you run your own mumble server, these will be self explanatory.

## Hardware Requirements
* (1) Raspberry Pi Zero W
* (1) [SeeedStudio ReSpeaker 2-Mic Pi HAT](http://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/)
* (1) [Stereo speaker (3W, 8Ω) with JST connector](https://core-electronics.com.au/stereo-enclosed-speaker-3w-8.html)
* (1) MicroSD card, 16GB, class 10 with SD card adapter
* (1) Power adapter with micro USB cable
* (4) [M2.5 Brass Standoffs](https://core-electronics.com.au/brass-m2-5-standoffs-for-pi-hats-black-plated-pack-of-2.html)
* (4) M2.5 x 6 mm screws

## Install Operating System

Download the latest version of [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/); older versions available [here](http://downloads.raspberrypi.org/raspbian_lite/images/)

These instructions have been tested with:

~~~
Based on: Debian Stretch
Version: November 2017
Release date: 2017-11-29
Kernel version: 4.9
~~~

Write the image to a miroSD card with [Etcher](https://etcher.io/). Note, after the write is complete Windows will complain that the microSD card needs to be formatted. This message can be ignored. It is just that Windows cannot read the Linux partitions, but Windows can read the **Boot** partition which is the only one we need access to. 

Create two files on the **Boot** partition of the microSD card:

~~~
wpa_supplicant.conf
ssh
~~~

Below is a template for the wpa_supplicant.conf file. Please edit these details to match one or more wireless networks you intend to use. You can define as many networks as you like, if using only one network please delete the second network definition. Also, the two letter country code should match the country you are operating in.

Note, the file must be formatted with Unix style line end characters (i.e. LF not CR LF). [Notepad++](https://notepad-plus-plus.org/) is an example of a Windows based text editor that supports this.

~~~
country=AU
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    id_str="descriptive name for network 2"
    ssid="SSID2"
    psk="password2"
    priority=2
}

network={
    id_str="descriptive name for network 1"
    ssid="SSID1"
    psk="password1"
    priority=1
}
~~~

The ssh file should be empty.

Insert the microSD card into the RPi and apply power. The RPi will boot and attempt to connect one of the WiFi networks defined above with SSH enabled.

Use [Advanced Port Scanner](https://www.advanced-port-scanner.com/) or your router’s web interface to find the IP address of the RPi.

Use [PuTTY](http://www.putty.org/) to make an SSH connection to the RPi.

The default login credentials are:

~~~
rappberrypi login: pi
password: raspberry
~~~

Update repositories and upgrade software with:

~~~
$sudo apt update
$sudo apt upgrade
~~~

## Raspi-Config

Start the RPi configuration tool with this command:

~~~
$sudo raspi-config
~~~

Make the follow configuration changes:

1. Change User Password > Ok > YOUR_PASSWORD > YOUR_PASSWORD
1. Network Options > N1 Hostname > Ok > YOU_HOSTNAME > Ok
1. Localisation Options > I2 Change Timezone > YOUR_COUNTRY > YOUR_CITY
1. Localisation Options > I4 Change WiFi Country > YOUR_COUNTRY_CODE > Ok
1. Interfacing Options > P4 SPI > Yes > Ok
1. Advanced Options > A1 Expand Filesystem > Ok
1. Finish > Reboot? Yes

## Disable HDMI

To save power the HDMI output can be disabled at boot.

~~~
$sudo nano /etc/rc.local
~~~

Add the following lines above exit 0:

~~~
# Disable HDMI
/usr/bin/tvservice -o
~~~

## Git
Install Git with this command:

~~~
$sudo apt install git
~~~

## SeeedStudio ReSpeaker 2-Mic HAT
Install drivers with these commands:

~~~
$git clone https://github.com/respeaker/seeed-voicecard
$cd seeed-voicecard
$sudo ./install.sh
$sudo reboot
~~~

### Optional Audio Tests

Test the speaker and mic with:

~~~
$arecord -f cd | aplay
~~~

Test speakers with the following command (CTRL-C to quit).

~~~
$speaker-test
~~~

Test the microphone with:

~~~
$arecord test.wav
$aplay test.wav
$rm test.wav
~~~

Audio configuration is done with:

~~~
$alsamixer
~~~

## Install Prerequisites

Install prerequisites with:
~~~
$sudo apt install libopenal-dev libopus-dev
~~~

## Golang

Install Golang with:

~~~
$wget https://storage.googleapis.com/golang/go1.9.3.linux-armv6l.tar.gz
$sudo tar -C /usr/local -xzf go1.9.3.linux-armv6l.tar.gz
$mkdir ~/go
$cd go
$mkdir src
$mkdir bin
$cd
~~~

Edit the ~/.profile file with:

~~~
$nano ~/.profile
~~~

Add the following text to the end of the file:

~~~
export PATH=$PATH:/usr/local/go/bin:~/go/bin
export GOPATH=$HOME/go
export GOBIN=$HOME/go/bin
export PATH=$PATH:$GOPATH/bin
~~~

Reload the configuration with:

~~~
$source ~/.profile
~~~

## Install TalkiePi

Install TalkiePi with:

~~~
$cd $GOPATH/src
$go get periph.io/x/periph/cmd/...
$go get github.com/dchote/gopus
$go get github.com/CustomMachines/talkiepi
$go build -o $GOPATH/bin/talkiepi $GOPATH/src/github.com/CustomMachines/talkiepi/cmd/talkiepi/main.go
~~~

The RPi Zero needs libopenal complied without ARM NEON support. These packages can be found in the [workarounds](https://github.com/CustomMachines/talkiepi/blob/master/workarounds) directory of this repo. They can be installed over the existing libopenal libraries.

~~~
$cd ~/go/src/github.com/CustomMachines/talkiepi/workarounds
$sudo apt install ./libopenal-dev_1.17.2-4_armhf.deb
$sudo apt install ./libopenal1_1.17.2-4_armhf.deb
$sudo apt install ./libopenal-data_1.17.2-4_all.deb
$sudo apt install ./libopenal1-dbg_1.17.2-4_armhf.deb
~~~

## Create a Certificate

This is optional, mainly if you want to register your talkiepi against a mumble server and apply ACLs.

~~~
$cd
$openssl genrsa -aes256 -out key.pem
~~~

Enter a simple passphrase, its ok, we will remove it shortly...

~~~
$openssl req -new -x509 -key key.pem -out cert.pem -days 1095
~~~

Enter your passphrase again, and fill out the certificate info as much as you like, its not really that important if you're just hacking around with this.

~~~
$openssl rsa -in key.pem -out nopasskey.pem
~~~

Enter your password for the last time.

~~~
$cat nopasskey.pem cert.pem > mumble.pem
~~~

## Systemd Service

Copy the service file to the required folder.

~~~
$cd
$sudo cp /home/pi/go/src/github.com/CustomMachines/talkiepi/conf/systemd/mumble.service /etc/systemd/system/mumble.service
~~~

Edit the service file with:

~~~
$sudo nano /etc/systemd/system/mumble.service
~~~

Edit the **YOUR_SERVER:PORT** to match the one you wish to use. 

Edit the **YOUR_USERNAME** match the one created above.

~~~

[Unit]
Description = Mumble Client
Requires = systemd-user-sessions.service network.target sound.target
After = multi-user.target

[Service]
User = pi
Group = pi
Type = simple
ExecStart = /home/pi/go/bin/talkiepi -server YOUR_SERVER:PORT -username YOUR_USERNAME -certificate /home/pi/mumble.pem -channel Root
Restart = always
RestartSec = 5

[Install]
WantedBy = multi-user.target

~~~

Test the TalkiePi application with:

~~~
$$GOPATH/bin/talkiepi -server YOUR_SERVER:PORT -username YOUR_USERNAME -certificate /home/pi/mumble.pem -channel Root
~~~

To quit the application press (CTRL+C).

If all is well the Systemd service can be enabled with:

~~~
$sudo systemctl enable mumble.service
~~~

To start the service without rebooting run:

~~~
$sudo systemctl daemon-reload
$sudo systemctl restart mumble.service
~~~
