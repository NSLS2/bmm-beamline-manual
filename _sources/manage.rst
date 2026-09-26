..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. role:: key
   :class: key


.. _manage:

Managing the beamline
=====================


In this section, some high-level concepts of operations at BMM are
explained.

See :numref:`Section %s <iocs>` for details about the IOCs running the
beamline instrumentation.


.. _start_end:

Starting and ending an experiment
---------------------------------

When a new experiment begins, run this command::

  BMMuser.begin_experiment(name='Betty Cooper', date='2026-02-29', gup=123456, saf=654321)

This will:

1. Identify the correct proposal folder,
   i.e. ``/nsls2/data/bmm/proposals/<cycle>/pass-123456`` 
2. Make a local working folder |nd| ``~/Workspace/Visitors/Betty
   Cooper/2026-02-29`` |nd| and populate it with templates for
   :numref:`automation spreadsheets (Section %s) <automation>` and
   example :numref:`INI files (Section %s) <ini>`.
3. Configure detectors and cameras to use the assets folders in the
   proposal folder.
4. Run the ``sync_experiment()`` command, identifying the user in the
   run engine's metadata dictionary.
5. Configures the bot that writes Slack messages to use the channels
   for this proposal.  Writes a message to the proposal Slack channel
   explaining how to access data and subscribes the NSLS2 Operations
   Monitor to the Slack channel for beamline messages.
6. Sets the proposal number and safety approval form number as
   common metadata for all measurements.  Fetches the names of all
   experimenters on the proposal and sets that as common metadata.
7. Sets the |bsui| command line prompt to it's green-text state,
   indicating that an experiment is active.


The ``name`` should be the PI's full name, preferably transliterated
into normal ASCII.  The ``date`` should be the starting day of the
experiment in the ``YYYY-MM-DD`` format.  The ``GUP`` and ``SAF``
numbers can be found on the posted safety approval form.

Once the experiment is finished, run this command::

  BMMuser.end_experiment()

This will undo all the above and set the |bsui| prompt to its red-text
state, indicating that no experiment is active.


Change energy
-------------

Changing energy is simple.  Usually, it is :numref:`as simple as doing
(Section %s) <change_edge>`

.. code-block:: python
		
   RE(change_edge('Fe'))

replacing the two-letter element symbol with the element you actually
want to measure. This command will move the monochromator, put the
photon delivery system in the correct mode, move the M2 bender to
approximately the correct setting, run a rocking curve scan,
optimize the slit height, move the reference foil holder to the
correct position (if configured), and select the correct ROI channel
(if configured).


If you want to reproduce this by hand, here is the command sequence:


#. First move the DCM to the new energy position.  It is usually a
   good idea to move a bit above the target edge energy.  Here's an
   example for moving 50 eV above the iron K edge energy:

   .. code-block:: python

      RE(mv(dcm.energy, 7112+50))

#. Put the beamline in the correct photon delivery system mode.  (See
   the table just above.)  Continuing with the example of the iron K
   edge, for unfocused beam:

   .. code-block:: python

      RE(change_mode('E'))

   If the new edge energy is in the same energy range according to the
   table above, you can skip this step.  For example, Mn and Fe are
   both in mode E (or mode C).  The ``change_mode()`` command does not
   need to be run to move between those edges.

#. Measure a :numref:`rocking curve scan (Sec %s) <special-linescans>`
   to verify that the second crystal of the rocking curve is parallel
   to the first crystal.  This is more important for large energy
   changes.  You may find that you can skip this step if you are
   changing between nearby edges.

   .. code-block:: python

      RE(rocking_curve())

   At the end of the scan, the mono pitch will be moved to the top of
   the rocking curve.

#. If using focused beam, make sure that the mirror bender is in the
   correct position.  For focusing at the XAS table, ``m2_bender``
   should be at about 212000 counts.  For focusing at the position of
   the goniometer, ``m2_bender`` should be about 112000 counts.

   .. code-block:: python

      RE(mv(m2_bender, 212000))

#. Next, verify that the pitch of the final mirror is optimized to
   deliver the brightest part of the beam into the :numref:`end
   station slits (Section %s) <slits3>`.  In principle, this should be
   very close after changing photon delivery system mode.  But it
   doesn't hurt to verify.

   .. code-block:: python

      RE(mirror_pitch(mirror='m3'))

   At the end of the scan, you will need to pluck the peak position
   from the plot.

   The scanned mirror should be M3 for all :numref:`photon delivery
   modes (Table %s) <pds_modes>` except for Mode A, in
   which case you should scan M2.

#. Next, if you are using a reference foil, you should move the
   reference foil holder to the slot containing the correct foil.  The
   command is something like:

   .. code-block:: python

      RE(reference('Fe'))

   choosing the correct element for your measurement.

