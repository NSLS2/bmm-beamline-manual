..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. role:: strike
   :class: strike

.. |simdet|     replace:: :strike:`simDetector`

.. _vms:

IOCs on Virtual Machines
========================

As of February 2026, most IOCs at BMM are running on centrally
maintained virtual machines.  The prior IOC server, ``xf06bm-ioc2``,
is still there and can be fallen back upon should that be needed.

This section documents which IOC is running on which VM server.

Detectors: xf06bm-det-ioc1
--------------------------

=============== =======================================================
 IOC Name        Purpose
=============== =======================================================
 eiger-det1      Eiger4 area detector
 mythen-det2     Mythen strip detector
 |simdet|        :strike:`simulated detector` (deprecated)
 xs3-4-1         XSpress3X with calibration for the 4-element detector
 xs3-7-1         XSpress3X with calibration for the 7-element detector
 xs3-8ch         generic XSpress3 
=============== =======================================================

.. note:: The Pilatus100k IOC needs to move from ``xf06bm-ioc1`` to
          ``xf06bm-det-ioc1``


Instruments: xf06bm-inst-ioc1
-----------------------------

=============== =======================================================
 IOC Name        Purpose
=============== =======================================================
cas-switch       channel access security
diode            DIODE boxes
F460             FMBO electrometer on DM2
flag1            front end flag (not in use)
I400             FMBO electrometer on DM3
lakeshore331     Temperature controller used with Displex
linkam3          Linkam stage
MC02             Motor controller 2 (DCM)
MC03             Motor controller 2 (slits2, DM2)
MC04             Motor controller 2 (M2)
MC05             Motor controller 2 (M3)
MC06             Motor controller 2 (slits3, DM3)
MC07             Motor controller 2 (XAS stages)
MC08             Motor controller 2 (XAS stages)
MC09             Motor controller 2 (XAS stages)
MC11             Motor controller 2 (goniometer motors)
MC12             Motor controller 2 (goniometer motors)
MC13             Motor controller 2 (goniometer motors)
omega_i_series   ???
onewire          OneWire temperature sensors in FOE
piE625-M2        M2 pitch piezo controller
piE625-M3        M3 pitch piezo controller
piE625-mono      DCM second crystal pitch piezo controller
plc1             EPS controls
quadEM-1         black box QuadEM (normally in use)
quadEM-2         black box QuadEM (spare)
recsyncIOC       ???
va-1             vacuum controls
xf06bmAlarmIOC   alarm server
=============== =======================================================

.. admonition:: Future Tech!
		
   MC10 will be the next motor controller for the XAS end station,
   should it be needed


Cameras: xf06bm-cam-ioc1
------------------------


=============== =======================================================
 IOC Name        Purpose
=============== =======================================================
axis-caproto-5   XRD webcam controller
axis-caproto-6   XAS webcam controller
cam01            DM1 beam diagnostic
cam02            DM2 beam diagnostic
cam03            DM3 beam diagnostic
cam04            unused prosilica
cam07            
cam08            Mako gigE camera (XAS end station)
cam09            Mako gigE camera (XAS end station)
=============== =======================================================


.. admonition:: Future Tech!

   cam10 - cam11 to replace USB cameras at xrd end stations

   cam04 may be used with the direct beam sensor for goniometer
   alignment once that item is procured

.. note:: The ``logitechF710`` IOC continues to run on ``xf06bm-ioc2``
	  for now.  It might be moved to a meerkat.  It requires a USB
	  connection, which precludes running on a VM.
