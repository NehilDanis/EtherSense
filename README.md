# EtherSense
Ethernet client and server for RealSense using python's Asyncore.

## Prerequisites

* Python 3.10 

## How to Start?
In the jetson board clone this repo with a specific branch and go into the directory. This branch is created for using ethersense on Jetson board

### Cloning to repo with correct branch
```
git clone --branch ethersense-for-jetson https://github.com/NehilDanis/EtherSense.git
cd EtherSense/
```

Still check if you are in the correct branch by running:

```
git branch
```
You should see ethersense-for-jetson shown in green.If not please switch to this branch.

### Create python environment

If you are not already in the EtherSense directory go into that, and run the following commands.

```
python3.10 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Once the virtual environment is created and the requiremenets are installed, run the following on server and client machines.

#### On Server machine

This repo assumes that you are using aarch64 machine on the Server side. The realsense was not pip installed but pre compiled for aarch64 achitecture with python3.10 support. But in can you use x86 or so, then I believe you can directly add realsese as requirement to the requirements txt file and you do not need to use the pre compiled binaries for realsense.

Connect a realsense device to the server machine. Currently this version does not support selecting different devices, so if you have multiple realsense devices connected to server machine the program will pick one of them.

Once in the virtual environment run the following:

```
python EtherSenseServer.py
```

#### On Client machine

Client machine does not need to have realsense, hence does not matter if you are on x86 or aarch64. In my case I had my server on aarch64 and client on x86 machines. 

Once in the virtual environment run the following:

```
python EtherSenseClient.py
```

On the client machine you should be able to see the depth and rgb images captured from the camera.


## Overview
Mulicast broadcast is used to establish connections to servers that are present on the network. 
Once a server receives a request for connection from a client, Asyncore is used to establish a TCP connection for each server. 
Frames are collected from the camera using librealsense pipeline. It is then resized and send in smaller chucks as to conform with TCP.

### UpBoard PoE 
Below shows use of a PoE switch and PoE breakout devices(avalible from online retailers) powering each dedicated UpBoard: 
This configuration should allow for a number of RealSense cameras to be connected over distances greater then 30m 
![Example Image](https://github.com/krejov100/EtherSense/blob/master/UpBoardSwitch.JPG)
The 5 RealSense cameras are connected to each UpBoard using the provided USB3 cables.

### Client Window
Below shows the result of having connected to five cameras over the local network: 
![Example Image](https://github.com/krejov100/EtherSense/blob/master/MultiCameraEthernet.png)
The window titles indicate the port which the frames are being received over.

## Error Logging
Errors are piped to a log file stored in /tmp/error.log as part of the command that is setup in /etc/crontab

## NOTES

### Power Considerations
The UpBoards require a 5v 4Amp power supply. When using PoE breakout adaptors I have found some stability issues, for example the device kernel can crash when the HDMI port is connected. As such I recommend running the UpBoard as a headless server when using PoE. 

### Network bandwidth
It is currently very easy to saturate the bandwidth of the Ethernet connection I have tested 5 servers connected to the same client without issue beyond limited framerate:

cfg.enable_stream(rs.stream.depth, 640, 480, rs.format.z16, 30)

self.decimate_filter.set_option(rs.option.filter_magnitude, 2)

There are a number of strategies that can be used to increase this bandwidth but are left to the user for brevity and the specific tradeoff for your application, these include:

Transmitting frames using UDP and allowing for frame drop, this requires implementation of packet ordering.

Reducing the depth channel to 8bit.

Reducing the resolution further.

The addition of compression, either frame wise or better still temporal.

Local recording of the depth data into a buffer, with asynchronous frame transfer.
 
## TroubleShooting Tips

I first of all suggest installing and configuring openssh-server on each of the UpBoards allowing remote connection from the client machine.

Check that the UpBoards are avalible on the local network using "nmap -sP 192.168.2.*"

Check that the server is running on the UpBoard using "ps -eaf | grep "python EtherSenseServer.py"

Finally check the log file at /tmp/error.log

There might still be some conditions where the Server is running but not in a state to transmit, help in narrowing these cases would be much appreciated. 