#. Finally,  select the correct ROI channel:

   .. code-block:: python

      BMMuser.verify_roi(xs, 'Fe', 'K')


As a reminder, here is the table of operating modes.

.. _pds_modes:
.. table:: Photon delivery modes
   :name:  pds-modes2
   :align: left

   ====== ============ ========================= 
   Mode   focused      energy range
   ====== ============ ========================= 
   A      |checkmark|  above 8 keV
   B      |checkmark|  below 6 keV
   C      |checkmark|  6 keV |nd| 8 keV
   D      |xmark|      above 8 keV
   E      |xmark|      6 keV |nd| 8 keV
   F      |xmark|      below 6 keV
   XRD    |checkmark|  above 8 keV
   ====== ============ ========================= 


..
 Change mode
 -----------

 Suppose that you want to change from high-energy, unfocused operations
 to low energy, focused.  That is, you are changing from mode D to mode
 B, for example moving from a large sample at the yttrium K edge to a
 small sample at the vanadium K edge.

 .. code-block:: python

		 RE(change_mode('B'))
		 RE(mv(dcm.energy, 5465+50))
		 RE(rocking_curve())
		 RE(slit_height())

		 
    #. If the beam has recently been focused at the XRD station, you will
       also need to adjust the bender on M2 to optimize vertical focus at
       the XAS station (or vice versa).  This is best done with the small
       CCD camera sitting in the XAS sample stage.
       
    #. Again, iterating the optimization of the rocking curve and slit
       height might be necessary.

Change crystals
---------------

Suppose you wanted to change from the Pt L3 edge (11564 eV) on the
Si(111) crystal to the same energy on the Si(311) crystal.

.. code-block:: python

   RE(change_xtal('311'))

This will move the lateral motor of the DCM and optimize the roll and
pitch of the second crystal.  It will then move the DCM to the energy
that you started at with the other crystal set and run a rocking curve
scan.

Note that some of these motions can be a bit surprising in the sense
that the monochromator will briefly report itself as being outside the
normal operating range of the beamline.  They will, however,
eventually return to sensible places.


.. _xas-to-xrd:

Change XAS |larr| XRD
---------------------

To move the photon delivery system to delivery of focused beam to the
goniometer:

.. code-block:: python

   RE(xrd_mode())

That command is a thin wrapper around this call to the change_edge command

.. code-block:: python

   RE(change_edge('Ni', xrd=True, energy=8600, slits=True, mirror=False))

The element symbol in the first argument is not actually used in any
way when ``xrd=True`` is used, however the funtion requires
`something` as its first argument.  Setting ``xrd=True`` forces the
``focus=True`` and ``target=0`` arguments to the ``change_edge()``
command to be set.  This will move to the specified energy, place the
photon delivery mode in `XRD` mode, optimize the second crystal and
the slit height, and move to an appropriate M2 bender position.

To do all of that by hand, you would do the follow commands:

.. code-block:: python

   RE(change_mode('XRD'))
   RE(mv(dcm.energy, 8600))
   RE(rocking_curve())
   RE(mirror_pitch(mirror='m2'))

This change of mode should have the beam in good focus at the position
of the goniometer.  8600 eV is the nominal operating energy for the
goniometer.  If a higher energy is required, substitute the correct
energy for ``8600`` in the second line.

It is prudent to adjust the yaw of the focusing mirror to optimize
the shape and inclination of the focused beam at the goniometer.  This
is done by placing the direct beam camera in the beam path just down
stream of the sample position at the center of the goniometer.  Then
do

.. code-block:: python

   RE(mvr(m2.yaw, 0.05))

or

.. code-block:: python

   RE(mvr(m2.yaw, -0.05))

while observing the beam on the camera.


.. todo:: Work with DSSI BLOP team to optimize DCM roll and M2 yaw &
	  lateral for all energies above 8000 eV at the XRD end
	  station.

.. todo:: Determine look-up table for lower energy operations using
	  both M2 and M3.  This will require a new XAFS table and
	  adjustments to the limit switches on ``m3_ydo`` and
	  ``m3_ydi``.


.. _use333:

XAFS with Si(333)
-----------------

Using the Si(111) monochromator, it is possible to use the third
harmonic |nd| the Si(333) reflection |nd| to measure XAS with slightly
higher energy resolution.  In this section, we explain how to set up
the beamline to measure the Ge K edge at 11103 eV using the Si(333).

You cannot use the ``change_edge()`` command to do this.  Use of the
Si(111) (or Si(311)) is hard-wired into that plan.  You have to set up
the beamline by hand.

