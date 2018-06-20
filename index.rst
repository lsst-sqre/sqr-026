..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is


   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

Introduction
============

The rapid cadence and scale of the LSST observing program requires that each visit is scheduled shortly before it is observed to make optimal use of conditions that vary with time like observing conditions, observatory and telescope parameters, survey completion, etc. That requires automated data quality feedback from the *Data Management System (DMS)/Prompt Quality Control* to the *Observatory Control System (OCS)/Scheduler*	to	assess	*whether	visits	acquired	should	be	scored	as	successful	and	to	assess	the	general	observing	quality	– e.g.,	weather	and	seeing	– across	the	sky*. It also requires automated but human-readable feedback to monitor data quality, in the form of near-realtime dashboards for observers and end-of-night reports for DM staff.

During observations, the *Prompt Processing* (see section 3 in `LDM-151`_) will perform a Single Frame Processing, from which data quality metrics can be derived for each CCD in a visit for each visit.

Data quality metrics will be published to the OCS/Scheduler throught the Telemetry Gateway at the Base, as defined in `LDM-148`_, which will then convert the messages into telemetry for the OCS Service Abstraction Layer (SAL) as specified by **OCS-DM-COM-ICD-0017** in `LSE-72`_.

In particular, **OCS-DM-COM-ICD-0019** provides a minimum set	of	events	and	telemetry	to	be	published to the OCS.

From **OCS-DM-COM-ICD-0021** the imediate data quality feedback to the OCS/Scheduler is important as it impacts the short term scheduling.

While the detailed implementation of the Telemetry Gateway is not yet defined, we assume that data quality metrics, with a granularity level suitable for the OCS/Scheduler, will be published and also persited in the EFD (see **OCS-DM-COM-ICD-0020**).

.. note::
  We need a requirement that specifies the frequency of the data quality feedback to OCS/Scheduler, as it impacts the Prompt Processing pipeline design and the maximum time the Scheduler will wait for that information before fallback.

.. note::
  The current plan is that DMS *will not* publish to the OCS/Scheduler a pass/fail data quality flag to determine if a visit needs to be scheduled for reobservation.

The Prompt Quality Control software will run at NCSA and will require its own database. The Prompt QC database will be connected to data quality dashboards for near-realtime moniritoring, available to the observers. The Prompt QC database will store the data quality metrics for each processed CCD in a visit for each visit. Additional parameters like observing conditions, observatory and telescode parameters will be obtained from the image header and stored in the Prompt QC database (see Appendix A). The image header will be the main mechanism for OCS-DMS communication in near-realtime.

The Prompt QC database should be exposed to the DAX services at the Data Access Centers as it contains useful information to science users.

End-of-night Data Quality reports will be generated at NCSA and will require information from both the Prompt QC database and from the DM EDF.

Figure 1 presents an overview diagram of the Prompt QC components in the context of the interfaces described above.

.. figure:: /_static/qc_components.png
  :name: Prompt Quality Control components and DMS-OCS interfaces

A detailed description of each component is given in the next sections.


Collecting data quality metrics
===============================

The Single Frame Processing task of the Prompt Processing pipeline is responsible for the reduction of raw
or camera-corrected image data to calibrated exposures, the detection and measurement of
Sources, the characterization of the point-spread-function (PSF), and the generation of an astrometric solution for an image.

Quality metrics are collected instrumenting these tasks with the LSST Verification framework or `lsst.verify` (see `SQR-019`_).

Currently `lsst.verify` is being used to define and collect Key Performance Metrics derived from the LSST Science Requirements Document (`LPM-17`_).


Storing data quality metrics
============================

Metric values computed by pipeline tasks and collected by `lsst.verify` will be uploaded to the Prompt QC database through a REST API.


Publishing data quality metrics to the OCS
==========================================

Monitoring quality metrics in near-realtime dashboards
======================================================

Prompt Data Quality Reports
===========================

The minimum content for the Prompt Data Quality report is specified in  **DMS-REQ-0097** (see section 1.3.14 in `LSE-61`_).

**Specification:** *The DMS shall produce a Level 1 Data Quality Report that contains indicators of data quality that result from running the DMS pipelines, including at least: Photometric zero point vs. time for each utilized filter; Sky brightness vs. time for each utilized filter; seeing vs. time for each utilized filter; PSF parameters vs. time for each utilized filter; detection efficiency for point sources vs. mag for each utilized filter.*

**Discussion:** *The seeing report is intended as a broad-brush measure of image quality. The PSF parameters provide more detail, as they include asymmetries and field location dependence.*

Also from **DMS-REQ-0096** (see section 2.2.10 in `LSE-61`_) the Prompt Data Quality Report must be generated in less than 4h after the end of the nigthly Prompt Processing in order to evaluate *whether changes to hardware, software, or procedures are needed for the following night’s observing.*

.. note::
  Include DMS-REQ-0099 - Level 1 Performance Report Definition and DMS-REQ-0101 - Level 1 Calibration Report Definition here.


As a reference, see the `Dark Energy Survey Night Summary`_ report.


Acessing the QC and EFD databases
---------------------------------


The data quality metrics specified in **DMS-REQ-0097** will be derived from the *Single Frame processing* . However, a richer report also needs to correlate those metrics with the state of the instrument, observing conditions, observatory parameters obtained from the DM EFD. Also, we'll need to correlate that information with observer comments obtained from the observatory electronic log system.


Implementation
--------------
The Data Quality Report will be implemented as jinja templates that will produce rst. It does not require much processing since the information is pre-computed and accessible from the Prompt QC and DM EFD API's.


Appendix A - Data quality metrics and parameters to store
=========================================================

Here we list the data quality metrics and parameters that we should store for each LSST visit in the Prompt QC database.

The data quality metrics measured by the Prompt Quality Control software for each individual CCD in visit. These metrics are also aggregated at the visit level.

  - PSF FWHM
  - PSF Ellipticiy
  - Sky brightness
  - Zeropoint


The data quality parameters obtained from the image header.

  - Visit Start Time (UTC): date and time in UTC when the visit acquisition started.
  - Visit End Time (UTC): date and time in UTC when the visit acquisition ended.
  - Visit Mid Time (UT): date and time in UTC at the midpoint of acquisition.
  - Visit ID: unique identifier of the visit.
  - RA, Dec: Telescope boresight pointing.
  - Airmass: Even though it can be calculated from RA, Dec it is useful to store Airmass since it determines the expected atmospheric contribution.
  - Filter: One of the five LSST observing filters, ugrizy.
  - Focus: The donut estimate of focus error for the visit.
  - Guider DeltaRA, DeltaDec: Guider displacements for TCS correction
  - DIMM seeing: seeing determined by the observatory
  - Wind Speed vector: anamometer data from site weather stations


.. note::
  Not clear if/when focus and guider information is available; should add to ICD if required





References
==========

.. target-notes::

.. _`LDM-151`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LDM-151
.. _`LSE-72`: https://docushare.lsst.org/docushare/dsweb/Get/LSE-72
.. _`LSE-70`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LSE-70
.. _`LDM-148`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LDM-148
.. _`LSE-61`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LSE-61
.. _`DMTN-050`: https://dmtn-050.lsst.io
.. _`Dark Energy Survey Night Summary`: http://des-ops.fnal.gov:8080/nightsum
.. _`SQR-019`: https://sqr-019.lsst... important::
.. _`LPM-17`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LPM-17
