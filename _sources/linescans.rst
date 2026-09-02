..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. role:: key
    :class: key


.. _linescan:

Sample position scans
=====================

General line scans
------------------

In the nomenclature of BMM, a line scan is a simple scan of a single
motor axis with an on-screen plot.  At BMM this sort of scan is
typically done using the ``linescan()`` plan.  This is a wrapper
around BlueSky's `rel_scan()
<https://nsls-ii.github.io/bluesky/generated/bluesky.plans.rel_scan.html#bluesky.plans.rel_scan>`_.

In BMM's ``linescan()`` plan, the scan is always a relative scan
around the current position of the motor being scanned.  It works like
this::

    RE(linescan(xafs_x, 'it', -4, 4, 81))

The arguments are:

#. The motor axis to be scanned.  This can be either the motor's
   name in Bluesky.

#. A string indicating the detector for the plotted signal.  The
   choices are:

   * ``it``: display the ratio of ``It/I0``
   * ``if``: display the sum of four silicon drift channels normalized
     by ``I0``
   * ``i0``: display the signal on ``I0``
   * ``ir``: display the ratio of ``Ir/It``

#. The starting position of the motor scan, relative to the current
   position.

#. The ending position of the motor scan, relative to the current
   position.

#. The number of steps in the scan.

Note that the first two arguments can be in either order.  The plan
will sort it out for you.  These are equivalent::

  RE(linescan('it', xafs_x, -4, 4, 81))
  RE(linescan(xafs_x, 'it', -4, 4, 81))


At the end of the scan, you are prompted with the following question::

    Pluck motor position from the plot? yes: y the Enter (or just Enter), no: n then Enter

If you answer :key:`y` then :key:`Enter`, or simply hit
:key:`Enter`, you will be prompted to single click the left mouse
button :mark:`leftclick,.` on the plot.  

You will then be prompted a second time with the value you selected.

The motor that was scanned will then move to the motor position you
clicked on.

You can skip the "click for motor position" step by typing
:key:`n` and hitting :key:`Enter`.


.. _pluck:

Plucking a point from a line scan
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to repeat the action of clicking on a point and moving the
motor to the click-upon point, do::

  RE(pluck())

This will enable the left mouse :mark:`leftclick,.` click and
subsequent motion on the most recent plot.  

.. warning:: The ``pluck()`` command works on any plot made through
   the :numref:`Kafka plotter (Section %s) <plotting>`.  Be sure you
   are clicking on the plot you intend to pluck from!


Revisit a line scan
~~~~~~~~~~~~~~~~~~~

.. admonition:: Needs to be verified

   Does this still work post-data-security?

Retrieve data from the database using the database key or scan ID::

   ls2dat('/path/to/output.file', '<id>')

This writes from the database to an output file.  The output file is a
simple column data file without header or metadata.


The first argument is the name of the output data file.  The second
argument is either the scan's unique ID |nd| something like
``42447313-46a5-42ef-bf8a-46fedc2c2bd1`` |nd| or the scan's transient
id number.



.. _special-linescans:

Specific line scans
-------------------

Some scan types are performed often enough and always with the same
arguments that they have special names.

**Rocking curve scan**
   This command::

     RE(rocking_curve())

   does a scan of the pitch of the second mono crystal and plots the
   signal on I0.  At the end of the scan, it moves to the position of
   the peak of the rocking curve.  This scan is useful after a large
   change of energy.  It also opens the slits to 3 mm in vertical size
   before starting the scan, then returns the slits to their original
   height after the scan.

   You can put this scan in a macro using::

     yield from rocking_curve()

**Mirror pitch scan**
   This command::

     RE(mirror_pitch(mirror='m3'))
     
   does a scan of the pitch of the harmonic rejection mirror (M3) to
   center the beam onto the hutch slits.  Setting the ``mirror``
   argument to ``m2`` does the same with the focusing mirror (M2),
   which would be appropriate for the focused beam in mode A.  This
   scan is used to optimize the beam intensity after moving the slits
   to their tabulated value.

   You can put this scan in a macro using::

     yield from mirror_pitch(mirror='m3')

**Align ex situ sample holder**
   If the *ex situ* sample wheel is in approximately the right
   position such that X-rays are passing through a slot on the outer
   ring, you can center the slot around the beam with::

     RE(find_slot())

   This will run linescans in the X and Y directions, fitting a
   lineshape to each scan in order to find the center position.  The
   same can be done by hand by doing::

     RE(linescan(xafs_y, 'It',  -3,  3, 31))
     RE(linescan(xafs_x, 'It', -10, 10, 31))

   each time selecting the center when prompted.

   You can put this scan in a macro using::

     yield from find_slot()