First, put the photon delivery system in mode D (or mode A if using
the focusing mirror):

.. code-block:: python

   RE(change_mode('D'))

Next, move the monochromator to a few 10s of eV above the absorption
edge, as measured with the third harmonic.  The Ge K edge is at 11103
eV, so we need to move the monochromator to 11103/3 = 3701 eV.

.. code-block:: python

   RE(mv(dcm.energy, (11103+27)/3))

or simply

.. code-block:: python

   RE(mv(dcm.energy, 3701+9))

This will put the third harmonic energy 27 eV above the Ge K edge.

Now, run a rocking curve scan:

.. code-block:: python

   RE(rocking_curve())

This will produce a plot that looks something like this:

.. _fig-rocking333:
.. figure:: _images/software/rocking_curve_333_E=3716.png
   :target: _images/rocking_curve_333_E=3716.png
   :width: 70%
   :align: center

   A rocking curve scan with the photon delivery system in mode D and
   the mono at 3716 eV.

The broad base of this curve is the Si(111) rocking curve with photons
at 3710 eV. The sharp spike in the middle is the Si(333) rocking curve
with photons at 11130 eV.

Optimize the slit_height:

.. code-block:: python

   RE(slit_height())

You are ready to measure XAS with the Si(333) reflection!

Here's an example ``scan.ini`` file for XANES of elemental Ge:  

.. code-block:: ini

   [scan]
   experimenters = Bruce Ravel

   filename      = Ge
   sample        = elemental Ge, crystalline
   prep          = standard sample
   comment       = measured with Si(333) reflection, 25um Al foil in beam path before I0

   ththth        = True
   e0            = 11103
   element       = Ge
   edge          = K

   nscans        = 1
   start         = next

   ## mode is one of transmission, fluorescence, both, or reference
   mode       = transmission

   ## Ge Si(333)
   bounds     = -45    -18     -9      36    150
   steps      =      9     0.9     0.3    0.9
   times      =      0.5    0.5    0.5    0.5
 

Several things to note:

#. Note that the actual value for E0 is specified, not the divided-by-3 value.  
#. Actual energy bounds and steps are specified, the xafs scan plan
   will convert them to appropriately sized steps for the Si(111).
#. By setting the 333 flag to True, the correct thing will happen,
   including writing the correct energy axis to the output data file.
#. The on-screen plot will show the fundamental |nd| Si(111) |nd| energy, however.  
#. Also, you still need to set up the photon delivery system up by hand.



Motor controller kill switches
------------------------------

The MCS8 motor controllers supplied by FMBO have a kill switch for
power cycling the Phytron amplifier cards.  This is implemented by the
vendor as connector plugged into the back of the chassis which shorts
the two leads of the receptacle.  To kill the amplifiers, this plug is
removed and reinserted.

That's fine, but the motor controllers are on top of the FOE |nd| not
a convenient location.

The new kill switch system uses DIODE to close the kill switch
circuit. Two-conductor cable is run from each motor controller to a
remote DIODE box mounted on the inboard wall of the end station.

The Bluesky interface is defined `here
<https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/BMM/killswitch.py>`__



From the docstring of the class: 

.. code-block:: none

   A simple interface to the DIODE kill switches for the Phytron
   amplifiers on the FMBO Delta Tau motor controllers.

   In the BMM DIODE box, these are implemented on channels 0 to 4 of
   slot 4.

   attributes
   ----------
   dcm 
     kill switch for MC02, monochromator
   slits2
     kill switch for MC03, DM2 slits
   m2
     kill switch for MC04, focusing mirror
   m3
     kill switch for MC05, harmonic rejection mirror
   dm3
     kill switch for MC06, hutch slits and diagnostics

   methods
   -------
   kill(mc)
     disable Phytron
   enable(mc)
     activate Phytron
   cycle(mc)
     disable, wait 5 seconds, reactivate, then re-enable all motors

   Specify the motor controller as a string, i.e. 'dcm', 'slits2', 'm2', 'm3', 'dm3'
   
   Here is a common problem which is resolved using a kill switch.

      BMM E.111 [36] ▶ RE(mvr(m2.pitch, 0.05))
      INFO:BMM_logger:    Moving m2_pitch to 2.550

      Moving m2_pitch to 2.550
      ERROR:ophyd.objects:Motion failed: m2_yu is in an alarm state status=AlarmStatus.STATE severity=AlarmSeverity.MAJOR
      ERROR:ophyd.objects:Motion failed: m2_yu is in an alarm state status=AlarmStatus.STATE severity=AlarmSeverity.MAJOR
      ERROR:ophyd.objects:Motion failed: m2_ydi is in an alarm state status=AlarmStatus.STATE severity=AlarmSeverity.MAJOR
      ERROR:ophyd.objects:Motion failed: m2_ydi is in an alarm state status=AlarmStatus.STATE severity=AlarmSeverity.MAJOR
      Out[36]: ()

   This is telling you that the amplifiers for two of the M2 jacks
   went into an alarm state. In the vast majority of cases, this
   simply requires killing and reactivating those amplifiers.

   The solution to this one is:

      BMM E.111 [1] ▶ ks.cycle('m2')
      Cycling amplifiers on m2 motor controller
      killing amplifiers
      reactivating amplifiers
      enabling motors

