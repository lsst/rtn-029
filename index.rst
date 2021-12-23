..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
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

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is not yet published.**

   We describe the procedure we followed at FrDF for creating a butler repository from scratch for performing data release processing for the needs of Data Preview 0.2. We include the butler commands used, the scripts which use the butler's Python API as well as the details on the input datasets used for creating the repository.

Introduction
============

In this note we document the required input datasets and the procedure we followed at the Rubin French Data Facility (FrDF) for creating a butler repository for performing data release processing for the needs of Data Preview 0.2 :cite:`RTN-001`. We include the butler commands we used, the scripts which use the butler's Python API as well as the details on the datasets used for populating the repository.

The details for preparing this note are extracted from `PREOPS-711 <https://jira.lsstcorp.org/browse/PREOPS-711>`__.

Input Datasets
==============

For creating this repository we used four kinds of datasets:

- raw images
- calibrations
- reference catalogs
- skyMap

In the subsections below we document the source of each of those datasets and the transformations we applied to them specifically for creating the repository, if any.

Raw images
----------

For Data Preview 0.2 we use a subset of the simulated raw images produced by the Dark Energy Science Collaboration (DESC) Data Challenge 2 :cite:`2021ApJS..253...31L`. Specifically, we use the subset known as WFD, composed of 19,852 visits, one exposure per visit. The visit identifiers of were obtained from `DR6_Run2.2i_WFD_visits.txt <https://github.com/lsst-dm/gen3_shared_repo_admin/blob/master/python/lsst/gen3_shared_repo_admin/data/dc2/DR6_Run2.2i_WFD_visits.txt>`__.

The simulated images are stored at FrDF under path

.. code-block:: none
   
   /sps/lsst/datasets/rubin/previews/dp0.2/raw

and are organized by year subdirectory (named ``y1-wfd`` to ``y5-wfd``), as the original DESC DC2 simulated raw dataset. Within a given year there is a subdirectory per visit, which contains all the files beloging to that visit, typically one ``.fits`` file per sensor, e.g:

.. code-block:: bash

    $ tree -L 1 /sps/lsst/datasets/rubin/previews/dp0.2/raw/y1-wfd/00261426
    /sps/lsst/datasets/rubin/previews/dp0.2/raw/y1-wfd/00261426
    |-- _index.json
    |-- lsst_a_261426_R03_S02_r.fits
    |-- lsst_a_261426_R03_S11_r.fits
    |-- lsst_a_261426_R03_S12_r.fits
    ...

There is one file named ``_index.json`` associated to each visit. Each of those index files contains metadata extracted from all the sensor files of the same visit. When ingesting the raw images, the ``butler ingest-raws`` command looks for index files and if present extracts from them the relevant information to populate the registry database without actually reading the ``.fits`` files themselves, which can help to speed up (re)ingestion of raw images.

Those index files can be generated via the `astro_metadata_translator <https://astro-metadata-translator.lsst.io>`__ package, which is included in the LSST Science Pipelines distribution. We used weekly **w_2021_42** for generating those indexes, via a command similar to:

.. code-block:: bash

    $ astrometadata --packages lsst.obs.lsst write-index --content metadata /sps/lsst/datasets/rubin/previews/dp0.2/raw

The generated index files can be reused at any facility ingesting the same raw images. You can download them at https://me.lsst.eu/lsstdata/dp02_raw_wfd_indexes.tar.gz (1.5 GB)

Calibration data
----------------

Calibration data was extracted from an existing butler repository located under ``/repo/dc2`` at NCSA, using the command below (weekly **w_2021_42**):

.. code-block:: bash

    # Executed at NCSA
    $ butler export-calibs /repo/dc2 gen3-repo-calibs 2.2i/calib

Note however that for this command to work the `default datastore template <https://github.com/lsst/daf_butler/blob/ac63b1862508ff15b39a6f6be096f4af46b21807/python/lsst/daf/butler/configs/datastores/fileDatastore.yaml#L8>`__ was modified to replace ``detector.full_name`` by ``detector``. 

The resulting calibration data is organized at FrDF as follows:

.. code-block:: bash
   
    $ tree -L 5 -F /sps/lsst/datasets/rubin/previews/dp0.2/calib
    /sps/lsst/datasets/rubin/previews/dp0.2/calib
    ├── 2.2i/
    │   └── calib/
    │       ├── DM-30694/
    │       │   ├── curated/
    │       │   │   └── 19700101T000000Z/
    │       │   └── unbounded/
    │       │       └── camera/
    │       └── gen2/
    │           ├── 20220101T000000Z/
    │           │   ├── bias/
    │           │   └── dark/
    │           ├── 20220806T000000Z/
    │           │   └── flat/
    │           └── 20231201T000000Z/
    │               └── sky/
    └── export.yaml

