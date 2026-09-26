..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. |nbsp| unicode:: 0xA0 
   :trim:

.. _motors:

Moving and querying motors
==========================

To get an overview of the state of the beamline motors, do ``%m`` at
the |bsui| command line.  Here is an example:

.. code-block:: text

   In [1897]: %m
   ==============================================================================
   Energy = 19300.1   reflection = Si(111)   mode = fixed
        Bragg =  5.87946   2nd Xtal Perp = 15.0792   2nd Xtal Para = 146.4328

   M2
        vertical =   6.000 mm           YU  =   6.000
        lateral  =   0.000 mm           YDO =   6.000
        pitch    =   0.000 mrad         YDI =   6.000
        roll     =  -0.001 mrad         XU  =  -0.129
        yaw      =   0.200 mrad         XD  =   0.129

   M3
        vertical =   0.000 mm           YU  =  -1.167
        lateral  =  15.001 mm           YDO =   1.167
        pitch    =   3.500 mrad         YDI =   1.167
        roll     =   0.000 mrad         XU  =  15.001
        yaw      =   0.001 mrad         XD  =  15.001

   Slits3:   vsize  vcenter  hsize   hcenter     top    bottom    outboard  inboard
             1.350   0.000   8.000  -0.000      0.675   -0.675    4.000   -4.000

   DM3_BCT:  45.004 mm

   XAFS table:
        vertical  pitch    roll   YU     YDO     YDI
        132.000   0.000   0.000 132.000 132.000 132.000

   XAFS stages:
             x        y     roll    pitch    linxs    roth    wheel     rots
           9.224  115.000   0.840   0.000  -45.000    0.000  -59.000    0.000
   ==============================================================================


See :numref:`Section %s <iocs>` for details about the IOCs running the
beamline instrumentation.



.. _sample_stages:

Sample stages
-------------

.. |widdershins| replace:: `widdershins <https://en.wiktionary.org/wiki/widdershins>`__


These stages sit on top of the XAFS optical table.  

.. table:: XAFS experimental stages
   :name:  xafs-stages
   :align: left

   =============== ===========  =========  =======================  =====================================
   motor           type         units      notes                    directions
   =============== ===========  =========  =======================  =====================================
   ``xafs_x``      linear       mm         main sample stage        |plus| outboard, - inboard
   ``xafs_y``      linear       mm         main sample stage        |plus| up, - down
   ``xafs_wheel``  rotary       degrees    *ex situ* sample wheel   |plus| clockwise, - |widdershins|
   ``xafs_detx``   linear       mm         detector mount           |plus| away from sample, - closer
   ``xafs_dety``   linear       mm         detector mount           |plus| up, - down
   ``xafs_detz``   linear       mm         detector mount           |plus| downstream, - upstream
   ``xafs_ref``    rotary       degrees    reference stage          |plus| clockwise, - |widdershins|
   ``xafs_refx``   linear       mm         reference X stage        |plus| outboard, - inboard
   ``xafs_refy``   linear       mm         reference Y stage        |plus| up, - down
   ``xafs_pitch``  tilt         degrees    Huber tilt stage         |plus| more positive
   ``xafs_roll``   tilt         degrees    Huber tilt stage         |plus| more positive
   ``xafs_rots``   rotary       degrees    small rotary stage       |plus| clockwise, - |widdershins|
   ``xafs_garot``  rotary       degrees    g.a. rotary stage        |plus| clockwise, - |widdershins|
   ``xafs_spare``  linear       mm         spare linear stage       
   ``xafs_bsx``    linear       mm         beam stop stage          |plus| outboard, - inboard
   ``xafs_bsy``    linear       mm         beam stop stage          |plus| up, - down
   ``xafs_adx``    linear       mm         area detector stage      |plus| outboard, - inboard
   ``xafs_ady``    linear       mm         area detector stage      |plus| up, - down
   ``xafs_yu``     linear       mm         u.s. table jack          |plus| up, - down
   ``xafs_yd``     linear       mm         d.s. table jack          |plus| up, - down
   =============== ===========  =========  =======================  =====================================

g.a. = glancing angle |blackcircle|
o.b. = outboard  |blackcircle|
i.b. = inboard  |blackcircle|
u.s. = upstream  |blackcircle|
d.s. = downstream

Note that the ``xafs_pitch`` stage is not right handed in the beamline
coordinate system.  This is the more intuitive setting for how that
stage is typically used.

``xafs_yd`` is the motor driving the coupled downstream jacks.

----


Here are some photos identifying these axes:


