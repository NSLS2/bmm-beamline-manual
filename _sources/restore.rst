..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. _restore:

Power outage recovery
=====================

.. note::

   This section was started after the scheduled power shutdown of
   April 20, 2024.  I am trying to capture the things I did to bring
   the beamline back to life that seemed non-obvious or were a source
   of friction.  YMMV and this section should be expanded in the future.

Channel Access
--------------

By default, channel access security is set such that access to
beamline PVs is disabled.  This means that motors cannot be moved,
detectors cannot be triggered, and so on.

To enable channel access, do the following:

.. code-block:: bash

		caget XF:06BM-CT{}Prmt:RemoteExp-Sel 1

Thus **must** be done as yourself, not as the beamline operator
account.

If that PV is being reported as disconnected |nd| which is indicated
by the "Workstation Access" buttons on the BMM Main CSS screen or by
the command above returning ``Channel connect timed out`` |nd| then
you need to restart the CAS Switch IOC.

To do that, ssh to xf06bm-ioc2 as youself (not as the operator
account) and do 

.. code-block:: bash

		dzdo manage-iocs restart cas-switch

Once that IOC restarts, try again to set
``XF:06BM-CT{}Prmt:RemoteExp-Sel``. 

Redis
-----

.. note::

   Now using redis service at ``xf06bm-bmm-redis1.nsls2.bnl.gov``

   Will need to solicit help from DSSI if that VM does not come back
   to life after a power failure.



Xspress3
--------

To re-power the Xspress3 and its associated server:

#. Verify that the power button on the back of the Xspress3 unit is
   switched on.  It should be glowing red.
#. Press the front power button.
#. Once running, restart the relevant IOC on ``xf06bm-ioc2``.  For the
   seven element detector, use ``xs3-7-1``.  For the four element
   detector, use ``xs3-4-1``.


Other IOCs
----------

The startup acceptance tests in the |bsui| profile may eventually fail
when trying to connect to instruments.  For example, this:

.. code-block:: text

   TimeoutError: XF:06BM-ES:{LINKAM}:MODEL could not connect within 10.0-second timeout.

indicates that the Linkam controller is powered off and/or the
``linkam3_socat`` IOC is not running.  After verifying power to the
instrument, ssh to ``xf06bm-inst-ioc1`` and do:

.. code-block:: bash

   dzdo manage-iocs restart linkam3_socat

To get a list of all IOCs and their status, do:

.. code-block:: bash

   manage-iocs status

Find the name of the relevant IOC and restart it using the
``manage-iocs rastart`` command.

It sometimes helps to know what port number each IOC is communicating
on:

.. code-block:: bash

   dzdo manage-iocs report

.. _iocs:

IOCs at BMM
-----------

IOCs are mostly run from the virtual machines: ``xf06bm-inst-ioc1``,
``xf06bm-det-ioc1``, ``xf06bm-cam-ioc1``.

Previously ``xf06bm-ioc2`` was the main IOC server at BMM.  It is a
much beefier machine than ``xf06bm-ioc1``.

Here is a list of all the IOCs at BMM, what they do, and where they
are running.  The color coding has no meaning other that to make it
easier to see which IOCs run on the same server.

================  ================================================= ==========================
IOC name           purpose                                           location
================  ================================================= ==========================
 axis-caproto5     XRD Axis web camera                               :blue:`xf06bm-cam-ioc1`
 axis-caproto6     XAS Axis web camera                               :blue:`xf06bm-cam-ioc1`
 cam01             Prosilica camera #1 (DM1)                         :blue:`xf06bm-cam-ioc1`
 cam02             Prosilica camera #1 (DM2)                         :blue:`xf06bm-cam-ioc1`
 cam03             Prosilica camera #3 (DM3)                         :blue:`xf06bm-cam-ioc1`
 cam04             Prosilica                                         :blue:`xf06bm-cam-ioc1`
 cam08             Mako GigE                                         :blue:`xf06bm-cam-ioc1`
 cam09             Mako GigE                                         :blue:`xf06bm-cam-ioc1`
 cas-switch        channel access security management                xf06bm-inst-ioc1
 dante             Dante controller for Ge detector                  :green:`xf06bm-det-ioc1`
 diode             DIODE controller (filters, spinner stage)         xf06bm-inst-ioc1
 eiger-det1        Eiger 4M                                          :green:`xf06bm-det-ioc1`
 F460              FMBO current monitor (not in use)                 xf06bm-inst-ioc1
 flag1             Front end flag (not in use)                       xf06bm-inst-ioc1
 I400              FMBO electrometer (not in use)                    xf06bm-inst-ioc1
 lakeshore331      LakeShore temperature controller (Displex)        xf06bm-inst-ioc1
 linkam3_scoat     Linkam controller                                 xf06bm-inst-ioc1
 logitechF710      Game controllers                                  :red:`xf06bm-ioc1`
 MC01              Collimating mirror                                xf06bm-inst-ioc1
 MC02              Monochromator                                     xf06bm-inst-ioc1
 MC03              Slits2                                            xf06bm-inst-ioc1
 MC04              Focusing mirror                                   xf06bm-inst-ioc1
 MC05              Harmonic rejection mirror and DM1 filters         xf06bm-inst-ioc1
 MC06              DM3 diagnostics and slits3                        xf06bm-inst-ioc1
 MC07              xafs_* motors                                     xf06bm-inst-ioc1
 MC08              xafs_* motors                                     xf06bm-inst-ioc1
 MC09              xafs_* motors                                     :red:`xf06bm-ioc2`
 MC11              goniometer motors                                 xf06bm-inst-ioc1
 MC12              goniometer motors                                 xf06bm-inst-ioc1
 MC13              goniometer motors                                 xf06bm-inst-ioc1
 mythen-det2       Mythen                                            :green:`xf06bm-det-ioc1`
 omega_i_series    ??
 onewire           1Wire temperature sensors near mono               xf06bm-inst-ioc1
 piE625-M2         M2 piezo controller                               xf06bm-inst-ioc1
 piE625-M3         M3 piezo controller                               xf06bm-inst-ioc1
 piE625-mono       mono piezo controller                             xf06bm-inst-ioc1
 Pilatus100K       Pilatus using NFS mounts                          :red:`xf06bm-ioc1`
 plc1              PLC IOC                                           xf06bm-inst-ioc1
 pscdrv            ??
 quadEM-1          QuadEM box 1                                      xf06bm-inst-ioc1
 quadEM-2          QuadEm box 2                                      xf06bm-inst-ioc1
 recsyncIOC        ??                                                xf06bm-inst-ioc1
 simDetector       ??                                                :green:`xf06bm-det-ioc1`
 va-1              Vacuum controllers and gauges                     xf06bm-inst-ioc1
 xf06bmAlarmIOC    Alarm server                                      xf06bm-inst-ioc1
 xs3-8ch           :silver:`deprecated XSpress3 server, do not run`  :green:`xf06bm-det-ioc1`
 xs3-7-1           XSpress3 server for use with 7-element detector   :red:`xf06bm-ioc2`
 xs3-4-1           XSpress3 server for use with 4-element detector   :green:`xf06bm-det-ioc1`
