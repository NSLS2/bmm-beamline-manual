..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. _automation:

Beamline automation
===================

.. caution:: Spreadsheets with version number earlier than 13 **will not
	     work** as of 1 March, 2024.  Grid spreadsheets changed
	     significantly between 16 and 17.



BMM currently supports six categories of spreadsheet-based automation:

#. Sample wheels (|download| `spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/wheel.xlsx>`__)
#. Linkam stage temperature control (|download| `spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/linkam.xlsx>`__)
#. LakeShore 331 controller for Displex cryostat (|download| `spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/lakeshore.xlsx>`__)
#. Glancing angle stage (|download| `spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/glancing_angle.xlsx>`__)
#. Generic motor grids (|download| `spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/grid.xlsx>`__)
#. Resonant relfectivity (|download| `spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/reflectivity.xlsx>`__)
   (This is a work in progress.)

The latest spreadsheets for each of these can always be found at
https://github.com/NSLS2/bmm-profile-collection/tree/main/startup/xlsx


Each of the spreadsheets looks much like this, although there are some
differences in the columns corresponding to the different instruments.

.. _fig-wheel-spreadsheet:
.. figure:: _images/spreadsheets/wheel_spreadsheet.png
   :target: _images/wheel_spreadsheet.png
   :width: 90%
   :align: center

   Example spreadsheet for running an experiment on an *ex situ*
   sample wheel.

Always use the most up-to-date version of the spreadsheet from one of
the links above.

.. note:: The current spreadsheet version is **17**, as of 7
          July, 2025.  You should *always* use a current
          spreadsheet.

.. caution:: Spreadsheets with version number earlier 13 **will not
	     work** as of 1 March, 2024.  Grid spreadsheets changed
	     significantly between 16 and 17.


Common features
---------------

Default information
~~~~~~~~~~~~~~~~~~~

All the spreadsheets use the concept of "default" scan information,
that is, information that is expected to be used for most or all of the
indicated measurements.  In :numref:`Figure %s
<fig-wheel-spreadsheet>`, the defults are entered into the row with the
green background.  All rows underneath the green line are used to
describe individual measurements.

For each individual measurement:

+ If a white cell is left blank, the default value from the
  corresponding green cell will be used.  
+ If a white cell is filled in, that value will be used for that
  measurement.

Some columns do not use the green default line:

+ The filename must be explicitly specified.  A blank filename cell is
  interpreted as a row to be skipped.
+ Motor columns, if used, must be filled in.  A blank cell means not
  to move that motor.  Motor columns indicate that the green cell is
  ignored by displaying a bold X through the green cell, as shown in 
  :numref:`Figure %s <fig-motor_columns>`.

.. _fig-motor_columns:
.. figure:: _images/spreadsheets/motor_columns.png
   :target: _images/motor_columns.png
   :width: 75%
   :align: center

   Motor columns in spreadsheets show that the green cell is not used
   by displaying a bold X through the green cells in those
   columns. This example is from the grid spreadsheet.



Experimenters
~~~~~~~~~~~~~

.. note::

   As of summer 2024, with the implementation of data security, the
   beamline now has access to information from the proposal and SAF.
   It is no longer necessary to specify the names of the
   experimenters.  All names on the proposal will be put in the
   metadata of every scan.

.. _spreadsheet_options:

Measurement options
~~~~~~~~~~~~~~~~~~~

Beneath the experimenter cell, there are three drop-down menus for
setting aspects of the sequence of measurements described in the
spreadsheet.  These are shown in :numref:`Figure %s
<fig-measurement_options>`.


#. A yes/no menu for telling Bluesky to close the shutter at the end
   of the measurement sequence.

#. A menu of options for modifying filenames to contain information
   about things like absorber element, edge symbol, Linkam stage
   temperature, and so on.  This simplifies data entry into the
   ``filename`` column of the spreadsheet.

#. A place for specifying the number of repetitions of the entire
   spreadsheet. This is different from the column labeled
   "repetitions", which specifies the number of repeated XAS scans of
   the sample in that row of the spreadsheet.  


.. _fig-measurement_options:
.. figure:: _images/spreadsheets/measurement_options.png
   :target: _images/measurement_options.png
   :width: 85%
   :align: center

   Measurement options at the top of the spreadsheet.