An archive of the calibration data is available at https://me.lsst.eu/lsstdata/dp02_calib.tar.gz (136 GB)

Reference catalogs
------------------

For DP0.2, we use same reference catalogs that were used for processing the DESC DC2 data. Those catalogs are located at FrDF and organized as follows

.. code-block:: none
   
    $ tree -L 1 /sps/lsst/datasets/desc/DC2/reference_catalogs/Run2.2i/cal_ref_cat
    /sps/lsst/datasets/desc/DC2/reference_catalogs/Run2.2i/cal_ref_cat
    |-- 141440.fits
    |-- 141443.fits
    |-- 141825.fits
    ...

There are 1,213 ``.fits`` files which we copied under 

.. code-block:: none

    /sps/lsst/datasets/rubin/previews/dp0.2/refcats/cal_ref_cat

The reference catalogs data is organized at FrDF as follows:

.. code-block:: bash

    $ tree -L 1 -F /sps/lsst/datasets/rubin/previews/dp0.2/refcats
    /sps/lsst/datasets/rubin/previews/dp0.2/refcats
    ├── cal_ref_cat/
    └── refcat.ecsv

An archive file containing the ``.fits`` files are available at https://me.lsst.eu/lsstdata/dp02_refcat.tar.gz (1.8 GB).

skyMap
------

The sky map configuration file was copied unmodified from `DC2.py <https://github.com/lsst-dm/gen3_shared_repo_admin/blob/master/python/lsst/gen3_shared_repo_admin/config/skymaps/DC2.py>`__ and stored under:

.. code-block:: bash

    $ tree -F /sps/lsst/datasets/rubin/previews/dp0.2/skymaps
    /sps/lsst/datasets/rubin/previews/dp0.2/skymaps
    └── DC2.py


Input datasets layout
---------------------

The four datasets prepared in the previous steps are organized as follows:

.. code-block:: bash

    $ tree -L 1 -F /sps/lsst/datasets/rubin/previews/dp0.2
    /sps/lsst/datasets/rubin/previews/dp0.2
    ├── calib/
    ├── raw/
    ├── refcats/
    └── skymaps/

Creating the repository
=======================

In this section we present the step-by-step procedure we use for creating the repository using release **v23.0.0**.

For conciseness, hereafter we refer to the location of the repository the via the environment variable ``$REPO``. In addition, we use some environment variables which have the values shown below:

.. prompt:: bash

    export DP02_TOP='/sps/lsst/datasets/rubin/previews/dp0.2'
    export DP02_CALIB="$DP02_TOP/calib"
    export DP02_RAW="$DP02_TOP/raw"
    export DP02_REFCATS="$DP02_TOP/refcats"
    export DP02_SKYMAP="$DP02_TOP/skymaps"


Create an empty repository
--------------------------

.. prompt:: bash

    butler create --seed-config butler-dp02.yaml --override $REPO

To configure the butler to use a file-based data store and a PostgreSQL registry database we use a seed configuration file ``butler-dp02.yaml``  similar to:

.. code-block:: bash

    $ cat butler-dp02.yaml
    datastore:
      cls: lsst.daf.butler.datastores.fileDatastore.FileDatastore
    registry:
      db: postgresql://user@host:1234/databasename

Import calibration data
-----------------------

.. prompt:: bash

    butler import --export-file "$DP02_CALIB/export.yaml" $REPO $DP02_CALIB

For this repository, importing calibration data needs to be performed before registering instruments (see below), otherwise an error is produced.

Register instrument
-------------------

To register the instrument for this repository we use:

.. prompt:: bash

    butler register-instrument $REPO lsst.obs.lsst.LsstCamImSim

