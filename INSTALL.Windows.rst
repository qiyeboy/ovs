..
      Licensed under the Apache License, Version 2.0 (the "License"); you may
      not use this file except in compliance with the License. You may obtain
      a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

      Unless required by applicable law or agreed to in writing, software
      distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
      WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
      License for the specific language governing permissions and limitations
      under the License.

      Convention for heading levels in Open vSwitch documentation:

      =======  Heading 0 (reserved for the title in a document)
      -------  Heading 1
      ~~~~~~~  Heading 2
      +++++++  Heading 3
      '''''''  Heading 4

      Avoid deeper levels because they do not render well.

=======================
Open vSwitch on Windows
=======================

.. _windows-build-reqs:

Build Requirements
------------------

Open vSwitch on Linux uses autoconf and automake for generating Makefiles.  It
will be useful to maintain the same build system while compiling on Windows
too.  One approach is to compile Open vSwitch in a MinGW environment that
contains autoconf and automake utilities and then use Visual C++ as a compiler
and linker.

The following explains the steps in some detail.

- Mingw

  Install Mingw on a Windows machine by following the instructions on
  `mingw.org <http://www.mingw.org/wiki/Getting_Started>`__.

  This should install mingw at ``C:\Mingw`` and msys at ``C:\Mingw\msys``.  Add
  ``C:\MinGW\bin`` and ``C:\Mingw\msys\1.0\bin`` to PATH environment variable
  of Windows.

  You can either use the MinGW installer or the command line utility
  ``mingw-get`` to install both the base packages and additional packages like
  automake and autoconf(version 2.68).

  Also make sure that ``/mingw`` mount point exists. If its not, please
  add/create the following entry in ``/etc/fstab``::

      'C:/MinGW /mingw'.

- Python

  Install the latest Python 2.x from python.org and verify that its path is
  part of Windows' PATH environment variable.

- Visual Studio

  You will need at least Visual Studio 2013 (update 4) to compile userspace
  binaries.  In addition to that, if you want to compile the kernel module you
  will also need to install Windows Driver Kit (WDK) 8.1 Update.

  It is important to get the Visual Studio related environment variables and to
  have the $PATH inside the bash to point to the proper compiler and linker.
  One easy way to achieve this for VS2013 is to get into the "VS2013 x86 Native
  Tools Command Prompt" (in a default installation of Visual Studio 2013 this
  can be found under the following location: ``C:\Program Files (x86)\Microsoft
  Visual Studio 12.0\Common7\Tools\Shortcuts``) and through it enter into the
  bash shell available from msys by typing ``bash --login``.

  There is support for generating 64 bit binaries too.  To compile under x64,
  open the "VS2013 x64 Native Tools Command Prompt" (if your current running OS
  is 64 bit) or "VS2013 x64 Cross Tools Command Prompt" (if your current
  running OS is not 64 bit) instead of opening its x86 variant.  This will
  point the compiler and the linker to their 64 bit equivalent.

  If after the above step, a ``which link`` inside MSYS's bash says,
  ``/bin/link.exe``, rename ``/bin/link.exe`` to something else so that the
  Visual studio's linker is used. You should also see a 'which sort' report
  ``/bin/sort.exe``.

- pthreads-win32

  For pthread support, install the library, dll and includes of pthreads-win32
  project from `sourceware
  <ftp://sourceware.org/pub/pthreads-win32/prebuilt-dll-2-9-1-release>`__ to a
  directory (e.g.: ``C:/pthread``). You should add the pthread-win32's dll path
  (e.g.: ``C:\pthread\dll\x86``) to the Windows' PATH environment variable.

- OpenSSQL

  To get SSL support for Open vSwitch on Windows, you will need to install
  `OpenSSL for Windows <http://www.openssl.org/related/binaries.html>`__

  Note down the directory where OpenSSL is installed (e.g.:
  ``C:/OpenSSL-Win32``) for later use.

Install Requirements
--------------------

* Share network adaptors

  We require that you don't disable the "Allow management operating system to
  share this network adapter" under 'Virtual Switch Properties' > 'Connection
  type: External network', in the HyperV virtual network switch configuration.

* Checksum Offloads

  While there is some support for checksum/segmentation offloads in software,
  this is still a work in progress. Till the support is complete we recommend
  disabling TX/RX offloads for both the VM's as well as the HyperV.

Bootstrapping
-------------

This step is not needed if you have downloaded a released tarball. If
you pulled the sources directly from an Open vSwitch Git tree or got a
Git tree snapshot, then run boot.sh in the top source directory to build
the "configure" script::

    > ./boot.sh