Detector position
~~~~~~~~~~~~~~~~~

On the right hand side of each spreadsheet, there is a column for
specifying the position of the fluorescence detector.  See
:numref:`Figure %s <fig-motor_columns>` for an example from the grid
spreadsheet.

A smaller value is closer to the sample.

The fully retracted position is 205.

The detector position is set on a sample-by-sample basis, allowing the
best possible measurement |nd| not saturating the detector while
maximizing the signal for samples of different absorber concentrations
|nd| for each sample.  For many experiments, most of the set up work
involves moving from sample to sample and setting the values of this
column.


Fine tuning sample position and slits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On the right hand side of each spreadsheet, there are columns for
specifying specific positions for sample X and Y and for slit width
and height.  This allows you to fine tune the sample position and beam
size on a per-sample basis.

Some notes about how motors are interpreted for specific spreadsheet
types.

:motor grid: The motor grid spreadsheet offers five columns for
	     specifying motor positions.  Two of them are mandatory,
	     three are optional and may be specified as "none".  

	     The motors associated with those columns are specified by
	     the user.  The column headings |nd| row 5 |nd| are drop
	     down menus. The two mandatory columns |nd| R and S |nd|
	     can be any xafs stage except for the rotation stages for
	     the *ex situ* sample wheel, the glancing angle rotation
	     stage, or the reference wheel.

	     Those three rotary stages can be selected as optional
	     motors.  For ``xafs_wheel`` or ``xafs_ref``, the value
	     should be an integer between 1 and 24.  For
	     ``xafs_garot``, the value should be an integer betwene 1
	     and 8.  Any other value for these motors will fail with
	     an error message.

	     In this way, a grid of 2 to 5 motors can programmed.

:glancing angle: The glancing angle spreadsheet has the columns for
		 specifying slit width and height.  It also has
		 columns for specifying sample Y and pitch (X and
		 pitch when in perpendicular mode) when manual
		 alignment rather than automated alignment is
		 selected.  There is no column for specifying the X
		 position (or Y when in perpendicular mode) as use of
		 the glancing angle stage presumes that all the
		 samples are mounted at the centers of the spinners.


.. _xlsx:

Selecting a spreadsheet
-----------------------

All spreadsheets are imported using the :file:`xlsx()` command.  The
spreadsheets are self-identifying.  Every spreadsheet has an
identifying string spanning cells B1:C1.  This is the cell with the
pink background.  

.. danger:: **Never** change the text in the pink cell or your
            spreadsheet will likely be interpreted incorrectly.

Import a spreadsheet
~~~~~~~~~~~~~~~~~~~~

To convert a spreadsheet into a macro then run the macro, do the
following:

.. sourcecode:: python

   xlsx()

This will show a numbered list of all :file:`.xlsx` files in your data
folder, something like this:

.. sourcecode:: text

  Select your xlsx file:

    1: 20210127-KB1.xlsx
    2: 20210127-KB3.xlsx
    3: 20210128-KB2.xlsx
    4: 20210128-KB4.xlsx
    5: 20210128-KB5.xlsx
    6: wheel_template.xlsx

    r: return

  Select a file > 

Select the :file:`.xlsx` file you want to import.  Based on the
content of the pink identifying cell, your spreadsheet will be
interpreted appropriately.

You may have multiple tabs in the spreadsheet file.  If the file you
selected from the menu shown above has multiple tabs, you will be
presented with a menu of tabs, something like this:

.. sourcecode:: text

  Select a sheet from yourfile.xlsx:

    1: tab1
    2: tab2
    3: tab3

    r: return

  Select a file > 

Enter the number corresponding to the tab to be measured.

The menu of tab selections will only be presented if there is more
than one tab in the spreadsheet file.

You may organize your experiment in a single file with multiple tabs
or in multiple files (each with one or more tabs).  That is enturely
up to you.

Generating Bluesky instructions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The tab on the selected spreadsheet file will be parsed, then a macro
file generated called :file:`<tab>_macro.py` and an INI file called
:file:`<tab>.ini`, where :file:`<tab>` is the name of the tab from
which the instructions were read.

