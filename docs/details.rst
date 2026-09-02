..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. role:: strike
    :class: strike

.. role:: key
    :class: key

.. role:: darkyellow

.. _details:

Instrumentation Details
=======================

This section is a scattershot assortment of details about things at
the beamline, presented in no special order.  Subsection titles have
been alphabetized to make them a bit easier to find when reading this
page.  This is basically an attempt to capture institutional knowledge
... someplace.

All DNS names and IP addresses at 6BM are recorded at
https://docs.nsls2.bnl.gov/docs/admin/network/beamline_ipaddresses.html#bm-bmm-ip-addresses

See :numref:`Section %s <iocs>` for details about the IOCs running the
beamline instrumentation.


Analog Video Capture
--------------------

.. admonition:: Deprecated
   :class: note

   This camera is still in place and displayed on the screen to the
   left of the main control computer.  The REDGO dongle is still
   plugged into xf06bm-ws3 and can be used to capture images.  But
   images are no longer captured as part of the dossier as there is no
   way to write those images to project folders and capture the
   resource link in Tiled.  (May, 2026)


Implementing `the REDGO Video Audio VHS VCR USB Video Capture Card to
DVD Converter Capture Card Adapter
<https://www.amazon.com/REDGO-Video-Capture-Converter-Adapter/dp/B01E5ITE2W>`__
to capture video from the small analog cameras in the hutch took a bit
of doing.

First, the adapter must be plugged directly into the computer.  Using
a USB hub makes for an unreliable interface to the camera.

Second, the file ``/etc/udev/rules.d/99-usb-camera-capture.rules`` is
needed to set permissions on ``/dev/video0`` correctly when the adapter is
plugged in.

.. code-block:: none

   ACTION!="add|change", GOTO="webcam_capture_end"
   SUBSYSTEM=="usb", ATTRS{idVendor}=="534d", ATTRS{idProduct}=="0021", MODE="0666"
   KERNEL=="video*", ATTRS{idVendor}=="534d", ATTRS{idProduct}=="0021", GROUP="video", MODE="0666"
   KERNEL=="video*", ATTRS{idVendor}=="534d", ATTRS{idProduct}=="0021", ATTRS{avoid_reset_quirk}=1
   KERNEL=="video*", ATTRS{idVendor}=="534d", ATTRS{idProduct}=="0021", ATTRS{quirks}=0x100
   LABEL="webcam_capture_end"

Putting this file in place will require assistance from DSSI. Beamline
staff do not have permission to make a file in that folder. See `this
Jira ticket <https://jira.nsls2.bnl.gov/browse/HXSS-779>`__ for an
example of what to ask for.

This recognizes the vendor and product IDs of the specific adapter
that I bought.  When inserted, it sets the device to RW for all users
and sets a couple of possibly relevant attributes.  (This udev rules
file was adapted from the rules file that comes with the easycap dc60
package – info and links `here
<http://easycap.blogspot.com/p/easycap-dc60.html>`__).

Next a small function was written as a wrapper around `fswebcam
<https://github.com/fsphil/fswebcam>`__ to grab frames from the
camera. The function is basically a wrapper around a call to
``fswebcam`` like so:

.. code-block:: sh

   fswebcam -d /dev/video0 -r 640x480 -S 30 -F 5 foo.jpg

along with some image processing using python's ``wand`` package. 

Required packages:

+ ``fswebcam``
+ ``python-wand``
+ ``imagemagick``

This whole setup is filled with quirk.  There is a delay accessing the
video capture.  The ``-S`` switch builds in a 1 second delay, giving the
capture device enough time to begin displaying the image.  The ``-F``
switch tells the script how many frames to accumulate for good signal.
5 is probably overkill.

In any case, it is now possible to grab screen shots of the currently
displayed analog video while collecting data.

All of this is implemented in ``BMM/camera_device.py`` for use in
Bluesky. The heart of the implementation is a system call to
``fswebcam``. From there, the image is saved as an asset and correctly
pointed to in databroker.  See:

+ `BMM/user_ns/detectors.py <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/BMM/user_ns/detectors.py#L253>`__
+ `BMM/camera_device.py <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/BMM/camera_device.py#L62-L164>`__

While this resembles a properly integrated camera and counting on the
``anacam`` object will get recorded in the database, the resource file
|nd| the jpeg image |nd| gets written to the local machine, thus is
not recoverable via Tiled.  A future todo would be to make a caproto
IOC as a wrapper around ``fswebcam`` and give it proper authority to
write to storage.

.. admonition:: Update
   :class: note

   As of December 2024, this has been captured in BMM's ansible
   configuration.  Thus xf06bm-ws3 should always have this udev rule
   available, even after a system upgrade or installation.





Beamline maintenance tools
--------------------------

(The next three subsections were copied verbatim from the BMM wiki
page.  Links and other information should be verified.  February 2025)

Motor Controllers
~~~~~~~~~~~~~~~~~

On the Windows Laptop, use PEWIN.  PMAC User's Manual

The motor controllers are 10.6.130.81, .82, etc.  You can communicate
using PEWIN with the laptop on the 10.6.130.X network, :strike:`with
the laptop plugged into a DHCP port (it's address will be
10.6.128.213)`, or using a USB cable.  Unplug the network adapter from
the Delta Tau before plugging in the USB cable. The USB cable is the
sort with an A-type plug on one end and a B-type on the other end.

.. _fig_USB:
.. figure:: _images/infrastructure/USB.png
   :target: _images/USB.png
   :width: 30%
   :align: center


MOXA Configuration
~~~~~~~~~~~~~~~~~~

On the Windows Laptop, use the NPort Admin Suite.  It is pretty
self-explanatory.  Put the computer on the 10.6.130.X network, then
ask the suite to do a search.  When it finds a MOXA device (they
should be 10.6.130.51, .52, etc), click on the configure button.  It
will query the device, then give a fairly self-explanatory set of tabs
for configuring things on the device.

`NPort5200 User Manual
<https://www.moxa.com/doc/manual/nport/5200/NPort5200_v1.pdf>`__ --
The small Moxa devices supplied by FMBO are in this series. 

.. note:: The 5200 series is a discontinued Moxa product.

Camera Configuration
~~~~~~~~~~~~~~~~~~~~

On the Windows Laptop, Vimba is the tool for the Allied Vision cameras
supplied by FMBO (also the Prosilica GC in use at the XRD end station)
for looking at the fluorescence screens in DM1, DM2, and end station
FS.

Vimba can also be installed on a Linux machine. From the Quick Start
document:

.. code-block:: text

   Installing Vimba SDK on Linux Necessary runtime libraries for
   executing Vimba Viewer are available with the Vimba download.

   + Vimba ships as a tarball. Uncompress the archive with the command
     ``tar -xf ./AVTVimba.tgz`` to a directory you have writing
     privileges for. This creates a directory named ``AVTVimba``.

   + Navigate to ``AVTVimba/AVTGigETL`` and execute the shell script
     Install.sh with root privileges (for example ``dzdo
     ./Install.sh``).

   + Vimba Viewer is now ready to use, and it can be found in
     ``Vimba/Viewer/Bin``.

The second step is essential or the viewer will be unable to find the cameras on the network.

+ `Allied Vision homepage <https://www.alliedvision.com/en/products/software.html>`__
+ `Prosilica GC technical documentation
  <https://www.alliedvision.com/en/support/technical-documentation/prosilica-gc-documentation.html>`__

.. note:: DSSI should be able to configure Prosilica and Make GigE
   cameras.  The information above is only needed if, for some reason,
   DSSI is not or cannot be involved.


BNC Cable Map
-------------

Here is an explanation of the BNC and SHV patch panels going between
rack D at the control station, Rack C on the roof of the hutch, and
the in-hutch patch panel.


.. _fig-bncpatch:
.. figure:: _images/infrastructure/Bnc_map.png
   :target: _images/Bnc_map.png
   :width: 100%
   :align: center



.. _unfinished_controls:

Controls chores, unfinished
---------------------------

This is a list of things that never got done back in 2017-2018.  For
the most part, none of these are a problem in any way.  The beamline
operates just fine without these.  This section is simply an effort to
capture these for posterity.

+ PI piezo controllers on ``dcm.pitch``, ``m2.pitch``, and
  ``m3.pitch``. Some primitive controls are in place.  There is a box
  on the BMM CSS Main Page for piezo controls.  But active feedback
  was never developed for any of these three items. It is not hard to
  imagine the ways that having active feedback might be helpful, but
  these are not currently available.

* Fully fleshed out trigonometry for M2 and M3.  This would include
  careful calibration of distances.

* Photon delivery system mode B.  This is the low energy, focused
  mode.  The last time I looked at this, it seemed that the hard
  limits on the M3 downstream axes were set just a bit too high.  It
  is also not clear whether the XAS table can go low enough for this
  delivery mode.

* The VME crate is plugged into an unmaganged switch, which is then
  plugged into the managed switch.  This is silly and unnecessary, but
  has never been enough of a problem to fix.

* The outlets in the FOE should be moved to the insides of the blue
  stand to avoid having plugs get kicked when people are moving around
  the back alley.  This seems expensive and not completely necessary,
  but sure would be nice.

* Encoder pizza box in Rack A is unconfingured.  It will likely never
  get used.


See also :numref:`Section %s <future_instruments>` for the related
topic of planned instrumentation upgrades.


DCM 1st Xtal Equilibration
--------------------------

The temperature of the first crystal is measured by a K type
thermocouple pressed against the crystal by .... (verify how this is done)

With the front end slits set to 8 mm x 1.5 mm, the DCM first crystal
saturates at about 30.5 C from the base cooling water temperature of
about 28 C.

This takes about 50 minutes. 

.. _fig-dcm1stxtal:
.. figure:: _images/instrumentation/dcm1stxtal.png
   :target: _images/dcm1stxtal.png
   :width: 70%
   :align: center




DI Water Flow
-------------

The DI water is controled by manual valves, which should only be
operated by the utilities group, and by solenoid valves in the FOE.
The solenoid valves are triggered by a water-sensing strip along the
floor of the FOE. They are also actuated by switches on the CSS
utilities screen. These toggles are the ones circled in pink inthe
screenshot on the left. 