Area scans
----------


.. todo:: A raster scan type |nd| i.e. ``RE(raster())`` |nd| that does
   an areascan as described here, makes a nice figure using
   `matplotlib's contourf
   <https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.contourf.html>`_,
   exports data in formats used by popular plotting programs,
   and writes a dossier was tested in 2022-2.  Current status:
   dossier is written, scan works, documentation needs to be written.


An area scan is a simple scan of a two motor axes with an on-screen
heat map.  At BMM this sort of scan is typically done using the
``areascan()`` plan.  This is a wrapper around BlueSky's
`rel_grid_scan()
<https://nsls-ii.github.io/bluesky/generated/bluesky.plans.rel_grid_scan.html#bluesky.plans.rel_grid_scan>`_.
Because the sample stages at BMM do not have encoders, the area scan
is made by `retreading the direction
<http://nsls-ii.github.io/bluesky/tutorial.html#scan-multiple-motors-in-a-grid>`_
of the fast motor rather than snaking back and forth.

In BMM's ``areascan()`` plan, the scan is always a relative scan
around the current positions of both motors being scanned.  It works
like this::

    RE(areascan('it', '<slow_motor>', -4, 4, 81, '<fast_motor>', -2, 2, 41))

The arguments are:

#. The slow motor axis.  This can be either the motor's
   BlueSky name or the nickname in :numref:`Table %s <xafs-stages>`.  So,
   these are equivalent::

     RE(areascan('it', 'x', -4, 4, 81, 'y', -2, 2, 41))
     RE(areascan('it', xafs_x, -4, 4, 81, xafs_y, -2, 2, 41))
     RE(areascan('it', xafs_x, -4, 4, 81, 'y', -2, 2, 41))

   For a motor that does not have a nickname, you must use the BlueSky
   name, as in this very silly example::

     RE(areascan('it', slits3_outboard, -1, 1, 21, dcm_pitch, -2, 2, 41))

#. The starting position of the slow motor, relative to the current
   position.

#. The ending position of the slow motor, relative to the current
   position.

#. The number of steps to take on the slow motor.

#. The fast motor axis.  This can be either the motor's
   BlueSky name or the nickname in :numref:`Table %s <xafs-stages>`.

#. The starting position of the fast motor, relative to the current
   position.

#. The ending position of the fast motor, relative to the current
   position.

#. The number of steps to take on the fast motor.

#. The detector for the plotted signal.  The choices are ``it``,
   ``if``, and ``i0``.  For the ``it`` choice, the plot will display
   the ratio of It/I0.  Similarly for the ``if`` choice, the plot will
   display the sum of four silicon drift channels normalized by I0.
   For the ``i0`` choice, the signal on the I0 chamber will be plotted.

At the end of the scan, you are prompted with the following question::

    Pluck motor position from the plot? [Yn]

If you answer :key:`Y`, or simply hit :key:`Return`, you will be
prompted to single click the left mouse button :mark:`leftclick,.` on
the plot.  Both motors will then move to the position you clicked on.

You can skip the "click for motor position" step by typing
:key:`n` and hitting :key:`Return`.


Plucking a point from an area scan
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to repeat the action of clicking on a point and moving the
motor to the click-upon point, do::

  RE(pluck())

This will enable the left mouse click :mark:`leftclick,.` and
subsequent motion on the most recent plot.  The ``pluck()`` command
*only* works on the most recent plot.  You may not pluck from an older
plot that is still displayed on the screen.

Of course, an older plot remains active in the sense that you can pass
the cursor over the plot and read the mouse coordinates in the bottom,
left corner of the plot window.  You can find a point in this way,
then do a movement command::

  RE(mv(xafs_x, 28.31, xafs_y, 113.97))



Revisit an area scan
~~~~~~~~~~~~~~~~~~~~

.. admonition:: Needs to be verified

   Does this still work post-data-security?

Retrieve data from the database using the database key or scan ID::

   as2dat('/path/to/output.file', '<id>')

This writes from the database to an output file.  The output file is a
simple column data file.  The format of this data file is columns with
datablocks (i.e. rows of constant value of the slow motor) separated by
blank lines.  This is a format that `works with Gnuplot
<http://gnuplot.sourceforge.net/docs_4.2/node331.html>`_ and other
plotting programs.

The first argument is the name of the output data file.  The second
argument is either the scan's unique ID |nd| something like 
``42447313-46a5-42ef-bf8a-46fedc2c2bd1`` |nd| or the scan's transient
id number.