It is, therefore, a good idea to give your tabs names that
indicate something about the experiment being described on that tab.

The INI file (:numref:`Section %s <ini>`) contains the default values
from the green line (see :numref:`Figure %s <fig-wheel-spreadsheet>`).
The macro file is imported into the BlueSky session, providing a new
with the name of the spreadsheet file.  If the tab in the spreadsheet
was called :file:`mysamples`, the new BlueSky command is called
``mysamples_macro()``.

.. admonition:: Future Tech!

   Convert spreadsheets to Bluesky |qs| input.


.. _sample_wheel_automation:

Sample wheel automation
-----------------------

The standard *ex situ* sample holder at BMM is a plastic wheel mounted
on a rotation stage.  Examples are shown in figures
:numref:`fig-doublewheel`.  The rotation stage is mounted on an XY
stage, so when one slot on the sample wheel is aligned, all the slots
are aligned.  Alignment details |harr| :numref:`Section %s <special-linescans>`


.. _fig-doublewheel:
.. figure:: _images/instrumentation/double_wheel_sm.jpg
   :target: _images/double_wheel_sm.jpg
   :width: 50%
   :align: center

   Double-ring sample wheels with 48 sample positions.  There
   are options for both wheel styles with 13mm x 3 mm slots or 13mm
   diameter holes.  The rings on the double wheel are 26 mm apart
   (center to center of slots/holes).

The automation concept is that a measurement at an edge on a slot on
the sample wheel is described by a row in the spreadsheet.  Each
column of the spreadsheet carries one parameter of the XAFS scan.


.. _fig-doublewheel-spreadsheet:
.. figure:: _images/spreadsheets/doublewheel_spreadsheet.png
   :target: _images/doublewheel_spreadsheet.png
   :width: 70%
   :align: center

   Example spreadsheet for running an experiment from a wheel with a
   two sample rings.  Link: `wheel spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/wheel.xlsx>`__.


If you have read :numref:`Section %s <ini>` about the INI file, then
most of the columns in this spreadsheet will be quite familiar.  Most
of the columns are used to specify the same set of parameters as in
the INI file |nd| file name, element, edge, and so on.

The green cell in the first row is used to input the names of all the
people involved in the experiment, as explained above.

As explained above, row 6, the row with the lime-green background,
is used to specify the default values for all the parameters.  The
concept here is to try to avoid having to input repetitive
information.  For instance, in this case, all measurements will be
made at the Fe K edge.  The element and edge are all specified in the
green row.  Those cells are left blank for all subsequent rows, so the
default values will be used.

In short, any cell that is left blank will use the value from the
green, default row.  Any cell for which a value is specified will be
used in the macro that gets generated.

The first column is used to specify the slot number for each sample on
the sample wheel.

The second column is a simple way of excluding the slot from
measurement simply by specifying *No*.

The next several columns correspond to lines in the INI file as
explained in :numref:`Section %s <ini>`.

Energy changes can be included in the macro by specifying different
values for element and/or edge in a row.  When specified
and different from the previous row, a call to the ``change_edge()``
command (:numref:`Section {number} <pds>`) is inserted into the macro.

Not shown in :numref:`Figure %s <fig-wheel-spreadsheet>` are columns
for tweaking the ``xafs_x`` and ``xafs_y`` positions, adjusting the
horizontal and vertical size of :numref:`slits3 (see Section %s)
<slits3>`, and adjusting the fluorescence detector position.


Again, assuming the tab in the spreadsheet was called ``mysamples``,
you can then run the macro generated from the spreadsheet by::

   RE(mysamples_macro())


Here are the first few lines of the macro generated from this
spreadsheet. Note that for each sample, the macro first moves using
the ``slot()`` command, then measures XAS using the ``xafs()``
command.  The ``xafs()`` command uses the INI file (:numref:`Section
{number} <ini>`) generated from the green default line and has
explicit arguments for the filled-in spreadsheet cells.