The valves themselves are the large yellow and black boxes mounted
high on the back wall of the FOE.  The valve indicators are the rods
with orange markings.  When the valves are open, the orange marks are
facing downstream.  When closed, the orange marks are rotated towards
the wall.  Opening and closing those valves is managed through CSS.
They must be open for the utilities group to do their work on the DI
delivery to the mono and the filter assemblies.  

.. subfigure::  ABC
   :layout-sm: ABC
   :gap: 8px
   :subcaptions: above
   :name: fig-diwater
   :class-grid: outline

   .. image:: _images/infrastructure/Water_flow_CSS.png

   .. image:: _images/infrastructure/Water_flow_valves_1.jpg

   .. image:: _images/infrastructure/Water_flow_valves_2.jpg

   (Left) The CSS utilities screen where the water valve controls are
   found.  (Middle) A view into the FOE.  (Right) The inboard wall
   where the physical valve is found.

Disabling MCS8 axes after move
------------------------------

From Adam Young at FMBO
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   The motors can be disabled after a movement and this can be set at the
   Delta Tau level.

   First you will need to connect to each MCS8+ with the beamline laptop
   and start PEWin.

   Then please do the following:

   + Click on the 'View' menu at the top of the window. Then click
     'Program/PLC Status (and upload)'. 
   + Select PLC1 and click 'Upload'. An editor showing PLC1 will appear.
   + Scroll down to find the variables P105 to P805. The '1' to '8' part
     of these variables represent axis 1 to 8 on the MCS8+. The value of
     these variables determines whether or not the motors will be
     disabled after a move. They are likely all set to '0' meaning power
     stays on. The lateral motors are on axis 4 and 5 so P405 and P505
     should be set to '1'.
   + Click on the yellow downwards pointing arrow on the toolbar in the
     editor. This downloads the modified PLC1 from the editor to the
     Delta Tau. Close the editor. 
   + In the terminal window issue a 'save' to save the modified
     configuration to the Delta Tau non-volatile memory and issue '$$$'
     to refresh the controller. 

A follow up from Graeme Elliner, FMBO
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   Just done a fast scan of the config file and I think it is probably
   because P302=1.

   Px02 and Px05 (where x is the motor number) are special Pvars for
   setting the final state of the motor once it has stopped moving, they
   are used in PLC1x and set as you know in PLC1

   If Px02=1 the PLC to check if the motor is in position and its
   desired velocity is zero, if these two conditions are set a Flag is
   set, If the conditions are still met 1second later then the motor
   is put into OPEN LOOP. This means the motor is still enabled but
   will ignore the encoder and the motor will hold its current rotary
   location. This is useful for the motors that have DPTs pushing
   against them in flexures (trapezoidal roll and pitch assy on the
   DCMs), it gives a firm base for the DPT to push against but will
   not try to hold position (as it would in closed loop) when the DPT
   pushes the top part of the stage and moves the encoder.  If Px05=1
   then the PLC checks to see if the motor is in position and has zero
   velocity, then 1second later it will kill that motor

   Due to the way the code is ordered (it looks for thePx02 first) it
   will enter Px02 check first, when the conditions are met it will
   set the first Flag After that check it then see the Px05 check and
   kills the motor. However on the next pass through the PLC it will
   again enter the Px02 check, see that the first flag has been set
   then trigger the open loop command, re-enabling the motor.

   Hence by setting P302=0 in PLC1, it will not go into the check and
   not accidentally enable the motor.  If this does not fix it then
   the issue is in EPICS

Conclusion
~~~~~~~~~~

The above suggestions were done for ``dm3_bct``, a motor that was
showing the re-enable behavior.  This made that motor tricky to
operate in bluesky. Setting ``P302=0`` and ``P305=1`` did the trick.


DM3 CAT6 Patch Panel
--------------------

13 more CAT6 ports for use in the hutch. Note that ports listed as
SCI/EPICS are tagged ports on both subnets.

This is needed by workstations (like ``xf06bm-ws5``), display machines
running CSS (like ``xf06bm-disp1``), and machines running IOCs (like
``xf06bm-xspress3``).

Note that ``xf06bm-em1`` needs to be on an INST port while the ion
chambers are on EPICS ports. The difference is that the ion chambers
are running their own on-board IOCs, making them more like IOC servers
than instruments.

.. todo:: Update me!


+-----------+----------+--------------------+----------------+---------------------+-------------------+
| **Patch** | **Port** |  **xf06bm-a port** |  **Network**   |  **Role**           |  **Cable number** |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
| **DM3-A** |  1       |  44                |  EPICS         |  xf06bm-ic1         |  200235           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  2       |  45                |  EPICS         |  xf06bm-ic2         |  200236           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  3       |  46                |  EPICS         |  xf06bm-ic3         |  200237           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  4       |  06bm-agg 36       |  INST          |  xf06bm-em1         |  200238           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
| **DM3-B** |  1       |  unused            |                |                     |  200239           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  2       |  18                |  SCI/EPICS     |  xf06bm-disp1       |  200240           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  3       |  28                |  CAM           |  xf06bm-cam8        |  200241           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  4       |  27                |  CAM           |  xf06bm-cam8        |  200242           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
| **DM3-C** |  1       |  37                |  INST          |  xf06bm-edxd1       |  200243           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  2       |  06bm-agg 29       |  INST          |  xf06bm-hiden1      |  200244           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  3       |  ??                |                |                     |  200245           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  4       |  unused            |                |                     |  200246           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
| **DM3-D** |  1       |  24                | CAM            |  xf06bm-cam7        |  200247           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  2       |                    | SCI/EPICS      |  xf06bm-xspress3    |  200248           |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  3       |                    |                |  unused             |                   |
+-----------+----------+--------------------+----------------+---------------------+-------------------+
|           |  4       |                    |                |  unused             |                   |
+-----------+----------+--------------------+----------------+---------------------+-------------------+


Some photos of the patch panel:

.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-dm3cat6
   :class-grid: outline

   .. image:: _images/infrastructure/DM3_patch_panel.jpg

   .. image:: _images/infrastructure/DM3_first_cat6.jpg

   (Left) CAT6 patch panel at DM3.  (Right) Lowest numbered label on
   the CAT6 cables in the DM3 patch panel


.. _dm3-bct:

DM3_BCT
-------

In early May, 2026, after a power failure, the motors on MC06 had to
be rehomed.  ``dm3_bct``, the vertical stage for the slit assembly,
failed in a very low position, but still some distance from the lower
limit switch.

The reason for the failure turned out to be that the encoder read
strip had delaminated at the end relevant to the lower range of the
stage (i.e. the top-most part of the read strip).  There is also a
noticeable kink in the read strip about 2 cm from the end.

After assisting the homing of the stage by manually pressing the lower
limit switch to allow the homing sequence to complete, the problem was
resolved by:

1. Resecuring the faulty portion of the read strip to the metal frame
   with some double sided tape.

2. Raising the lower limit switch by about 40 mm.

The lower limit switch was *very* low, far below the functional range
of motion of the slit assembly, even in the low energy modes of the
photon delivery system.  So, raising the limit switch should
have no impact on operations.  Doing so will avoid having the encoder
read head pass over the damaged part of the read strip.


.. _eiger:

Eiger2 Si 4M
------------

.. |br| raw:: html

      <br>

.. note:: Note that the vendor supplied camera server,
   ``xf06bm-eiger1``, is found in rack C2


.. _fig-eiger:
.. figure:: _images/detectors/eiger.jpg
   :target: _images/eiger.jpg
   :width: 70%
   :align: center

   The Eiger2 Si 4M detector with the front panel off, revealing the
   aluminized myler sheet protecting the detector surface.

:Coolling water:

   The detector requires that the chiller is running continuously.
   Connect the chiller lines by attaching the quick-connect fixtures.
   Note that the chiller should be filled with 2/3 water and 1/3
   ethylene glycol.  Dectris recommends `MotorX M5.0 coolant
   <https://www.motorexusa.com/products/coolant-m5-0-ready-to-use>`__
   as it also has corrosion protection chemicals in its formulation.
   That product is a 50/50 mix of water and ethylene glycol, so it
   needs to be diluted with 1 bottle-full of water for every two
   bottles of coolant.

   |br|
   
:Dry nitrogen:

   The detector also requires dry nitrogen flowing into the air fixture
   on the back of the box.  This protects the space around the detector
   from humidity by floding it with dry gas.

   |br|
   
:Lifting:

   When lifting the detector into position on the XAS table or
   goniometer, it is a good idea to use the hoist.  The hook can be
   connected directly to the eyebolt on the top of the detector.

   |br|
   
:Uncover:

   To remove the cover from the front of the detector, use a 2.5 mm
   hex driver in the two small holes about 1/3 from the top on each
   side.  These are a bit surprising.  Turn the screws `inward` to
   disengage the cover.  When replacing the cover, turn the screws
   outward.  This seems opposite to my intution, at least!  Take care
   not to apply much force on the screws |nd| simply turn them all the
   way in or out without applying any force at the end of the range of
   travel.
   
**To start the detector:**
   
#. Turn on the power switch on the power supply for the Eiger.  Once
   the green light on the back of the box begins blinking, press in the
   blue button on the back.
#. At a beamline computer, point a browser at http://xf06bm-eiger1.nsls2.bnl.local/#/system
#. Click the :key:`Reboot` button to restart the camera server.
#. Verify connections with the :key:`Check connections` button.
#. Click the :key:`Initialize detector` button.
#. Restart the Eiger ioc on ``xf06bm-det-ioc``.  This can be done by
   clicking the :key:`Reboot` button on the Eiger CSS screen.
#. Edit the ``BMM_configuration.ini`` file to enable the Eiger Ophyd
   object.  Restart |bsui| by doing

   .. code-block:: bash

      cd ~/.ipython/profile_collection
      pixi run start



   



Encoder loss second crystal roll
--------------------------------

