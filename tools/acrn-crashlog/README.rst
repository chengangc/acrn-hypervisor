ACRN-Crashlog
#############

Introduction
============

ACRN-Crashlog is a debug tool to collect any useful logs/information after each crash/event occurred on the platform, including hypervisor, SOS and AaaG. It provides abundant logs for developers to debug/analyze the crashes. And it provides a flexiable way to config which crash/event to be concerned with xml file.

Building the ACRN-Crashlog
==========================

Build dependencies
^^^^^^^^^^^^^^^^^^

- libevent-dev
- openssl-dev
- libxml2-dev
- telemetrics-client-dev
- check-dev
- systemd-dev

Build
^^^^^

To build the ACRN-Crashlog

.. code-block:: console

   make

To Clean the build:

.. code-block:: console

   make clean

Installing the ACRN-Crashlog
============================

Install dependencies
^^^^^^^^^^^^^^^^^^^^

Since ACRN-Crashlog runs as a probe of `telemetrics-client`_, it needs `telemetrics-client`_ (>=1.6.0) has been installed at first.

Install
^^^^^^^

To install the build

.. code-block:: console

   sudo make install

Usage
=====

ACRN-Crashlog works as a probe of `telemetrics-client`_. It is autostarted, running as daemon and sending the crashlog path to `telemetrics-client`_.
`telemetrics-client`_ records events of interest and reports them to the backend using telemd telemetrics daemon. The type of events/logs is implemented as probe. The work flow of acrnprobe and `telemetrics-client`_ is:

.. code-block:: console

            crashlog path                   log content
   acrnprobe------------->telemetircs-client----------->backend

For more details of telemetrics, please refer the `telemetrics-client`_ and `telemetrics-backend`_ website.

Crashlog can be retrieved with telem_journal command:

.. code-block:: console

   telem_journal -i

ACRN-Crashlog also provides a tool debugger to dump the specific process information:

.. code-block:: console

   debugger <pid>

PS: need to run as root role

Source Code
===========

The source code structure:

.. code-block:: console

   acrn-crashlog/
   ├── acrnprobe
   │   └── include
   ├── common
   │   └── include
   ├── data
   └── usercrash
       └── include

- acrnprobe: to probe our tool on `telemetrics-client`_, and gathers all the crash/event logs on the platform. For the logs on hypervisor, it's gotten with acrnlog. For the log on SOS, the userspace crash log is gotten with usercrash, and the kernel crash log is gotten with the inherent mechanism like ipanic, pstore and etc. For the log on AaaG, it's gotten with monitoring the change of related folders on the sos image, like /data/logs/. acrnprobe also provides a flexible way to allow users to config which crash/event they want to collect through xml file easily.
- common: some utils for logs, exec and string.
- data: configuration file, service files and shell script.
- usercrash: to implement the tool which get the crash info for the crashing process in userspace.

acrnprobe
^^^^^^^^^

The acrnprobe is a probe of `telemetrics-client`_. It detects all critical events on the platform and collects specific information for debug purpose. These information would be storaged as logs, and the log path would be deliverd to `telemetrics-client`_ as a record.
For more detail on arcnprobe, please refer acrnprobe/README.rst.

usercrash
^^^^^^^^^

The usercrash is a tool to get the crash info for the crashing process in userspace. It works in Cient/Server model. Server is autostarted, and client is configured in core_pattern, which will be triggered once crash occurred in userspace.
For more detail on usercrash, please refer usercrash/README.rst.

.. _`telemetrics-client`: https://github.com/clearlinux/telemetrics-client
.. _`telemetrics-backend`: https://github.com/clearlinux/telemetrics-backend
