..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.

.. |itchamber|     replace:: I\ :sub:`t`


.. _refl:

Reflectivity Set-Up
===================

We are still in the early stages of resonant reflectivity experiments
at BMM.  This section of the beamline manual documents how to set up
for one of these experiments.

:red:`This section is a work in progress.`

From a setup perspective, the reflectivity experiment requires this
instrumentation configuration:

#. The sample is mounted on the :numref:`Huber tilt stage (Section %s)
   <tilt-stage>`, which is mounted on the sample XY stage.
#. The |itchamber| chamber is mounted on a translation stage in X so
   it can be translated in and out of the beam.
#. A beam stop is positioned downstream of the |itchamber| chamber.
#. An area detector |nd| either the Pilatus or the Eiger |nd| is on an
   XY stage near the downstream end of the table.

.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-refl_setup
   :class-grid: outline

   .. image:: _images/reflectivity/refl_sideview.jpg

   .. image:: _images/reflectivity/refl_topview.jpg

   (Left) The reflectivity setup viewed from the side. (Right) The
   reflectivity set up from above.  

.. attention:: :numref:`Figure %s <fig-refl_setup>` was a setup during
   the 2025-3 cycle.  That was prior to the new ``xafs_y`` and
   ``xafs_y`` stages and the repurposing of the old stages as
   ``xafs_adx`` and ``xafs_ady``


Translatable It Chamber
-----------------------


The |itchamber| chamber needs to be on a stage.  Sample alignment uses
the |itchamber| signal, but the actual measurement needs to have the
scattered signal be unobstructed by the bulk of the chamber.

One solution to this problem is to bolt the ``xafs_spare`` stage to
the table as shown in :numref:`Figure %s <fig-refl_it>`.

A small piece of rail is bolted to the ``xafs_spare`` carriage.  Note
that this uses a particular piece of railing that has been modified
for this use.

Be very mindful of the bundle of ethernet cable, power cable, and gas
line.  It is way too easy to get something in that bundle to catch on
the edge of the table or some other obstruction.  In :numref:`Figure
%s <fig-refl_it>`, a piece of X-rail has been secured to the table to
provide support for the cables.


.. admonition:: Future Tech!

   Set in and out positions as attributes of the ``refl`` object.
   Current using values set in ``BMM_configuration.ini``.


.. code-block:: python

   RE(it_in())
   RE(it_out())



.. subfigure::  AB
   :layout-sm: AB
   :gap: 8px
   :subcaptions: above
   :name: fig-refl_it
   :class-grid: outline

   .. image:: _images/reflectivity/it.jpg

   .. image:: _images/reflectivity/it_retracted.jpg

   (Left) The |itchamber| chamber on the ``xafs_spare`` stage.
   (Right) |itchamber| fully retracted.  Note that the bundle of
   ethernet cable, power cable, and gas line are carefully placed to
   not catch on an edge when moving in the +X direction.


Mounting the beam stop
----------------------

.. todo::
   Take a lot of photos the next time Evan is here.

``xafs_bsx`` and ``xafs_bsy``

Put a phosphor screen somewhere below the detector.  Put the beam on
that screen.

Raise ``xafs_bsy`` until it just obstructs that beam.  Lower
``xafs_ady`` into the beam and use the area detector to fine-tune
the position of the beam stop.

Set limits as needed on ``xafs_bsx`` and ``xafs_bsy``


Mounting the Pilatus 100k
-------------------------

Use ``xafs_adx`` and ``xafs_ady``.  Use the L-bracket labeld as
being for the Pilatus.

See :numref:`Section %s <pilatus>` for instructions on getting the
Pilatus 100k up and running.


.. admonition:: Future Tech!

   Use the Eiger 4M.


Bluesky configuration
---------------------

Do stuff with ``BMM_configuration.ini``...


Experiments
-----------

Resonant relfectivity spreadsheet (|download| `spreadsheet
<https://github.com/NSLS2/bmm-profile-collection/blob/main/startup/xlsx/reflectivity.xlsx>`__)


.. todo::
   Need to document how interactions with area detector ROIs works.

.. todo::
   Need to document what information goes into the spreadsheet.
