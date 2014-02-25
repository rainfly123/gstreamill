# INTRODUCTION

## Overview

gstreamill is an open source, GPL licensed "stream mill" based on gstreamer-1.0. To send query to a gstreamill server to provoke a live encode, transcode or recoder job. query is carried in http post, post body is the job to be provoke, job is descripted in json.

## Highlight

   * hls, http progressive streaminig, udp output.
   * hls push mode output via webdav.
   * Multi-Rate with GOP Alignment.
   * RESTful management interface, allowing easy integration into operator environment.
   * Job is descript in json.
   * Job run in subprocess, and auto restart on error.
   * Base on gstreamer and easy to extend.

## Application

    IP --------+ 
               |                      +------- UDP
    CVBS ------+    +------------+    |               +---- http put
               +----+ gstreamill +----+------ M3U8----+
    SDI -------+    +------+-----+    |               +---- http get
               |           |          +------ HTTP
    LIVE ------+           |
                           |
             REST interface (json over http post)

# INSTALL

gstreamill have been tested in ubuntu 13.10.

## Prerequisites

   * gnome-common
   * autoconf
   * automake
   * libtool
   * libgstreamer1.0-dev
   * libgstreamer-plugins-base1.0-dev

## Compile and install

    ./autogen.sh
    ./configure (--help)
    make
    make install

# USING GSTREAMILL

## Prerequisites

   * gstreamer1.0-plugins-ugly
   * gstreamer1.0-plugins-bad
   * gstreamer1.0-plugins-good
   * gstreamer1.0-plugins-base

## command line

* help

    gstreamill -h

        Usage:
          gstreamill [OPTION...]
    
          Help Options:
            -h, --help                        Show help options
            --help-all                        Show all help options
            --help-gst                        Show GStreamer Options
    
          Application Options:
            -j, --job                         -j /full/path/to/job.file: Specify a job file, full path is must.
            -l, --log                         -l /full/path/to/log: Specify log path, full path is must.
            -m, --httpmgmt                    -m http managment service address.
            -a, --httpstreaming               -a http streaming service address.
            -s, --stop                        Stop gstreamill.
            -v, --version                     display version information and exit.

* start gstreamill

    gstreamill

* stop gstreamill

    gstreamill -s

* invoke a job

default management port is 20118, invoke test job as flowing:

    curl -H "Content-Type: application/json" --data @examples/test.job http://localhost:20118/start

* invode a job in foreground:

    gstreamill -j job_descript_file

* access output use vlc

    http://localhost:20119/live/test/encoder/0

or

    http://localhost:20119/live/test/playlist.m3u8

## management interface

* start a job over http use curl

    curl -H "Content-Type: application/json" --data @examples/test.job http://localhost:20118/start

test.job is job description in json, can be found in examples directory.

* stop a job

    curl http://localhost:20118/stop/job_name

job_name is value of the "name" field in job description.

* query gstreamill stat:

    curl http://localhost:20118/stat/gstreamill
    curl http://localhost:20118/stat/gstreamill/livejob/test

* query gstreamer information:

    curl http://localhost:20118/stat/gstreamer[/plugin]

## output

job name is the value of name of job description.

* http progressive streaming

    http://localhost:20119/live/job name/encoder/encoder_index

* hls

    http://localhost:20119/live/job name/playlist.m3u8

    http://localhost:20119/live/job name/encoder/encoder_index/playlist.m3u8

* udp

    udp://@ip:port

## JOB DESCRIPTION

JOB file structure:

    {
        'name' : 'cctv2',
        'debug' : '3',
        'source' : {
            ...
        },
        'encoders' : [
            ...
        ],
        'm3u8streaming' : {
            ...
        }
    }

name : job name

debug : debug option, reference gst-launch gst-debug, optional.

source : source of encoders.

encoders : encoders.

m3u8streaming : m3u8 streaming

structure of source:

    "source" : {
        "elements" : {
            ...
        },
        "bins" : [
            ...
        ]
    }

structure of elements:

    "elements" : {
        "element" : {
            "property" : {
                ...
            },
            "caps" : "..."
        },
        ...
    }

structure of bins:

    "bins" : [
        bin,
        bin,
        ...
    ]

bins is an array of bin, syntax of bin is like gst-launch, for example:

    "udpsrc ! queue ! tsdemux name=demuxer",
    "demuxer.video ! queue ! mpeg2dec ! queue ! appsink name = video",
    "demuxer.audio ! mpegaudioparse ! queue ! mad ! queue ! appsink name = audio" 

in this example, first bin with tsdemux has sometimes pads, second and third bin link with first bin: demuxer.video and demuxer.audio. second bin with appsink named video and third bin with appsink named audio. source bins must have bin with appsink that is corespond endoders' source.

structure of encoders:

    "encoders" : [
        encoder,
        ...
    ]

encoders is an array of encoder, encoder is an object, every encoder correspneds an encode output, that means gstreamill support mutilty bitrate output.

structure of encoder:

    {
        "elements" : {
            ...
        },
        "bins" : [
            ...
        ],
        "udpstreaming" : "uri"
    }

elements and bins is just the same as source structure in syntax, the differnce is encoder bins must have bins with appsrc element, appsrc must have name property, the value of name is the same as appsink name value in source bins. udpstreaming uri is udp streaming output uri, it's optional.

m3u8streaming is hls output, it's optional:

    "m3u8streaming" : {
        "version" : 3,
        "window-size" : 10,
        "segment-duration" : 3.00,
        "push-server-uri" : "http://192.168.56.3/test"
    }

There are examples in examples directory of source.

Talk about gstreamill on [gstreamill](https://groups.google.com/forum/#!forum/gstreamill) or report a bug on [issues](https://github.com/zhangping/gstreamill/issues) page.