.. _fig-xy:
.. figure:: _images/stages/samplexy_stages.jpg
   :target: _images/samplexy_stages.jpg
   :width: 50%
   :align: center

   The two-axis main sample stage: ``xafs_x`` and ``xafs_y``.

.. _fig-det_xyz:
.. figure::  _images/stages/det_xyz.jpg
   :target: _images/det_xyz.jpg
   :width: 70%
   :align: center

   The three-axis stage holding the fluorescence detector:
   ``xafs_detx``, ``xafs_dety``, and ``xafs_detz``.

.. _fig-detref:
.. figure:: _images/stages/reference_stages.jpg
   :target: _images/reference_stages.jpg
   :width: 50%
   :align: center

   The three-axis stage holding the reference wheel: ``xafs_ref``,
   ``xafs_refx``, and ``xafs_refy``.


.. subfigure::  AB
   :layout-sm: AB
   :subcaptions: above
   :gap: 8px
   :name: fig-pr_rots
   :class-grid: outline

   .. image:: _images/stages/tilt_stage.jpg

   .. image:: _images/stages/small_rotation_stage.jpg

   (Left) The pitch and roll stage: ``xafs_pitch`` and ``xafs_roll``.
   (Right) The small rotation stage: ``xafs_rots``

.. _fig-garot:
.. figure:: _images/stages/xafs_garot.jpg
   :target: _images/xafs_garot.jpg
   :width: 70%
   :align: center

   The glancing angle rotary stage, here sitting on top of the pitch
   and roll stage: ``xafs_garot``.

.. _fig-spare:
.. figure:: _images/stages/xafs_spare.jpg
   :target: _images/xafs_spare.jpg
   :width: 70%
   :align: center

   The spare linear stage: ``xafs_spare``.

.. _fig-beamstop:
.. figure:: _images/stages/beamstopxy_stages.jpg
   :target: _images/beamstopxy_stages.jpg
   :width: 70%
   :align: center

   The beam stop stages: ``xafs_bsx`` and ``xafs_bsy``.



.. subfigure::  AB
   :layout-sm: AB
   :subcaptions: above
   :gap: 8px
   :name: fig-areadetector
   :class-grid: outline

   .. image:: _images/stages/areadetectorxy_stages.jpg

   .. image:: _images/stages/detector_mounts.jpg

   (Left) The area detector stages: ``xafs_adx`` and ``xafs_ady``.
   (Right) Mounting rigs for the Eiger and Pilatus detectors.


.. _fig-coupled_xafs_yd:
.. figure:: _images/stages/coupled_xafs_yd.gif
   :target: _images/coupled_xafs_yd.gif
   :width: 70%
   :align: center

   The inboard and outboard jacks on the downstream end of the
   XAFS table, coupled together to be run by a single motor.

   



Basic motor commands
--------------------


Configuration and position of the motors can be queried easily.  In
the following examples, the ``xafs_y`` motor is used.  The commands
are the same for all sample stage motors.

**Querying position**
   The position of a motor can be queried with a command line like ::

     %w xafs_y 

   or ::

     xafs_y.position

**Moving to a new position**
   Always move motors through the run engine, for example: ::

     RE(mvr(xafs_y, 10))

   ``mvr`` is the relative move command |nd| the numerical argument is
   the amount by which the motor will move from the current position.

   ``mv``, as in::

     RE(mv(xafs_y, 37.63))

   is the absolute move command.  The numerical argument is the
   position to which the motor will move.

..   All movements are logged in the :numref:`experimental log (Section %s) <logfile>`

**Moving to a new position in a plan**
   To move a sample stage as part of a :numref:`macro (Section %s)
   <macro>` , do::

     yield from mv(xafs_y, 37.36)

   You can combine motions of two or more motors in a single
   synchronous movement::

     yield from mv(xafs_y, 37.36, xafs_x, 15.79)

   Similarly::

     yield from mvr(xafs_y, 5)

**Querying soft limits**
   To know the soft limits on a sample stage, do ``xafs_y.limits`` or
   ``xafs_y.llm.get()`` or ``xafs_y.hlm.get()`` to query the low or
   high limits individually.

**Setting soft limits**
   To set the soft limits on a sample stage, do something like
   ``xafs_y.llm.put(5)`` or ``xafs_y.hlm.put(85)``