On 9 January, 2018, when attempting to home the mono motors following
the schduled power outage in December, the 2\ :sup:`nd` crystal roll
motor moved to its negative limit, then reported an encoder loss.
With Graeme Elliner's (an FMB-O controls engineer) help, I came to a
resolution of the problem.  It has left that axis in an unusual state
that needs to be documented.

Executive summary: that axis does not use its encoder.  It homes by
running to its negative limit, then running back to it's home
position.  It does this by counting controller pulses rather than
encoder counts.

Here are a couple of useful emails from Graeme to me from January 11
and 12, 2018.

.. code-block:: text

   Hi Bruce

   We now need to work out where the problem is.
   NOTE you will not be able to drive anything while you're doing this or you potentially can break more.
   1: Unplug the Disable Plug from the back of the DCM (this will
      force all motors to be disabled) - it’s the small black connector
      (bottom right as you look at the back) 
   2: Disconnect PL102 & SK102 from IF2
   3: Disconnect PL103 & SK103 from IF3
   4: Connect PL103 & SK103 to IF2
   5: Connect PL102 & SK102 to IF3

   IF the Red light on the Interpolator stayed with IF3 then there is
   a problem with Interpolator - Need to put motor into Open Loop 

   IF the Red light on the Interpolator has moved to IF2 then the
   Interpolator is fine and it is cabling somewhere - GOTO STEP 6 

   6: SWAP PL102 and PL103

   IF the red light has moved back to IF3 then the problem is between
   PL103 to the read head on the Xtal2 Roll stage - GOTO STEP 7 

   IF the red light has stayed with IF2 then the problem is between SK103 to the MCS8
   
   This cabling is Pin to Pin so a simple continuity test on each pin should identify what has broken

   7: SWAP SK102 and SK103. The cabling should now be back to the original layout
   8: SWAP SK203-2 and SK203-3 at the feedthroughs on the DCM (FD3-2 & FD3-3 respectively)

   IF the red light has moved to IF2 then the problem is INSIDE the
   DCM vessel - Need to put motor into Open Loop (and ultimately open
   the vessel to find it) 

   If the red light has stayed on IF3 then there is a problem with the
   cable to the DCM. This cable is should be Pin to Pin so a simple
   continuity test on each pin should identify what has broken 


   To Put the Roll Axis into Open Loop
   Have you got PeWin working now??
   Using Pewin backup the config for the DCM and send it to me please.

A lot of the cable swapping Graeme called for was to try to isolate a
bad connection.  The connection between read-head and motor controller
is rather lengthy, with a vacuum feedthrough, a feedtrough on the side
of the service box, and two connections to the interpolators inside
the service box.

Following the steps laid out by Graeme, I isolated the problem to
being inside the vacuum vessel.  Drat! Using the `old MC02
configuration
<https://github.com/NSLS-II-BMM/BMM-beamline-configuration/blob/master/MCS8/mc02-11Jan2018.cfg>`__
I saved to a file, Graeme made some edits as described below and sent
me `a new configuration file
<https://github.com/NSLS-II-BMM/BMM-beamline-configuration/blob/master/MCS8/mc02-12Jan2018.cfg>`__.

.. code-block:: text

   Hi Bruce
   I have modified the config file to now not use the encoder for
   position feedback. I have tested that it downloads with no errors 

   Details of the mods are listed at the top of the file and below, I
   have marked all modifications with either GRE+ (for added code) or
   GRE- (for commented code) 

   In PLC1
   P446=0  this disables encoder loss detection for axis4


   In the Ivars
   I430=700        changed to default stepper gain for no encoder
   I432=0          changed to default value for no encoder
   I7040=8 this forces the system to use steps for feedback

   Use restore config from the backup menu in PEWin to install this
   CHECK that the box at the bottom reports NO ERRORS,
   in the terminal window you will need to "SAVE" and "$$$".

   You will now find that the position scaling will be completely
   different now that you are not using the encoder. This means that
   your jog speeds will also be different 

   I strongly suggest NOT trying to use EPICS imediately.

   Use the PeWin terminal (or the Jog Ribbon) to move axis 4 to the -ve
   limit ("#4j-") at the -ve limit type "#4HMZ" to zero the postion
   display and then to the +ve limit ("#4j+"). 

   This will tell you how many steps there are between the limits.

   Using this info and the data for the encoded version you should be
   able to move th axis to approximately the correct location. 

   I have noticed that in PLC14 (the homing PLC for axis 4) that even
   when the axis was using the encoder the home routine was not using
   the encoder home refernce. 

   It is moving to the -ve limit then moving off 51926 encoder counts,
   then setting this to be HOME  -  search for GRE*** in the file. 

   This will not be correct now the system is using steps and might
   actually be more than you have measured as the range in steps. 

   You will need to change this value before you can use EPICS to home the axis.

   Once all this is working in PeWin you can test the homing routine
   by entering M1416=1 in the Pewin terminal. 

Following this set of instructions, I found that there are 1,218,299
steps between the two limits on the 2\ :sup:`nd` crystal roll motor.
It would seem that there are about 10 or 12 steps per encoder count.
The homing procedure works in the sense of finding the negative limit,
then moving to a home position.  But that home position seems to be
about 1/10 of the way between the negative limit and the
home-using-encoder-counts.

Inert Gas Plumbing
------------------

Needle valves are mounted on the outboard side of DM3. Quick connect
outlets for the gases are mounted on the upstream/inboard corner of
the XAFS table.  

.. admonition::  Gaseous nitrogen supply
   :class: note
   
   BMM no longer uses a nitrogen cylinder as the supply of N\ :sub:`2`
   for the ion chambers.  The house GN2 supplies N\ :sub:`2` to the
   needle valves.

.. _fig-inertgas:
.. figure:: _images/infrastructure/Gas_handling.png
   :target: _images/Gas_handling.png
   :width: 100%
   :align: center

Vendor link for quick-disconnect fixture: https://www.mcmaster.com/5012K122/

In practice, the H\ :sub:`2`/N\ :sub:`2` and N\ :sub:`2`/Ar mixing
channels are not much used.  Unless measuring with the incident beam
below 5 keV or above 21 keV, it is a poor use of time to make changes
to the gas content of the ion chambers.  This is because it takes
quite some time for the volume of the ion chamber to equillibrate.

N\ :sub:`2` is adequate for almost all experiments at BMM.  For Tc or
Ru, it is helpful to use about 20% Ar.  For Sc or lower, 50% He might
be helpful.  But remember that purging the ion chambers takes
**hours**.










Logitech controller
-------------------

.. caution::

   The Logitech controllers are handy, but scary.  They are not a toy.
   If you play around with one, you are likely to ruin your experiment.


There are two Logitech controllers configured to move particular
motors at the beamline.  One controller lives permanently inside the
hutch, usually located on the white box hanging from the side of
diagnostic module 3.  The other lives permanently outside the hutch in
the clear plastic box next to the control station.

The purpose of the Logitech controllers is to facilitate rough
alignment of sample in the beam. The assumption is that careful
alignment will be done with X-rays.

.. _fig-logitech:
.. figure:: _images/instrumentation/Logitech.png
   :target: _images/Logitech.png
   :width: 100%
   :align: center


:green:`Green A button`
   Open the photon shutter (outside controller only)

:red:`Green B button`
   Open the photon shutter (outside controller only)

:blue:`Blue X button`
   Fully retract the detector, move ``xafs_detx`` to 205

:darkyellow:`Yellow Y button`
   Disable amplifiers on ``dcm.pitch``, ``dcm.roll``, and
   ``m2.bender``, equivalent to ``dcm.kill()`` + ``m2.bender.kill()``
   at the |bsui| command line

Right joystick
   Control the sample XY stage

Left joystick
   Control the detector Z stage (left/right, up/down do nothing,
   inside controller only)

Right trigger buttons
   Rotate the reference wheel

Left trigger buttons
   Rotate the *ex situ* sample wheel

Logitech button
   Wake up the controller


.. todo::

   Explain how to configure controls in CSS


M2 Bender
---------

The homing sequence on the bender (MC04, channel 6) seems to have the
wrong parity.  Instead of moving to the negative limit, it moves to
the positive limit.

Rather than running the homing procedure, step to the negative limit.
This can be done via EPICS or by issuing the ``#6j-`` command in the
PEWIN terminal.  Adam Young from FMBO indicated that the home marker
is about +6000 steps from the negative limit, so move there and zero
out the display |nd| ``#6HMZ`` in PEWIN.  This will leave the bender
reporting to EPICS that it is not homed, but it can be moved sensibly
to a bend value.

=============  ========================== 
 End station    Bender position in steps
=============  ========================== 
 XAS            212225
 XRD            107240 
=============  ========================== 

MC09 patch panel
----------------

Photo and explain...

In hutch, patch panel for MC10 is in place but unfinished


MCS8 network configuration
--------------------------

Among the curiosities of managing the FMBO MCS8 TurboPMAC motor
controllers delivered as part of the original construction of the
beamline, there is the matter of network configuration.  While there
is a tool on the NIST Windows 8 laptop for configuring the IP address
of the MCS8 over a USB cable, there is a bit of work that has to be
done for IP configuration to take effect.

The MCS8 units have something called EEPROM Write Protection which
keeps the IP configuration from being changed by accident.  At least,
I suppose that's the purpose...

To bypass write protection, there is a jumper on one of the circuit
boards that has to be in place.  Here is :download:`a PDF file
explaining how to do this <_static/MCS8-E8-jumper.pdf>`.

MC01 through MC06 are FMBO-supplied controllers.  These cover the
entire photon delivery system up to diagnostic module 3.  All of the
XAS or XRD end station motors use NSLS-II standard Geobricks.

Monochromator notes
-------------------

The Bragg axis drive train has been a recurring source of trouble at
BMM.  In the past, the point of coupling between the drive motor and
the Bragg goniometer has been the main issue.  This coupling is made
between the spindle of the drive motor and the goniometer's work gear
via a bellows.  Despite assurances from the mono vendor that this
drive train has been reliable on all other similar goniometers
(including 3 others here at NSLS-II), BMM has is on its fifth bellows
as of January 2026.

