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

Introduction
============

In this note we document the required input datasets and the procedure we followed at the Rubin French Data Facility (FrDF) for creating and populating a butler repository for the needs of image processing for preparing Data Preview 0.2 :cite:`RTN-001`.

We include the command line tools and scripts we used as well as the details on the datasets used for populating the repository.

The details for preparing this note were extracted from `PREOPS-711 <https://jira.lsstcorp.org/browse/PREOPS-711>`__.

How to give feedback
--------------------

If you notice errors in this document or want to help improve it, please feel free to `create an issue <https://github.com/lsst/rtn-029/issues>`__.

Input Datasets
==============

For populating this repository we used the dataset types listed below:

- raw images
- calibrations
- reference catalogs
- skymap
- truth summary

In the following subsections we document the source of each of those dataset types and the transformations we applied for ingesting them into the repository.

Raw images
----------

For Data Preview 0.2 we use a subset of the simulated raw images produced by the Dark Energy Science Collaboration (DESC) Data Challenge 2 :cite:`2021ApJS..253...31L`. Specifically, we use the subset known as WFD, composed of 19,852 visits, one exposure per visit. The visit identifiers were obtained from `DR6_Run2.2i_WFD_visits.txt <https://github.com/lsst-dm/gen3_shared_repo_admin/blob/master/python/lsst/gen3_shared_repo_admin/data/dc2/DR6_Run2.2i_WFD_visits.txt>`__.

The simulated images are stored at FrDF under path

.. code-block:: none
   
   /sps/lsst/datasets/rubin/previews/dp0.2/raw

and are organized by year subdirectory (named ``y1-wfd`` to ``y5-wfd``), as the original DESC dataset. Within a given year there is a subdirectory per visit, which contains all the files beloging to that visit, typically one ``.fits`` file per sensor. For instance, for visit with identifier 261426 we have:

.. code-block:: bash

    $ tree -L 1 /sps/lsst/datasets/rubin/previews/dp0.2/raw/y1-wfd/00261426
    /sps/lsst/datasets/rubin/previews/dp0.2/raw/y1-wfd/00261426
    |-- _index.json
    |-- lsst_a_261426_R03_S02_r.fits
    |-- lsst_a_261426_R03_S11_r.fits
    |-- lsst_a_261426_R03_S12_r.fits
    ...

There is one file named ``_index.json`` associated to each visit. Each of those index files contains metadata extracted from all the sensor files of the same visit. When ingesting the raw images, the ``butler ingest-raws`` command looks for index files and if present extracts from them the relevant information to populate the registry database without actually reading the ``.fits`` files themselves, which can help to speed up (re)ingestion of raw images (see :ref:`ingest-raw-exposures`).

Those index files can be generated via the `astro_metadata_translator <https://astro-metadata-translator.lsst.io>`__ package, which is included in the LSST Science Pipelines distribution. We used weekly **w_2021_42** for generating those indexes, via a command similar to:

.. code-block:: bash

    $ astrometadata --packages lsst.obs.lsst write-index --content metadata \
          /sps/lsst/datasets/rubin/previews/dp0.2/raw

The generated index files can be reused at any facility ingesting the same raw images. You can find them at

   https://me.lsst.eu/lsstdata/dp02_raw_wfd_indexes.tar.gz (1.5 GB)


Calibration data
----------------

Calibration data was extracted from an existing butler repository located at NCSA under ``/repo/dc2``, using the command below (weekly **w_2021_42**):

.. code-block:: bash

    # Export of calibration data executed at NCSA
    $ butler export-calibs /repo/dc2 gen3-repo-calibs 2.2i/calib

.. warning::

  For this command to work the `default datastore template <https://github.com/lsst/daf_butler/blob/ac63b1862508ff15b39a6f6be096f4af46b21807/python/lsst/daf/butler/configs/datastores/fileDatastore.yaml#L8>`__ was modified to replace ``detector.full_name`` by ``detector``. This export issue is being tracked via ticket `DM-32061 <https://jira.lsstcorp.org/browse/DM-32061>`__.

The resulting exported calibration data was transferred and stored FrDF as follows:

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

An archive of the calibration data is available at

   https://me.lsst.eu/lsstdata/dp02_calib.tar.gz (136 GB)

See :ref:`import-calibration-data` for details on how we imported this dataset into the repository.

