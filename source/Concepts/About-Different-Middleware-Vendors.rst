.. redirect-from::

    DDS-and-ROS-middleware-implementations

About different ROS 2 DDS/RTPS vendors
======================================

ROS 2 is built on top of DDS/RTPS as its middleware, which provides discovery, serialization and transportation.
`This article <https://design.ros2.org/articles/ros_on_dds.html>`__ explains the motivation behind using DDS implementations, and/or the RTPS wire protocol of DDS, in detail.
In summary, DDS is an end-to-end middleware that provides features which are relevant to ROS systems, such as distributed discovery (not centralized like in ROS 1) and control over different "Quality of Service" options for the transportation.

`DDS <https://www.omg.org/omg-dds-portal>`__ is an industry standard which is implemented by a range of vendors, such as RTI's implementation `Connext <https://www.rti.com/products/>`__, ADLINK's implementation `OpenSplice <https://github.com/ADLINK-IST/opensplice>`__ or `Eclipse's Cyclone DDS <https://projects.eclipse.org/projects/iot.cyclonedds>`__.
RTPS (a.k.a. `DDSI-RTPS <https://www.omg.org/spec/DDSI-RTPS/About-DDSI-RTPS/>`__\ ) is the wire protocol used by DDS to communicate over the network.
There are implementations of that which do not fulfill the full DDS API, but provide sufficient functionality for ROS 2, such as eProsima's implementation `Fast RTPS <http://www.eprosima.com/index.php/products-all/eprosima-fast-rtps>`__.

ROS 2 supports multiple DDS/RTPS implementations because it is not necessarily "one size fits all" when it comes to choosing a vendor/implementation.
There are many factors you might consider while choosing a middleware implementation: logistical considerations like the license, or technical considerations like platform availability, or computation footprint.
Vendors may provide more than one DDS or RTPS implementation targeted at meeting different needs.
For example, RTI has a few variations of their Connext implementation that vary in purpose, like one that specifically targets microcontrollers and another which targets applications requiring special safety certifications (we only support their standard desktop version at this time).

In order to use a DDS/RTPS implementation with ROS 2, a "\ **R**\ OS **M**\ iddle\ **w**\ are interface" (a.k.a. ``rmw`` interface or just ``rmw``\ ) package needs to be created that implements the abstract ROS middleware interface using the DDS or RTPS implementation's API and tools.
It's a lot of work to implement and maintain RMW packages for supporting DDS implementations, but supporting at least a few implementations is important for ensuring that the ROS 2 codebase is not tied to any one particular implementation, as users may wish to switch out implementations depending on their project's needs.

Supported RMW implementations
-----------------------------

.. list-table::
   :header-rows: 1

   * - Product name
     - License
     - RMW implementation
     - Status
   * - eProsima *Fast RTPS*
     - Apache 2
     - ``rmw_fastrtps_cpp``
     - Full support. Default RMW in Foxy and before. Packaged with binary releases.
   * - Eclipse *Cyclone DDS*
     - Eclipse Public License v2.0
     - ``rmw_cyclonedds_cpp``
     - Full support. Default RMW since Galactic. Packaged with binary releases.
   * - RTI *Connext*
     - commercial, research
     - ``rmw_connext_cpp``
     - Full support. Support included in binaries, but Connext installed separately.
   * - RTI *Connext* (dynamic implementation)
     - commercial, research
     - ``rmw_connext_dynamic_cpp``
     - Support paused. Full support until alpha 8.*
   * - ADLINK *Opensplice*
     - Apache 2, commercial
     - ``rmw_opensplice_cpp``
     - Partial support. Support included in binaries before Foxy, but OpenSplice installed separately.

*"Partial support" means that one or more of the features required by the rmw interface is not implemented.*

For practical information on working with multiple RMW implementations, see the `"Working with multiple RMW implementations" <../Guides/Working-with-multiple-RMW-implementations>` tutorial.

Multiple RMW implementations
----------------------------

The ROS 2 binary releases for currently active distros have built-in support for several RMW implementations out of the box (Fast DDS, RTI Connext Pro, ADLINK Eclipse Cyclone DDS).
Since Galactic, the default is Cyclone DDS, which works without any additional installation steps, because we distribute it with our binary packages.
Prior to Galactic, the default was Fast DDS, which works without any additional installation steps.

Other RMWs like Fast-DDS or Connext can be enabled by `installing additional packages <../Installation/DDS-Implementations>`, but without having to rebuild anything or replace any existing packages.

A ROS 2 workspace that has been built from source may build and install multiple RMW implementations simultaneously.
While the core ROS 2 code is being compiled, any RMW implementation that is found will be built if the relevant DDS/RTPS implementation has been installed properly and the relevant environment variables have been configured.
For example, if the code for the `RMW package for RTI Connext <https://github.com/ros2/rmw_connext/tree/master/rmw_connext_cpp>`__ is in the workspace, it will be built if an installation of RTI's Connext Pro can also be found.

For many cases you will find that nodes using different RMW implementations are able to communicate, however this is not true under all circumstances.
Here is a list of inter-vendor communication configurations that are not supported:

- Fast-DDS <-> Connext
   - does not support communication over pub/sub
   - ``WString`` published by Fast-DDS can't be received correctly by Connext on macOS
- OpenSplice <-> OpenSplice
   - does not support ``WString``
   - ``WString`` is mapped to ``String`` which has a different wire representation
- Connext <-> Cyclone DDS
   - does not support pub/sub communication for ``WString``
- Connext Dynamic <-> Connext Dynamic
   - does not support C services

Default RMW implementation
--------------------------

If a ROS 2 workspace has multiple RMW implementations, the default RMW implementation since Galactic is selected as Cyclone DDS if it's available.
If the Cyclone DDS RMW implementation is not installed, the RMW implementation with the first RMW implementation identifier in alphabetical order will be used.
The implementation identifier is the name of the ROS package that provides the RMW implementation, e.g. ``rmw_cyclonedds_cpp``.
For example, if both ``rmw_fastrtps_cpp`` and ``rmw_connext_cpp`` ROS packages are installed, ``rmw_connext_cpp`` would be the default.
If ``rmw_cyclonedds_cpp`` is ever installed, it would be the default.

See the `guide <../Guides/Working-with-multiple-RMW-implementations>` for how to specify which RMW implementation is to be used when running the ROS 2 examples.