.. _windows-configuring:

Configuring
-----------

Configure the package by running the configure script.  You should provide some
configure options to choose the right compiler, linker, libraries, Open vSwitch
component installation directories, etc. For example::

    > ./configure CC=./build-aux/cccl LD="$(which link)" \
        LIBS="-lws2_32 -liphlpapi" --prefix="C:/openvswitch/usr" \
        --localstatedir="C:/openvswitch/var" \
        --sysconfdir="C:/openvswitch/etc" \
        --with-pthread="C:/pthread"

.. note::
  By default, the above enables compiler optimization for fast code.  For
  default compiler optimization, pass the ``--with-debug`` configure option.

To configure with SSL support, add the requisite additional options::

    > ./configure CC=./build-aux/cccl LD="`which link`"  \
        LIBS="-lws2_32 -liphlpapi" --prefix="C:/openvswitch/usr" \
         --localstatedir="C:/openvswitch/var"
         --sysconfdir="C:/openvswitch/etc" \
         --with-pthread="C:/pthread" \
         --enable-ssl --with-openssl="C:/OpenSSL-Win32"

Finally, to the kernel module also::

    > ./configure CC=./build-aux/cccl LD="`which link`" \
        LIBS="-lws2_32 -liphlpapi" --prefix="C:/openvswitch/usr" \
        --localstatedir="C:/openvswitch/var" \
        --sysconfdir="C:/openvswitch/etc" \
        --with-pthread="C:/pthread" \
        --enable-ssl --with-openssl="C:/OpenSSL-Win32" \
        --with-vstudiotarget="<target type>"

Possible values for ``<target type>`` are: ``Debug`` and ``Release``

.. note::
  You can directly use the Visual Studio 2013 IDE to compile the kernel
  datapath.  Open the ovsext.sln file in the IDE and build the solution.

Refer to the `installation guide <INSTALL.rst>` for information on additional
configuration options.

.. _windows-building:

Building
--------

Once correctly configured, building Open vSwitch on Windows is similar to
building on Linux, FreeBSD, or NetBSD.

#. Run make for the ported executables in the top source directory, e.g.::

       > make

   For faster compilation, you can pass the ``-j`` argument to make.  For
   example, to run 4 jobs simultaneously, run ``make -j4``.

   .. note::

     MSYS 1.0.18 has a bug that causes parallel make to hang. You can overcome
     this by downgrading to MSYS 1.0.17.  A simple way to downgrade is to exit
     all MinGW sessions and then run the below command from MSVC developers
     command prompt.::

         > mingw-get upgrade msys-core-bin=1.0.17-1

#. To run all the unit tests in Open vSwitch, one at a time::

       > make check

   To run all the unit tests in Open vSwitch, up to 8 in parallel::

       > make check TESTSUITEFLAGS="-j8"

#. To install all the compiled executables on the local machine, run::

       > make install

  .. note::

    This will install the Open vSwitch executables in ``C:/openvswitch``.  You
    can add ``C:\openvswitch\usr\bin`` and ``C:\openvswitch\usr\sbin`` to
    Windows' PATH environment variable for easy access.

The Kernel Module
~~~~~~~~~~~~~~~~~

If you are building the kernel module, you will need to copy the below files to
the target Hyper-V machine.

- ``./datapath-windows/x64/Win8.1Debug/package/ovsext.inf``
- ``./datapath-windows/x64/Win8.1Debug/package/OVSExt.sys``
- ``./datapath-windows/x64/Win8.1Debug/package/ovsext.cat``
- ``./datapath-windows/misc/install.cmd``
- ``./datapath-windows/misc/uninstall.cmd``

.. note::
  The above path assumes that the kernel module has been built using Windows
  DDK 8.1 in Debug mode. Change the path appropriately, if a different WDK
  has been used.

Now run ``./uninstall.cmd`` to remove the old extension. Once complete, run
``./install.cmd`` to insert the new one.  For this to work you will have to
turn on ``TESTSIGNING`` boot option or 'Disable Driver Signature
Enforcement' during boot.  The following commands can be used::

    > bcdedit /set LOADOPTIONS DISABLE_INTEGRITY_CHECKS
    > bcdedit /set TESTSIGNING ON
    > bcdedit /set nointegritychecks ON

.. note::
  You may have to restart the machine for the settings to take effect.

In the Virtual Switch Manager configuration you can enable the Open vSwitch
Extension on an existing switch or create a new switch.  If you are using an
existing switch, make sure to enable the "Allow Management OS" option for VXLAN
to work (covered later).

