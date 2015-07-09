gstreamer
=========

Debugging
---------

Dump pipeline::

  GST_DEBUG_DUMP_DOT_DIR=$PWD gst-launch-1.0 ...

Debugs::

  GST_DEBUG=4 gst-launch-1.0 ...

Selective debug::

  gst-launch-1.0 --gst-debug=imxv4l2videosrc:5 imxv4l2videosrc ...


Streaming
---------

RTP
~~~

As H.264 using i.MX VPU encoder::

  GST_DEBUG=3 gst-launch-1.0 --gst-debug=imxv4l2videosrc:5 imxv4l2videosrc device=/dev/video0 ! \
      imxipuvideotransform ! video/x-raw,width=1280,height=720 ! imxvpuenc_h264 ! rtph264pay ! \
      typefind ! udpsink host=172.27.0.197 port=9999

RTSP
~~~~

Use `gstreamer1.0-rtsp-server`. Example binary `test-launch` can
launch any pipeline and provide RTSP endpoint.

HTTP
~~~~

Use `http-launch`: https://github.com/sdroege/http-launch