Reference catalogs
------------------

For DP0.2 we use same reference catalogs that were used for processing the DESC DC2 data. Those catalogs are located at FrDF and organized as follows

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

Details on the contents and procedure to create the file ``refcat.ecsv`` are provided in :ref:`ingest-reference-catalog-data`. 

An archive file containing the ``.fits`` files are available at

   https://me.lsst.eu/lsstdata/dp02_refcat.tar.gz (1.8 GB).

SkyMap
------

The skymap configuration file was copied unmodified from `DC2.py <https://github.com/lsst-dm/gen3_shared_repo_admin/blob/master/python/lsst/gen3_shared_repo_admin/config/skymaps/DC2.py>`__ and stored under:

.. code-block:: bash

    $ tree -F /sps/lsst/datasets/rubin/previews/dp0.2/skymaps
    /sps/lsst/datasets/rubin/previews/dp0.2/skymaps
    └── DC2.py

See :ref:`register-sky-map` for details on how we set the repository to use this configuration.

Truth summary
-------------

The DESC DC2 truth summary tables need to be present in the butler repository so that they can be used in pipeline tasks. More context on the motivation to include this dataset into the repository can be found in issue `DM-32895 <https://jira.lsstcorp.org/browse/DM-32895>`_.

The 166 files (aggregated volume 63 GB) are located under 

.. code-block:: none

    /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_parquet

They are named like ``truth_patchint_tract<NNNN>.parq`` where ``<NNNN>`` is the identifier of a tract (e.g. ``truth_patchint_tract4643.parq``). Those files were imported unmodified to FrDF from NCSA. Their original location at NCSA was the directory

.. code-block:: bash

    # Location at NCSA of parquet files
    /project/shared/DC2/truth_parquet


The file 

.. code-block:: none

    /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_tables.csv
    
contains a table necessary for ingestion into a butler repository. An excerpt of its contents is shown below:

.. code-block:: bash

    $ head -5 /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_tables.csv 
    file URI,skymap,tract
    /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_parquet/truth_patchint_tract3449.parq,DC2,3449
    /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_parquet/truth_patchint_tract3832.parq,DC2,3832
    /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_parquet/truth_patchint_tract4433.parq,DC2,4433
    /sps/lsst/datasets/rubin/previews/dp0.2/truth_summary/truth_parquet/truth_patchint_tract3080.parq,DC2,3080

That file was adapted from the original at NCSA so that the file paths reflect the location of those files at FrDF. The original file was located at NCSA at the location below:

.. code-block:: bash

    # Location at NCSA of truth summary table file
    /project/shared/DC2/truth_tables.csv 

See :ref:`ingest-truth` for details on how we ingest this dataset into the repository.

An archive of the truth summary dataset is available at

   https://me.lsst.eu/lsstdata/dp02_truth_summary.tar (63 GB)

Input datasets layout
---------------------

The datasets prepared in the previous steps are organized as follows:

.. code-block:: bash

    $ tree -L 1 -F /sps/lsst/datasets/rubin/previews/dp0.2
    /sps/lsst/datasets/rubin/previews/dp0.2
    |-- calib/
    |-- raw/
    |-- refcats/
    |-- skymaps/
    `-- truth_summary/


Creating and populating the repository
======================================

In this section we present the step-by-step procedure we use for creating and populating the repository using the `LSST Science Pipelines <https://pipelines.lsst.io>`__ releases in the series **v23.0.x**.

For conciseness, hereafter we refer to the location of the repository the via the environment variable ``$REPO`` and we use environment variables to refer to the location of the input datasets. Those variables have the values shown below:

.. prompt:: bash

    export REPO='/path/to/the/desired/location/of/butler/repository'
    export DP02_TOP_DIR='/sps/lsst/datasets/rubin/previews/dp0.2'
    export DP02_CALIB="$DP02_TOP_DIR/calib"
    export DP02_RAW="$DP02_TOP_DIR/raw"
    export DP02_REFCATS="$DP02_TOP_DIR/refcats"
    export DP02_SKYMAP="$DP02_TOP_DIR/skymaps"
    export DP02_TRUTH_SUMMARY="$DP02_TOP_DIR/truth_summary"


.. _create-empty-repository:

Create an empty repository
--------------------------

