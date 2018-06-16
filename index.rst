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

The rapid cadence and scale of the LSST observing program requires that each Visit is scheduled shortly before it is observed to make optimal use of conditions that vary with time like observing conditions, observatory and telescode parameters, survey completion, etc. That requires automated feedback from the *Data Management System (DMS)* to the *Observatory Control System (OCS)* to predict the next observation, and to determine if a given Visit needs be scheduled for re-observation. It also requires automated but human-readable feedback to monitor data quality, in the form of near-realtime dashboards for observers and end-of-night reports for DM staff.

During observations, the *Prompt Processing* (see section 3 in `LDM-151`_) will perform a Single Frame Processing in near-realtime, from which data quality metrics can be derived for each CCD and Visit.

From the *Data Management – OCS Software Communication Interface* **OCS-DM-COM-ICD-0017** (see section 3.1 in `LSE-72`_ ) the DMS shall use the OCS Service Abstraction Layer (SAL) to communicate with the OCS. This communication interface is defined in the *System Communication Protocol Interface* `LSE-70`_ which contains the requirements to *publish* data quality metrics to OCS as telemetry and also to *subscribe* to telemetry events from the OCS.

.. note::
  I don't understand SAL and DDS yet, can this be configured between NCSA and the Summit?  not sure about OCS-DMS telemetry, is this equivalent to querying the EFD (in LSE-72 there is specification for a query interface to EFD) or will DMS subscribe to OCS events?. My understanding is that for near-realtime telemetry we cannot use the T-EFD.


In particular, **OCS-DM-COM-ICD-0019** (see section 3.1.2 in `LSE-72`_) will provide a complete list of events and telemetry required by the OCS from the DMS.

From **OCS-DM-COM-ICD-0021** (see section 3.1.2.2 in `LSE-72`_) the data quality metrics will be used by the OCS to determine if the observation meets the *scheduler criteria*. In this sense, the imediate feedback from the DMS to the OCS is important as it impacts the short term scheduling.

In addition to the the data quality metrics, the DMS should be able to compute and store a passed/failed data quality flag to determine if a given Visit needs to be scheduled for re-observation.

.. note::
  `LSE-61` does not have an explicit requirement for that.  Which data quality critera will be used to define the passed/failed flag? Other Surveys like DES use a combination of the sky brightness estimates, PSF FWHM and atmosperic transparency as a single data quality metric called *effective exposure time* that is used for this purpose.

While the details of the OCS-DMS interface are not completely defined, we assume that data quality metrics for each CCD and Visit will be persisted in the "OCS database" (see section 3.1.2 in `LDM-151`_). We call that the Prompt Quality Control (QC) database, from which telemetry information to OCS can be generated.

A component like the Prompt QC database is also required for generating a human-readable near-realtime dashboard and a Data Quality Report.


The minimum content for the Prompt Data Quality report is specified in  **DMS-REQ-0097** (see section 1.3.14 in `LSE-61`_).

**Specification:** *The DMS shall produce a Level 1 Data Quality Report that contains indicators of data quality that result from running the DMS pipelines, including at least: Photometric zero point vs. time for each utilized filter; Sky brightness vs. time for each utilized filter; seeing vs. time for each utilized filter; PSF parameters vs. time for each utilized filter; detection efficiency for point sources vs. mag for each utilized filter.*

**Discussion:** *The seeing report is intended as a broad-brush measure of image quality. The PSF parameters provide more detail, as they include asymmetries and field location dependence.*

Also from **DMS-REQ-0096** (see section 2.2.10 in `LSE-61`_) the Prompt Data Quality Report must be generated in less than 4h after the end of the nigthly Prompt Processing in order to evaluate *whether changes to hardware, software, or procedures are needed for the following night’s observing.*

.. note::
  Should we consider DMS-REQ-0099 - Level 1 Performance Report Definition and DMS-REQ-0101 - Level 1 Calibration Report Definition here?

The data quality metrics specified in **DMS-REQ-0097** can be derived from the *Single Frame processing* alone. However, a richer report will need to correlate those metrics with the state of the instrument, observing conditions, observatory parameters obtained from the Transformed EFD (see `DMTN-050`_). Also, we'll need to correlate that information with observer comments obtained from the observatory electronic log system.

As a reference, see the `Dark Energy Survey Night Summary`_ report.

In this technote we evaluate existing DM tools, and discuss the the need of developing new tools for:

  - Collecting the data quality metrics from the *Single Frame Processing* pipeline;
  - Storing the data quality metrics in the Prompt QC database;
  - Publishing data quality metrics to the OCS;
  - Monitoring data quality metrics in near-realtime dashboards;
  - Accessing the Prompt QC and EFD databases;
  - Implementing and running the Prompt Data Quality Report;
  - Publishing the Prompt Data Quality Reports.

Figure 1 presents an overview diagram of the Prompt QC components described in more detail in the next sections.

.. figure:: /_static/qc_components.png
  :name: Prompt Quality Control components.

In the Appendix A we list the data quality metrics and parameters used by the Prompt QC software.



Collecting data quality metrics
===============================

The Single Frame Processing Pipeline is responsible for reducing raw
or camera-corrected image data to calibrated exposures, the detection and measurement of
Sources, the characterization of the point-spread-function (PSF), and the generation of an astrometric solution for an image.

Quality metrics are collected instrumenting these tasks with the LSST Verification framework or `lsst.verify` (see `SQR-019`_).

Currently `lsst.verify` is being used to define and collect Key Performance Metrics derived from the LSST Science Requirements Document (`LPM-17`_).


Storing data quality metrics
============================

Metric values computed by pipeline tasks and collected by `lsst.verify` will be uploaded to the Prompt QC database through a RESTful API.


Publishing data quality metrics to the OCS
==========================================

Data quality metrics will be published to the OCS through telemery events using SAL.

.. note::
  While LSE-72 proposes SAL for the DMS-OCS interface, I don't know if that is possible between NCSA and the Summit.  See also LDM-148 about the "Telemetry Gateway" which proposes that RabbitMQ is used to transfer status and quality metrics to the gateway over the international network.

Monitoring quality metrics in near-realtime dashboards
======================================================

SQuaSH like dashboards can be used to visualize the data quality metrics enabling *remote observing* in near-realtime.

Prompt Data Quality Reports
===========================

Acessing the QC and EFD databases
---------------------------------
Correlating data quality metrics with observing conditions, with the state of the instrument, observing conditions, observatory parameters etc.

Implementation
--------------
The Data Quality Report will be implemented as jinja templates that will produce rst. It does not require much processing since the information is pre-computed and accessible from the Prompt QC REST API and the EFD.

Publication
-----------
The publication is done automatically using the LTD infrastructure to generate static pages from the rst documents generated by the jinja templates.

Appendix A - Data quality metrics and parameters
================================================

Table 1 - Data quality metrics measured, collected and stored by the Prompt Quality Control software.

+---------------------+--------------+-----+
| Data quality metric |  Description | Use |
+---------------------+--------------+-----+

Table 2 - Data quality parameters obtained from the EFD.

+------------------------+-------------+-----+
| Data quality parameter | Description | Use |
+------------------------+-------------+-----+


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