================  ================================================= ==========================

IOC Notes
~~~~~~~~~

:Pilatus100K:
   The Pilatus100K IOC does a lot of file I/O, so it seemed like a good idea to
   isolate it from the other IOCs.  And it needs to be on a server that
   can NFS mount the RAM disk at  ``xf06bm-pilatus100k``.

   All other IOCs on ``xf06bm-ioc1`` must be in the ``stopped`` state.

:MC09:
   This was moved back to ``xf06bm-ioc2`` when save/restore issues
   were causing problems on ``xf06bm-inst-ioc1``.  This needs to be revisited.

:xs3-7-1:
   This should be running on ``xf06bm-det-ioc1``.  Needs investigations.

:logitechF710:
   This needs to run on a local server with a USB port.

Motor controllers
-----------------

FMBO MCS8
~~~~~~~~~

Save/restore will not correctly remember motor positions on any opf
the FMBO-supplied axes (i.e. everything except the XAFS and XRD end
stations).  

Restore power to the motor controllers.  It should not necessary to
restart the IOCs (MC02 through MC06), but do so if motors are not
moving after powering up the controllers.

The steps below are the commands in |bsui| for homing sets of axes.  The
``ks.cycle()`` steps are not, strictly speaking, necessary.  But it is
a good idea to be sure the amplifiers are in a good state.  If any
amplifier faults trigger upon starting the homing process, the motors
will be left in a confused state.


.. code-block:: python

   ks.cycle('slits2')
   RE(recover_slits2())

   ks.cycle('dm3')
   RE(recover_slits3())
   RE(recover_diagnostics())

   ks.cycle('m2')
   RE(recover_m2())

   ks.cycle('m3')
   RE(recover_m3())

   ks.cycle('dcm')
   RE(dcm.recover())


After homing, the monochromator should be at 7134.3 eV, which is an
energy within photon delivery mode E.  The mirrors and ``dm3_bct``
should be at positions consistent with mode E.

Some of these take quite a while to go through their homing procedure.
The diagnostics recovery takes almost an hour because a couple of the
motors are **very** slow and have a long way to go to hit their limit
switches.

The M2 bender does not have a homing routine.  To verify its
position, move it by hand to its negative limit:

.. code-block:: python

   RE(mvrbender(-10000))

That command is a wrapper around killing the amplifier, then moving by
the specified amount.  Feel free to take larger steps.

Once it hits the negative limit, reset its offset 

.. code-block:: python

   reset_offset(m2_bender, 0)

then move it back to position and kill the amplifier:

.. code-block:: python

   RE(mvbender(BMMuser.bender_xas))
   m2_bender.kill()

For reference, the XAS position for the bender is around 212,000.  The
XRD position is around 107,000.


.. note:: 

   **Never** home M1, the collimating mirror.  It is close enough to the
   right position and should never be moved.  In fact, there is no
   reason to power up the motor controller.

   The fear is that an axis might fail far from the correct position.

   The M1 motor controller is in rack MC7-RG-E4 on the mezzanine.  It
   is near the bottom and is the only one with FMBO branding.

Homing MSC8s via PEWIN
~~~~~~~~~~~~~~~~~~~~~~

If homing from the |bsui| command line fails, your best bet is to find
the laptop with PEWIN and connect to the motor controller with a USB
cable.

First, go to ``xf06bm-inst-ioc1`` and stop the reelvant IOC.

Fire up PEWIN.  In the PEWIN command console, issue this command:
``M1x16=1``, where ``x`` is a number from 1 to 8 and indicates the
axis that you want to home.

You can home multiple axes simultaneously by issuing ``M1x16=1``
instruction while other axes are in the process of homing.  PEWIN is
happy to multitask. 

Note that any axes that involve coordinated motion |nd| mirror
vertical, mirror horizontal, slit vertical or horizontal |nd| work
such that all coordinated axes are triggered for homing when any
individual axis is triggered.  For example, to home the M3 vertical
axes, you do not need to do ``M1116=1``, ``M1216=1``, and
``M1316=1``.  When you issue any one of those three instructions, all
three axes will begin moving.



geobrick
~~~~~~~~

Few or none of the motors on the older NSLS-II geobricks are
equipped with home or limit switches.  This includes the motor
controllers in racks RGC-1 and RGC-2.  MC09 is a newer geobrick and
controls encoded axes.

Save/restore should remember positions.