The command to create a new switch named 'OVS-Extended-Switch' using a physical
NIC named 'Ethernet 1' is::

    PS > New-VMSwitch "OVS-Extended-Switch" -AllowManagementOS $true \
        -NetAdapterName "Ethernet 1"

.. note::
  You can obtain the list of physical NICs on the host using 'Get-NetAdapter'
  command.

In the properties of any switch, you should should now see "Open vSwitch
Extension" under 'Extensions'.  Click the check box to enable the extension.
An alternative way to do the same is to run the following command::

    PS > Enable-VMSwitchExtension "Open vSwitch Extension" OVS-Extended-Switch

.. note::
  If you enabled the extension using the command line, a delay of a few seconds
  has been observed for the change to be reflected in the UI.  This is not a
  bug in Open vSwitch.

Starting
--------

.. important::
  The following steps assume that you have installed the Open vSwitch utilities
  in the local machine via 'make install'.

Before starting ovs-vswitchd itself, you need to start its configuration
database, ovsdb-server. Each machine on which Open vSwitch is installed should
run its own copy of ovsdb-server. Before ovsdb-server itself can be started,
configure a database that it can use::

    > ovsdb-tool create C:\openvswitch\etc\openvswitch\conf.db \
        C:\openvswitch\usr\share\openvswitch\vswitch.ovsschema

Configure ovsdb-server to use database created above and to listen on a Unix
domain socket::

    > ovsdb-server -vfile:info --remote=punix:db.sock --log-file \
        --pidfile --detach

.. note::
  The logfile is created at ``C:/openvswitch/var/log/openvswitch/``

Initialize the database using ovs-vsctl. This is only necessary the first time
after you create the database with ovsdb-tool, though running it at any time is
harmless::

    > ovs-vsctl --no-wait init

.. tip::
  If you would later like to terminate the started ovsdb-server, run::

      > ovs-appctl -t ovsdb-server exit

Start the main Open vSwitch daemon, telling it to connect to the same Unix
domain socket::

    > ovs-vswitchd -vfile:info --log-file --pidfile --detach

.. tip::
  If you would like to terminate the started ovs-vswitchd, run::

      > ovs-appctl exit

.. note::
  The logfile is created at ``C:/openvswitch/var/log/openvswitch/``

Validating
----------

At this point you can use ovs-vsctl to set up bridges and other Open vSwitch
features.

Add bridges
~~~~~~~~~~~

Let's start by creating an integration bridge, ``br-int`` and a PIF bridge,
``br-pif``::

    > ovs-vsctl add-br br-int
    > ovs-vsctl add-br br-pif

.. note::
  There's a known bug that running the ovs-vsctl command does not terminate.
  This is generally solved by having ovs-vswitchd running.  If you face the
  issue despite that, hit Ctrl-C to terminate ovs-vsctl and check the output to
  see if your command succeeded.

Validate that ports are added by dumping from both ovs-dpctl and ovs-vsctl::

    > ovs-dpctl show
    system@ovs-system:
            lookups: hit:0 missed:0 lost:0
            flows: 0
            port 2: br-pif (internal)     <<< internal port on 'br-pif' bridge
            port 1: br-int (internal)     <<< internal port on 'br-int' bridge

    > ovs-vsctl show
    a56ec7b5-5b1f-49ec-a795-79f6eb63228b
        Bridge br-pif
            Port br-pif
                Interface br-pif
                    type: internal
        Bridge br-int
            Port br-int
                Interface br-int
                    type: internal

.. note::
  There's a known bug that the ports added to OVSDB via ovs-vsctl don't get to
  the kernel datapath immediately, ie. they don't show up in the output of
  ``ovs-dpctl show`` even though they show up in output of ``ovs-vsctl show``.
  In order to workaround this issue, restart ovs-vswitchd. (You can terminate
  ovs-vswitchd by running ``ovs-appctl exit``.)

Add physicals NICs (PIF)
~~~~~~~~~~~~~~~~~~~~~~~~

Now, let's add the physical NIC and the internal port to ``br-pif``. In OVS for
Hyper-V, we use the name of the adapter on top of which the Hyper-V virtual
switch was created, as a special name to refer to the physical NICs connected
to the Hyper-V switch, e.g. if we created the Hyper-V virtual switch on top of
the adapter named ``Ethernet0``, then in OVS we use that name (``Ethernet0``)
as a special name to refer to that adapter.

.. note::
  we assume that the Hyper-V switch on which OVS extension is enabled has a
  single physical NIC connected to it.

