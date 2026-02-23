# Dual Screen Raspberry Pi Video Setup

### IMPORTANT: There are two ways to launch it. Below is a quick one, after is really the one that never stops and never sleeps. 

Making a video kiosk out of Raspberry Pi to play 2 videos on startup.

## 1. Check if OMXPlayer exists

```
omxplayer
```

## 2. Copy Video Files to Desktop

```
/home/pi/Desktop/video-1.mp4

/home/pi/Desktop/video-2.mp4 
```

## 3. Create a Launch Parallel File

This script will launch 2 videos at the same time. 

In the home folder: `launch-parallel.sh`

Then:

```
#!/bin/bash

for cmd in "$@"; do {
  echo "Process \"$cmd\" started";
  $cmd & pid=$!
  PID_LIST+=" $pid";
} done

trap "kill $PID_LIST" SIGINT

echo "Parallel processes have started";

wait $PID_LIST

echo
echo "All processes have completed";

```

## Launching 2 Videos in Parallel

Creating a file `play.sh` in the homne folder that will launch the videos. This file contains:

```
/home/pi/launch_parallel.sh "omxplayer --no-keys --display 2 --loop /home/pi/Desktop/video-1.mp4" "omxplayer --no-keys --display 7 --loop /home/pi/Desktop/video-2.mp4"
```

where `--display 2` means HDMI0 port and `--display 7` means HDMI1 port (on Pi 4).


## Making it launch on startup

In `/etc/systemd/service` you are creating the `player.service` file:

```
nano /etc/systemd/service/player.service`
```

Then adding the following:

```
[Unit]
Description=Launches two videos players
Wants=graphical.target
After=graphical.target

[Service]
Environment=DISPLAY=:0.0
Environment=XAUTHORITY=/home/pi/.Xauthority
Type=simple
ExecStart=/home/pi/launch_parallel.sh "xterm -e `/bin/omxplayer --no-keys --display 2 --loop /home/pi/Desktop/1-EightOS-Confluence-AI.mp4`" "lxterminal -e `/bin/omxplayer --no-keys --display 7 --loop /home/pi/Desktop/1-EightOS-Confluence-AI.mp4`"
Restart=always
User=pi
Group=pi

[Install]
WantedBy=graphical.target
```

Then 

```
systemctl enable player.service
dystemctl activate player.service

```


## Quitting the Service

Usually, it should work to quit if you:
press q
press Alt+F2 or Alt+F4

However, it doesn't always work. 

In that case, you should have access to Pi via SSH (e.g. if you plug it in Ethernet or if it catches WiFi) and run 

```
sudo systemctl disable player.service
```

If there's no possibility to go via SSH, install `Paragon extFS` on your Mac, then mount the SD card on the Mac, and open manually:

```
ls /Volumes/rootfs/etc/systemd/system/
```

Where `/rootfs` is the path to the SD card rom Mac

This lists `systemctl` services.

Then you "mask" the service so it doesn't auto-launch by creating a fake symlink to it:

```
sudo ln -s /dev/null /Volumes/rootfs/etc/systemd/system/graphical.target.wants/player.service
```

NOTE: This above may not work, then use:

```
sudo mv /Volumes/rootfs/etc/systemd/system/player.service /Volumes/rootfs/etc/systemd/system/player.service.bak
```

(if you're in SSH, run):

```
sudo systemctl mask player.service
```

Then to "unmmask" you run:

```
sudo rm /etc/systemd/system/graphical.target.wants/player.service
```

(if you're in SSH, run):

```
sudo systemctl unmask player.service
```

## No Sleep Setup

This is additional setup that avoids the service going to sleep and relaunches it. For some reason it also automatically starts the service upon startup. 

Below we assume we're doing it from Pi. If we're on Mac, add `/Volumes/rootfs` prefix (or whatever the disk is).

1. Check if there's a `nosleep.service` file somewhere:

```
find /lib/systemd/system/ -name "nosleep*"
find /etc/systemd/system/ -name "nosleep*"
```

2. Once found, go to step 3). Or create if it doesn't exist:

```
nano /etc/systemd/system/nosleep.service
```

3. Add the following to the file

```
[Unit]
Description=Chromium Kiosk
Wants=graphical.target
After=graphical.target

[Service]
Environment=XAUTHORITY=/home/pi/.Xauthority
Type=simple
ExecStart=/bin/bash /home/pi/nosleep.sh
Restart=on-abort
User=pi
Group=pi

[Install]
WantedBy=graphical.target

```

4. Create a symlink in `graphical.target.wants`

```
sudo systemctl daemon-reload
sudo systemctl enable nosleep.service
```

5. Create the `nosleep.sh` script

```
nano /home/pi/nosleep.sh
```

6. Add this to the file

```
#!/bin/bash
xset s noblank
xset s off
xset -dpms

sleep 30

/home/pi/play.sh
```

4. Make it executable:
```
chmod +x /home/pi/nosleep.sh
```

5. Check if exists / or create the actual `play.sh` file:

```
nano /home/pi/play.sh
```

Then add this content there:
```
/home/pi/launch_parallel.sh "omxplayer --no-keys --display 2 --loop /home/pi/Desktop/BMML-Body.mp4" "omxplayer --no-keys --display 7 --loop /home/pi/Desktop/BBML-Machine.mp4"
```

IMPORTANT - if you want to be able to quit the video, do not add --no-keys

```
/home/pi/launch_parallel.sh "omxplayer --no-keys --display 2 --loop /home/pi/Desktop/BMML-Body.mp4" "omxplayer --display 7 --loop /home/pi/Desktop/BBML-Machine.mp4"
```

Or also just comment out for now:

```
# /home/pi/launch_parallel.sh "omxplayer --no-keys --display 2 --loop /home/pi/Desktop/BMML-Body.mp4" "omxplayer --display 7 --loop /home/pi/Desktop/BBML-Machine.mp4"
```

6. Check if exists / create the `launch_parallel.sh` file:

```
nano /home/pi/launch_parallel.sh
```

Add this: 

```
#!/bin/bash

for cmd in "$@"; do {
  echo "Process \"$cmd\" started";
  $cmd & pid=$!
  PID_LIST+=" $pid";
} done

trap "kill $PID_LIST" SIGINT

echo "Parallel processes have started";

wait $PID_LIST

echo
echo "All processes have completed";
```

Make it executable:
```
chmod +x /home/pi/launch_parallel.sh
```

6. To verify it's enabled:
```
sudo systemctl status nosleep.service
```