.. sourcecode:: python
   :linenos:

   yield from slot(1)
   yield from xafs('MnFewheel.ini', filename='Fe-Rhodonite', sample='MnSiO3', comment='ID:93 Russia')
   close_last_plot()

   yield from slot(2)
   yield from xafs('MnFewheel.ini', filename='Fe-Johannsonite', sample='CaMnSi2O6 - LT', comment='B –Iron Cap Mine; Graham Country, Arizona')
   close_last_plot()

   yield from slot(3)
   yield from xafs('MnFewheel.ini', filename='Fe-Spessartine', sample='Mn3Al2(SiO4)3', comment='Grants Mining District; New Mexico')
   close_last_plot()

   ## and so on....


Mounting samples on the sample wheel
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The sample wheels are intended to be an easy way of mounting and
automating *ex situ* measurements.  There are, however, some rules to
their use.

1. Note that the slots are numbered from 1 to 24.  This indexing is
   indicated by the numbers 1, 7, 13, and 19 cut into the plastic.

2. As shown in :numref:`Figure %s <fig-doublewheel-spreadsheet>`, the
   inner and outer slots are labeled as such.  Thus there is an inner
   slot #1 and an outer slot #1.  The slots are **not** numbered from
   1 to 48.  The numbering is inner 1 to inner 24 and outer 1 to
   outer 24.

3. The side the wheel on which the numbers are read normally is the
   side that faces the beam.  The side facing the beam is refered to
   as the "front" of the wheel.

4. Samples, when mounted on the wheel using tape as in the left side
   of :numref:`Figure %s <fig-mountsamplewheel>`, should be taped to
   the front of the wheel.

5. As shown in :numref:`Figure %s <fig-mountsamplewheel>`, samples can
   fill the slots or they can be affixed to the front of the wheel
   over the slots.  The choice is yours.  Do which ever seems more
   convenient for your samples.

6. Note that there are lines scored into the plastic going through the
   centers of the slots.  Take care to mount your samples such that
   they are well-centered in each slot.  The better a job you do
   aligning the mounted samples, the easier it will be to correctly
   align each sample in the beam.

7. While completely filling or covering each slot makes the wheel very
   easy to use, that is not always possible.  Slit size and sample
   stage positions can be adjusted in the spreadsheet, as needed for
   each sample individually.  That said, centering your samples
   carefully in each slot is especially helpful when the sample is
   smaller than the slot.

8. Feel free to write directly on the wheel with a Sharpie.  The
   sample holder will be in view of multiple cameras.  Clearly
   readable labels seen in the cameras are helpful when setting up the
   experiment and provide a :numref:`useful visual record of the
   experiment (Section %s) <snap>`.



.. subfigure::  AB
   :layout-sm: AB
   :subcaptions: above
   :gap: 8px
   :name: fig-mountsamplewheel
   :class-grid: outline

   .. image:: _images/instrumentation/ontop.jpg

   .. image:: _images/instrumentation/filled.jpg

   (Left) An example of samples taped to the front of the sample wheel
   (Right) An example of samples filling the slots in the sample wheel.



.. admonition:: Do not make up your own numbering scheme!
   :class: caution

   The spreadsheet automation system expects the indexing as described
   above.  If you make up your own indexing scheme, you **will** get
   confused at the beamline and you run the risk of mislabeling your
   data.

.. _auto_linkam:

Linkam stage automation
-----------------------

One of the temperature control options at BMM is a `Linkam stage
<https://www.linkam.co.uk/thms600>`_.  Ours is the kind that can cool
using liquid nitrogen flow or heat up to 600 C using a resistive
heater.  The linkam stage is typically mounted upright on top fo the
XY stage.


.. subfigure::  AB
   :layout-sm: AB
   :subcaptions: above
   :gap: 8px
   :name: fig-linkamstage
   :class-grid: outline

   .. image:: _images/instrumentation/linkam.jpg

   .. image:: _images/instrumentation/dewar.jpg

   (Left) The Linkham stage mounted for transmission on the sample
   stage.  (Right) The 25 L dewar used for cooling the Linkam stage.


The automation concept for the Linkam stage is quite similar to the
*ex situ* sample holder.  Instead of specifying the slot position of the
sample, you will specify the target temperature for the measurement.
There is also a column for specifying the holding time after arriving
at temperature before beginning the XAFS measurement.

The feature described in :numref:`Section %s <spreadsheet_options>`
for modifying filenames is particularly useful in this context.  It
can be used to put the measurement temperature in the filename,
allowing you to simply specify a default filename, leaving that cell
in each row blank.  The generated data files will then have sensible
names. 