**Reference wheel** 
   :numref:`The reference stage (Section %s) <foilholder>` is a
   rotation stage with a sample wheel holding up to 48 reference foils
   or stable oxides.  It is calibrated such that the beam passes
   through the center of a slot every 15 degrees.  The slots are
   indexed such that they can be accessed by the symbol of the element
   being measured.  To move to a new reference foil::

     RE(reference('Fe'))

   To see the available foils, do ``%se`` or look at the value of
   ``xafs_ref.mapping``.

   :numref:`See Section %s for a full explanation of the the reference wheel contents. <reference-wheel>`

   `Here is a complete list of standards
   <https://nsls2.github.io/bmm-standards/BMM-standards.html>`__
   in BMM's collection.  These standards are mounted on sample wheels
   and stored in the hutch for ready access by users.


.. _wheel:

Sample wheel
------------

The ``xafs_wheel`` motor is a rotary stage that is typically mounted
on the XY stage.  It can be mounted face-on to the beam or at 45
degrees for use with the fluorescence detector.

Sample plates laser cut from plastic sheet (initially we used `Delrin
<https://en.wikipedia.org/wiki/Polyoxymethylene>`_, since COVID made
supply difficult, we use whatever we can get) are attached to the
rotation stage.  The single-ring version of these plates have 24 slots
arranged around the periphery, evenly spaced 15 degree apart.  The
double-ring version has concentric rings of 24 slots each.  These are
still 15 degrees apart.  The radius of the outer ring is 26 mm larger
than the radius of the inner ring.

While you can move from slot to slot in increments of 15 degrees, i.e.

.. code-block:: python

   RE(mvr(xafs_wheel, 15*3))

it is somewhat easier to move by slot number.  The sample plates are
cut with sample numbers for slots 1, 7, 13, and 19, making it clear
which slot is which.  The wheel is mounted such that the numbers can
be read normally on the side facing the beam.

To move, for instance, to slot 5, do:

.. code-block:: python

   RE(slot(5))

In a macro, do

.. code-block:: python

   yield from slot(5)

To move to the inner or outer ring, do

.. code-block:: python

   RE(xafs_wheel.inner())
   RE(xafs_wheel.outer())

This translates ``xafs_x`` by |nbsp|  |pm| 26 mm.

In a macro, do

.. code-block:: python

   yield from xafs_wheel.inner()
   yield from xafs_wheel.outer()


..
   Sample spinner
   --------------

   The sample spinner is a 12 volt CPU cooling fan mounted on a plate
   which is mounted on the tilt stage.  It is used to spin crystalline
   samples in an effort to suppress Bragg peaks which might enter the
   fluorescence detector.

   To turn the spinner on and off::

     fan.on()
     fan.off()

   To turn the spinner on or off in a :numref:`macro (Section %s) <macro>`::

     yield from fan.on_plan()
     yield from fan.off_plan()

   The spinner should **always** be turned off before entering the end
   station.  It is a good idea to always have a camera pointed at the
   spinner while it is use.


Glancing angle stage
--------------------

The glancing angle stage, shown in :numref:`Figure %s <fig-gastage>`,
can hold up to eight samples and allows each sample to spin
independently.  The spinning allows spurious diffraction from a
crystalline substrate into the fluorescence detector to be suppressed.

.. _fig-gastage:
.. figure:: _images/instrumentation/glancing_angle_stage.jpg
   :target: _images/glancing_angle_stage.jpg
   :width: 50%
   :align: center

   The glancing angle stage with 8 sample positions.

To move to a sample position::

  RE(ga.to(3))

where the argument is a number from 1 to 8, as shown by the labels in
:numref:`Figure %s <fig-gastage>`.  This command will turn off all
other spinners, rotate that sample into the beam path, and start the
sample spinning.

To turn a spinner on or off, where the argument is a number from 1 to 8::

  RE(ga.on(3))
  RE(ga.off(3))

To turn off all spinners::

  RE(ga.alloff())

In a plan::

  yield from ga.on_plan()
  yield from ga.off_plan()
  yield from ga.alloff_plan()



Sample alignment
~~~~~~~~~~~~~~~~

