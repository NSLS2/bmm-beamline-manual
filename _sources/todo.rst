..
   This document was developed primarily by a NIST employee. Pursuant
   to title 17 United States Code Section 105, works of NIST employees
   are not subject to copyright protection in the United States. Thus
   this repository may not be licensed under the same terms as Bluesky
   itself.

   See the LICENSE file for details.


.. role:: strike
    :class: strike

.. _todo_list:

To Do List
==========

Chapters needed:

#. Dealing with XRF spectra, including ``%xrf``, ``xrfat()``, saving
   XRF spectra 
#. Valence maps
#. Document why M1 is the way it is, explain why moving it is a BAD
   idea, explain front end slits


This is an aggregate list of things missing from the BMM beamline
documentation manual.

#. Replace ``db.v2`` with explicit use of Tiled catalog
#. Document use of ``XDI_record`` to control what motors get recorded
   in the XDI header
#. Flesh out documentation on using automation + spreadsheets
#. Explain contents of dossier more fully
#. Current batch of cameras
#. Document using the Xspress3
#. Better abstraction of spreadsheet-to-macro components, less cargo-cult code
#. Document how redis is used at the BL, explain state+redis+BMMuser object
#. Document timescan and SEAD, discuss in plotting section
   Citation: A Filipponi et al 1998 J. Phys.: Condens. Matter 10 235 DOI:
   10.1088/0953-8984/10/1/026


This is an aggregate list of things mentioned in this document that
the BMM data collection system needs to do.

#. Mode C lookup table
#. Lookup table for E < 8 keV at goniometer


How to add a scan.ini parameter
-------------------------------

#. Add its entry into ``scan_metadata()`` in :file:`BMM/energyscan.py`
#. Add its default value to the ``BMM_configuration``  class in :file:`BMM/modes.py`
#. Add it to ``html_dict`` in :file:`BMM/energyscan.py` around line 912
#. Correspondingly, add to the call arguments of
   ``scan_sequence_static_html()`` in :file:`BMM/energyscan.py`
#. make corresponding entries in the various automation spreadsheets

and, of course, use the new parameter in whatever way it needs to be used.

