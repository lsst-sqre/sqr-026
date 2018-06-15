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

The rapid cadence and scale of the LSST observing program requires that each Visit is scheduled shortly before it is observed to make optimal use of conditions that vary with time like observing conditions, observatory and telescode parameters, survey completion, etc. That requires 1) automated feedback from the *Data Management System (DMS)* to the *Observatory Control System (OCS)* Scheduler. Also, observers need automated but human-readable feedback about data quality, in the form of 2) near-realtime data quality dashboards and 3) end-of-night data quality reports.

During observations, the *Prompt Processing* (see section 3 in `LDM-151`_ ) will perform a Single Frame Processing in near-realtime, from which data quality metrics can be derived for each CCD and Visit.

From the *Data Management – OCS Software Communication Interface* **OCS-DM-COM-ICD-0017** (see section 3.1 in `LSE-72`_ ) the DMS shall use the OCS Service Abstraction Layer (SAL) to communicate with the OCS. This communication interface is defined in the *System Communication Protocol Interface* `LSE-70`_ which contains the requirements to *publish* data quality metrics to OCS as telemetry and also to *subscribe* to telemetry events from the OCS.

.. note::
  I don't understand SAL and DDS yet, can this be configure between NCSA and the Summit? perhaps a RESTful API would be enough for the DMS to OCS  communication, not sure about OCS to DMS telemetry, is this equivalent to querying the EFD (in LSE-72 there is specification for a query interface to EFD) I don't think the TEFD can replace near-realtime telemery from the OCS if that's required.

In particular, **OCS-DM-COM-ICD-0019** (see section 3.1.2 in `LSE-72`_) will provide a complete list of events and telemetry required by the OCS from the DMS.

From **OCS-DM-COM-ICD-0021** (see section 3.1.2.2 in `LSE-72`_) the data quality metrics will be used by the OCS to determine if the observation meets the *scheduler criteria*. In this sense, the imediate feedback from the DMS to the OCS is important as it impacts the short term scheduling.

In addition to the the data quality metrics, the DMS should be able to compute and store a passed/failed data quality flag to determine if a given Visit needs to be repeated.

.. note::
  `LSE-61` does not have an explicit requirement for that. How DMS will notify OCS that a given Visit needs to be repeated? Which data quality critera will be used to define the passed/failed flag? Other Surveys like DES use a combination of the sky brightness estimates, PSF FWHM and atmosperic transparency as a single data quality metric called *effective exposure time* that is used for this purpose.

While the details of the OCS-DMS interface are not completely defined, we assume that data quality metrics for each CCD and Visit will be persisted in the "OCS database" (see section 3.1.2 in `LDM-151`_). We'll call that the Prompt Quality Control (QC) database, from which telemetry information to OCS could be generated.

A component like the Prompt QC database is also required for generating of human-readable and possibly machine-readable Prompt Data Quality Reports. The minimum content for this report is specified in  **DMS-REQ-0097** (see section 1.3.14 in `LSE-61`_).

**Specification:** *The DMS shall produce a Level 1 Data Quality Report that contains indicators of
data quality that result from running the DMS pipelines, including at least: Photometric zero
point vs. time for each utilized filter; Sky brightness vs. time for each utilized filter; seeing vs.
time for each utilized filter; PSF parameters vs. time for each utilized filter; detection efficiency
for point sources vs. mag for each utilized filter.*

**Discussion:** *The seeing report is intended as a broad-brush measure of image quality. The
PSF parameters provide more detail, as they include asymmetries and field location dependence.*

Also from **DMS-REQ-0096** (see section 2.2.10 in `LSE-61`_) the Prompt Data Quality Report must be generated in less than 4h after the end of the nigthly Prompt Processing in order to evaluate *whether changes to hardware, software, or procedures are needed for the following night’s observing.*

.. note::
  Should we consider DMS-REQ-0099 - Level 1 Performance Report Definition and DMS-REQ-0101 - Level 1 Calibration Report Definition here?

The data quality metrics specified in **DMS-REQ-0097** can be derived from the *Single Frame processing* alone. However, a richer report will need to correlate those metrics with the state of the instrument, observing conditions, observatory parameters etc. obtained from the Transformed EFD, see `DMTN-050`_ as well as with observer comments obtained from the observatory electronic log system.

As a reference, see the `Dark Energy Survey Night Summary`_ report.

In this technote we evaluate existing DM tools, and discuss the the need of developing new tools for:

  - Collecting the data quality metrics from the *Single Frame Processing* pipeline;
  - Storing the data quality metrics in the Prompt QC database;
  - Publishing data quality metrics to the OCS;
  - Monitoring data quality metrics in near-realtime dashboards;
  - Accessing the Prompt QC and EFD databases;
  - Implementing the Prompt Data Quality Report;
  - Executing and publishing static versions of the Prompt Data Quality Report.

Figure 1 presents an overview diagram of the Prompt QC components described in more detail in the next sections.

.. figure:: /_static/qc_components.png
  :name: Prompt Quality Control components.

  Prompt processing > Prompt QC database > OCS Telemetry > LSP Notebook > Executing the Data Quality Report > Publishing Data Quality Report.

Collecting data quality metrics
===============================

The Single Frame processing pipeline consists in Instrument Signature Removal (ISR), PSF and Backgorund Characterization and Photometric and WCS Calibration tasks.

Quality metrics are collected instrumenting these tasks with the LSST Verification framework (see `SQR-019`_).

The LSST verification framework `lsst.verify` is being used to define and collect Key Performance Metrics derived from the LSST Science Requirements Document (`LPM-17`_).


Storing data quality metrics
============================

Metric values computed by pipeline tasks and collected by `lsst.verify` are currently uploaded to the SQuaSH database using a RESTful API.

The current infrastructure is generic and we recommend the SQuaSH database and its RESTful API for storing and publishing data quality metrics.

Publishing data quality metrics to the OCS
==========================================

Data quality metrics metrics can be published to the OCS through the REST API instead of using SAL and DDS.

Monitoring quality metrics in near-realtime dashboards
======================================================

SQuaSH like dashboards can be used to visualize the data quality metrics enabling *remote observing* in near-realtime.


Prompt Data Quality Reports
===========================

Acessing the QC and EDF databases
---------------------------------
Correlating data quality metrics with observing conditions, with the state of the instrument, observing conditions, observatory parameters etc.


Implementation
--------------
Reports are implemented as Jupyter notebooks using the LSP and accessing the Prompt QC REST API.

Execution
---------
The notebook is executed automatically as a kubernetes job.

Publication
-----------
The publications is done automatically using the LTD infrastructure to generate static pages from the Jupyter notebook outputs.

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



..

  .. figure:: /_static/filename.ext
    :name: fig-label

    Caption text.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa
