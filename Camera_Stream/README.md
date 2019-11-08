# Stream over HTTP from the OT2 camera

The OT2 has a webcam! It would be great to stream it over HTTP.

These instructions rely on your OT2 running Raspbian as described [here](https://github.com/theosanderson/Advanced_OT2/tree/master/Raspbian_OT2).

Install motion:
```sudo apt-get install motion```

Make a configuration file:
```
mkdir ~/.motion
nano ~/.motion/motion.conf
```

Paste these settings in:

```
webcam_port 8081
webcam_localhost off
stream_port 8080
stream_localhost off
output_pictures off
framerate 20
stream_quality 70
width 640
height 480
ffmpeg_video_codec mpeg4
stream_maxrate 10
```

Start the stream:
```
motion
```

Then access the camera through your browser at:
```
http://[YOUR_IP_ADDRESS]:8080/
```
