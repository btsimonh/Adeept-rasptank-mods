# video capture, processing and streaming

##Summary:

On RPi3b with bullseye, ~45% cpu to stream h264 to webrtc with decent low latency with raw video frames passing through OpenCV (640x480).  ~1.5Mbits
(648x480@25 -> ~45%cpu on node)
(648x480@60 -> ~98%cpu on node)

using @u4/opencv4nodejs to build opencv (~2.5 hours, make swap 2048 first).
using sudo apt-get install janus to get janus.
editing /etc/janus/janus.jcfg for ip.
uncomment h264 example in /etc/janus/janus.plugin.streaming.jcfg
restart janus (sudo service janus restart), or stop and run on command line for debug.

OpenCV capture/writer lines:
```
const Vcap = new cv2.VideoCapture("v4l2src ! video/x-raw, format=BGR, width=640, height=480, framerate=25/1 ! appsink", cv2.CAP_GSTREAMER);

// to local janus
const sink = new cv2.VideoWriter("appsrc  caps=video/x-raw,format=BGR,width=640,height=480,framerate=25/1 ! videoconvert ! v4l2h264enc ! video/x-h264,level=(string)3 ! h264parse config-interval=1 ! rtph264pay ! udpsink host=127.0.0.1 port=8004 sync=false", 0, 25, new cv2.Size(640, 480));
```

video viewer:
https://github.com/agonza1/WebRTC-Live-Streaming-with-AI
clone repo, and edit js/streaming.js:
```
if(window.location.protocol === 'http:')
    server = "http://"+window.location.hostname+":8088/janus";
else
    server = "https://"+window.location.hostname+":8089/janus";
```


OpenCV doing edge detect (640x480@25, brg->grey->canny->grey->bgr) -> ~190% cpu for node.
Still gets 25fps....

Note: using writeAsync reduces normal cpu use to 25% !!


## todo

