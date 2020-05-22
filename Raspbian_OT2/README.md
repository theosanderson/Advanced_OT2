## Installing Raspbian on the OT2

The OT2 is designed for a reliable user-friendly experience for relatively non-technical users. It therefore runs on a Raspberry Pi in a very locked down version of the Alpine Linux distribution. This is great if you just want to interact with it through the OT app. But if you want to do much crazier things it can be complex. It's hard to access the files on the filesystem, and most parts of the operating system will be reset whenever the machine is rebooted. You can't install "packages", as one typically does on Debian-based distributions.

Luckily, since the OT software is open-source it's not too hard to install your own distribution of choice on the Raspberry Pi. Here are some instructions of what I did to achieve that. This is definitely for advanced users only - there's no reason to do it unless there's something you can't do with the normal system!
## Steps

### Install Raspbian on a spare SD card

Get a spare mini SD card, so you can keep your old OT2 SD card intact and swap back whenever you like.

Install Raspbian Lite on the new SD card using the [Raspberry Pi imager](https://www.raspberrypi.org/downloads/).

With the SD card still connected to your computer (you might need to reinsert it to mount it), make an empty file called `ssh` in the `boot` part of the SD card. This will allow the Pi to be connected to over SSH so that you don't need a monitor.


### Swap the SD cards
At this point you should be in a position to put the SD card currently in the external Raspberry Pi into your robot. Turn off the robot first. The OT website has a [guide](https://support.opentrons.com/en/articles/1841108-changing-sd-card-in-ot-2) on how to do this which you can follow. I have a slightly different model of robot, and perhaps as a result I found it easier to remove the bolts holding the Raspberry Pi board to the back of the robot so it was loose enough to access the SD card.

### Initial connection

Now connect to the OT2 over USB. Note that this actually means connecting to it through its ethernet port, since the USB port is connected to a USB-ethernet adapter which is connected to the Pi's USB port. Turn on the OT2 power and wait a while for the connection to be established (you will see it in your network settings). It may say that your computer has a self-assigned IP. That's OK. Now connect to the Pi (this requires Bonjour, if you don't have a mac, install it on your computer).
```
ssh pi@raspberrypi.local
```

The password is `raspberry`. 

Type `passwd` to set a new password for security. 

Use `raspi-config` and enable WiFi by selecting your country. Unfortunately this setup won't work for enterprise Wifi (see below).

### Get it networked

Get your Pi connected to a network you have access to. Again, this is a bit out of scope. Connecting the Pi to enterprise WiFi networks can actually be a really annoying process, and a wired connection is more reliable. We ended up plugging a USB to ethernet adapter into the Pi, and had to get its MAC address whitelisted by our IT department. We also added:

```
auto eth1
iface eth1 inet dhcp
```

to the bottom of our `/etc/network/interfaces` file to make sure this second connection got initialised. We were fortunate that our IT department assigned a static IP address to this device. (Note: if you do need enterprise WiFi try `sudo apt install network-manager` (you'll want it later anyway) and follow [these instructions](https://askubuntu.com/a/839763)).




## Make robot-specific customizations
### SSH into your robot
You should now be able to access your robot by SSH, as before with the Pi. Now we need to change some things to allow the Pi to talk to robot componenents.

### Enable UART
We need to enable the UART port which the Pi uses to talk to the SmoothieBoard that controls the robots motors. To do this edit `/boot/config.txt` (i.e. `sudo nano /boot/config.txt`) and edit the section at the bottom. We need to add `enable_uart=1` (replacing any other enable_uart line) and to add `dtoverlay=pi3-disable-bt`. The first will enable the UART port, the second will disable the Pi's bluetooth which normally uses the same pins.


### Install OpenTrons API
It's probably good practice to do this in a virtual environment, so let's get virtualenv:
```
sudo apt-get install python3-pip
sudo pip3 install virtualenv 
```

Now let's make our new environment
```
cd ~
virtualenv ot_env
```

And get into it:
```
source ot_env/bin/activate
```

and now install the Opentrons API.

```
pip install opentrons
```

Now we are very nearly there!

We might want to install Jupyter

```
pip install jupyter
```

```
jupyter notebook --ip=YOUR_IP_HERE
```

When writing robot scripts there are a few things we need to do on startup before the robot is ready to use. These are listed in [this example notebook](https://github.com/theosanderson/Advanced_OT2/blob/master/Raspbian_OT2/Example%20script.ipynb) and reflect the things that in a normal robot are carried out by [blinkenlights](https://github.com/Opentrons/buildroot/blob/opentrons-develop/board/opentrons/ot2/rootfs-overlay/usr/bin/ot-blinkenlights) on startup. You could also make your own version of blinkenlights.

### API V2
It helps to set some environmental variables, especially if you want to use the V2 API, so you could add these to your `.bashrc`.
```
export RUNNING_ON_PI=true
export OT_SMOOTHIE_ID=AMA
```

Also the V2 API expects something in `/etc/VERSION.json` (I'm yet to determine how important the value is)

```
sudo echo '{"buildroot_version":1}' > /etc/VERSION.json
```

and then it wants to write stuff to `/data` so it better exist:

```
sudo mkdir /data
sudo chmod 777 /data
```


## OT2 server

Once you've done the steps above you can control your robot in Jupyter or Python but you can't connect using the OT app. The commands below are a work in progress and a service would be better, but they let you use the OT app (you'll need to manually add the robot's IP in advanced settings)!

```
sudo apt install network-manager
```

```
sudo apt install nginx
wget https://raw.githubusercontent.com/Opentrons/buildroot/opentrons-develop/board/opentrons/ot2/rootfs-overlay/etc/nginx/nginx.conf
sudo mv nginx.conf /etc/nginx/nginx.conf
sudo /etc/init.d/nginx restart
```

Simulate balena to change how logs are written:

```
sudo -i
export OT_SYSTEM_VERSION=1
export RUNNING_ON_PI=true
export OT_SMOOTHIE_ID=AMA
source ~pi/ot_env/bin/activate
python -m opentrons.main -U /run/aiohttp.sock --hardware-server
```