An internal port is the virtual adapter created on the Hyper-V switch using the
``AllowManagementOS`` setting.  This has already been setup while creating the
switch using the instructions above.  In OVS for Hyper-V, we use a the name of
that specific adapter as a special name to refer to that adapter. By default it
is created under the following rule ``vEthernet (<name of the switch>)``.

As a whole example, if we issue the following in a powershell console::

    PS C:\package\binaries> Get-NetAdapter | select Name,MacAddress,InterfaceDescription
    Name                   MacAddress         InterfaceDescription
    ----                   ----------         --------------------
    Ethernet1              00-0C-29-94-05-65  Intel(R) PRO/1000 MT Network Connection
    vEthernet (external)   00-0C-29-94-05-5B  Hyper-V Virtual Ethernet Adapter #2
    Ethernet0              00-0C-29-94-05-5B  Intel(R) PRO/1000 MT Network Connection #2

    PS C:\package\binaries> Get-VMSwitch
    Name     SwitchType NetAdapterInterfaceDescription
    ----     ---------- ------------------------------
    external External   Intel(R) PRO/1000 MT Network Connection #2

We can see that we have a switch(external) created upon adapter name
'Ethernet0' with an internal port under name ``vEthernet (external)``. Thus
resulting into the following ovs-vsctl commands::

    > ovs-vsctl add-port br-pif Ethernet0
    > ovs-vsctl add-port br-pif "vEthernet (external)"

Dumping the ports should show the additional ports that were just added::

    > ovs-dpctl show
    system@ovs-system:
            lookups: hit:0 missed:0 lost:0
            flows: 0
            port 4: vEthernet (external) (internal) <<< 'AllowManagementOS'
                                                         adapter on
                                                         Hyper-V switch
            port 2: br-pif (internal)
            port 1: br-int (internal)
            port 3: Ethernet0                       <<< Physical NIC

    > ovs-vsctl show
    a56ec7b5-5b1f-49ec-a795-79f6eb63228b
        Bridge br-pif
            Port "vEthernet (external)"
                Interface "vEthernet (external)"
            Port br-pif
                Interface br-pif
                    type: internal
            Port "Ethernet0"
                Interface "Ethernet0"
        Bridge br-int
            Port br-int
                Interface br-int
                    type: internal

Add virtual interfaces (VIFs)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Adding VIFs to openvswitch is a two step procedure.  The first step is to
assign a 'OVS port name' which is a unique name across all VIFs on this
Hyper-V.  The next step is to add the VIF to the ovsdb using its 'OVS port
name' as key.

First, assign a unique 'OVS port name' to the VIF. The VIF needs to have been
disconnected from the Hyper-V switch before assigning a 'OVS port name' to it.
In the example below, we assign a 'OVS port name' called ``ovs-port-a`` to a
VIF on a VM ``VM1``.  By using index 0 for ``$vnic``, the first VIF of the VM
is being addressed.  After assigning the name ``ovs-port-a``, the VIF is
connected back to the Hyper-V switch with name ``OVS-HV-Switch``, which is
assumed to be the Hyper-V switch with OVS extension enabled.::

    PS> import-module .\datapath-windows\misc\OVS.psm1
    PS> $vnic = Get-VMNetworkAdapter <Name of the VM>
    PS> Disconnect-VMNetworkAdapter -VMNetworkAdapter $vnic[0]
    PS> $vnic[0] | Set-VMNetworkAdapterOVSPort -OVSPortName ovs-port-a
    PS> Connect-VMNetworkAdapter -VMNetworkAdapter $vnic[0] \
          -SwitchName OVS-Extended-Switch

Next, add the VIFs to ``br-int``::

    > ovs-vsctl add-port br-int ovs-port-a