We use the seed configuration file ``dp02-butler-seed.yaml`` shown below to create a butler repository composed of a PostgreSQL registry database and a file-based data store (the default):

.. code-block:: bash

    $ cat dp02-butler-seed.yaml
    registry:
      db: "postgresql://host.example.com:5432/my_database"
      namespace: "dp02_v23_0_0"

The value associated to the ``db`` key above specifies the URL of the PostgreSQL database we want to use for this repository. The ``namespace`` key tells the butler to use the `PostgreSQL schema <https://www.postgresql.org/docs/current/ddl-schemas.html>`__ named ``dp02_v23_0_0`` within database ``my_database``.

Connexion details for the registry database server can be provided via a protected file by default located at path ``$HOME/.lsst/db-auth.yaml`` or at a path pointed to by the environment variable ``LSST_DB_AUTH``. The contents of that file is similar to:

.. code-block:: bash

    $ cat $LSST_DB_AUTH
    - url: "postgresql://host.example.com:5432/my_database"
      username: "user"
      password: "secret_password"

Alternative syntax for providing registry database connexion details can be found `here <https://github.com/lsst/daf_butler/blob/main/tests/config/dbAuth/db-auth.yaml>`__.

.. note::

  It is also possible to seed the butler with a data store which exposes other access protocols (e.g. WebDAV or S3). In that case, the seed configuration file needs to be extended to contain also a ``datastore`` entry with a ``root`` key pointing to the top directory of the store, e.g.:

  .. code-block:: bash
     
      datastore:
        root: "https://webdav.example.com:1234/path/to/root/dir"

  See also the `butler datastore configuration <https://pipelines.lsst.io/v/weekly/modules/lsst.daf.butler/datastores.html>`__ document for details on more configuration options.

See also `Configuring a Butler <https://pipelines.lsst.io/v/weekly/modules/lsst.daf.butler/configuring.html>`__ for additional configuration details.

To create the repository at location ``$REPO`` we use the command:

.. prompt:: bash

    butler create --seed-config dp02-butler-seed.yaml --override $REPO

.. _register-instrument:

Register instrument
-------------------

To register the instrument for this repository we use the command below:

.. prompt:: bash

    butler register-instrument $REPO 'lsst.obs.lsst.LsstCamImSim'

.. _import-calibration-data:

Ingest calibration data
-----------------------

To ingest calibration data we use the command below:

.. prompt:: bash

    butler import --export-file "$DP02_CALIB/export.yaml" \
       --skip-dimensions 'instrument,detector,physical_filter,band' $REPO  $DP02_CALIB

Note that it is possible to add option ``--transfer direct`` to this command to avoid copying or creating symbolic links to the calibration files within the repository's data store.

.. _add-instrument-calibrations:

Add instrument's curated calibrations
-------------------------------------

To ingest the known calibration data for instrument ``LSSTCam-imSim`` we use the command below:

.. prompt:: bash

    butler write-curated-calibrations $REPO 'LSSTCam-imSim'

.. _register-sky-map:

Register SkyMap
----------------

To register the skymap configuration we use the command below:

.. prompt:: bash

    butler register-skymap --config-file "$DP02_SKYMAP/DC2.py" $REPO

.. _ingest-reference-catalog-data:

Ingest reference catalog data
-----------------------------

Ingestion of reference catalogs requires an `Astropy table <https://docs.astropy.org/en/stable/api/astropy.table.Table.html>`__ associating each file path of the reference catalog and its dimension. We use the script below to create that table and store it in file ``refcat.ecsv``.

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

.. code-block:: bash

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

The generated table file is available at

   https://me.lsst.eu/lsstdata/dp02_refcat.ecsv.tar.gz (7.8 KB)

To register and ingest reference catalog data we use the commands below:

.. code-block:: bash

    # Register reference catalog data with dataset type 'cal_ref_cat_2_2',
    # storage class 'SimpleCatalog' and dimension 'htm7'
    $ butler register-dataset-type $REPO cal_ref_cat_2_2 SimpleCatalog htm7

    # Ingest dataset of type 'cal_ref_cat_2_2' into run 'refcats' using information
    # (e.g. paths, dimensions) present in table 'refcat.ecsv'
    $ butler ingest-files --transfer direct $REPO cal_ref_cat_2_2 refcats $DP02_REFCATS/refcat.ecsv