The part in question is shown in :numref:`Figure %s <fig-braggaxis>`.
An example of a sheared bellows is shown in :numref:`Figure %s
<fig-shearedbellows>`.


.. _fig-braggaxis:
.. figure:: _images/infrastructure/bragg_axis.jpg
   :target: _images/bragg_axis.jpg
   :width: 70%
   :align: center

   The Bragg axis drive train. The bellows coupler is inside the part
   indicated with the red arrow and circle.

.. _fig-shearedbellows:
.. figure:: _images/infrastructure/sheared_bellows.jpg
   :target: _images/sheared_bellows.jpg
   :width: 70%
   :align: center

   An example of a sheared bellows

Replacement bellows purchased from HUCO are in cabinet #4 in a gray
bin labeled "Photon delivery system spare parts". 

These bellows can be purchased directly from HUCO:
`HUCO 530-41, 41 mm Diameter Bellows Coupling
<https://www.huco.com/shop/couplings/bellows-couplings/bellows-flex-b/530-41?b1=24&b2=24>`__
with one bore size of |half| inch and the other |quarter| inch.

Bragg axis failure
~~~~~~~~~~~~~~~~~~

In the first half of the 2026-1 cycle, we spent some time repairing
several problems with the monochromator.

The troubles began with a failure of the Bragg axis to move without
encoder loss error.  Some preliminary investigation showed that the
drive shaft was behaving as though the worm gear was losing contact
on every revolution with the teeth of the goniometer stage.

After removing the whole crystal cage from the FOE and disassembling
the mono, we found three problems that together resulted in the loss
of the Bragg axis:

1. The o-rings on the vacuum feed-through were abraded, causing
   additional resistance to rotation.  New o-rings were purchase,
   installed a lubricated.  Additional 0-rings can be found in the
   photon delivery supplies bin in Cabinet 8.

2. The lubricant on the spring inside the pre-load assembly for the
   Bragg drive train was dried up and crusty, causing the spring not
   to retract and release freely.  This spring was cleaned and
   relubricated. 

3. The lateral retaining clamps on the work gear |nd| two small steel
   plates bolted to the side of the worm gear assembly |nd| were
   broken.  New plates were procured from the goniometer vendor, `RPI
   <https://www.rpiuk.com/>`__, and installed.  Two additional plates
   are in the photon delivery supplies bin in Cabinet 8.

The likely result of these three issues was that the enhanced friction
on the rotary feed-through along with the sticky pre-load was causing
the gearing to become disengaged on every revolution.

Once all this was repaired and replaced, the Bragg axis moved well.

.. admonition:: Bragg axis coordinate system
   :class: note

   Note that the coordinate system of the Bragg axis is not consistent
   with the NSLS-II coordinate system.  The rotation sense of the
   Bragg axis is left-handed!  This means the parity of the limit
   switches is opposite of what you might expect.


.. _fig-braggcoordinate:
.. figure:: _images/monochromator/bragg_coordinate_system.png
   :target: _images/bragg_coordinate_system.png
   :width: 75%
   :align: center

   The coordinate system used by the monochromator motor controller,
   MC02.  Figure taken from the manual provided by the vendor, XDS
   Oxford.


.. admonition:: Encoder setting for the Bragg axis
   :class: note

   The encoder directional sense must be set correctly for the primary
   and secondary Bragg encoders.  The PMAC variables ``I7010`` and
   ``I7020`` both must be set to 7.  This detail is recorded here as
   it proved to be a source of confusion as we were troubleshooting.

Bragg encoder
~~~~~~~~~~~~~

One problem we had during the 2026-1 repair effort involved the
encoder for the Bragg axis.  :numref:`Figure %s <fig-axialencoder>`
shows a confusing view of the Bragg axis.  This view is behind the
mounting plate for the crystal cage, looking down the beam direction
along the door of the vacuum vessel.  

.. _fig-axialencoder:
.. figure:: _images/monochromator/axial_encoder.jpg
   :target: _images/axial_encoder.jpg
   :width: 75%
   :align: center

   The encoder for the Bragg axis.  The encoder read-head is indicated
   by the red arrow.  The magnet for the Hall sensor is circled in
   magenta.


As we started doing motion testing, the cable leading to the read head
was not dressed correctly.  As we moved to a high angle (i.e. a low
energy orientation), the cable caught on a tab holding the magnet for
the Hall sensor in the encoder read-head.  This stretched and damaged
the cable, breaking communication with the interpolator.  This was
fixed by replacing the read-head.

However, this read-head (model number RGH20F30M03C) is long obsolete.
Renishaw no longer sells this model and it was (at the time of the
2026-1 repair effort) unavailable on eBay.  Happily, XDS Oxford had an
unopened spare in their shop which they kindly gave to us.

In future interventions |nd| for instance scheduled maintenance of the
rotary feed through |nd| it is **essential** that great care is taken
to dress that cable correctly.  Replacing it again with the same model
will almost certainly be impossible.


Pitch motor cable
~~~~~~~~~~~~~~~~~

As part of the 2026-1 repairs, the motor cable to  the pitch axis was
partly re-wired and is now using a different DB15 vacuum feedthrough
compared to what is documented in the original vendor supplied
documentation. 

The DB15 feedthrough is correctly labeled on the exterior of the
vacuum vessel.

This is mentioned to document this discrepancy with the vendor
documentation.



Motor controllers
-----------------

This section is a big, long list of all the motor PV names at BMM.

Some motors have aliases.  The alias is an alternate, easier-to-type
name for the axis.  These are equivalent:

.. code-block:: sh

   caget XF:06BMA-OP{Mono:DCM1-Ax:Bragg}Mtr
   caget xafs_bragg

Aliases work with motor record fields, as well.  The following are
also equivalent:

.. code-block:: sh

   caget XF:06BMA-OP{Mono:DCM1-Ax:Bragg}Mtr.VELO
   caget xafs_bragg.VELO


The following tables give PV name and alias, a brief description of
the purpose of the motor, the controller and location of that
controller, and the channel number in the controller.  A few
abbreviations are used:  

:us: upstream
:ds: downsteam
:ib: inboard
:ob: outboard
:para: parallel
:perp: perpendicular


Collimating mirror, M1
~~~~~~~~~~~~~~~~~~~~~~

================================  =========  =========================  ======================  ==============
PV                                alias      Motor Description          controller              motor number
================================  =========  =========================  ======================  ==============
XF:06BM-OP{Mir:M1-Ax:YU}Mtr       m1_yu      us jack                    MC01 (mezzanine)        1
XF:06BM-OP{Mir:M1-Ax:YDO}Mtr      m1_ydo     ds, outboard jack          MC01 (mezzanine)        2
XF:06BM-OP{Mir:M1-Ax:YDI}Mtr      m1_ydi     ds, inboard jack           MC01 (mezzanine)        3
XF:06BM-OP{Mir:M1-Ax:XU}Mtr       m1_xu      us lateral                 MC01 (mezzanine)        4
XF:06BM-OP{Mir:M1-Ax:XD}Mtr       m1_xd      ds lateral                 MC01 (mezzanine)        5 
================================  =========  =========================  ======================  ==============

Filters, DM1
~~~~~~~~~~~~

================================  ============  =========================  ======================  ==============
PV                                alias         Motor Description          controller              motor number
================================  ============  =========================  ======================  ==============
XF:06BMA-BI{Fltr:01-Ax:Y1}Mtr     dm1_filters1  assembly #1                MC05 (RGA)              6
XF:06BMA-BI{Fltr:01-Ax:Y2}Mtr     dm1_filters2  assembly #2                MC05 (RGA)              7 
================================  ============  =========================  ======================  ==============

DCM
~~~

===================================  ============  ====================  ======================  ==============
PV                                   alias         Motor Description     controller              motor number
===================================  ============  ====================  ======================  ==============
XF:06BMA-OP{Mono:DCM1-Ax:Bragg}Mtr   dcm_bragg     DCM Bragg             MC02 (RGA)              1
XF:06BMA-OP{Mono:DCM1-Ax:Bragg2}Mtr  dcm_bragg2    Bragg 2nd encoder     MC02 (RGA)
XF:06BMA-OP{Mono:DCM1-Ax:P2}Mtr      dcm_pitch     2nd xtal pitch        MC02 (RGA)              3
XF:06BMA-OP{Mono:DCM1-Ax:R2}Mtr      dcm_roll      2nd xtal roll         MC02 (RGA)              4
XF:06BMA-OP{Mono:DCM1-Ax:Per2}Mtr    dcm_para      2nd xtal perp         MC02 (RGA)              5
XF:06BMA-OP{Mono:DCM1-Ax:Par2}Mtr    dcm_perp      2nd xtal para         MC02 (RGA)              6
XF:06BMA-OP{Mono:DCM1-Ax:X}Mtr       dcm_x         lateral               MC02 (RGA)              7
XF:06BMA-OP{Mono:DCM1-Ax:Y}Mtr       dcm_y         vertical              MC02 (RGA)              8 
===================================  ============  ====================  ======================  ==============

Slits 2, DM2
~~~~~~~~~~~~

==============================  ============  ====================  ======================  ==============
PV                              alias         Motor Description     controller              motor number
==============================  ============  ====================  ======================  ==============
XF:06BMA-OP{Slt:01-Ax:O}Mtr     dm2_slits_o   outboard              MC03 (RGA)              1
XF:06BMA-OP{Slt:01-Ax:I}Mtr     dm2_slits_i   inboard               MC03 (RGA)              2
XF:06BMA-OP{Slt:01-Ax:T}Mtr     dm2_slits_t   top                   MC03 (RGA)              3
XF:06BMA-OP{Slt:01-Ax:B}Mtr     dm2_slits_b   bottom                MC03 (RGA)              4 
==============================  ============  ====================  ======================  ==============


DM2 fluorescence screen
~~~~~~~~~~~~~~~~~~~~~~~

==============================  ============  ====================  ======================  ==============
PV                              alias         Motor Description     controller              motor number
==============================  ============  ====================  ======================  ==============
XF:06BMA-BI{Diag:02-Ax:Y}Mtr    dm2_fs        vertical              MC04 (RGA)              7 
==============================  ============  ====================  ======================  ==============