Dumping the ports should show the additional ports that were just added::

    > ovs-dpctl show
    system@ovs-system:
            lookups: hit:0 missed:0 lost:0
            flows: 0
            port 4: vEthernet (external) (internal)
            port 5: ovs-port-a
            port 2: br-pif (internal)
            port 1: br-int (internal
            port 3: Ethernet0

    > ovs-vsctl show
    4cd86499-74df-48bd-a64d-8d115b12a9f2
        Bridge br-pif
            Port "vEthernet (external)"
                Interface "vEthernet (external)"
            Port "Ethernet0"
                Interface "Ethernet0"
            Port br-pif
                Interface br-pif
                    type: internal
        Bridge br-int
            Port br-int
                Interface br-int
                    type: internal
            Port "ovs-port-a"
                Interface "ovs-port-a"

Add patch ports and configure VLAN tagging
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Windows Open vSwitch implementation support VLAN tagging in the switch.
Switch VLAN tagging along with patch ports between ``br-int`` and ``br-pif`` is
used to configure VLAN tagging functionality between two VMs on different
Hyper-Vs.  To start, add a patch port from ``br-int`` to ``br-pif``::

    > ovs-vsctl add-port br-int patch-to-pif
    > ovs-vsctl set interface patch-to-pif type=patch \
        options:peer=patch-to-int

Add a patch port from ``br-pif`` to ``br-int``::

    > ovs-vsctl add-port br-pif patch-to-int
    > ovs-vsctl set interface patch-to-int type=patch \
        options:peer=patch-to-pif

Re-Add the VIF ports with the VLAN tag::

    > ovs-vsctl add-port br-int ovs-port-a tag=900
    > ovs-vsctl add-port br-int ovs-port-b tag=900

Add tunnels
~~~~~~~~~~~

The Windows Open vSwitch implementation support VXLAN and STT tunnels. To add
tunnels. For example, first add the tunnel port between 172.168.201.101 <->
172.168.201.102::

    > ovs-vsctl add-port br-int tun-1
    > ovs-vsctl set Interface tun-1 type=<port-type>
    > ovs-vsctl set Interface tun-1 options:local_ip=172.168.201.101
    > ovs-vsctl set Interface tun-1 options:remote_ip=172.168.201.102
    > ovs-vsctl set Interface tun-1 options:in_key=flow
    > ovs-vsctl set Interface tun-1 options:out_key=flow

...and the tunnel port between 172.168.201.101 <-> 172.168.201.105::

    > ovs-vsctl add-port br-int tun-2
    > ovs-vsctl set Interface tun-2 type=<port-type>
    > ovs-vsctl set Interface tun-2 options:local_ip=172.168.201.102
    > ovs-vsctl set Interface tun-2 options:remote_ip=172.168.201.105
    > ovs-vsctl set Interface tun-2 options:in_key=flow
    > ovs-vsctl set Interface tun-2 options:out_key=flow

Where ``<port-type>`` is one of: ``stt`` or ``vxlan``

.. note::
  Any patch ports created between br-int and br-pif MUST be be deleted prior to
  adding tunnels.

Windows Services
----------------

Open vSwitch daemons come with support to run as a Windows service. The
instructions here assume that you have installed the Open vSwitch utilities and
daemons via ``make install``.

.. note::
  The commands shown here can be run from MSYS bash or Windows command prompt.

To start, create the database::

    > ovsdb-tool create C:/openvswitch/etc/openvswitch/conf.db \
        "C:/openvswitch/usr/share/openvswitch/vswitch.ovsschema"

Create the ovsdb-server service and start it::

    > sc create ovsdb-server \
        binpath="C:/openvswitch/usr/sbin/ovsdb-server.exe \
          C:/openvswitch/etc/openvswitch/conf.db \
          -vfile:info --log-file --pidfile \
          --remote=punix:db.sock --service --service-monitor"
    > sc start ovsdb-server

.. tip::
  One of the common issues with creating a Windows service is with mungled
  paths.  You can make sure that the correct path has been registered with the
  Windows services manager by running::

      > sc qc ovsdb-server

Check that the service is healthy by running::

    > sc query ovsdb-server

Initialize the database::

    > ovs-vsctl --no-wait init

Create the ovs-vswitchd service and start it::

    > sc create ovs-vswitchd \
      binpath="C:/openvswitch/usr/sbin/ovs-vswitchd.exe \
      --pidfile -vfile:info --log-file  --service --service-monitor"
    > sc start ovs-vswitchd

Check that the service is healthy by running::

    > sc query ovs-vswitchd

To stop and delete the services, run::

    > sc stop ovs-vswitchd
    > sc stop ovsdb-server
    > sc delete ovs-vswitchd
    > sc delete ovsdb-server

Windows CI Service
------------------

`AppVeyor <www.appveyor.com>`__ provides a free Windows autobuild service for
opensource projects.  Open vSwitch has integration with AppVeyor for continuous
build.  A developer can build test his changes for Windows by logging into
appveyor.com using a github account, creating a new project by linking it to
his development repository in github and triggering a new build.

TODO
----

* Investigate the working of sFlow on Windows and re-enable the unit tests.

* Investigate and add the feature to provide QoS.

* Sign the driver & create an MSI for installing the different OpenvSwitch
  components on Windows.