.. _fig-linkam-spreadsheet:
.. figure:: _images/spreadsheets/linkam_spreadsheet.png
   :target: _images/linkam_spreadsheet.png
   :width: 70%
   :align: center

   Example spreadsheet for running a temperature-dependent experiment
   using the Linkam stage.  Link to the `Linkam spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/linkam.xlsx>`_

.. _auto_lakeshore:

LakeShore/Displex automation
----------------------------

For extremely low temperature experiments, BMM has a Displex crystat
which uses a two-stage helium compressor to cool the cold head down as
low as 10K with temperature control between 10K and 500K using a
resistive heater and a `LakeShore temperature controller
<https://www.lakeshore.com/products/categories/overview/discontinued-products/discontinued-products/model-331-cryogenic-temperature-controller>`__.

This is a somewhat unusual version of the Displex system in that it is
suitable for low-vibration applications.  The compressor is
mechanically decoupled from the cold head, reducing the motion of the
sample. As a result of this cooling system, it is somewhat
time-consuming to temperature cycle and replace samples.  Expect that
cooling from room temperature to 10K will take about 2 hours and
budget up to an an hour for returning to room temperature and changing
samples. 

.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-displex
   :class-grid: outline

   .. image:: _images/instrumentation/cryostat.jpg

   .. image:: _images/instrumentation/lakeshore331.png

   (Left) The Displex cryostat and it's compressor.  (Right) The
   LakeShore 331 controller, used to control temperature for the
   cryostat shown to the left.


The automation for the LakeShore 331 works much the same as for the
Linkam stage.  Again, you will specify the target temperature for the
measurement.  And there is a column for specifying the holding time
after arriving at temperature before beginning the XAFS measurement.

There is a column for specifying the power level of the heater in the
cryostat.  There are three power settings.  You probably want to use
the high power setting.  The controller is pretty well tuned for the
cryostat.  It is unlikely to overshoot the when raising temperature.

.. _fig-lakeshore-spreadsheet:
.. figure:: _images/spreadsheets/lakeshore_spreadsheet.png
   :target: _images/lakeshore_spreadsheet.png
   :width: 70%
   :align: center

   Example spreadsheet for running a temperature-dependent experiment
   using the Displex cryostat and the LakeShore 331. Link to the
   `LakeShore spreadsheet <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/lakeshore.xlsx>`_.


..
  .. _fig-lakeshoreCSS:
  .. figure::  _images/instrumentation/lakeshoreCSS.png
     :target: _images/lakeshoreCSS.png
     :width: 30%
     :align: center

   The CSS screen for the LakeShore 331.


.. _auto_ga:

Glancing angle stage automation
-------------------------------

This stage is used to automate measurement at glancing angle, usually
on thin film samples.  The stage can be mounted horizontally or
vertically, allowing measurement of in- or out-of-plane strain in thin
films.

.. _fig-glancinganglestage:
.. figure:: _images/instrumentation/glancing_angle_stage.jpg
   :target: _images/glancing_angle_stage.jpg
   :width: 50%
   :align: center

   The glancing angle stage with 8 sample positions.

This stage is mounted on a rotation stage to move between samples.
The rotation stage is mounted on a tilt stage to set the incident
angle of the beam relative to the sample surface.  This entire set up
is mounted on the XY stage for alignment on the beam.

Each sample is affixed to a sample spinner (which is simply a cheap, 24 VDC
CPU fan).  The 8 spinners are independently controlled via slip ring
electrical connection that runs through the axis of the rotation
stage.  In practice, only the sample that is being measured is spinning.

Again, the automation concept is very similar to the *ex situ* sample
wheel.  Instead of specifying slot number, the spinner number is
specified on each row.  There is also a yes/no menu for specifying
whether the sample spins during measurement.

.. _fig-glancingangle-spreadsheet:
.. figure:: _images/spreadsheets/glancingangle_spreadsheet.png
   :target: _images/glancingangle_spreadsheet.png
   :width: 70%
   :align: center

   Example spreadsheet for running an experiment using the glancing
   angle stage.  Link to the `glancing angle spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/glancing_angle.xlsx>`_.