.. todo::
  
    Do we also need to register instrument ``lsst.obs.lsst.LsstCamPhoSim`` ? If we don't register it and we import calibration data *after* registering only ``lsst.obs.lsst.LsstCamImSim`` we get the error below:

    .. code-block:: bash

       sqlalchemy.exc.IntegrityError: (sqlite3.IntegrityError) UNIQUE constraint failed: instrument.name
       [SQL: INSERT INTO instrument (name, visit_max, exposure_max, detector_max, class_name) VALUES (?, ?, ?, ?, ?)]
       [parameters: ('LSSTCam-imSim', 9999999, 9999999, 1000, 'lsst.obs.lsst.LsstCamImSim')]
       (Background on this error at: https://sqlalche.me/e/14/gkpj)

Add instrument's curated calibrations
-------------------------------------

.. prompt:: bash

    butler write-curated-calibrations $REPO 'LSSTCam-imSim'

Register sky map
----------------

.. prompt:: bash

    butler register-skymap -C "$DP02_SKYMAP/DC2.py" $REPO

Ingest reference catalog data
-----------------------------

Ingestion of reference catalogs requires that an `Astropy table <https://docs.astropy.org/en/stable/api/astropy.table.Table.html>`__ associating each file path of the reference catalog and its dimension be provided. We use the script below to create that table and store it in file ``refcat.ecsv``.

.. code-block:: python

    import os
    import re
    from astropy.table import Table
         
    refcat_dir = '/sps/lsst/datasets/rubin/previews/dp0.2/refcats/cal_ref_cat'

    pattern = re.compile(r'[0-9]{6}.fits')
    rows = []
    for file in os.listdir(refcat_dir):
       if pattern.match(file):
          filename = os.path.splitext(file)[0]
          filepath = os.path.join(refcat_dir, file)
          rows.append( (filepath, int(filename)) )
        
    table = Table(rows=rows, names=['filename', 'htm7'])
    table.write('refcat.ecsv')

An excerpt of the contents of the generated table file is shown below:

.. code-block:: none

    $ head -10 refcat.ecsv 
    # %ECSV 1.0
    # ---
    # datatype:
    # - {name: filename, datatype: string}
    # - {name: htm7, datatype: int64}
    # schema: astropy-2.0
    filename htm7
    /sps/lsst/datasets/rubin/previews/dp0.2/refcats/cal_ref_cat/146812.fits 146812
    /sps/lsst/datasets/rubin/previews/dp0.2/refcats/cal_ref_cat/141991.fits 141991
    /sps/lsst/datasets/rubin/previews/dp0.2/refcats/cal_ref_cat/146919.fits 146919

The generated table file is available at https://me.lsst.eu/lsstdata/dp02_refcat.ecsv.tar.gz (7.8 KB).

Register and ingest reference catalogs data:

.. code-block:: bash

    # Register reference catalog data with dataset type 'cal_ref_cat_2_2',
    # storage class 'SimpleCatalog' and dimensions 'htm7'
    $ butler register-dataset-type $REPO cal_ref_cat_2_2 SimpleCatalog htm7

    # Ingest dataset of type 'cal_ref_cat_2_2' into run 'refcats' using information
    # (e.g. paths, dimensions) present in table 'refcat.ecsv'
    $ butler ingest-files --transfer direct $REPO cal_ref_cat_2_2 refcats refcat.ecsv

Ingest raw exposures
--------------------

.. prompt:: bash
    
    butler ingest-raws --transfer direct $REPO $DP0_RAW/y{1..5}-wfd

Note that there are many ways to perform the ingestion of raws concurrently, for instance launching an ingestion command per year and by specifying the number of processes to use for each command, such as:

.. prompt:: bash
    
    butler ingest-raws --transfer direct -j 16 $REPO $DP0_RAW/y1-wfd

At FrDF we use ingestion in place via the option ``--transfer direct`` to avoid copying raw exposure data to the repository location.

Define visits
-------------

Define visits from the exposures already present in the repository in collection ``LSSTCam-imSim/raw/all`` for instrument ``LSSTCam-imSim``:

.. prompt:: bash
    
    butler define-visits --collections 'LSSTCam-imSim/raw/all' $REPO 'LSSTCam-imSim'

Creating collections
--------------------

We create a chained collection with the Python code below:

.. code-block:: python

    #!/usr/bin/env python
    import os
    import sys
    from lsst.daf.butler import Butler, CollectionType

    repo = os.getenv('REPO')
    if len(sys.argv) > 1:
        repo = sys.argv[1]

    parent='2.2i/defaults'
    children = ['LSSTCam-imSim/raw/all', '2.2i/calib', 'skymaps', 'refcats']

    butler = Butler(repo, writeable='True')
    butler.registry.registerCollection(name=parent, type=CollectionType.CHAINED)
    butler.registry.setCollectionChain(parent=parent, children=children)

.. todo::
  
    Add a sentence on why creating this chain is needed / desirable / convenient


.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
   :style: lsst_aa