Focusing mirror, M2
~~~~~~~~~~~~~~~~~~~


==============================  ============  ====================  ======================  ==============
PV                              alias         Motor Description     controller              motor number
==============================  ============  ====================  ======================  ==============
XF:06BMA-OP{Mir:M2-Ax:YU}Mtr    m2_yu         us jack               MC04 (RGA)              1
XF:06BMA-OP{Mir:M2-Ax:YDO}Mtr   m2_ydo        ds, outboard jack     MC04 (RGA)              2
XF:06BMA-OP{Mir:M2-Ax:YDI}Mtr   m2_ydi        ds, inboard jack      MC04 (RGA)              3
XF:06BMA-OP{Mir:M2-Ax:XU}Mtr    m2_xu         us lateral            MC04 (RGA)              4
XF:06BMA-OP{Mir:M2-Ax:XD}Mtr    m2_xd         ds lateral            MC04 (RGA)              5
XF:06BMA-OP{Mir:M2-Ax:Bend}Mtr  m2_bender     bender                MC04 (RGA)              6 
==============================  ============  ====================  ======================  ==============

Harmonic rejection mirror, M3
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

==============================  ============  ====================  ======================  ==============
PV                              alias         Motor Description     controller              motor number
==============================  ============  ====================  ======================  ==============
XF:06BMA-OP{Mir:M3-Ax:YU}Mtr    m3_yu         us jack               MC05 (RGA)              1
XF:06BMA-OP{Mir:M3-Ax:YDO}Mtr   m3_ydo        ds, outboard jack     MC05 (RGA)              2
XF:06BMA-OP{Mir:M3-Ax:YDI}Mtr   m3_ydi        ds, inboard jack      MC05 (RGA)              3
XF:06BMA-OP{Mir:M3-Ax:XU}Mtr    m3_xu         us lateral            MC05 (RGA)              4
XF:06BMA-OP{Mir:M3-Ax:XD}Mtr    m3_xd         ds lateral            MC05 (RGA)              5 
==============================  ============  ====================  ======================  ==============

Slits 3, DM3
~~~~~~~~~~~~

==============================  ============  ====================  ======================  ==============
PV                              alias         Motor Description     controller              motor number
==============================  ============  ====================  ======================  ==============
XF:06BM-BI{Slt:02-Ax:O}Mtr      dm3_slits_o     outboard            MC06 (RGC1)             5
XF:06BM-BI{Slt:02-Ax:I}Mtr      dm3_slits_i     inboard             MC06 (RGC1)             6
XF:06BM-BI{Slt:02-Ax:T}Mtr      dm3_slits_t     top                 MC06 (RGC1)             7
XF:06BM-BI{Slt:02-Ax:B}Mtr      dm3_slits_b     bottom              MC06 (RGC1)             8
==============================  ============  ====================  ======================  ==============

DM3
~~~

==============================  ============  ====================  ======================  ==============
PV                              alias         Motor Description     controller              motor number
==============================  ============  ====================  ======================  ==============
XF:06BM-BI{FS:03-Ax:Y}Mtr       dm3_fs        fluorescent screen     MC06 (RGC1)            1 
XF:06BM-BI{Fltr:01-Ax:Y}Mtr     dm3_foils     foils actuator         MC06 (RGC1)            4
XF:06BM-BI{BCT-Ax:Y}Mtr         dm3_bct       vertical stage         MC06 (RGC1)            3
XF:06BM-BI{BPM:1-Ax:Y}Mtr       dm3_bpm       NanoBPM                MC06 (RGC1)            2 
==============================  ============  ====================  ======================  ==============

XAFS Table
~~~~~~~~~~

========================================  ====================  ========================= ======================  ==============
PV                                        alias                 Motor Description         controller              motor number
========================================  ====================  ========================= ======================  ==============
XF:06BMA-BI{XAFS-Ax:Tbl_YU}Mtr            xafs_yu               xafs table y us            MC07 (RGC1)             1
XF:06BMA-BI{XAFS-Ax:Tbl_YDO}Mtr           <unused>              <available>                MC07 (RGC1)             2
XF:06BMA-BI{XAFS-Ax:Tbl_YDI}Mtr           xafs_yd               xafs table y ds            MC07 (RGC1)             3
:strike:`XF:06BMA-BI{XAFS-Ax:Tbl_XU}Mtr`  :strike:`xafs_xu`     :strike:`xafs table x us`  :strike:`MC07 (RGC1)`   :strike:`4`
XF:06BMA-BI{XAFS-Ax:Tbl_XD}Mtr            :red:`xafs_detx`      xafs detector stage x      MC07 (RGC1)             5
========================================  ====================  ========================= ======================  ==============

.. note::

   As part of a problem in 2024 involving a shorted motor coil on the
   ``xafs_detx`` stage, the table horizontal motors got repurposed.
   Axis 4 suffered a damaged amplifier.  After replacing the motor,
   the ``xafs_detx`` was placed on axis 5.

   Axis 4 on MC07 is out of service. ``xafs_xu`` and ``xafs_xd`` are
   not currently connected to a motor controller.

   In Summer 2026, ``xafs_ydi`` and ``xafs_ydo`` were coupled together
   after leveling the table.  ``xafs_ydi`` was renamed to ``xafs_yd``.
   The controls for the :numref:`XAFS table (see Section %s)
   <xafs_table>` were rewritten to use the single motor and to remove
   roll from the list of coordinated motions.  Axis 2 on MC07 is
   currently unused and available.


XAFS Stages
~~~~~~~~~~~

Note that, over time, some of the consistency between the EPICS
nomenclature and the ophyd nomenclature has diverged.  For example,
the EPICS and ophyd names for ``xafs_pitch`` and ``xafs_roll`` are
mismatched due to how we chose to mount the :numref:`Huber tilt stage
(see Section %s) <tilt-stage>`.

More recently, the sample XY stages were replaced with encoded stages.
The older sample stages |nd| channels 1 and 2 on MC08 |nd| were
repurposed for holding the :numref:`Eiger (section %s) <eiger>` or
:numref:`Pilatus (section %s) <pilatus>` area detectors.  The new,
encoded stages are on channels 6 and 7 on MC09.  Happily, the EPICS
names on channel 9 do not try to capture the purpose of the axis.  The
semantic naming of those axes is better left for the ophyd layer.

.. note::
   In cabinet 4, there are some useful stages that are unused.  

   #. a Huber theta/2theta stage
   #. a Huber theta stage
   #. a set of 4-axis slits.

XAFS stages on MC08
*******************

======================================  ==================  ===============================  ======================  ==============
PV                                      alias               Motor Description                controller              motor number
======================================  ==================  ===============================  ======================  ==============
XF:06BMA-BI{XAFS-Ax:LinY}Mtr            xafs_ady            area detector y                  MC08 (RGC1)             1
XF:06BMA-BI{XAFS-Ax:LinX}Mtr            xafs_adx            area detector x                  MC08 (RGC1)             2
:strike:`XF:06BMA-BI{XAFS-Ax:LinS}Mtr`  :strike:`xafs_det`  :strike:`xafs reference stage`   :strike:`MC08 (RGC1)`   :strike:`3`
XF:06BMA-BI{XAFS-Ax:LinXS}Mtr           xafs_refy           xafs reference y                 MC08 (RGC1)             4
XF:06BMA-BI{XAFS-Ax:Pitch}Mtr           xafs_roll           xafs roll stage                  MC08 (RGC1)             5
XF:06BMA-BI{XAFS-Ax:Roll}Mtr            xafs_pitch          xafs pitch stage                 MC08 (RGC1)             6
XF:06BMA-BI{XAFS-Ax:Ref}Mtr             xafs_ref            xafs reference wheel             MC08 (RGC1)             7
XF:06BMA-BI{XAFS-Ax:Mtr8}Mtr            xafs_garot          glancing rotation                MC08 (RGC1)             8
======================================  ==================  ===============================  ======================  ==============

XAFS stages on MC07
*******************

======================================  ==================  ===============================  ======================  ==============
PV                                      alias               Motor Description                controller              motor number
======================================  ==================  ===============================  ======================  ==============
XF:06BMA-BI{XAFS-Ax:Tbl_RefX}Mtr        xafs_refx           xafs reference x                 MC07 (RGC1)             6
XF:06BMA-BI{XAFS-Ax:Tbl_RotB}Mtr        xafs_wheel          xafs wheel stage                 MC07 (RGC1)             7
XF:06BMA-BI{XAFS-Ax:Tbl_RotS}Mtr        xafs_rots           xafs small rot stage             MC07 (RGC1)             8
======================================  ==================  ===============================  ======================  ==============


XAFS stages on MC09
*******************

======================================  ==================  ===============================  ======================  ==============
PV                                      alias               Motor Description                controller              motor number
======================================  ==================  ===============================  ======================  ==============
XF:06BMA-BI{MC:09-Ax:1}Mtr              xafs_dety           xafs detector stage y            MC09 (RGC1)             1
XF:06BMA-BI{MC:09-Ax:2}Mtr              xafs_detz           xafs detector stage z            MC09 (RGC1)             2
XF:06BMA-BI{MC:09-Ax:3}Mtr              xafs_spare          spare xafs stage                 MC09 (RGC1)             3
XF:06BMA-BI{MC:09-Ax:4}Mtr              xafs_bsx            beam stop stage y                MC09 (RGC1)             4
XF:06BMA-BI{MC:09-Ax:5}Mtr              xafs_bsy            beam stop stage y                MC09 (RGC1)             5
XF:06BMA-BI{MC:09-Ax:6}Mtr              xafs_x              xafs sample stage x              MC09 (RGC1)             4
XF:06BMA-BI{MC:09-Ax:7}Mtr              xafs_y              xafs sample stage y              MC09 (RGC1)             5
======================================  ==================  ===============================  ======================  ==============

Gonimeter circles
~~~~~~~~~~~~~~~~~