.. _ingest-raw-exposures:

Ingest raw exposures
--------------------

To ingest raw exposures we use the command:

.. prompt:: bash
    
    butler ingest-raws --transfer direct $REPO $DP02_RAW/y{1..5}-wfd

Note that there are many ways to perform the ingestion of raws concurrently, for instance by launching an ingestion command per year and by specifying the number of processes to use for each command, such as:

.. prompt:: bash
    
    butler ingest-raws --transfer direct --processes 16 $REPO $DP02_RAW/y1-wfd

At FrDF we use ingestion in place via the option ``--transfer direct`` to avoid copying (or symlinking) raw exposure data to the repository location.


.. _define-visits:

Define visits
-------------

To define visits from the exposures previously ingested into the repository in collection ``LSSTCam-imSim/raw/all`` for instrument ``LSSTCam-imSim`` we use the command below:

.. prompt:: bash
    
    butler define-visits --collections 'LSSTCam-imSim/raw/all' $REPO 'LSSTCam-imSim'


.. _ingest-truth:

Ingest truth summary data
-------------------------

To register the dataset type named ``truth_summary`` with storage class ``DataFrame`` and dimensions ``skymap`` and ``tract`` we use the command:

.. prompt:: bash
    
    butler register-dataset-type $REPO 'truth_summary' DataFrame skymap tract

To ingest the files of dataset type ``truth_summary`` registered above into run ``2.2i/truth_summary`` using the contents of table file ``$DP02_TRUTH_SUMMARY/truth_tables.csv`` we used the command:

.. prompt:: bash

    butler ingest-files --transfer direct $REPO 'truth_summary' '2.2i/truth_summary' \
        $DP02_TRUTH_SUMMARY/truth_tables.csv

Details on how to ingest this dataset into the butler repository were extracted from `PREOPS-584 <https://jira.lsstcorp.org/browse/PREOPS-584>`_.


.. _create-collections:

Create collections
------------------

In accordance to the conventions for organizing data repositories described in `DMTN-167 <https://dmtn-167.lsst.io>`__, we create a chained collection with parent ``2.2i/defaults`` and children collections ``LSSTCam-imSim/raw/all``, ``2.2i/calib``, ``skymaps``, ``refcats,2.2i/truth_summary`` using the command below:

.. prompt:: bash

    butler collection-chain $REPO '2.2i/defaults' \
       'LSSTCam-imSim/raw/all,2.2i/calib,skymaps,refcats,2.2i/truth_summary'



Inspecting the repository
=========================

In this section we present some mechanisms to inspect the contents of the created repository using the `butler command line interface <https://pipelines.lsst.io/modules/lsst.daf.butler/scripts/butler.html>`_. It is also possible to inspect the repository using the `Butler Python API <https://pipelines.lsst.io/py-api/lsst.daf.butler.Butler.html>`_ (see for instance the `Introduction to the LSST data Butler <https://github.com/rubin-dp0/tutorial-notebooks/blob/main/04_Intro_to_Butler.ipynb>`_ notebook of the `DP0.1 Tutorials <https://dp0-1.lsst.io/tutorials-examples/index.html>`_ series.)

Collections
-----------

.. code-block:: bash

    $ butler query-collections --chains 'TREE' $REPO
                        Name                         Type   
    -------------------------------------------- -----------
    2.2i/calib/DM-30694/curated/19700101T000000Z RUN        
    2.2i/calib/DM-30694/unbounded                RUN        
    2.2i/calib/gen2/20220101T000000Z             RUN        
    2.2i/calib/gen2/20220806T000000Z             RUN        
    2.2i/calib/gen2/20231201T000000Z             RUN        
    2.2i/calib/DM-30694                          CALIBRATION
    2.2i/calib/gen2                              CALIBRATION
    2.2i/calib                                   CHAINED    
      2.2i/calib/DM-30694                        CALIBRATION
      2.2i/calib/gen2                            CALIBRATION
      2.2i/calib/DM-30694/unbounded              RUN        
    LSSTCam-imSim/calib                          CALIBRATION
    LSSTCam-imSim/calib/unbounded                RUN        
    LSSTCam-imSim/calib/curated/19700101T000000Z RUN        
    skymaps                                      RUN        
    refcats                                      RUN        
    LSSTCam-imSim/raw/all                        RUN        
    2.2i/truth_summary                           RUN        
    2.2i/defaults                                CHAINED    
      LSSTCam-imSim/raw/all                      RUN        
      2.2i/calib                                 CHAINED    
        2.2i/calib/DM-30694                      CALIBRATION
        2.2i/calib/gen2                          CALIBRATION
        2.2i/calib/DM-30694/unbounded            RUN        
      skymaps                                    RUN        
      refcats                                    RUN        
      2.2i/truth_summary                         RUN        


