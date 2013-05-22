graphping
=========

Send ping statistics for multiple hosts to Graphite

Description
-----------

This Python program takes a list of hostnames to collect ping statistics for.
It uses the 'fping' program to do the real work, and send the results to
Graphite. By default, it detaches itself from the console and continues to run
in the background.

Until stopped, the program starts an 'fping' for all the given target and sends
30 ICMP ping packets. The results (min, max, average and mean ping round-trip
times, as well as packet loss percentage) are sent to a Graphite server, which
by default should be running on localhost:2003. It then sleeps until the next
minute on the system clock before it starts another 'fping'. This way, you get
one measurement per minute. This is not configurable at this time.

Requirements
------------

* fping
* python-daemon
* A Graphite server

Configuration
-------------

Set up a Carbon/Whisper retention schema for the ping statistics, keeping one
sample per minute for some time, and optionally some aggregations:

```
[ping]
pattern = ^ping\.
retentions = 1m:1d,5m:31d,60m:366d
```

At the top of the script, there are some configurable items:

```
# number of ping packets to send
packets = 30

# Graphite
g_host = 'localhost'
g_port = 2003
g_prefix = 'ping'

# System
fping = '/usr/sbin/fping'
logfile = '/tmp/ping-graphite.log'
```

Usage
-----

`./graphping &lt;target&grt; [&lt;target&grt; ...]`
