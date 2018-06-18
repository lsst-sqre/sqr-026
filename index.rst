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

In this technote we describe how LSST DM/SQuaRE infrastructure will be used to generate the Data Management System (DMS) end-of-night report as specified in the `LSE-61`_ (DMS-REQ-0097) and discuss the adoption of this toolchain as the common reporting infrastructure to be shared accross the LSST subsystems, as requested in `LCR-1203`_.

**DMS-REQ-0097**  specifies the minimum content for the DMS end-of-night report:

**Specification:** *The DMS shall produce a Level 1 Data Quality Report that contains indicators of data quality that result from running the DMS pipelines, including at least: Photometric zero point vs. time for each utilized filter; Sky brightness vs. time for each utilized filter; seeing vs. time for each utilized filter; PSF parameters vs. time for each utilized filter; detection efficiency for point sources vs. mag for each utilized filter.*

Also, from **DMS-REQ-0096** the Prompt Data Quality Report must be generated in less than 4h after the end of the nigthly Prompt Processing in order to evaluate *whether changes to hardware, software, or procedures are needed for the following night’s observing.*

During observations, the *Prompt Processing* pipeline (see section 3 in `LDM-151`_) will perform Single Frame Processing from which the data quality metrics specified in **DMS-REQ-0097** can be derived for each CCD and Visit. However, a more complete end-of-night report will require correlation of those metrics with the state of the instrument, observing conditions, observatory parameters obtained from the DM-EFD (see `DMTN-050`_), and probably with observer comments obtained from the observatory electronic log system. As a reference, see the `Dark Energy Survey Night Summary`_ reports.

Figure 1 presents an overview diagram of the DMS end-of-night report components, mapped to existing tools developed by LSST DM/SQuaRE.

.. figure:: /_static/report_components.png
  :name: DMS end-of-night report components.

The LSST DM/SQuaRE toolchain is described in more details in the following sections. In Appendix A we present an initial list of the data quality metrics and parameters needed by the DMS end-of-night report.

Collecting data quality metrics with `lsst.verify`
==================================================

The Single Frame Processing pipeline is responsible for the reduction of raw or camera-corrected image data to calibrated exposures, the detection and measurement of Sources, the characterization of the point-spread-function (PSF), and the generation of an astrometric solution for an image.

Data quality metrics can be collected during the processing of each CCD in a visit (eg. zeropoints) or by an afterburner task. Normal processing tasks or QA-specific afterburner tasks can be instrumented by the LSST Verification Framework used to define and collect measurements of those metrics as demonstrated in `SQR-019`_.

Currently `lsst.verify` is being used to measure Key Performance Metrics derived from the LSST Science Requirements Document (`LPM-17`_) but it is generic and will be used to publish ad hoc metrics.

Metrics measured through `lsst.verify` can be uploaded to the LSST SQuaSH infrastructure for curation and display.

Storing data quality metrics with SQuaSH
========================================

`SQuaSH`_ is a metric store and monitoring system. Metrics are uploaded to SQuaSH using the `lsst.verify` framework. These metrics and their measured values are stored and tracked, making it possible to monitor regressions via a dahsboard interface or through alerts, including Slack notifications.

SQuaSH has a microservice architecture that makes it easier to reuse some of its components for the DMS end-of-night report. Here we decribe the use of the SQuaSH database for storing the data quality metrics and its RESTful API as a programatic interface to access the results.

The `Bokeh`_ plotting library is the data visualization technology used in `SQuaSH`_. We plan to use it for the end-of-night reports for creating rich and interactive visualizations as it integrates very well with the notebook environment and with the publication infrastructure as described below. See also `SQR-022`_ for an example of a custom chart implemented in Bokeh.

Implementing the end-of-night reports as Jupyter notebooks
==========================================================

We intend to implement the end-of-night reports as Jupyter notebooks using the notebook aspect of the `LSST Science Platform`_. The notebook will contain the code necessary to access the results from programatic APIs, the visualization code and the text narrative in a single place.

The notebooks can be executed on demand or on a schedule (e.g. via a cronjob) and the result will be published as a static page using the `LSST the Docs`_, the same infrastructure used to publish this technote. In `SQR-019`_, we demonstrate how a notebook can be used as a source to a published document.

This will allow users to consume it as a document report if they do not need the interactive analytical capabilities offered by SQuaSH and the notebook framework.

Correlating with data in the DM-EFD
===================================

The DMS end-of-night report will need access to the DM-EFD to correlate the data quality metrics with EFD-derived data such as observing conditions, state of the instrument, observatory parameters etc.

The DM-EFD will have a programatic interface through the DAX system.  These interfaces will be accessible from the LSP environment.

Conclusions
===========

This technote summarizes LSST DM/SQuaRE’s toolchain and demonstrates its potential use to generate the DMS end-of-night report in particular, and more generally as a common reporting infrastructure to be shared accross the LSST subsystems. We describe the current tools showing examples of their use and integration. We suggest using `lsst.verify` to define and collect measurements of ad hoc metrics; SQuaSH as the metric store; the notebook aspect of the LSP as the development environment for implementing the reports powered by programatic APIs for accessing SQuaSH and the DM-EFD databases; and finally using the LSST the Docs infrastructure to produce the static reports.

Appendix A - DMS end-of-night report: data quality metrics and parameters to store
==================================================================================

DMS-REQ-0097 makes reference to data quality metrics from the Single Frame Processing pipeline for each individual CCD in a Visit and for each filter. Calculating these values is the responsibility of the Science Pipelines team. Using the SQuaSH metric store, these metrics can also be aggregated at the Visit level.

  - PSF FWHM
  - PSF Ellipticiy
  - Sky brightness
  - Zeropoint

A number of relevant data quality parameters can be obtained from the DM-EFD:

  - Start time (UTC): date and time in UTC when the Visit acquisition started.
  - Visit ID: unique identifier of the Visit.
  - RA, Dec: Telescope pointing.
  - Airmass: Even though it can be calculated from RA, Dec it is useful to store Airmass since it determines the expected atmospheric contribution to the image quality.
  - Filter: One of the five LSST observing filters, ugrizy.
  - Focus: The donut estimate of focus error for the Visit.
  - Guider DeltaRA, DeltaDec: Guider displacements for TCS correction
  - DIMM seeing: seeing determined by the observatory
  - Wind vector: anamometer data from site weather stations


References
==========

.. target-notes::

.. _`LSE-61`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LSE-61
.. _`LCR-1203`: https://project.lsst.org/groups/ccb/node/2174
.. _`LDM-151`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LDM-151
.. _`DMTN-050`: https://dmtn-050.lsst.io
.. _`Dark Energy Survey Night Summary`: http://des-ops.fnal.gov:8080/nightsum
.. _`SQR-019`: https://sqr-019.lsst.io
.. _`LPM-17`: https://docushare.lsstcorp.org/docushare/dsweb/Get/LPM-17
.. _`SQuaSH`: https://squash.lsst.codes/
.. _`Bokeh`: https://bokeh.pydata.org/en/latest/
.. _`SQR-022`: https://sqr-022.lsst.codes/
.. _`LSST Science Platform`: https://nb.lsst.io/
.. _`LSST the Docs`: https://sqr-006.lsst.io/
.. _`SQR-009`: https://sqr-009.lsst.io
..