Dataset types
-------------

.. code-block:: bash

    $ butler query-dataset-types --verbose $REPO
          name                                  dimensions                               storage class    
    --------------- ----------------------------------------------------------------- --------------------
                bfk                                        ['instrument', 'detector'] BrighterFatterKernel
               bias                                        ['instrument', 'detector']            ExposureF
    cal_ref_cat_2_2                                                          ['htm7']        SimpleCatalog
             camera                                                    ['instrument']               Camera
               dark                                        ['instrument', 'detector']            ExposureF
               flat             ['band', 'instrument', 'detector', 'physical_filter']            ExposureF
                raw ['band', 'instrument', 'detector', 'physical_filter', 'exposure']             Exposure
                sky             ['band', 'instrument', 'detector', 'physical_filter']            ExposureF
             skyMap                                                        ['skymap']               SkyMap
      truth_summary                                               ['skymap', 'tract']            DataFrame


Dimension records
-----------------

The command ``butler query-dimension-records`` displays detailed information of which we only present some excerpts noted by the presence of the ``...`` in the command's output.

.. code-block:: bash

    $ butler query-dimension-records $REPO 'detector'
      instrument   id full_name name_in_raft raft purpose
    ------------- --- --------- ------------ ---- -------
    LSSTCam-imSim   0   R01_S00          S00  R01 SCIENCE
    LSSTCam-imSim   1   R01_S01          S01  R01 SCIENCE
    LSSTCam-imSim   2   R01_S02          S02  R01 SCIENCE
    ...

.. code-block:: bash

    $ butler query-dimension-records $REPO 'instrument'
         name     visit_max exposure_max detector_max         class_name        
    ------------- --------- ------------ ------------ --------------------------
    LSSTCam-imSim   9999999      9999999         1000 lsst.obs.lsst.LsstCamImSim

.. code-block:: bash

    $ butler query-dimension-records $REPO 'physical_filter'
      instrument     name   band
    ------------- --------- ----
    LSSTCam-imSim g_sim_1.4    g
    LSSTCam-imSim i_sim_1.4    i
    LSSTCam-imSim r_sim_1.4    r
    LSSTCam-imSim u_sim_1.4    u
    LSSTCam-imSim y_sim_1.4    y
    LSSTCam-imSim z_sim_1.4    z


Datasets
--------

The command ``butler query-datasets`` displays detailed information of which we only present some excerpts noted by the presence of the ``...`` in the command's output.