=============================  =============  =====================  ======================  ==============
PV                                alias         Motor Description     controller             motor number
=============================  =============  =====================  ======================  ==============
XF:06BM-ES{SixC-Ax:VTTH}Mtr    6bm:sixc_vtth   Vertical two theta      MC11 (RGC2)            1
XF:06BM-ES{SixC-Ax:VTH}Mtr     6bm:sixc_vth    Vertical theta          MC11 (RGC2)            2
XF:06BM-ES{SixC-Ax:CHI}Mtr     6bm:sixc_chi    Chi                     MC11 (RGC2)            3
XF:06BM-ES{SixC-Ax:PHI}Mtr     6bm:sixc_phi    Phi                     MC11 (RGC2)            4
XF:06BM-ES{SixC-Ax:HTH}Mtr     6bm:sixc_hth    Horizontal theta        MC11 (RGC2)            5
XF:06BM-ES{SixC-Ax:HTTH}Mtr    6bm:sixc_htth   Horizontal two theta    MC11 (RGC2)            6
XF:06BM-ES{SixC-Ax:ANAL}Mtr    6bm:sixc_anal   Analyzer                MC11 (RGC2)            7
XF:06BM-ES{SixC-Ax:DET}Mtr     6bm:sixc_det    Detector                MC11 (RGC2)            8
=============================  =============  =====================  ======================  ==============

Goniometer motors
~~~~~~~~~~~~~~~~~

==============================  ==============  ====================  ======================  ==============
PV                                alias         Motor Description     controller               motor number
==============================  ==============  ====================  ======================  ==============
XF:06BM-ES{SixC-Ax:DETHOR}Mtr   6bm:sixc_det_h  det horiz               MC12 (RGC2)            1
XF:06BM-ES{SixC-Ax:WHEEL1}Mtr   6bm:sixc_wh1    wheel 1                 MC12 (RGC2)            2
XF:06BM-ES{SixC-Ax:WHEEL2}Mtr   6bm:sixc_wh2    wheel 2                 MC12 (RGC2)            3
XF:06BM-ES{SixC-Ax:SAMX}Mtr     6bm:sixc_samx   sample X                MC12 (RGC2)            4
XF:06BM-ES{SixC-Ax:SAMY}Mtr     6bm:sixc_samy   sample Y                MC12 (RGC2)            5
XF:06BM-ES{SixC-Ax:SAMZ}Mtr     6bm:sixc_samz   sample Z                MC12 (RGC2)            6
XF:06BM-ES{SixC-Ax:Tbl_YD}Mtr   6bm:sixc_tyd    table Y ds              MC12 (RGC2)            7
XF:06BM-ES{SixC-Ax:Tbl_YUI}Mtr  6bm:sixc_tyui   table Y us ib           MC12 (RGC2)            8
XF:06BM-ES{SixC-Ax:...}Mtr                      detector ...            MC14 (RGC2)            1
XF:06BM-ES{SixC-Ax:...}Mtr                      detector ...            MC14 (RGC2)            2
==============================  ==============  ====================  ======================  ==============



Goniometer table
~~~~~~~~~~~~~~~~

==============================  ===============  ====================  ======================  ==============
PV                                alias          Motor Description     controller               motor number
==============================  ===============  ====================  ======================  ==============
XF:06BM-ES{SixC-Ax:Tbl_YUO}Mtr  6bm:sixc_tyuo    table Y us ob          MC13 (RGC2)             1
XF:06BM-ES{SixC-Ax:Tbl_XU}Mtr   6bm:sixc_txu     table X us             MC13 (RGC2)             2
XF:06BM-ES{SixC-Ax:Tbl_XD}Mtr   6bm:sixc_txd     table X ds             MC13 (RGC2)             3
XF:06BM-ES{SixC-Ax:Tbl_Z}Mtr    6bm:sixc_tz      table Z                MC13 (RGC2)             4
XF:06BM-ES{SixC-Ax:Slt1_T}Mtr   6bm:sixc_slt1_t  top slit               MC13 (RGC2)             5
XF:06BM-ES{SixC-Ax:Slt1_B}Mtr   6bm:sixc_slt1_b  bottom slit            MC13 (RGC2)             6
XF:06BM-ES{SixC-Ax:Slt1_I}Mtr   6bm:sixc_slt1_i  inboard slit           MC13 (RGC2)             7
XF:06BM-ES{SixC-Ax:Slt1_O}Mtr   6bm:sixc_slt1_o  outboard slit          MC13 (RGC2)             8
==============================  ===============  ====================  ======================  ==============



Shutters and screen
~~~~~~~~~~~~~~~~~~~

================================  =========  ======================  ======================  ==============
PV                                alias      Motor Description       controller              motor number
================================  =========  ======================  ======================  ==============
XF:06BM-PPS{Sh:FE}Pos-Sts                    front end shutter       PPS     
XF:06BM-PPS{Sh:A}Pos-Sts                     A hutch shutter         PPS     
XF:06BMA-OP{FS:1}Pos-Sts                     fluorescent screen      EPS     
================================  =========  ======================  ======================  ==============



Front-end slits
~~~~~~~~~~~~~~~

================================  =========  ======================  ======================  ==============
PV                                alias      Motor Description       controller              motor number
================================  =========  ======================  ======================  ==============
FE:C06B-OP{Slt:12-Ax:X}size                  horizontal size         geobrick (mezzanine)    virtual
FE:C06B-OP{Slt:12-Ax:X}center                horizontal center       geobrick (mezzanine)    virtual
FE:C06B-OP{Slt:12-Ax:Y}size                  vertical size           geobrick (mezzanine)    virtual
FE:C06B-OP{Slt:12-Ax:Y}center                vertical center         geobrick (mezzanine)    virtual
FE:C06B-OP{Slt:1-Ax:Hrz}Mtr                  Slit 1 horizontal       geobrick (mezzanine)    
FE:C06B-OP{Slt:1-Ax:Inc}Mtr                  Slit 1 incline          geobrick (mezzanine)    
FE:C06B-OP{Slt:1-Ax:O}Mtr                    Slit 1 X outboard       geobrick (mezzanine)    
FE:C06B-OP{Slt:1-Ax:T}Mtr                    Slit 1 Y top            geobrick (mezzanine)    
FE:C06B-OP{Slt:2-Ax:Hrz}Mtr                  Slit 2 horizontal       geobrick (mezzanine)    
FE:C06B-OP{Slt:2-Ax:Inc}Mtr                  Slit 2 incline          geobrick (mezzanine)    
FE:C06B-OP{Slt:2-Ax:I}Mtr                    Slit 2 X inboard        geobrick (mezzanine)    
FE:C06B-OP{Slt:2-Ax:B}Mtr                    Slit 2 Y bottom         geobrick (mezzanine)
================================  =========  ======================  ======================  ==============










Network configuration
---------------------

When introducing a new device to the beamline network:

+ Open a Jira ticket to request a new IP address and device name for
  DNS.  As an example, a new Moxa terminal server was introduced.  In
  the Jira ticket, we requested an IP address of 10.68.42.83 and a DNS
  entry of ``xf06bm-tsrv13`` (full name:
  ``xf06bm-tsrv13.nsls2.bnl.local``). 

+ In the case of a device that one in s series of similar devices, be
  sure that the name matches the similar devices and that the number
  is incremented.  In the case of the new Moxa server, there are 12
  others at the beamline.  The numbers were incremented correctly for
  both the name and the IP address

+ Identify a port on one of the servers that is available and
  configured for the correct network.  The networks are:

  + **SCI** (650): 10.68.40.xxx
  + **CAM** (651): 10.68.41.xxx
  + **INST** (652): 10.68.42.xxx
  + **EPICS** (653): 10.68.43.xxx

When configuring an individual device, here are the network configurations:

+ The gateway address on each network is ``xz.yy.zz.2``.  For example,
  the gateway on the SCI network is 10.68.40.2. 

* The netmask is ``255.255.255.0``.

+ There are two DNS servers:

  + DNS1: 10.65.2.25
  + DNS2: 10.65.2.26




.. _pilatus:   

Pilatus 100K
------------


The Pilatus camserver is running on ``xf06bm-pilatus100k-651``, which
is in the rack on wheels in the hutch.  ``xf06bm-pilatus100k-651`` is
``10.68.41.29`` and must be on the CAM network.  The network cable
should be plugged into the port on the left on the back of the server
in the rack.

The IOC for the pilatus is running on ``xf06bm-ioc1``.  Note that this
is the only IOC running on ``xf06bm-ioc1``.


Starting the system
~~~~~~~~~~~~~~~~~~~

After logging in as the ``det`` user:

+ Start the NFS server on ``xf06bm-pilatus100k-651``.  This can be
  done with the YAST GUI or by ``sudo /etc/init.d/nfsserver restart``
+ Start the NTP server on ``xf06bm-pilatus100k-651``.  This can be
  done with the YAST GUI or by ``sudo /etc/init.d/ntp restart``
+ Make sure that ``/disk2`` on ``xf06bm-pilatus100k-651`` is mounted
  by ``xf06bm-ioc1`` at ``/disk2`` on that machine.  That mount should
  happen at boot and be available as long as the Pilatus server is on
  the network.
+ Turn on the detector by clicking the switch on the back.
+ Start the camserver on ``xf06bm-pilatus100k-651``.  This is done in
  a terminal by ``start-camserver`` in the ``det`` home directory.
+ Start (or restart) the IOC on ``xf06bm-ioc1`` by clicking the
  :key:`Reboot` button on the Pilatus screen or by doing ``dzdo
  manage-iocs restart Pilatus100K`` at the command line.

Overview of file saving
~~~~~~~~~~~~~~~~~~~~~~~

+ The camserver writes tiff files to ``/disk2`` on
  ``xf06bm-pilatus100k-651``.
+ That folder is NFS mounted on ``xf06bm-ioc1`` as ``/disk2``.
+ The tiff and hdf5 AD plugins read individual images from that
  location and write tiff or hdf5 files to proposal directories.
+ In |bsui|, there are ``pilatus`` and ``pilatus_tiff`` objects.  We
  normally use ``pilatus``, which writes images to HDF5 files.  The
  ``pilatus_tiff`` object is helpful for testing tiff file writing,
  which is used by IBM.