Unresponsive motor controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If all the motors on a single motor FMB MCS8 controller are
non-responsive, it is possible that the kill switch for that
controller has been triggered inadvertently.  This happened, for
example, one time that a ``change_edge()`` command was interrupted by
:key:`Control`-:key:`c` :key:`Control`-:key:`c`.

In that case, check the front panel of the motor controller,
:numref:`Figure %s <fig-disabled_light>`.  If the light labeled
"Disabled" is illuminated, then it is likely that kill switch for that
controller is open.

.. _fig-disabled_light:
.. figure:: _images/infrastructure/disabled_light.jpg
   :target: _images/disabled_light.jpg
   :width: 50%
   :align: center

   Front panel of an FMBO MCS8 controller.

In that case, try enabling that kill switch.  At the |bsui| command
line, do

.. code-block:: python

   ks.enable('m2')

That example re-enables the kill switch for the focusing mirror
controller.  The other possible arguments for the enable command are 

+ ``dcm``: motor controller 2, monochromator
+ ``slits2``:  motor controller 3, post-mono slits
+ ``m2``: motor controller 4, focusing mirror
+ ``m3``: motor controller 5, harmonic rejection mirror
+ ``dm3``: motor controller 6, diagnostic module 3 and hutch slits

After running the ``ks.enable()`` command, check the "Disabled" light
on the front of the controller.  If it is off, the motors should be
responsive again.



Old kill switch system
~~~~~~~~~~~~~~~~~~~~~~

.. note:: The panel and cabling for the old kill switch system is
	  still in place, but the controller-side ends of the cables
	  are at the bottom of the rack.  Should the DIODE system
	  somehow fail, this can be redeployed easily.

There is a row of switches on rack D, the rack next to the control
station, that are used to disable the amplifiers for the MCS8 motor
controllers.  See :numref:`Figure %s <fig-killswitches>`.

.. _fig-killswitches:
.. figure:: _images/infrastructure/Kill_switches.jpg
   :target: _images/Kill_switches.jpg
   :width: 70%
   :align: center

   The manual kill switch system


When you suspect that a motor has an amplifier fault, toggle the
appropriate switch to the off position.  Wait 10 seconds (to be very
safe...).  Then toggle the switch back to the on position. The motor
should be ready to go. These switches replace the shorted plugs that
came attached to the "disable" port on the back side of the MCS8s.

=======  ==========================  ===============================   =================================
 MCS8     RGA label                   RGD label                         motors
=======  ==========================  ===============================   =================================
 MC02     6BM-100149-RG:A1-PT1B3-A    6BM-100149-RG:A1-PT1B3-B	        DCM
 MC03     6BM-100150-RG:A1-PT1B3-A    6BM-100150-RG:A1-PT1B3-B	        slits2
 MC04     6BM-100151-RG:A1-PT1B3-A    6BM-100151-RG:A1-PT1B3-B	        M2 + DM2 FS
 MC05     6BM-100152-RG:A1-PT1B3-A    6BM-100152-RG:A1-PT1B3-B	        M3 + Filters
 MC06                                 <installed, not yet labeled>      DM3 (bct,bpm,fs,foils)+ Slits3
=======  ==========================  ===============================   =================================

In the situation where toggling the switch does not clear the
amplifier fault, the next troubleshooting step is to power cycle the
MCS8.  This is done by toggling the red, illuminated switch on the
front of the MCS8.  Wait for the red amplifier lights to stop
flickering after turning off the MCS8, then turn the MCS8 back on.

After power cycling the MCS8, it is necessary to re-home all the
motors controlled by the MCS8.



MCS8 Connector
~~~~~~~~~~~~~~

The disable plug on the back of the MCS8 controllers is a Binder RS
connector, part number 468-885. `Here's a vendor example
example. <https://uk.rs-online.com/web/p/industrial-automation-circular-connectors/0468885/?sra=pstk>`__
(Accessed July 2025)

And here is the wiring diagram.  Short the prongs on the side opposite
to the alignment groove.

