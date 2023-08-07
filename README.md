# Dual Screen Raspberry Pi Video Setup

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

In `/etc/systemd/service`:

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


## Cheatsheet

To quit: press q

If the above works, you can now 