1/ make opencv4nodejs available to node-red.
    move to global folder?  or to node-red install?
     this worked:
     cp -r node_modules/* ../.node-red/node_modules/

     ensure in settings.js:
        functionGlobalContext: {
            require:require,
        },

2/ proxy janus API in node-red.
    working.
3/ reproduce viewing page served from node-red.
    done.
4/ auto-start video.
    Needs a gesture, so enabled controls - you must hit play.
5/ attempt higher resolution.   
    at 1280x720, the system crashed (driver blows up, stalling node-red, required reboot - reboot takes AGES)
    try increase gpu_memory in /boot/config.txt from 128 to 256? - does not fix crash.



# Notes made whilst investigating

## Example of gstreamer input pipelines for opencv:

"rpicamsrc exposure-mode=off shutter-speed=50 preview=false ! "
            "video/x-raw, width=(int)" + std::to_string(capture_width) + ", height=(int)" + std::to_string(capture_height) + ", framerate=(fraction)" + std::to_string(framerate) +"/1 !"
            " videoscale ! video/x-raw,"
            " width=(int)" + std::to_string(display_width) + ","
            " height=(int)" + std::to_string(display_height) + " ! "
            " videoconvert ! video/x-raw, format=(string)BGR ! appsink";

"rpicamsrc exposure-mode=off shutter-speed=50 preview=false ! 
video/x-raw, width=(int)640, height=(int)480, framerate=(fraction)30/1 !
videoscale ! video/x-raw, width=(int)640, height=(int)480 !
 videoconvert ! video/x-raw, format=(string)BGR ! appsink"




! x264enc ! h264parse ! mpegtsmux ! rtpmp2tpay ! udpsink



We want to take the raw input in opencv, but we'd also like to stream H264 to a browser with low latency.

(but use omxh264enc for hardware encoding.- depreciated)

bullseye info:  video changed a lot!!!
https://qengineering.eu/install-gstreamer-1.18-on-raspberry-pi-4.html


tests:

"v4l2src device=/dev/video0/ ! video/x-raw,width=1280,height=720 ! x264enc speed-preset=ultrafast tune=zerolatency byte-stream=true bitrate=3000 threads=1 ! h264parse config-interval=1 ! rtph264pay ! udpsink host=127.0.0.1 port=8004"

working:
// working stream to vlc - 14% CPU on RPi3b
// works at 640x480, but not at 1280x720
// latency ~1.5s
gst-launch-1.0 -vvv v4l2src device=/dev/video0 ! 'video/x-raw,width=640,height=480,framerate=30/1,format=UYVY' ! videoconvert ! videoscale ! queue ! v4l2h264enc ! 'video/x-h264,level=(string)4' ! h264parse  config-interval=-1 ! mpegtsmux ! udpsink host=192.168.1.163 port=5555


working (send to two): 
gst-launch-1.0 -vvv v4l2src device=/dev/video0 ! 'video/x-raw,width=640,height=480,framerate=30/1,format=UYVY' ! videoconvert ! videoscale ! queue ! v4l2h264enc ! 'video/x-h264,level=(string)4' ! h264parse  config-interval=-1 ! mpegtsmux ! tee name=t ! queue ! udpsink host=192.168.1.163 port=5555 t. ! queue ! udpsink host=192.168.1.163 port=5556

works, 
gst-launch-1.0 -vvv v4l2src device=/dev/video0 ! 'video/x-raw,width=640,height=480,framerate=30/1,format=UYVY' ! videoconvert ! videoscale ! tee name=t ! queue ! v4l2h264enc ! 'video/x-h264,level=(string)4' ! h264parse  config-interval=-1 ! mpegtsmux ! queue ! udpsink host=192.168.1.163 port=5555 t. ! queue ! fakesink



also see:
https://www.linux-projects.org/uv4l/

https://github.com/tryolabs/vierjavibot


Works in chrome:
uv4l --auto-video_nr --driver raspicam --server-option '--port=9000'

kill with:
pkill uv4l


// install opencv c bindings (4.5.1): !!! not good for opencv4nodejs !!!!
NOT GOOD... sudo apt install libopencv-dev

libraries are here:
/usr/lib/arm-linux-gnueabihf

setting swap size:
https://nebl.io/neblio-university/enabling-increasing-raspberry-pi-swap/
(increased to 2048 - seems to build better now)
also, stop node-red before building...


Building from source attempt1:
Swap still at 2Gbytes
node-red-stop
mkdir ocv4n
cd ocv4n
npm init
npm install --save @u4/opencv4nodejs
build-opencv --version 4.6.0 rebuild
(wait ~2.5 hours)
(works!!!)



Gstreamer direct to VLC (some lag, 14% cpu):
gst-launch-1.0 -v v4l2src do-timestamp=true ! video/x-raw,framerate=30/1,width=640,height=480 ! v4l2h264enc ! 'video/x-h264,level=(string)3' ! h264parse config-interval=1 ! mpegtsmux ! udpsink host=192.168.1.163 port=5555 sync=false

(note that the single quotes are ONLY needed for the command line version!!!!)

NodeJS working (39% cpu on RPi3 with frame number written to frame):

```
let cv2 = require('@u4/opencv4nodejs');
const Vcap = new cv2.VideoCapture("v4l2src ! video/x-raw, format=BGR, width=640, height=480, framerate=25/1 ! appsink", cv2.CAP_GSTREAMER);
// for janus
//const sink = new cv2.VideoWriter('appsrc ! videoconvert ! video/x-raw,format=I420,width=640,height=480,framerate=25/1 ! x264enc ! rtph264pay ! udpsink host=192.168.1.163 port=5555', 0, 25, new cv2.Size(640, 480));

// stream direct to VLC over TS with some lag:
const sink = new cv2.VideoWriter("appsrc  caps=video/x-raw,format=BGR,width=640,height=480,framerate=25/1 ! videoconvert ! v4l2h264enc ! video/x-h264,level=(string)3 ! h264parse config-interval=1 ! mpegtsmux ! udpsink host=192.168.1.163 port=5555 sync=false", 0, 25, new cv2.Size(640, 480));

let frames = 0;
let start = (new Date()).valueOf();

let readone = ()=>{
    Vcap.readAsync()
	.then((frame)=>{
      const text = ''+frames;
      const org = new cv2.Point(20, 20);
      const fontFace = cv2.FONT_HERSHEY_SIMPLEX;
      const fontScale = 0.5;
      const textColor = new cv2.Vec(255, 0, 0);
      const thickness = 2;

      // put text on the object
      frame.putText(text, org, fontFace, fontScale, textColor, thickness);

	sink.write(frame);
	frames++;
	let now = (new Date()).valueOf();
	let delay = (now - start)/1000;
	if (delay === 0) delay += 0.04;
	let fps = frames/delay;
	console.log('fps'+fps);
	setTimeout(readone, 1);
	})
	.catch((e)=>{console.error(e)});
}

readone();
```

sudo apt-get install janus ?
Seems to have worked....
and started a service - config in /etc/janus/
Janus example:
https://github.com/agonza1/WebRTC-Live-Streaming-with-AI
here, janus is accepting data on rtph264pay ! host=127.0.0.1 port=8004
(un-comment h264-sample in sudo nano /etc/janus/janus.plugin.streaming.jcfg)

working sending to janus line:
const sink = new cv2.VideoWriter("appsrc  caps=video/x-raw,format=BGR,width=640,height=480,framerate=25/1 ! videoconvert ! v4l2h264enc ! video/x-h264,level=(string)3 ! h264parse config-interval=1 ! rtph264pay ! udpsink host=127.0.0.1 port=8004 sync=false", 0, 25, new cv2.Size(640, 480));

note h264parse config-interval=1 is important....

Overall on RPi3b node=38% cpu, janus 6% cpu/stream. (inserting frame number on OpenCV passthrough).