.. _fig-killswitcheconnector:
.. figure:: _images/infrastructure/Kill_switch_connector.png
   :target: _images/Kill_switch_connector.png
   :width: 30%
   :align: center

Tutorial for how to put together the Binder connectors: :download:`PDF <_static/Binder-instructions.pdf>`


.. _windows_vm:

Windows VM and BioLogic
-----------------------

We use the `BioLogic EC_lab software
<https://www.biologic.net/support-software/ec-lab-software/>`__ to run
the `VSP-300 potentiostat
<https://www.biologic.net/products/vsp-300/>`__.  Since there is not a
dedicated Windows machine at BMM, EC-Lab is run on a virtual machine
that is spun up when needed.  

Similarly, Hiden's control software is a Window's only produce.  It is
run on the same firtual machine.

Here are the instructions for starting and interacting with the
the VM.

Starting the virtual machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+ At a command line, do ``xfreerdp /v:xf06bm-srv2 /u:xf06bm /size:75%wh``
+ You will be prompted for a password.  Normal BNL credentials do not
  work. Log in using the password known by beamline staff.
+ This will open a new window and display a Windows desktop. 
+ The Windows desktop might start with a full-screen management
  application that looks like the figure below. You can close
  or minimize that window.
+ Double-click on the EC-lab or Hiden icon.
+ Do some electrochemistry or mass spectrometry.
+ Save your electrochemistry or mass spectrometry data to the assets
  folder as explained below.

.. _fig-winvm:
.. figure:: _images/WinVM/Winvm_startup.png
   :target: _images/Winvm_startup.png
   :width: 70%
   :align: center

   VM management window. You can minimize or close this.

Storing electrochemistry or mass spectrometry data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Windows VM has permission to connect to central storage with
permissions to write files from EC-lab or the Hiden software to the
correct location.

To start a new experiment, you first have to disconnect the old drive
(if connected).  This is likely mounted as the ``Z:`` drive.

.. _fig-winstaledrive:
.. figure:: _images/WinVM/Windows_stale_folder.png
   :target: _images/Windows_stale_folder.png
   :width: 70%
   :align: center

   An example of a stale folder from a previous experiment mounted as
   the ``Z:`` drive.

Disconnect the stale ``Z:`` drive by right clicking on its entry in
the side bar and selecting "Disconnect".

.. _fig-windisconnect:
.. figure:: _images/WinVM/Windows_disconnect.png
   :target: _images/Windows_disconnect.png
   :width: 70%
   :align: center

   Disconnect the stale ``Z:`` drive.

Next, click on "This PC" in the sidebar, then click on the button that
says "Map network drive".

.. _fig-winmapnewdrive:
.. figure:: _images/WinVM/Windows_map_new_drive.png
   :target: _images/Windows_map_new_drive.png
   :width: 70%
   :align: center

   Map a new network drive to the VM.

On the "Map Network Drive" page, you need to fill in the path to the
current experiment's proposal folder.  Suppose the current cycle is
2024-3 and the current proposal number is 316832.  Using ``Z:`` as the
drive letter, enter the following as the "Folder" 

.. code-block:: none

   \\storage.bmm.nsls2.bnl.gov\

From the mounted Z drive, you can navigate to
``\proposals\<cycle>\pass-<GUP>``, replacing ``<cycle>`` and ``<GUP>``
with the correct cycle number and proposal number for the experiment.

Note that the backslashes are important.  Also substitute the correct
cycle and proposal numbers.

Be sure to leave "Reconnect at sign-in" checked.  Note that "Connect
using different credentials" should be unchecked.


.. _fig-winspecifynewdrive:
.. figure:: _images/WinVM/Windows_specify_new_drive.png
   :target: _images/Windows_specify_new_drive.png
   :width: 70%
   :align: center

   Map a new network drive to the VM.

Click the finish button.  The connection will take several seconds,
but then the new entry will show up in the side bar.

The new network drive can now be clicked into.

Configure EC-lab to write its data files into the
``assets\vsp300-1`` folder.  The full path, then, is
``Z:\proposals\<cycle>\pass-<GUP>\assets\vsp300-1``.

Configure the Hiden software to write its data files into the
``assets\hpr20-1`` folder.

.. _fig-winsassetsfolder:
.. figure:: _images/WinVM/Windows_assets_folder.png
   :target: _images/Windows_assets_folder.png
   :width: 70%
   :align: center

   The ``assets\vsp300-1`` folder is the correct place for data from
   EC-lab to be written.  The ``assets\hpr20-1`` folder is the correct
   place for data from the Hiden to be written.

By following this procedure, the electrochemistry data from EC-lab and
mass spectrometry data from the Hiden will be available to the user in
the :numref:`same manner as their XAS data (Section %s) <data>`.




