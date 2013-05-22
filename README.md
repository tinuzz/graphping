graphping
=========

Send ping statistics for multiple hosts to Graphite

Description
-----------

This Python program takes a list of hostnames to collect ping statistics for.
It uses the 'fping' program to do the real work, and send the results to
Graphite. By default, it detaches itself from the console and continues to run
in the background.

Until stopped, the program starts an 'fping' for all the given targets and sends
30 ICMP ping packets. The results (min, max, average and mean ping round-trip
times, as well as packet loss percentage) are sent to a Graphite server, which
by default should be running on localhost:2003. It then sleeps until the next
minute on the system clock before it starts another 'fping'. This way, you get
one measurement per minute. This is not configurable at this time.

By default, it uses 'ping' as a prefix for the metric name in Graphite,
followed by the hostname, with dots replaced by underscores. So for a host
named 'host.domain.tld', you would get the following metrics in Graphite, all
of which are gauges:

* ping.host_domain_tld.avgrtt
* ping.host_domain_tld.minrtt
* ping.host_domain_tld.medianrtt
* ping.host_domain_tld.maxrtt
* ping.host_domain_tld.packetloss

Requirements
------------

* fping
* python-daemon
* A Graphite server

The program uses regular expressions to get useful numbers from the fping ouput, so
it depends on specific formatting of the output. The following fping programs are
known to be compatible:

* version 3.4 on RHEL 6.4
* version 3.2 on Debian Wheezy

The output should look like this (for 2 hosts, 3 packets per host):

```
www.github.io  : [0], 84 bytes, 4.26 ms (4.26 avg, 0% loss)
www.google.com : [0], 84 bytes, 8.35 ms (8.35 avg, 0% loss)
www.github.io  : [1], 84 bytes, 3.25 ms (3.75 avg, 0% loss)
www.google.com : [1], 84 bytes, 7.84 ms (8.09 avg, 0% loss)
www.github.io  : [2], 84 bytes, 3.32 ms (3.61 avg, 0% loss)
www.google.com : [2], 84 bytes, 7.86 ms (8.01 avg, 0% loss)

www.github.io  : xmt/rcv/%loss = 3/3/0%, min/avg/max = 3.25/3.61/4.26
www.google.com : xmt/rcv/%loss = 3/3/0%, min/avg/max = 7.84/8.01/8.35
```

Configuration
-------------

Set up a [Carbon/Whisper retention schema](http://graphite.readthedocs.org/en/latest/config-carbon.html)
for the ping statistics, keeping one sample per minute for some time, and
optionally some aggregations:

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

If you want to run the program in the foreground instead of daemonized, search
the program for a line that says

```
self.daemonize = True
```

and change the value to `False`.

Usage
-----

`./graphping <target> [<target> ...]`

License
-------

Graphping is licensed under the Apache License, version 2.0.  A copy of the
license can be found in the 'COPYING' file and
[on the web](http://www.apache.org/licenses/LICENSE-2.0).
