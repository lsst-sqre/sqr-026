.. image:: https://img.shields.io/badge/sqr--026-lsst.io-brightgreen.svg
   :target: https://sqr-026.lsst.io
.. image:: https://travis-ci.org/lsst-sqre/sqr-026.svg
   :target: https://travis-ci.org/lsst-sqre/sqr-026
..
  Uncomment this section and modify the DOI strings to include a Zenodo DOI badge in the README
  .. image:: https://zenodo.org/badge/doi/10.5281/zenodo.#####.svg
     :target: http://dx.doi.org/10.5281/zenodo.#####

#################################################################
Periodic report generation and publication via notebook templates
#################################################################

SQR-026
=======

The purpose of this technote is to describe the software components that will produce the Data Quality report for the Prompt Processing Pipeline.

**Links:**

- Publication URL: https://sqr-026.lsst.io
- Alternative editions: https://sqr-026.lsst.io/v
- GitHub repository: https://github.com/lsst-sqre/sqr-026
- Build system: https://travis-ci.org/lsst-sqre/sqr-026


Build this technical note
=========================

You can clone this repository and build the technote locally with `Sphinx`_:

.. code-block:: bash

   git clone https://github.com/lsst-sqre/sqr-026
   cd sqr-026
   pip install -r requirements.txt
   make html

.. note::

   In a Conda_ environment, ``pip install -r requirements.txt`` doesn't work as expected.
   Instead, ``pip`` install the packages listed in ``requirements.txt`` individually.

The built technote is located at ``_build/html/index.html``.

Editing this technical note
===========================

You can edit the ``index.rst`` file, which is a reStructuredText document.
The `DM reStructuredText Style Guide`_ is a good resource for how we write reStructuredText.

Remember that images and other types of assets should be stored in the ``_static/`` directory of this repository.
See ``_static/README.rst`` for more information.

The published technote at https://sqr-026.lsst.io will be automatically rebuilt whenever you push your changes to the ``master`` branch on `GitHub <https://github.com/lsst-sqre/sqr-026>`_.

Updating metadata
=================

This technote's metadata is maintained in ``metadata.yaml``.
In this metadata you can edit the technote's title, authors, publication date, etc..
``metadata.yaml`` is self-documenting with inline comments.

Using the bibliographies
========================

The bibliography files in ``lsstbib/`` are copies from `lsst-texmf`_.
You can update them to the current `lsst-texmf`_ versions with::

   make refresh-bib

Add new bibliography items to the ``local.bib`` file in the root directory (and later add them to `lsst-texmf`_).

****

Copyright 2018 AURA/LSST

This work is licensed under the Creative Commons Attribution 4.0 International License. To view a copy of this license, visit http://creativecommons.org/licenses/by/4.0/.

.. _Sphinx: http://sphinx-doc.org
.. _DM reStructuredText Style Guide: https://developer.lsst.io/docs/rst_styleguide.html
.. _this repo: ./index.rst
.. _Conda: http://conda.pydata.org/docs/
.. _lsst-texmf: https://lsst-texmf.lsst.io