Calibrate the mono
------------------

The typical calibration procedure involves measuring the angular
position of the Bragg axis for the edge energies of 10 metals: Fe, Co,
Ni, Cu, Zn, Pt, Au, Pb, Nb, and Mo.  

The tabulated values of edge energies from Table 1 in `Kraft, et
al. <https://doi.org/10.1063/1.1146657>`__ are used in the
calibration.


#. Be sure that all 10 of these elements are actually mounted on the
   reference wheel and configured in the ``xafs_ref.mapping`` dict.
   (They should be.  It would be very unusual for any of these foils
   to have been removed from the reference wheel.)

#. Run the command 

   .. code-block:: python

      RE(calibrate(mono='111'))

   Use the ``mono='311'`` argument for the Si(311) monochromator.
   This will, in sequence, move to each edge and measure a XANES scan
   over a wide enough range that it should cover the edge (unless the
   mono is currently calibrated VERY wrongly).  This will write a file
   called :file:`edges111.ini` (or :file:`edges3111.ini`).  Each XANES
   scan uses the file
   :file:`/home/xf06bm/Data/Staff/mono_calibration/cal.ini` as the INI
   file.  Edge appropriate command line parameters will be added by
   the ``calibrate()`` plan.

#. Examine the data in |athena|. Make sure E\ :sub:`0` is selected
   correctly for all 10 edges. Copy those values into the first column
   of :file:`edges111.ini` (or :file:`edges311.ini`). 

   .. attention::

      It is no longer necessary to compute the angular positions of
      the monochromator.  Those will be computed from the edge energy
      values you edited into the INI file by hand.

   .. todo::

      Implement on-the-fly determination of E\ :sub:`0` to obviate the
      step of editing the INI file.  Pb is tricky.  Nb and Mo are kind
      of tricky.


   .. 
     Compute the
     angular positions using
     .. code-block:: python
	dcm.e2a(<energy values>)
     and copy those numbers into the :file:`edgeH11.ini` file.

#. Run the command

   .. code-block:: python

      calibrate_mono(mono='111')

   (or use the ``'311'`` argument).  This will show the fitting
   results and plot the best fit.  It will also print in a text box
   instructions for modifying the :file:`BMM/dcm-parameters.py` file
   to use the new calibration values.

   .. _fig-calibrate:
   .. figure:: _images/software/Calibration_111.png
      :target: _images/Calibration_111.png
      :width: 70%
      :align: center

      Example calibration curve
   

#. Edit :file:`BMM/dcm-parameters.py` as indicated.

#. Do 

   .. code-block:: python

      calibrate_pitch(mono='111')

   This performs a simple linear fit to the rocking curve peak
   positions for ``dcm_pitch`` found at each edge.  Use the fitted
   slope and offset to modify ``approximate_pitch`` in
   :file:`BMM/functions.py`.

#. Finally do

   .. code-block:: python

      %run -i 'home/xf06bm/.ipython/profile_collection/startup/BMM/dcm-parameters.py'

   then do

   .. code-block:: python

      dcm.set_crystal()

   Or simply restart |bsui|, which is usually the simpler solution.


The mono should now be correctly calibrated using the new calibration
parameters.




Manage Silicon Drift Detectors
------------------------------

The assumption in the data acquisition system is that one of the three
silicon drift detectors will be the primary detector in an experiment.
At the |bsui| command line (or in |qs|) the ``xs`` symbol should
point at the correct detector.  Also, a parameter is set in Redis
allowing other processes (such as the Kafka plotting agent) to know
which detector to be paying attention to.


.. code-block:: python

   xs = xspress3_set_detector(7)

where the argument to ``xspress3_set_detector`` is 1, 4, or 7.  Since
October 2024, use of the seven element detector is the default.

This sets ``xs`` to the selected detector object |nd| ``xs1``,
``xs4``, or ``xs7``.  

Also set is the Redis parameter ``BMM:xspress3``, which is set to 1,
4, or 7 (and represented as a b-string).

.. code-block:: python

   n_elements = int(rkvs.get('BMM:xspress3'))

Using Guacamole
---------------

To connect to a beamline computer via Guacamole, point a web browser
at https://remote.nsls2.bnl.gov. Authenticate with username, password,
and DUO.  From the menu of options, select "BMM (06-BM)".  The main
workstation for XAS experiments is "BMM Workstation 3".  Then select
the display you want to look at.

.. _fig-quac_options:
.. figure:: _images/software/guac_options.png
   :target: _images/guac_options.png
   :width: 50%
   :align: center

   The Guacamole options menu


