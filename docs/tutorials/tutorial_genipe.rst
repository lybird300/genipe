
.. contents:: Quick navigation
   :depth: 2


.. _genipe-tut-page:

Genome-wide imputation pipeline
================================

High-throughput genotyping platforms can assess up to five million variations
in thousands of samples. Depending on the genotyped data and the reference
panel used, genome-wide imputation tools can infer genotypes for more than 38
million variants (single nucleotides, insertions and deletions). Unfortunately,
genome-wide imputation requires high computation power and multiple data
processing steps.

The :py:mod:`genipe` pipeline automates the different steps for pre-phasing and
imputation for genome-wide data. The pipeline follows the guideline described
by IMPUTE2's best practices when analyzing genome-wide data (described
`by IMPUTE2 <https://mathgen.stats.ox.ac.uk/impute/impute_v2.html#prephasing>`_
and `by SHAPEIT <http://www.shapeit.fr/pages/m03_phasing/imputation.html>`_).


Preparing the data
-------------------

The :py:mod:`genipe` module provides a script to prepare all the required files
for this tutorial. It will download the appropriate binary files for Plink,
IMPUTE2 and SHAPETIT, the genotypes of the study cohort to impute, the
IMPUTE2's reference panels, and the indexed human reference (in *fasta*
format).

.. warning::

   You should review
   `IMPUTE2 licence <https://mathgen.stats.ox.ac.uk/impute/impute_v2.html#download>`_ and
   `SHAPEIT licence <https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#download>`_
   to make sure that you have the right to use these tools.

To following command will execute the script (once the virtual Python
environment is activated, see :ref:`genipe-pyvenv-activation` for more
details).


.. code-block:: bash

   genipe-tutorial

.. note::

   The script will create a directory called ``genipe_tutorial`` in the home
   directory. If another directory is required, the default path can be modify
   using the ``--tutorial-path`` option.

To have more information about the input data, have a look at the
:ref:`genipe-tut-more-details` section below. If the imputation is to be run on
a server using a job scheduler, have a look at the :ref:`genipe-tut-drmaa`
section.


Input file summary
-------------------

You should have the following directory structure:

.. code-block:: text

   $HOME/genipe_tutorial/
   │
   ├── 1000GP_Phase3/
   │   ├── 1000GP_Phase3_chr1.hap.gz
   │   ├── 1000GP_Phase3_chr2.hap.gz
   │   ├── ...
   │   ├── 1000GP_Phase3_chr1.legend.gz
   │   ├── 1000GP_Phase3_chr2.legend.gz
   │   ├── ...
   │   ├── 1000GP_Phase3.sample
   │   ├── genetic_map_chr1_combined_b37.txt
   │   ├── genetic_map_chr2_combined_b37.txt
   │   └── ...
   │
   ├── bin/
   │   ├── impute2
   │   ├── plink
   │   └── shapeit
   │
   ├── data/
   │   ├── hapmap_CEU_r23a_hg19.bed
   │   ├── hapmap_CEU_r23a_hg19.bim
   │   └── hapmap_CEU_r23a_hg19.fam
   │
   ├── genipe_config.ini  # OPTIONAL (--use-drmaa, --drmaa-config)
   │
   ├── hg19/
   │   ├── hg19.fasta
   │   └── hg19.fasta.fai
   │
   └── preamble.txt     # OPTIONAL (--use-drmaa, --preamble)


.. _genipe-tut-execute:

Executing the pipeline
-----------------------

Once all the input files are ready for analysis, you can finally execute the
pipeline. Make sure that the virtual Python environment was properly activated
(see :ref:`genipe-pyvenv-activation` for more details).

When automatically preparing the data using the ``genipe-tutorial`` script, a
new script named ``execute.sh`` will be created to execute the analysis. Here
is the content of the ``execute.sh`` script.

