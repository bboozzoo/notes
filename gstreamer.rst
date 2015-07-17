gstreamer
=========

Defined types:
http://gstreamer.freedesktop.org/data/doc/gstreamer/head/pwg/html/section-types-definitions.html

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

Coding
------

`multiudpsink`
~~~~~~~~~~~~

Parent class of `udpsink`. Supports streaming to multiple clients. Add
clients by triggering emission of `add`, remove by emitting `remove`
signal.

Connect to source named `stream`::

  Gst.Pipeline pipe = (Gst.Pipeline)
                        Gst.parse_launch("videotestsrc ! x264enc  ! rtph264pay name=stream");
  var endpoint = pipe.get_by_name("stream");
  var udpsink = Gst.ElementFactory.make("multiudpsink", null);
  // add to pipeline
  pipe.add(udpsink);
  // link to sink
  endpoint.link(udpsink);


Trigger client add::

  # C
  g_signal_emit_by_name(G_OBJECT(src), "add", host, port, NULL);

  # Vala
  Signal.emit_by_name(udpsink, "add", host, port);