A sample is aligned into the beam by moving the tilt stage to an
approximately flat position::

  RE(mv(xafs_pitch(0))

Then performing the following sequence::

  RE(linescan(xafs_y, 'It', -1, 1, 41))
  RE(linescan(xafs_pitch, 'It', -2, 2, 41))

At the and of the ``xafs_y`` scan, pick the position halfway down the
edge in the It signal.  At the end of the ``xafs_pitch`` scan, select
the peak position.  This will place the sample such that it is flat
relative to the incident beam direction and halfway blocking the beam.

You may choose to iterate those two scans.

Next move the sample to the measurement angle.  Suppose the
measurement angle is 2.5 degrees::

  RE(mv(xafs_pitch, 2.5))

Finally, position the sample so that the beam is hitting the center of
the sample::

  RE(linescan(xafs_y, 'If', -1, 1, 41))

Since the sample is not at the eucentric of the tilt stage, this final
vertical scan is always necessary.  When first aligning the sample,
you may need to center the sample in ``xafs_x`` as well::

  RE(linescan(xafs_x, 'If', -6, 6, 41))

You will almost certainly need to scan over a longer range.  Make sure
the detector is retracted far enough to allow for this motion.


Automated alignment
~~~~~~~~~~~~~~~~~~~

The sequence described above can be automated in many cases::

  RE(ga.auto_align(2.5))

This will run the sequence of alignment scans described above,
pitching the sample to the user-specified angle before the vertical
scan measuring the fluorescence signal.  This works by fitting an
error function to the ``xafs_y`` scan versus It, selecting the peak of
the pitch scan, then selecting the peak of the ``xafs_y`` scan versus
fluorescence.

.. _fig-ga_alignment_plot:
.. figure:: _images/software/spinner-alignment.png
   :target: _images/spinner-alignment.png
   :width: 50%
   :align: center

   If all goes well, the result of the sample alignment looks like
   this.  A picture like this is posted to :numref:`Slack (Section %s)
   <slack>`.


For very flat samples which are square or circular and about 5mm
across or larger, this alignment algorithm is very robust.  For oddly
shaped samples, verify that the automation works before relying upon
it.  Otherwise, you will need to do the alignment by hand.

.. _xafs_table:

Table motors
------------

Typically, table motors are not moved individually.  When changing
:numref:`photon delivery system modes (Section %s) <change-mode>`, the
table should be put into the correct orientation such that the beam
passes through the center of the ion chambers.  It is very easy to put
the beamline in a confusing state by changing the table motors outside
of the ``change_mode()`` command.

The lateral table motors |nd| and its yaw |nd| are normally disabled.
The downstream jacks are coupled and driven by a single motor.  Roll
is no longer available (as of Summer 2026).


.. table:: XAFS table motors
   :name:  xafs-table
   :align: left

   ===================  ========  =================================
   motor                units     notes
   ===================  ========  =================================
   xafs_yu              mm        upstream table jack
   xafs_yd              mm        downstream
   xafs_table.vertical  mm        coordinated linear motion
   xafs_table.pitch     degrees   coordinated table pitch
   ===================  ========  =================================


**Querying table position**
   The position of any motor can be queried with a command line like
   ``%w xafs_table``.

**Coordinated table movement**
   + ``RE(table_height(mode='A'))``: Move the table to its position
     for a specified mode, the modes :numref:`(see Section %s)
     <change-mode>` are ``'A'``, ``'B'``, ``'C'``, ``'D'``, ``'E'``,
     and ``'F'``.
   + ``RE(table_height(by=amount))``: Move the table vertically by an
     amount. This moves the three vertical motors the same amount.
   + ``RE(table_height(pitch=adjustment))``: Move the front by the
     specified amount, move the back the same in the opposite
     direction.  I.e. adjust the pitch of the table, except that the
     adjustment is in mm units, not degrees.

**Moving table motors**
   The normal movement commands work on the real and virtual motors,
   e.g.::

      RE(mvr(xafs_yd, 3))
      RE(mv(xafs_table.vertical, 107))

   Again, this is rarely necessary.  The mode changing plan should
   leave the table in the correct location for your experiment.

   All table movements are recorded in the :numref:`experimental log
   (Section %s) <log>`.

Examine Motor Axes
------------------

Some BlueSky functionality related to the axes controlled by the FMBO
MCS8 motor controllers.  These include:

+ Collimating mirror (``m1_*``)
+ Filter assemblies (``dm1_*``)
+ Monochromator (``dcm_*``)
+ Second diagnostic module (``dm2_*``)
+ Focusing mirror (``m2_*``)
+ Harmonic rejection mirror (``m3_*``)
+ Third diagnostic module (``dm3_*``)

(38 axes motors in total) but not any of the end station motors
(``xafs_*``), which are run using NSLS-II standard GeoBricks.

**Homing**
  Any of these axes can be homed with, for example, ``dm3_bct.home()``

**Summarize the status of a motor**
  To show the values of all the status flags, for example, ``dm3_bct.status()``

**Which motors have been homed?**
  Do this command: ``homed()``

**Which motors have their amplifiers enabled?**
  Do this command: ``ampen()``
