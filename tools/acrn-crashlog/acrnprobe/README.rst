#########
ACRNPROBE
#########

DESCRIPTION
===========
The **acrnprobe** is a *probe* of `telemetrics client`_. It detects all critical events
on the platform and collects specific informations for them. These informations would
be storaged as logs, and the log path would be deliverd to `telemetrics client`_ as a record.

USAGE
=====
The **acrnprobe** is launched as a service at boot. Also, it provides some basic options:

 Specify a configuration file for **acrnprobe**. If this option is unused, **acrnprobe** will
 use the configuration file located in CUSTOM CONFIGURATION PATH or INSTALLATION PATH
 (see `CONFIGURATION FILES`_).

 # acrnprobe -c [configuration_path]

 To see the version of **acrnprobe**.

 # acrnprobe -V

ARCHITECTURE
============

Syntax
******
channel : 
          Channel represents a way of detecting the system's events.
          There are 3 channels:
          1. oneshot, detect once while **acrnprobe** startup.
          2. polling, run a detecting job with fixed time interval.
          3. inotify, watch the change of file or dir.

event queue : 
          There is a global queue to receive all events detected.
          Generally, events are enqueued in channel, and dequeued in event handler.

event handler : 
          Event handler is a thread to handle events detected by channel.
          It's awakened by an enqueued event.

sender : 
         The sender corresponds to an exit of event.
         There are two senders:
         Crashlog is responsible to collect logs and save it locally.
         Telemd is responsible to send log records to `telemetrics client`_.

Description
***********
As a log collection mechanism to record critical events on the platform, **acrnprobe** have these functions:

1. detect event.
   Through experience, the occurrencs of an system event is usually accompanied
   by some effects. Effect could be a generated file, an error message in kernel's
   log, or a system reboot. To get these effects, some of them we can monitor a
   directory, some of them we might need to do a detection in time loop.
   *So we implement the channel, which represents a common method of detection.*

2. analyze event and determine the event type.
   Generally, a specific effect correspond to a particular type of events.
   However, it is the icing on the cake for analyzing the detailed event types
   according to some phenomena. *Crash reclassify is implemented for this purpose.*

3. collect information for detected events.
   This is for debug purpose. Events without information are meaningless, developers
   need to use this information to improve their system. *Sender crashlog is implemented
   for this purpose.*

4. archive these information as logs, and generate records.
   There must be a central place to tell user what was happened in system. *Sender telemd
   is implemented for this purpose.*

Diagram
*******
::

 +---------------------------------------------+
 | channel:   |oneshot|  |polling|   |inotify| |
 +--------------------------------------+------+
                                        |
 +---------------------+    +-----+     |
 | event queue         +<---+event+<----+
 +-+-------------------+    +-----+
   |
   v
 +-+----------------------------------------------------------------------------+
 |  event handler:                                                              |
 |                                                                              |
 |  event handler will handle internal event                                    |
 |      +----------+    +------------+                                          |
 |      |heart beat+--->+fed watchdog|                                          |
 |      +----------+    +------------+                                          |
 |                                                                              |
 |  call sender for other types                                                 |
 |      +--------+   +----------------+   +------------+   +------------------+ |
 |      |crashlog+-->+crash reclassify+-->+collect logs+-->+generate crashfile| |
 |      +--------+   +----------------+   +------------+   +------------------+ |
 |                                                                              |
 |      +------+    +-----------------+                                         |
 |      |telemd+--->+telemetics client|                                         |
 |      +------+    +-----------------+                                         |
 +------------------------------------------------------------------------------+


SOURCE FILES
============

- main.c
  Entry of **acrnprobe**.
- channel.c
  The implementation of *channel* (see `Syntax`_).
- crash_reclassify.c
  Analyzing the detailed types for crash event.
- probeutils.c
  Provide some utils **acrnprobe** needs.
- event_queue.c
  The implementation of *event queue* (see `Syntax`_).
- event_handler.c
  The implementation of *event handler* (see `Syntax`_).
- history.c
  There is a history_event file to manage all logs that **acrnprobe** archived. "history.c" provides the interfaces to modify the file in fixed format.
- load_conf.c
  Parse and load configuration.
- property.c
  **Acrnprobe** needs to know some HW/SW properties, such as board version, build version. Centrally manage these properties in this file.
- sender.c
  The implementation of *sender* (see `Syntax`_).
- startupreason.c
  This file provides the function to get system reboot reason from kernel command line.
- android_events.c
  Sync events detected by android crashlog.

CONFIGURATION FILES
===================

* ``/usr/share/defaults/telemetrics/acrnprobe.xml``

    If no custom configuration file is found, **acrnprobe** uses the
    settings in this file.

* ``/etc/acrnprobe.xml``

    Custom configuration file that **acrnprobe** reads.

.. _`telemetrics client`: https://github.com/clearlinux/telemetrics-client