Not shown in :numref:`Figure %s <fig-glancingangle-spreadsheet>` are
columns for specifying how sample alignment is handled.  The default
is to do automated alignment.  This works by following this script:

#. Move the stage to an incident angle that is close to flat for the
   sample and start the sample spinning.
#. Do a scan in the vertical direction, measuring the signal in the
   transmission chamber. Fit an error function to the transmission
   signal.  The centroid of that function is the position with the
   sample half-way in the beam.
#. Do a scan in pitch, measuring the signal in the transmission
   chamber.  The peak of that measurement is the position where the
   sample is flat relative to the beam direction.
#. Repeat steps 2 and 3.
#. Move the sample tilt to the angle specified by the user in the
   spreadsheet.
#. Do a scan in the vertical direction, measuring the signal in the
   fluorescence detector.  The center of mass of that measurement is
   the position where the beam is well-centered on the sample.

The result of this fully automated sequence is shown in
:numref:`Figure %s <fig-spinner_alignment>`.

.. _fig-spinner_alignment:
.. figure:: _images/software/spinner-alignment.png
   :target: _images/spinner-alignment.png
   :width: 50%
   :align: center

   This visual representation of the automated glancing angle
   alignment is posted to Slack and presented in the measurement
   :numref:`dossier (Section %s) <dossier>`.

For some samples, the automated alignment is unreliable, so there is
an option in the spreadsheet for manual alignment.  In that case, find
the ``xafs_y`` and ``xafs_pitch`` positions for the sample at its
measurement angle and well-aligned in the beam.  Enter those numbers
and they will be used by the macro rather than performing the
automated alignment.

Motor grid automation
---------------------

The final kind of automation-via-spreadsheet available is BMM is for a
generic motor grid.  The most common motor grid used for measurement
is the sample XY stage, ``xafs_x`` and ``xafs_y``.  However, most any
xafs stage can be used for the grid.

.. _fig-grid-spreadsheet:
.. figure:: _images/spreadsheets/grid_spreadsheet.png
   :target: _images/grid_spreadsheet.png
   :width: 70%
   :align: center

   Example spreadsheet for running an experiment on an XY grid.  Link
   to the `motor grid spreadsheet
   <https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/grid.xlsx>`_.


The columns in :numref:`Figure %s <fig-motor_columns>` |nd| to the
left of the view shown in :numref:`Figure %s <fig-grid-spreadsheet>`
|nd| are used to specify the axes in the grid.

The motors are identified by a drop down selection in the header cells
in row 5.

The first two motor columns are mandatory and must be selected.  These
can be any motors *except for* the *ex situ* sample wheel
(``xafs_wheel``), the glancing angle rotary stage (``xafs_garot``), or
the reference wheel (``xafs_ref``).

The three optional motors can be any stage, including the three rotary
stages.  Some details about the optional motors:

+ If ``none`` is selected from those drop-down menus, then that
  motor is not included in the automation.

+ If ``xafs_wheel`` or ``xafs_ref`` is selected, the value should be
  an integer between 1 and 24.  Any other value will trigger an error.

+ If ``xafs_garot`` is selected, the value should be an integer
  between 1 and 8.  Any other value will trigger an error.


.. todo:: What about ``xafs_refx`` when using ``xafs_ref``?  Maybe
	  ``xafs_ref`` could take values like "4i" and "12o"?  Maybe
	  exclude the ``xafs_refN`` stages entirely from the
	  automation?

In most ways |nd| the ``slot`` column being the notable exception |nd|
this spreadsheet is identical to the *ex situ* sample wheel
spreadsheet.

Resonant refectivity automation
-------------------------------

.. todo:: This is a work in progress


Future developments
-------------------

.. admonition:: Future Tech!

   Spreadsheets for:

   + Electrochemistry experiments using the BioLogic potentiostat
   + Chemistry experiments using the gas cart, including its mass flow
     controllers, valves, temperature controller, and mass spectrometer.
   + Anton-Parr
   + Automation of linear/pitch alignment for any pair of motors used
     for a glancing angle experiment


.. caution:: Spreadsheets with version number earlier than 13 **will
	     not work** as of 1 March, 2024.  Grid spreadsheets changed
	     significantly between 16 and 17.