.. code-block:: bash

   #!/usr/bin/env bash
   # Changing directory
   cd $HOME/genipe_tutorial

   # Launching the imputation with genipe
   genipe-launcher \
       --chrom autosomes \
       --bfile $HOME/genipe_tutorial/data/hapmap_CEU_r23a_hg19 \
       --shapeit-bin $HOME/genipe_tutorial/bin/shapeit \
       --impute2-bin $HOME/genipe_tutorial/bin/impute2 \
       --plink-bin $HOME/genipe_tutorial/bin/plink \
       --reference $HOME/genipe_tutorial/hg19/hg19.fasta \
       --hap-template $HOME/genipe_tutorial/1000GP_Phase3/1000GP_Phase3_chr{chrom}.hap.gz \
       --legend-template $HOME/genipe_tutorial/1000GP_Phase3/1000GP_Phase3_chr{chrom}.legend.gz \
       --map-template $HOME/genipe_tutorial/1000GP_Phase3/genetic_map_chr{chrom}_combined_b37.txt \
       --sample-file $HOME/genipe_tutorial/1000GP_Phase3/1000GP_Phase3.sample \
       --filtering-rules 'ALL<0.01' 'ALL>0.99' \
       --thread 4 \
       --report-title "Tutorial" \
       --report-number "Test Report"

When in the correct working directory, the following command should execute the
genome-wide imputation of the *HapMap* CEU dataset.

.. code-block:: bash

   cd $HOME/genipe_tutorial

   ./execute.sh

.. note::

   In the script ``execute.sh``, the ``--reference`` option is optional and
   might be removed.

.. note::

   It is possible to compress IMPUTE2's output files by adding the ``--bgzip``
   option (the ``bgzip`` software should be in the path).

.. note::

   It is possible to remove the ``--shapeit-bin``, the ``--impute2-bin`` and/or
   the ``--plink-bin`` options if the SHAPEIT, IMPUTE2 and/or the Plink
   binaries are in  the ``PATH`` variable.