The displays are numbered by this diagram showing the layout of the
monitors at the beamline.  The |bsui| terminal window is usually on
Display 1.  The web browser and spreadsheet are usually on Display 3.

.. code-block:: text

               +---------------+
	       |               |
	       |   Display 2   |
	       |               |
               +---------------+
    +---------------+      +---------------+
    |               |      |               |
    |   Display 1   |      |   Display 3   |
    |               |      |               |
    +---------------+      +---------------+


If you are BMM user and do not see the menu of options in
:numref:`Figure %s <fig-quac_options>`, ask BMM staff to grant access.


The main workstation for scattering experiments is workstation 2,
which has four displays in a 2x2 grid.


The Guacamole server
~~~~~~~~~~~~~~~~~~~~

If Guacamole is unresponsive when connecting to the URL above, try
restarting the server.  On xf06bm-ws3, do

.. code-block:: bash

   dzdo systemctl restart x11vnc@0 x11vnc@1 x11vnc@2 x11vnc-all xrdp

On xf06bm-ws2, do

.. code-block:: bash

   dzdo systemctl restart x11vnc@0 x11vnc@1 x11vnc@2 x11vnc@3 x11vnc-all xrdp


This requires authentication with username, password, and DUO.  This
restart can only be done by beamline or DSSI staff.

This is also sometimes necessary after a reboot when Guacamole has
connected with the workstation, but is showing a strange view of the
monitors in the three displays.


Granting and restricting access to the beamline machines via guacamole
is done using the `Science Network User Tools <https://docs.nsls2.bnl.gov/docs/explanations/tools/remote/N2SNUserTools.html#listing-users-who-can-login-to-instruments>`__.

+ List users: ``n2sn_list_users``
+ Search users: ``n2sn_search_users --surname <name>``
+ Add users: ``n2sn_add_user -l <username> USER,GUACCTRL``

``n2sn_add_user`` can only be run by BMM or DSSI staff.

See the link above for complete details.



Using Queueserver
-----------------

Server side
~~~~~~~~~~~

.. note:: Working on the server requires BMM or DSSI staff.

The |qs| runs on the virtual machine at
``xf06bm-bmm-qs1.nsls2.bnl.gov``.  When ssh-ing to that machine from
the beamline subnet, you must use the fully qualified path.  You
should ssh as yourself, not as the beamline operator, i.e.

.. code-block:: bash

   ssh bravel@xf06bm-bmm-qs1.nsls2.bnl.gov

substituting your username for Bruce's, of course.

Once there, you can check the status of a server instance with

.. code-block:: bash

   dzdo systemctl status bluesky-queueserve


You can start a stopped server instance with

.. code-block:: bash

   dzdo systemctl start bluesky-queueserve

If you need to change something about the |qs| configuration, for
example, the list of available plan names in
``existing_plans_and_devices.yaml``: 

.. code-block:: bash

   dzdo systemctl restart bluesky-queueserve

Once the |qs| is running, you can attached a |qm| or interact wiuth
the queue in any other way.


Client side
~~~~~~~~~~~

.. note:: The |qm| should be run as the beamline user, ``xf06bm``.


The machine operator account, ``xf06bm``, on all the workstations has
a folder at ``~/queueserver``.  In that location do

.. code-block:: bash

   pixi run qm

This will fire up the |qm| on that workstation.

.. _fig-qm_startup:
.. figure:: _images/qs/qm_startup.png
   :target: _images/qm_startup.png
   :width: 70%
   :align: center

   The |qm| at startup.

Clicking the :key:`Connect` button in the upper left will connect
the monitor to the configured instance of the |qs|.

.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-qm_connected
   :class-grid: outline

   .. image:: _images/qs/qm_connected.png

   .. image:: _images/qs/qm_connected2.png

   (Left) Connected to the |qs|, monitor tab.  (Right) connected to
   the |qs|, edit and control tab.


Next, click the :key:`Open` buton on the top left of the edit and
control tab.  This will begin loading the bluesky profile.


.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-qm_env
   :class-grid: outline

   .. image:: _images/qs/qm_env_start.png

   .. image:: _images/qs/qm_env_open.png

   (Left) Opening the bluesky environment tab.  Note that the same
   screen messages as we see in |bsui| are being written to the
   console on the monitor tab.  (Right) Ready to begin work.  Note the
   messages at the top of the edit and control tab indicate that the
   environment is open, but idle.


To open communications with the |qs|, you need to established an
authenticated connection via https.  Assuming you know an
authentication key, this bit of code will open the connection and
demonstrate connectivity via the ``ping`` method.

