# Ideas for tank software

## video
bring the video direct as hardware encoded H264 to the browser.
Draw overlays iin the browser, so we don;t need to use jpeg images.

refs:
[pi-h264-to-browser](https://github.com/dans98/pi-h264-to-browser)

[stream-html5](https://blog.ronnyvdb.net/2019/01/20/howto-stream-html5-video-h264-encoded-video-encapsulated-in-mp4-from-the-raspberry-pi-to-any-web-browser/)

[hardware h264 encode](https://forums.raspberrypi.com/viewtopic.php?t=72435)


Seems we may be able to use a gstreamer pipeline to make the H264, but split off the raw frames, and use [this](https://www.collabora.com/news-and-blog/blog/2017/11/17/ipcpipeline-splitting-a-gstreamer-pipeline-into-multiple-processes/) to send the frames to another gstreamer pipeline?
Then the opwncv can open it's own independent frame input, driven from the H264 pipeline.


maybe something like:
v4l2src device=/dev/video0/ ! video/x-raw,width=1280,height=720 ! x264enc speed-preset=ultrafast tune=zerolatency byte-stream=true bitrate=3000 threads=1 ! h264parse config-interval=1 ! rtph264pay ! udpsink host=127.0.0.1 port=8004

but use omxh264enc for hardware encoding.

[shared mem](https://stackoverflow.com/questions/57506349/gstreamer-pipeline-produce-and-consume-output-in-separate-processes)

should be able to test with

sudo gst-launch-1.0 v4l2src device=/dev/video0/ ! video/x-raw,width=1280,height=720,format=BGR ! gdppay ! shmsink &

and in existing tank code:

gstreamer_str = "sudo gst-launch-1.0 shmsrc ! gdpdepay ! appsink drop=1"
cap = cv2.VideoCapture(gstreamer_str, cv2.CAP_GSTREAMER)


gstreamer installation that worked (see last post):
https://forums.raspberrypi.com/viewtopic.php?t=297550