.. warning:: A commonly observed problem with the Pilatus is related
	     to the use of NFS to move images between the camserver
	     and the IOC server.  The NFS connection requires that the
	     time stamp on the image file not be more than 10 seconds
	     out of date.  Without a running NTP server on
	     ``xf06bm-pilatus100k-651``, the clock on that machine
	     will eventually fall more than 10 seconds behind the
	     clock on ``xf06bm-ioc1``.  This will manifest as
	     befuddling problems when trying to interact with the
	     detector |md| count times in |bsui| will be very long, the
	     image will not update on the CSS screen, the PVA and HDF5
	     plugins will report 0x0 image sizes, etc.  The solution
	     is to ssh to ``xf06bm-pilatus100k-651``, restart the NTP
	     server using the command above, and verify the clock time
	     against another computer.  After that, restart the IOC,
	     then restart |bsui| by doing

	     .. code-block:: bash

		cd ~/.ipython/profile_collection
		pixi run start



.. note:: There is a cron job running on ``xf06bm-ioc1`` which deletes
	  any tiff files in ``/disk2`` that are over an hour old.
	  The benefit is that this keeps ``/disk2`` from filling up.
	  The risk is that the HDF5 plugin for the Pilatus IOC must be
	  working correctly for data to be preserved properly.

Configuring the pilatus and tiff AD plugins
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


In CSS, configure the pilatus plugin parameters highlighted in yellow.
Note that in Bluesky all of this is handled automagically by the ophyd
object.

The file path **must** be set to ``/disk2``.

The file name needs to be something, but is unimportant.  TThese
images will be copied by the IOC to the proposal folder or written by
the IOC to an HDF5 file in the proposal folder.

Make sure the auto increment is set to Yes and that the filename
format is identical to the format used on the tiff plugin screen.
``%s%s_%3.3d.tiff`` is a good choice.

.. _fig-pilatus_plugin:
.. figure:: _images/Pilatus/pilatus_plugin.png
   :target: _images/pilatus_plugin.png
   :width: 70%
   :align: center

   Pilatus plugin configuration screen


On the tiff plugin configuration screen |nd| which is needed for the
``pilatus_tiff`` object but not for the ``pilatus`` object using the
HDF5 plugin |nd| several of the parameters must be set for any
measurement.

The file path must be a location in the user's proposal folder on
shared storage.  Thus, the path must be something like 

.. code-block:: text

   /nsls2/data/bmm/proposals/2025-1/pass-123456/assets/pilatus100k-1

where you would replace the cycle number (``2025-1`` in this example)
with the current cycle number and the proposal number (``123456`` in
this example) with the user's proposal number.

The IOC is only capable of writing to a location in the ``assets``
folder.  By good practice, it will write to the folder under
``assets`` created for it.  In the case of our Pilatus 100K, that
would be ``pilatus100k-1``.

The filename format **must** be the same as on the pilatus plugin
screen.  ``%s%s_%3.3d.tiff`` is a good choice.

The write mode **must** be ``stream`` and auto save **must** be
``yes``.

Much of this is handled in Bluesky by the ophyd object.

.. _fig-tiff_plugin:
.. figure:: _images/Pilatus/tiff_plugin.png
   :target: _images/tiff_plugin.png
   :width: 70%
   :align: center

   Tiff plugin configuration screen


Setting ROIs
~~~~~~~~~~~~

To set and use ROIs, the ROI and Stats plugins must be enabled in the
IOC.  This can be done on the appropriate CSS screen, but all
necessary plugins should become enabled when |bsui| starts and the
pilatus object is instantiated.


.. _fig-roi1:
.. figure:: _images/Pilatus/pilatus_rois_1.png
   :target: _images/pilatus_rois_1.png
   :width: 70%
   :align: center

   Initial configuration of the ROIs.  In this example, ROIs 1 and 2
   are being set to a subsection of the full image.

To interact with the ROIs, their visualization must be enabled by
clicking the corresponding buttons near the bottom of the CSS screen,
as shown in the figure below.  With those buttons clicked, the ROIs
will be outlined on the image.


.. _fig-roi2:
.. figure:: _images/Pilatus/pilatus_rois_2.png
   :target: _images/pilatus_rois_2.png
   :width: 70%
   :align: center
	   
   ROIs are displayed on top of the Pilatus image on the CSS screen.

You can then click on the outline in the image to move or resize the ROI.



Measurements with the Pilatus in bluesky
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In |bsui|, the total-counts statistic (from the STATS plugin) for ROI2
is called ``diffuse`` and is hinted.  This means that the total count
rate in the ROI will be recorded as a scalar in Tiled and that the
count rate will be reported in the best effort callback table printed
to the screen during the scan.

Similarly, the total count rate in ROI3 is called ``specular`` and is
hinted.  

Those names are chosen as mnemonics for obvious parts of the
scattering signal.

The header of the output XAFS scan file contains a metadata line
identifying the location of the HDF5 file containing the Pilatus image
stack for the scan.


Moving the detector between end stations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Pilatus and its rack typically live in the downstream, outboard
corner of the hutch, near the XRD end station.  For use on the XAS
table, it is convenient to move the entire rack closer to the XAS table.

You will need to power down the server, the rack-mount computer in the
rack.  Then:

#. Unplug the power cables to the power strip
#. Disconnect the network cable from the back of the server
#. Valve off the GN2 supply at the wall, then disconnect the purple
   polyflow tube
#. Remove the green grounding line from the back wall.

Now the rack can be moved.

There is a power outlet and an GN2 supply fixture on the back wall,
near the end of the XAS table.  There is also a copper grounding strip
near the floor that the green grounding cable can be clipped to.  Also
in that area is a blue network cable with a label that says "Pilatus @
XAS".  Plug that into the back of the server.

At the downstream network patch panel, unplug the network cable used
when the rack is in its m=normal location.  Search around for the
other end of the "Pilatus @ XAS" cable and plu it into the patch
panel.

Open flow of GN2 to the detector.  Power up the server.  Make sure the
power switch for the detector is on.

Once you have logged into the server as the ``det`` user, open a
terminal window and do ``../start_camserver`` to power up the
detector.

Make sure the time is correct on the Pilatus server

On ``xf06bm-ioc1``, restart the ``Pilatus100K`` IOC with

.. code-block:: bash

   dzdo manage-iocs restart Pilatus100K

You should be good to go.

..
  Time synchronization
  ~~~~~~~~~~~~~~~~~~~~

  .. note:: This was fixed May 2025 by setting the gateway to the
     correct subnet on ``xf06bm-pilatus100k-651`` then restarting the
     NTP server.

  Although the Pilatus server is configured to use an NTP server, it is
  (as of April 2025) unable to connect.  As a result, the system time is
  likely to be quite wrong, possibly by several minutes.

  The part of the IOC that moves images from ``/disk2`` and saves them
  on central storage either as HDF5 or tiff files requires that the
  system times on the two machines be no more than a few seconds apart.

  To synchronize the system time on ``xf06bm-pilatus100k-651`` with
  ``xf06bm-ioc1`` do this:

  #. On an ``xf06bm-ioc1`` command line, enter ``date``.  This will
     print a string like so:

     .. code-block:: text

	Wed Apr  2 07:35:49 PM EDT 2025

     Copy that string.

  #. On a command line on ``xf06bm-pilatus100k-651``, do this command:

     .. code-block:: text

	date -s "Wed Apr  2 07:35:49 PM EDT 2025"

     replacing that string with the correct date and time from
     ``xf06bm-ioc1``.

     That should do it.


.. _provision:

Provision a new beamline computer
---------------------------------

This is a list of notes on how to finish the provisioning of a new
beamline computer.

Firstly, make sure that ``/nsls2/data`` is a symlink to
``/nsls2/data3``.  If it is not, ask for help from DSSI.


install additional packages
~~~~~~~~~~~~~~~~~~~~~~~~~~~