.. code-block:: bash

    $ butler query-datasets $REPO

    type                     run                                       id                    instrument  detector
    ---- -------------------------------------------- ------------------------------------ ------------- --------
     bfk 2.2i/calib/DM-30694/curated/19700101T000000Z 10fe0425-6d33-4f9a-8487-bb505e478906 LSSTCam-imSim        0
     bfk 2.2i/calib/DM-30694/curated/19700101T000000Z 9634f25b-0421-414c-9a80-0206662ba573 LSSTCam-imSim        1
     bfk 2.2i/calib/DM-30694/curated/19700101T000000Z 2976df15-6233-4c8a-8911-a4fa107d0c45 LSSTCam-imSim        2
    ...

    type               run                                 id                    instrument  detector
    ---- -------------------------------- ------------------------------------ ------------- --------
    bias 2.2i/calib/gen2/20220101T000000Z 908f0761-f114-4d93-8c61-ef2913ddc0ee LSSTCam-imSim        0
    bias 2.2i/calib/gen2/20220101T000000Z edec132e-77ee-48e7-a0ff-eb6f107b7825 LSSTCam-imSim        1
    bias 2.2i/calib/gen2/20220101T000000Z 9519e665-c0da-4196-a3d1-c3a276ae602f LSSTCam-imSim        2
    ...

     type               run                               id                    instrument 
    ------ ----------------------------- ------------------------------------ -------------
    camera 2.2i/calib/DM-30694/unbounded b7e109f0-764c-492a-9cfc-04471a7171e3 LSSTCam-imSim
    camera LSSTCam-imSim/calib/unbounded 35c4bcc2-6f58-4e21-8a43-68fd775bbb06 LSSTCam-imSim

    type               run                                 id                    instrument  detector
    ---- -------------------------------- ------------------------------------ ------------- --------
    dark 2.2i/calib/gen2/20220101T000000Z c129b82b-02ea-4158-9166-6a0bbbc9eb12 LSSTCam-imSim        0
    dark 2.2i/calib/gen2/20220101T000000Z 22cf0a75-a29e-4bc9-9b91-9a4cafc90873 LSSTCam-imSim        1
    dark 2.2i/calib/gen2/20220101T000000Z 8101186b-b7ac-43f1-8112-b6b1e611bdcc LSSTCam-imSim        2
    ...

    type               run                                 id                  band   instrument  detector physical_filter
    ---- -------------------------------- ------------------------------------ ---- ------------- -------- ---------------
    flat 2.2i/calib/gen2/20220806T000000Z c8569842-4445-449a-90ea-c1ad341cf021    g LSSTCam-imSim        0       g_sim_1.4
    flat 2.2i/calib/gen2/20220806T000000Z a2bf9feb-b11e-43a0-882d-27f3c7870d0c    g LSSTCam-imSim        1       g_sim_1.4
    flat 2.2i/calib/gen2/20220806T000000Z f3021e4e-2d62-4691-8ad4-d789baaad27c    g LSSTCam-imSim        2       g_sim_1.4
    ...

    type               run                                 id                  band   instrument  detector physical_filter
    ---- -------------------------------- ------------------------------------ ---- ------------- -------- ---------------
     sky 2.2i/calib/gen2/20231201T000000Z 0d857fec-47be-40bf-bc07-518d13ebb19e    g LSSTCam-imSim        0       g_sim_1.4
     sky 2.2i/calib/gen2/20231201T000000Z 5a9fd70a-df96-436c-8cf9-82978e8d9023    g LSSTCam-imSim        1       g_sim_1.4
     sky 2.2i/calib/gen2/20231201T000000Z a13b6071-369b-4735-b724-244f387d0edc    g LSSTCam-imSim        2       g_sim_1.4
    ...

     type    run                    id                  skymap
    ------ ------- ------------------------------------ ------
    skyMap skymaps 7e392c32-2615-4249-8cd2-d40976ebda45    DC2

          type        run                    id                   htm7 
    --------------- ------- ------------------------------------ ------
    cal_ref_cat_2_2 refcats c05fa183-bb30-47a1-a859-339629255f4e 141440
    cal_ref_cat_2_2 refcats 49d3511b-d7dd-4001-9fe7-17471cefe6b2 141443
    cal_ref_cat_2_2 refcats e6265445-320f-4949-b7f9-99e0fe139b9f 141825
    ...

    type          run                           id                  band   instrument  detector physical_filter exposure
    ---- --------------------- ------------------------------------ ---- ------------- -------- --------------- --------
     raw LSSTCam-imSim/raw/all 8caedfed-a9ae-51e0-845a-0b3db1caeb72    i LSSTCam-imSim       19       i_sim_1.4   731614
     raw LSSTCam-imSim/raw/all 88a99f3d-b4f6-5886-842f-be0267cf6ef3    i LSSTCam-imSim       20       i_sim_1.4   731614
     raw LSSTCam-imSim/raw/all 2cb30d02-e0c3-5e27-ad85-0473e62ad3dd    i LSSTCam-imSim       21       i_sim_1.4   731614
    ...

         type            run                          id                  skymap tract
    ------------- ------------------ ------------------------------------ ------ -----
    truth_summary 2.2i/truth_summary a9d4e854-41fd-4274-a665-ceb02b1b1aa5    DC2  2723
    truth_summary 2.2i/truth_summary 26dab1ba-9d85-4b13-aa8d-88dee400b25e    DC2  2724
    truth_summary 2.2i/truth_summary a2de4b8a-efc7-4289-8e42-4d5472d6ba39    DC2  2725
    ...



.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
   :style: lsst_aa