.. code-block:: python

   from bluesky_queueserver_api import BPlan
   from bluesky_queueserver_api.http import REManagerAPI

   qs = REManagerAPI(http_server_uri="https://xf06bm-bmm-qs1.nsls2.bnl.gov:443")
   with open("/path/to/authentication_key", "r") as f:
	api_key = f.read().strip()
   qs.set_authorization_key(api_key=api_key)
   qs.ping()

With an established connection, plans can be pushed to the queue.  In
this example, the plan name is ``CMS_driven_measurement`` followed by
three arguments of the plan.

.. code-block:: python

   plan = BPlan('CMS_driven_measurement', composition, distance, time)
   qs.item_add(plan, pos=priority)

Here is an example of moving a motor by 10 mm at BMM:

.. code-block:: python

   plan = BPlan('mover', 'xafs_x', 10)
   qs.item_add(plan, pos=priority)


The ``pos`` argument to the ``item_add`` method takes either
``"front"`` or ``"back"`` and refers to the position in the queue to
which the plan will be pushed.


Using Zoom
----------

Zoom is available at the beamline and is often used by Bruce to do
beamline support from home.  Usually, Zoom is started remotely and all
the visitor needs to do is start talking.

Sometimes the audio settings need to be adjusted or the microphone
needs to be manually reset.  Because Bruce is not present when doing
remote support, he cannot know if the speaker is not working.  Neither
can he do a manual adjustment of the microphone.

In this section, *explicit and laborious* instructions are given for
how to reset audio at workstation 3, the main computer used for XAS
experiments.

To start, here are photos of the hardware.  There is a speaker bar.
It is located just below the monitors and shown on the left of
:numref:`Figure %s <fig-zoom_hardware>`.  It is the object circled in
red.  On the right hand side of the speaker bar is the volume knob.
It is indicated by the green arrow.  Turning that knob adjusts the
sound volume.


.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-zoom_hardware
   :class-grid: outline

   .. image:: _images/zoom/zoom_speaker.jpg

   .. image:: _images/zoom/zoom_mike.jpg

   (Left) The speaker at workstation 3.  Note the volume knob,
   indicated by the green arrow, on the right.  You may need to turn
   the volume on or up.  (Right) The microphone at workstation 3.  The
   USB cable on the rear of the microphone is marked in red.


The microphone is shown on the right of :numref:`Figure %s
<fig-zoom_hardware>`.  The USB cable that connects the speaker to the
computer is located on the rear end of the speaker casing and is
indicated by the red circle.

When everything is working correctly, all you need to do is look into
the camera (perched on top of one of the monitors) and speak into the
microphone.  Please remember that if you turn your head away from the
microphone, it may not pick up your voice.


You can pull up the audio settings dialog in Zoom by first clicking on
the upwards-pointing arrow next to the "Mute" button on the main Zoom
screen, as shown on the left of :numref:`Figure %s
<fig-zoom_audiodialog>`.  The audio setting dialog, shown on the right
of :numref:`Figure %s <fig-zoom_audiodialog>`, will appear.

.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-zoom_audiodialog
   :class-grid: outline

   .. image:: _images/zoom/zoom_audio.png

   .. image:: _images/zoom/zoom_audio_dialog.png

   (Left) Accessing the audio setting dialog in Zoom.  (Right) The
   audio settings dialog.  The microphone settings are indicated in cyan.


When everything is working correctly, the speaker selection will say
"Family 17h", as shown on the right of :numref:`Figure %s
<fig-zoom_audiodialog>`, and the microphone selection will be "Blue
Microphones Analog Stereo".  


Troubleshooting the speaker
   If you click the :key:`Test speaker` button, you should hear a tune
   coming from the speaker bar under the monitors (as shown on the
   left of :numref:`Figure %s <fig-zoom_hardware>`).  If you hear no
   sound or if the sound is feint, turn the volume knob on the right
   hand side of the speaker bar.  If you still do not hear the
   speaker, check the menu of speaker options to make sure that the
   "Family 17h" option is selected.

Troubleshooting the microphone
   If the microphone appears not to be working or if the selection is
   anything other than "Blue Microphones Analog Stereo", select "Blue
   Microphones Analog Stereo" from the menu.

   If the microphone still is not working, unplug the USB cable from
   the back of the microphone, wait a couple seconds, then plug it
   back in.  The selection in the audio dialog should change to "Blue
   Microphones Analog Stereo" on its own, but you may need to manually
   select it from the menu.


.. caution::

   **DO NOT** try to unplug the end of that USB cable that plugs into
   the computer.  You **DO NOT** have permission to go rooting through
   the cables on the back of the computer.

These instructions fix the most common audio problems.  If things
still aren't working, then Bruce will have to take a deeper dive the
next time he is physically present at the beamline.  :numref:`Figure
%s <fig-zoom_hardware>`.