.. note::

   The ``--chrom autosomes`` option will run the imputation only on the
   autosomes. If all chromosomes are required (including chromosomes 23 and 25
   (pseudo-autosomal region), then the ``--chrom`` option can be removed, and
   the imputation reference files for chromosome 23 PAR and nonPAR are required
   (see :ref:`this section <params-chrom-x>` for more information).

.. note::

   It is possible to add extra options to SHAPEIT and IMPUTE2 using the
   ``--shapeit-extra`` and ``--impute2-extra`` options, respectively.

The following table describes the options **used by** :py:mod:`genipe` **in the
previous command** (see the :ref:`genipe-usage` section for a full list):

.. table::

    +-----------------------+-------------------------------------------------+
    | Option                | Description                                     |
    +=======================+=================================================+
    | ``--chrom``           | The chromosome to impute (here, the autosomes). |
    +-----------------------+-------------------------------------------------+
    | ``--bfile``           | The genotypes of the study cohort to be imputed.|
    +-----------------------+-------------------------------------------------+
    | ``--shapeit-bin``     | The ``shapeit`` binary.                         |
    +-----------------------+-------------------------------------------------+
    | ``--impute2-bin``     | The ``impute2`` binary.                         |
    +-----------------------+-------------------------------------------------+
    | ``--plink-bin``       | The ``plink`` binary.                           |
    +-----------------------+-------------------------------------------------+
    | ``--reference``       | The *fasta* file containing the reference genome|
    |                       | for initial strand verification (*optional*).   |
    +-----------------------+-------------------------------------------------+
    | ``--hap-template``    | The template for *IMPUTE2*'s reference haplotype|
    |                       | files (``{chrom}`` will be replaced by the      |
    |                       | chromosome number).                             |
    +-----------------------+-------------------------------------------------+
    | ``--legend-template`` | The template for *IMPUTE2*'s reference legend   |
    |                       | files (``{chrom}`` will be replaced by the      |
    |                       | chromosome number).                             |
    +-----------------------+-------------------------------------------------+
    | ``--map-template``    | The template for *IMPUTE2*'s reference map      |
    |                       | files (``{chrom}`` will be replaced by the      |
    |                       | chromosome number).                             |
    +-----------------------+-------------------------------------------------+
    | ``--sample-file``     | The name of *IMPUTE2*'s reference sample file.  |
    +-----------------------+-------------------------------------------------+
    | ``--filtering-rules`` | Rules used by *IMPUTE2* to exclude sites from   |
    |                       | its reference files (using the legend files).   |
    |                       | Each terms are joined using a logical *OR*.     |
    |                       | Here, loci with an alternative allele frequency |
    |                       | lower than 0.01 and higher than 0.99 will be    |
    |                       | excluded from the imputation reference.         |
    +-----------------------+-------------------------------------------------+
    | ``--thread``          | The number of thread to use for the analysis.   |
    |                       | When using *DRMAA*, this will be the number of  |
    |                       | simultaneous tasks. Four threads will be used.  |
    +-----------------------+-------------------------------------------------+
    | ``--report-title``    | The title of the automatic report.              |
    +-----------------------+-------------------------------------------------+
    | ``--report-number``   | The number of the report (will appear as        |
    |                       | sub-title and in the footer of the automatic    |
    |                       | report).                                        |
    +-----------------------+-------------------------------------------------+


.. note::

   If the pipeline fails (*e.g.* not enough memory or the walltime exceeded on
   a computing cluster), re-running the pipeline (with different number of
   thread or different walltime) will only launch the task that were not
   completed.

   The pipeline checks if output files are missing. If an output file is
   deleted, the step producing this file will be run again (but not the
   subsequent steps).


.. note::

   Four options will modify the report content: ``--report-number``,
   ``--report-title``, ``--report-author`` and ``--report-background``.


.. _genipe-tut-compile-report:

Compiling the report
---------------------

A report containing useful information (such as quality metrics and execution
time, among others) is automatically generated once the imputation process is
completed. To compile the report, perform the following commands:

.. code-block:: bash

   cd $HOME/genipe_tutorial/genipe/report

   make && make clean

This will generate the following
`PDF report <http://pgxcentre.github.io/genipe/_static/tutorial/report.pdf>`_
(which is named ``report.pdf``). It is always possible to modify the original
``report.tex`` file to include analysis specific details (*e.g.* cohort
description).


.. _genipe-tut-output-files:

Output files
-------------

All results will be located in the ``genipe`` directory (or whatever
``--output-dir`` links to). Here is the directory tree summarizing the output
files.

.. code-block:: text

   genipe/
   │
   ├── chr1/
   │   ├── chr1.1_5000000.impute2
   │   ├── chr1.1_5000000.impute2_info
   │   ├── chr1.1_5000000.impute2_info_by_sample
   │   ├── chr1.1_5000000.impute2_summary
   │   ├── chr1.1_5000000.impute2_warnings
   │   ├── ...
   │   ├── chr1.final.bed
   │   ├── chr1.final.bim
   │   ├── chr1.final.fam
   │   ├── chr1.final.log
   │   ├── chr1.final.phased.haps
   │   ├── chr1.final.phased.ind.me
   │   ├── chr1.final.phased.ind.mm
   │   ├── chr1.final.phased.log
   │   ├── chr1.final.phased.sample
   │   ├── chr1.final.phased.snp.me
   │   ├── chr1.final.phased.snp.mm
   │   ├── ...
   │   │
   │   └── final_impute2/
   │       ├── chr1.imputed.alleles
   │       ├── chr1.imputed.completion_rates
   │       ├── chr1.imputed.good_sites
   │       ├── chr1.imputed.impute2.gz
   │       ├── chr1.imputed.impute2_info
   │       ├── chr1.imputed.imputed_sites
   │       ├── chr1.imputed.log
   │       ├── chr1.imputed.maf
   │       ├── chr1.imputed.map
   │       └── chr1.imputed.sample
   │
   ├── .../
   │
   ├── chromosome_lengths.txt
   ├── exclusion_summary.txt
   ├── frequency_pie.pdf
   ├── genipe.log
   ├── markers_to_exclude.txt
   ├── markers_to_flip.txt
   │
   ├── missing
   │   ├── missing.imiss
   │   ├── missing.lmiss
   │   └── missing.log
   │
   ├── report
   │   ├── frequency_pie.pdf
   │   ├── Makefile
   │   ├── references.bib
   │   ├── references.bst
   │   └── report.tex
   │
   └── tasks.db


``genipe`` directory
^^^^^^^^^^^^^^^^^^^^^

This directory contains all the chromosome specific analysis. The specific
directory content is describe below. The following files are created inside the
``genipe`` directory:

.. table::

    +----------------------------+--------------------------------------------+
    | File                       | Description                                |
    +============================+============================================+
    | ``chromosome_lengths.txt`` | The length of each chromosome (this        |
    |                            | information is fetched from Ensembl using  |
    |                            | its REST API and saved to file).           |
    +----------------------------+--------------------------------------------+
    | ``exclusion_summary.txt``  | A file containing the exclusion summary    |
    |                            | prior to phasing (*e.g* the number of      |
    |                            | ambiguous markers, the number of flipped   |
    |                            | markers, etc.).                            |
    +----------------------------+--------------------------------------------+
    | ``frequency_pie.pdf``      | This file contains a pie chart describing  |
    |                            | the minor allele frequency distribution of |
    |                            | the imputed markers. This file is generated|
    |                            | only if the :py:mod:`matplotlib` module is |
    |                            | installed.                                 |
    +----------------------------+--------------------------------------------+
    | ``genipe.log``               | The log file of the main pipeline.       |
    +----------------------------+--------------------------------------------+
    | ``markers_to_exclude.txt`` | The list of markers to exclude prior to    |
    |                            | phasing.                                   |
    +----------------------------+--------------------------------------------+
    | ``markers_to_flip.txt``    | The list of markers to flip prior to       |
    |                            | phasing.                                   |
    +----------------------------+--------------------------------------------+
    | ``tasks.db``               | The *sqlite* database containing           |
    |                            | information of all tasks (if it's          |
    |                            | completed, execution time, etc).           |
    +----------------------------+--------------------------------------------+


``genipe/chrN`` directories
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``chrN`` directories contain the intermediate files, created throughout the
pipeline. The most important files in these directories are the log files (for
errors and summary statistics). There will be one directory per autosomal
chromosomes.


.. _genipe-tut-output-files-final_impute2:

``genipe/chrN/final_impute2`` directories
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These ``final_impute2`` directories (located in the ``genipe/chrN``
directories) contain the final output files from the pipeline for each
autosomal chromosomes. They will contain the following files:

.. table::

    +-------------------------------+-----------------------------------------+
    | Extension                     | Description                             |
    +===============================+=========================================+
    | ``.imputed.alleles``          | Description of the reference and        |
    |                               | alternative allele at each sites.       |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.completion_rates`` | Number of missing values and completion |
    |                               | rate for all sites (using the           |
    |                               | probability threshold set by the user,  |
    |                               | where the default is higher and equal   |
    |                               | to 0.9).                                |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.good_sites``       | List of sites which pass the completion |
    |                               | rate threshold (set by the user, where  |
    |                               | the default is higher and equal to 0.98)|
    |                               | using the probability threshold (set by |
    |                               | the user, where the default is higher   |
    |                               | and equal to 0.9). The sites also pass  |
    |                               | the INFO threshold (set by the user,    |
    |                               | where the default is higher and equal to|
    |                               | 0).                                     |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.impute2`` or       | Imputation results (merged from the     |
    | ``.imputed.impute2.gz``       | individual segment files. This file     |
    |                               | might be compress (with the ``.gz``     |
    |                               | extension) if the ``--bgzip`` option was|
    |                               | used when launching the pipeline.       |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.impute2_info``     | Marker-wise information file with one   |
    |                               | line per marker and a single header line|
    |                               | at the begening. It contains, among     |
    |                               | others, the information value which is a|
    |                               | measure of the observed statistical     |
    |                               | information associated with the allele  |
    |                               | frequency estimate.                     |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.imputed_sites``    | List of imputed sites (excluding sites  |
    |                               | that were previously genotyped in the   |
    |                               | study cohort).                          |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.log``              | The log file of the merging step.       |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.maf``              | File containing the minor allele        |
    |                               | frequency (along with minor allele      |
    |                               | identification) for all sites using the |
    |                               | probabilitty threshold of 0.9. When no  |
    |                               | genotypes are available (because they   |
    |                               | are all below the threshold), the MAF is|
    |                               | ``NA``.                                 |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.map``              | A *map* file describing the genomic     |
    |                               | location of all sites.                  |
    +-------------------------------+-----------------------------------------+
    | ``.imputed.sample``           | The sample file generated by the phasing|
    |                               | step, which describe the sample ordering|
    |                               | in the IMPUTE2 files.                   |
    +-------------------------------+-----------------------------------------+


``genipe/missing`` directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``missing`` directory contains the missing rates for both samples
(``missing.imiss``) and genotypes markers (``missing.lmiss``). Those files are
generated by Plink.

``genipe/report`` directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This ``report`` directory contains the automatically generated report, which
provides valuable information about the imputation analysis. Such information
contains cross-validation statistics (as provided by IMPUTE2), frequency
statistics and completion rates according to user defined parameters.

The automatic report is generated in the ``LaTeX`` language (file
``report.tex``), and can be compile using the following command (as long as
``LaTeX`` is installed).

.. code-block:: bash

   cd $HOME/genipe_tutorial/genipe/report
   make && make clean

This will generate the following
`PDF report <http://pgxcentre.github.io/genipe/_static/tutorial/report.pdf>`_
(which is named ``report.pdf``). It is always possible to modify the original
``report.tex`` file to include analysis specific details (*e.g.* cohort
description).


.. _genipe-tut-drmaa:

Using genipe on a server
-------------------------

DRMAA configuration
^^^^^^^^^^^^^^^^^^^^

If the pipeline is to be launch on a computing server, the ``--use-drmaa``
option should be used. This will launch each step on the server using the DRMAA
api. On some cluster, supplemental information is required for each task
(*i.e.* execution time, number of nodes/processes to reserve). This
parametrization is done using a configuration (*ini*) file, describing these
parameters for each step.

When providing an empty *ini* file, the default walltime and number of
nodes/processes will be 15 minutes and 1/1, respectively. Otherwise, different
parameters can be used for each step. For example, the following configuration
will increase the walltime for all phasing tasks from 15 minutes to 3 hours. It
will also run each phasing tasks on one node using 12 processes.

.. code-block:: ini

   [shapeit_phase]
   walltime = 03:00:00
   nodes    = 1
   ppn      = 12

The following example has the same configuration as the previous example, but
will increase the walltime for chromosome 2 to 4 hours, with 1 node and 24
processes.

.. code-block:: ini

   [shapeit_phase]
   walltime = 03:00:00
   nodes    = 1
   ppn      = 12

   chr2_walltime = 04:00:00
   chr2_nodes    = 1
   chr2_ppn      = 24

Since imputation is performed on segments for each chromosome, it is possible
to modify the parameters for a single segment. This is usefull when a segment
doesn't have time to finish and its imputation requires a rerun. For example,
the following parameters will increase the walltime from 15 minutes to 3.5
hours for segment 10,000,001-15,000,000 on chromosome 1. Also, all segments
located on chromosome 2 will have a walltime of 4 hours.

.. code-block:: ini

   [impute2]
   chr1_10000001_15000000_walltime = 03:30:00

   chr2_walltime = 04:00:00

We provide a
`configuration example <http://pgxcentre.github.io/genipe/_static/tutorial/config_example.ini>`_
including all possible section. Also, here is a list of all possible section
(*i.e* pipeline step) that can be parametrized.

- ``plink_exclude``
- ``plink_missing_rate``
- ``shapeit_check_1``
- ``plink_flip``
- ``shapeit_check_2``
- ``plink_final_exclude``
- ``shapeit_phase``
- ``impute2``
- ``merge_impute2``
- ``bgzip``


Some cluster doesn't require any configuration at all. To skip configuration,
use the ``main`` section of the *ini* file as such:

.. code-block:: ini

   [main]
   skip_drmaa_config = yes

.. note::

   Keep in mind that lines starting with a ``#`` are comments and are not used
   in the DRMAA configuration. This is useful to describe what parameters are
   used for each step.


DRMAA preamble
^^^^^^^^^^^^^^^

When using the ``--use-drmaa`` option, the pipeline creates *bash* script that
are launched on the computing cluster. Some clusters require module to be
loaded and the python virtual environment to be loaded before executing a
script. This is done using the preamble file (the ``--preamble`` option).

The content of the file will be added between the first line of the temporary
*bash* script (the *shebang*) and the actual command. For example, the
following file will load the gcc module (version 4.8.2) and the python virtual
environment before launching the task.

.. code-block:: bash

   # Loading the required module
   module load gcc/4.8.2

   # The python virtual environment
   source $HOME/softwares/python_env/bin/activate

.. note::

   The preamble file is system dependent, but you should always at least
   activate the virtual python environment so that the tools provided by
   :py:mod:`genipe` are automatically in the system path.

.. warning::

   The preamble will be added **as-is** in the *bash* script that will be
   executed. Hence, always be careful of what is included in the preamble.


.. _genipe-tut-more-details:

Data in more details
---------------------

First, we will create a project directory where all the analysis will be
performed:

.. code-block:: bash

   mkdir -p $HOME/genipe_tutorial
   cd $HOME/genipe_tutorial


.. _genipe-tut-softwares:

Required softwares
^^^^^^^^^^^^^^^^^^^

The main :py:mod:`genipe` pipeline requires three external tools:
`Plink <http://pngu.mgh.harvard.edu/~purcell/plink/>`_,
`IMPUTE2 <https://mathgen.stats.ox.ac.uk/impute/impute_v2.html>`_ and
`SHAPEIT <https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html>`_.
These tools are not required to be located in your ``PATH`` variable, since you
can specify each of their location at runtime.

If some of the binaries are missing in you ``PATH`` variable, we will create a
directory where the missing binaries will go:

.. code-block:: bash

   mkdir -p $HOME/genipe_tutorial/bin
   cd $HOME/genipe_tutorial/bin

Plink is freely available. Go to
`Plink download page <http://pngu.mgh.harvard.edu/~purcell/plink/download.shtml>`_,
read the license, and download the most recent version. Extract the binary and
copy it to the ``bin`` directory previously created.

IMPUTE2 is freely available for academic use. Go to
`IMPUTE2 download page <https://mathgen.stats.ox.ac.uk/impute/impute_v2.html#download>`_,
read the license, and download the most recent version. Extract the binary and
copy it to the ``bin`` directory previously created.

SHAPEIT is freely available for academic use. Go to
`SHAPEIT download page <https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#download>`_,
read the license, and download the most recent version. Extract the binary and
copy it to the ``bin`` directory previously created.

The ``bin`` directory should now contain the missing binaries: ``impute2``,
``plink`` and/or ``shapeit``.


.. _genipe-tut-input-files:

Input files
^^^^^^^^^^^^

Genotypes
""""""""""

The genotypes of the study cohort have to be in Plink's binary pedfile format.
This format allows for time and disk space optimization. Each dataset is
comprised of three files: the ``.bed`` (binary file, genotype information),
``.fam`` (text file, sample information) and ``.bim`` (text file, marker
information) files. Those file are generated by Plink.

Genotype data are provided for this tutorial. They are a subset from the files
provided by Plink on its
`resources page <http://pngu.mgh.harvard.edu/~purcell/plink/res.shtml>`_.
The build was lifted over from *GRCh36* to *GRCh37* using
`USCS liftOver <https://genome.ucsc.edu/cgi-bin/hgLiftOver>`_. Only CEU samples
were kept, along with markers having a completion rate higher or equal to 0.98.
To download the data for this tutorial, execute the following command:

.. code-block:: bash

   mkdir -p $HOME/genipe_tutorial/data
   cd $HOME/genipe_tutorial/data

   wget http://pgxcentre.github.io/genipe/_static/tutorial/hapmap_CEU_r23a_hg19.tar.bz2
   tar -jxf hapmap_CEU_r23a_hg19.tar.bz2
   rm hapmap_CEU_r23a_hg19.tar.bz2


Reference panels
"""""""""""""""""

IMPUTE2 can use publicly available reference datasets. They provide such
dataset on their website. Go to IMPUTE2's
`reference page <https://mathgen.stats.ox.ac.uk/impute/impute_v2.html#reference>`_,
and download the most recent reference data (which is over 12Gb). Once the
reference is downloaded, extract it in the working directory
(``$HOME/genipe_tutorial``).

The following commands should download the reference files (1000 Genomes phase
3) and extract them in the required directory. Note that the specified URL
might change.

.. code-block:: bash

   cd $HOME/genipe_tutorial

   wget https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.tgz
   tar -zxf 1000GP_Phase3.tgz
   rm 1000GP_Phase3.tgz


Human reference (optional)
"""""""""""""""""""""""""""

The pipeline include an optional step to check for strand alignment with the
reference panel (using *SHAPEIT*). The drawback of this method is that it is
impossible to verify the strand of markers which are absent from the
*IMPUTE2*'s reference. We have introduce a way to check the strand using the
reference genome (in *fasta* format, indexed using *faidx*).

We have created such reference using the
`UCSC's human reference <http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/>`_.

.. code-block:: bash

   mkdir -p $HOME/genipe_tutorial/hg19
   cd $HOME/genipe_tutorial/hg19

   wget http://statgen.org/wp-content/uploads/Softwares/genipe/supp_files/hg19.tar.bz2
   tar -jxf hg19.tar.bz2
   rm hg19.tar.bz2