+ plasma-desktop (just ... better)
+ redis (essential for operation of |bsui|)
+ most (used as the pager in BMM's |bsui| profile)
+ ag (powerful ack-like grep alternative)
+ fswebcam (used to capture analog pinhole camera)
+ demeter and perl-Graphics-GnuplotIF (something silly, no doubt)
+ slack (communications)
+ ark (compression, useful in file manager)


To install these, do:

.. code-block:: sh

   dzdo dnf install redis most ag fswebcam demeter perl-Graphics-GnuplotIF slack ark
   dzdo dnf install --skip-broken --nobest @kde-desktop

The second command installs the KDE metapackage, skipping missing
packages. 

To finish installing ``sddm``, do 

.. code-block:: sh

   dzdo systemctl stop gdm
   dzdo systemctl start sddm

Desktop wallpaper
~~~~~~~~~~~~~~~~~

This may not be provisioned correctly out of the box.  Find the
beamline wallpapers in ``/usr/share/nsls2/wallpapers/beamlines``. 

Right click on the desktop and select "Configure Desktop and
Wallpaper".  Click on "Add Image" and navigate to the folder above.

Things to install from git
~~~~~~~~~~~~~~~~~~~~~~~~~~

+ BMM stuff: ``git clone git@github.com:NSLS-II-BMM/BMM-beamline-configuration.git``
  + then, ``cd ~/bin`` and ``ln -s ~/git/BMM-beamline-configuration/tools/run-cadashboard``
+ BMM user manual: ``git clone git@github.com:NSLS2/bmm-beamline-manual.git``
+ BMM standards: ``git clone git@github.com:NSLS2/bmm-standards.git``
+ Switch visualization: ``git clone git@github.com:NSLS-II-BMM/switch-pretty-printer.git``

Also do ``cd ~/bin`` and ``ln -s ~/.ipython/profile_collection/startup/consumer/run-consumer``

Workspace folders
~~~~~~~~~~~~~~~~~

Make the local data collection folders.  The
:numref:`BMMuser.begin_experiment() command (Section %s) <start_end>`
will make symlinks under those folders to the correct place on central
storage.

.. code-block:: text

   mkdir ~/Workspace
   mkdir ~/Workspace/Visitors
   mkdir ~/Workspace/Staff



.. _rack_cabinets:

Shelving Units and Cabinets
---------------------------

===============    =======    ======================================================
 Storage unit       Short      Purpose
===============    =======    ======================================================
 Shelf 1            S-1        Outside hutch |nd| this n that
 Shelf 2            S-2        Outside hutch |nd| BioLogic, Linkam, cat6 cables
 Shelf 3            S-3        Inside hutch |nd| experiment supplies
 Cabinet 1          C-1        IBM storage
 Cabinet 2          C-2        IBM storage
 Cabinet 3          C-3        IBM storage
 Cabinet 4          C-4        Cryostat parts, mono & mirror parts, yield detector
 Cabinet 5          C-5        Samples, old stuff
 Cabinet 6          C-6        Old instruments, Poly-Flo tubing, misc. cables
 Cabinet 7          C-7        IBM storage
 Cabinet 8          C-8        Books, games, office supplies, USB & video cables
===============    =======    ======================================================



.. _fig-racks_and_cabinets:
.. figure:: _images/infrastructure/racks_cabinets.png
   :target: _images/racks_cabinets.png
   :width: 100%
   :align: center

   Shelf and cabinet locations







.. _table_height:

Setting the table height
------------------------

This section outlines how the table height settings in the `Modes
lookup table
<https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/lookup_table/Modes.xlsx>`__
were found.

For starters, the positions of ``dm3_bct`` in each of the modes were
established.  This was done at 300 eV above the Pt L3 edge for modes
A, D, and E; the Fe K edge for mode C, and the Cr K edge for mode F.

Armed with set ``dm3_bct`` positions, the setup shown in
:numref:`Figure %s <fig-tableheightmeasurement>` was used to find the
positions of the XAFS table jacks.


.. _fig-tableheightmeasurement:
.. figure:: _images/stages/slits_table_height.jpg
   :target: _images/slits_table_height.jpg
   :width: 100%
   :align: center

   Table height measurement.

The I\ :sub:`r` detector is moved downstream of the two downstream
jacks on the XAFS table.  The I\ :sub:`t` detector is moved downstream
slightly to the other side of the upstream jack.

The manual slit assembly is mounted such that the opening is about 1/4
mm tall and centered in its housing.  The height of the slit opening
is carefully set to be at the same height as the centers of all the
ion chambers.

With the slits in front of I\ :sub:`t`, do a scan of the ``xafs_yu``
motor looking at the signal on I\ :sub:`t`.  Move to the peak of this
scan.

With the slits in front of the I\ :sub:`r` detector, adjust
``xafs_ydo`` and ``xafs_ydi`` in unison to optimize the signal on I\
:sub:`r`. 

Move the slits back to the first position and verify that the
``xafs_yu`` position still maximized the signal.

Record these positions in the `Modes lookup table
<https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/lookup_table/Modes.xlsx>`__.

For the lower energy edges, insert the flight path borrowed from the
XRD end station in between I\ :sub:`t`  and I\ :sub:`r`.  Run He
through to reduce the attenuation of the beam at lower energies.




.. _holding_current:

XAFS table holding current
--------------------------

The controller axes for the vertical motion of the XAFS table were
originally configured  without a holding current.  Over time, we
noticed that the XAFS table would lose its position by a couple
millimeters over the course of months.  In an effort to combat this
drift, a holding current was applied to those three axes in
January 2025. 

The three vertical table axes are channels 1, 2, and 3 on MC07, which
is a Geobrick Power PMAC.

This turned out to be a little complicated.  It turns out that the
holding current is controlled by PLC7 in the Geobrick Power PMAC
controllers.  That PLC uses several P-variables on the controller, as
explained in `the comment block at the top of the PLC code
<https://code.nsls2.bnl.gov/xf/06bm/iocs/xf06bm-ioc/mc07/-/blob/master/tpmacApp/pmc/xf06bma-mc07-plc07-power-down.pmc>`__.
(Note: that link requires logging onto the BNL VPN.)

Here is the text of that comment:

.. code-block:: text

   ; Note1: Geobrick controllers when killed effectively short the motor cables together providing an
   ; brake due to back EMF when the motor is rotated. Most axis can be safely killed without losing 
   ; position, this is certainly the case for most lead screw drives (ball screws may require a 
   ; holding current).
   ; 
   ; Note2: When using this PLC make sure the standard kill PLC (usually PLC7, sometimes PLC3) is removed from the geobrick.
   ; 
   ; Settings: 
   ; P701-P708
   ;   Define timeout period in milliseconds after which the axis will be powered 
   ;   down provided it has been idle for the whole period (note clock resolution below).
   ;   Set to zero to leave the amp powered continously. 
   ;   Set to one for (almost) immediate power down on motor stop.
   ;   Set to number of milliseconds for delayed power down after motor stops.
   ;   Typically this should be set to a few seconds.
   ; P733-740
   ;   Drive current & percentage for each axis.
   ;   Set to zero for axis that are to be killed or if controller is not a geobrick (i.e. does not support Ixx77 amp current).
   ;   For axis requiring a reduced holding current this contains ' normal_drive_current * 100 + power_down_percentage '
   ;   Note the drive current is defined in milliamps.
   ;   e.g. For motor axis 1 with 2000 milliamp drive current and 33% holding
   ;   current define P733 as 200033

OK!  So how do we make all that happen?  Firing up PEWIN on the
Windows laptop connected to MC07, open the windows for viewing the
I-variables and the P-variables.

All the relevant parameters were set in PEWIN as shown in the figures
below.  MC07 was then power cycled.  After power cycling, those three
axes have a good strong holding current in place.

A holding current of 30% of drive current was selected because the
table is rather heavy.  Jakub Wlodek's suggestion was "I usually go
with 10% unless it is lifting something heavy".  


.. subfigure::  ABC
   :layout-sm: ABC
   :subcaptions: above
   :gap: 8px
   :name: fig-holding-current
   :class-grid: outline

   .. image:: _images/infrastructure/timeout.png

   .. image:: _images/infrastructure/holding_current.png

   .. image:: _images/infrastructure/drive_current.png

   (Left) The P-variables window showing parameters P701-708.  The
   values P701, P702, and P703 are for axes 1, 2, and 3.  The value of
   5000 |nd| disable amplifier 5000 ms after a move |nd| are changed
   to the value of 0, thus leaving the amplifiers enabled.  (Middle)
   The P-variables window showing parameters P733-740.  The values
   P733, P734, and P735 are for axes 1, 2, and 3.  These are set to
   supply a holding current of 30% of the drive current using the
   specified formula.  (Right) The I-variables window showing the
   relevant parameter for axis 1.  The I177 parameters is set to 1800
   mA, as are I277 and I377.  This value was used to compute the value
   of the P733 - 735 parameters.

.. note::

   To effect this change in the holding current, it seems you have to
   press the "Upload To Editor" button at the top of the P-variables
   window.  In truth, I am not 100% sure about that step, but it seems
   as though the changes to the P-variables and the subsequent changes
   to the PLC do not take effect without that step.

   Maybe this could be revisited by someone with more knowledge of how
   PEWIN and the Power PMAC actually work...


Vortex pressure
---------------

Using a probe to measure the voltage on the IP port of the Vortex ME4.
This reading will tell you the internal pressure according to the
table in the snapshot below.  

.. _fig-votexpressure:
.. figure:: _images/detectors/Vortex_pressure.jpeg
   :target: _images/Vortex_pressure.jpeg
   :width: 40%
   :align: center


======================  ==========
 IP reading (Voltage)    Pressure
======================  ==========
 -0.01                   5E-9
 -0.1                    5E-8
 -1                      5E-7
 -10                     5E-6 
======================  ==========

Note that the voltages are positive on the 7-element and negative on
the 4-element.

Temperature reading on the 4-element should be 1.5 V when the TEC is
at proper temperature.  Temperature should be 0.6 V on the 7-element
detector.

`Vortex SDD manual
<https://www.aps.anl.gov/files/download/DET/Detector-Pool/Spectroscopic-Detectors/Vortex_SDD/Vortex_ME4/Vtx-ME4%20Multi-El%20User%20Manual%20Rev.4.pdf>`__
(link to copy at APS detector pool).

There is a copy of the Vortex manual at BMM.  Look in
``/nsls2/data3/bmm/legacy/products/ME7/``, the file is called
``Vtx-Multi-El User Manual Rev 15.0_Oct 16, 2023.pdf``.




Zoom calls in the hutch
-----------------------

``xf06bm-ws5`` (``10.68.40.225``) is the System 76 Meerkat mounted on
the inboard wall of the end station behind the XAS table and just
below the lower, right corner of the screen.  This machine is intended
to allow the users in the hutch to join a Zoom chat from within the
hutch.  This allows staff to provide user support from home.


.. _fig-ws5:
.. figure:: _images/infrastructure/xf06bm-ws5.jpg
   :target: _images/xf06bm-ws5.jpg
   :width: 70%
   :align: center


There are a number of peripherals attached to ``xf06bm-ws5``:

+ A wireless mouse and keyboard clearly labeled as being for this
  computer. These are normally sitting near the end of the XAS table.

+ A screen. This is the screen mounted in the back wall of
  the hutch using an articulated arm.

+ A reasonably loud speaker.  This is the black ball-shaped item which
  usually sits underneath the I\ :sub:`r` chamber.  Could be louder...

+ A good microphone. This is the Blue Yeti on a boom above the I\
  :sub:`r` chamber.  It has good noise cancellation so the din from the
  XSpress3 should not effect voice quality.  It is, however, important
  that the speaker face the microphone when speaking.

+ A decent camera. This is the Nexigo mounted on the inboard wall to 
  the right of the screen.


While these devices are all connected to ``xf06bm-ws5`` and powered
on, there are no long running processes that connect to the camera or
microphone. You are not being spied upon while in the hutch -- unless
you are on a Zoom call, in which case the Zoom session will be on
screen.

``xf06bm-ws5`` is available via Guacamole.  When needed for remote
support, Bruce will initiate the Zoom call and have the hutch computer
join in.




