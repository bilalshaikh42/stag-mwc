Running |full_name|
===================
You need to configure a workflow before you can run |full_name|. The code 
you downloaded in the previous ``git clone`` step includes a file called 
``config.yaml``, which is used to configure the workflow. 

Open ``config.yaml`` in your favorite editor and change the settings under the
``Run configuration`` heading: the input directory, the input filename pattern,
and the output directory, are the most important ones. They can be declared
using absolute or relative filenames (relative to the |full_name| repository
directory).

Next, configure the settings under the ``Pipeline steps included`` heading.
This is where you define what steps should be included in your workflow. Simply
assign ``True`` or ``False`` to the steps you want to include. Note that the
default configuration file already includes ``qc_reads`` and ``remove_human``.
These two steps are the primary read processing steps and most other steps
depends on human filtered reads (i.e. the output of the ``remove_human`` step).

.. note:: 

    You can create several copies of ``config.yaml``, named whatever you want,
    in order to manage several analysis from the same |full_name| directory.
    If you create a copy called e.g. ``microbime_analysis.yaml``, you can easily
    run the workflow with this configuration file by using the ``--configfile``
    commandline argument when running the workflow.

A reference database is required in order to run the ``remove_human`` step. If
you already have it downloaded somewhere, point |full_name| to the location
using the ``hg19_path`` parameter under the ``bbduk`` part of ``config.yaml``.
|full_name| can download and index the database for you, see `Downloading
databases` below. 


Downloading databases
*********************
Several of the tools used in |full_name| need special databases to work. Fortunately,
|full_name| makes it easy to download and prepare the required databases. The first
database you will need is the ``hg19`` reference database for use in the ``remove_human``
read processing step. If you do not have it available before using |full_name|, run
the following command to download and index the database for you::

    snakemake index_hg19

This will automatically download and index the BBMap masked hg19 file for you. The
database will be downloaded to the ``dbdir`` parameter specified in ``config.yaml``.
Note that creating the hg19 index requires at least 16GB of RAM, so it is typically
not recommended to do this on a laptop.

|full_name| can download several databases by typing ``snakemake <rule_name>``
using any of the following rules::

    index_hg19  (already shown above) 
    download_centrifuge_database
    download_kaiju_database
    build_metaphlan2_index
    create_megares_index

Make sure you update your ``config.yaml`` to reflect the location of the database
you want to use after downloading them.


Running
*******
It is recommended to run Snakemake with the ``-n``/``--dryrun`` argument before
starting an analysis for real. Executing a dryrun will let Snakemake check that
all the requirements are available and it will then print a summary of what it
intends to do, without actually doing anything. After finishing the
configuration by editing ``config.yaml``, test your configuration with::

    snakemake --use-conda --dryrun

If you are satisfied with the workflow plan output by the dryrun, you can run
the workflow. The typical command to run |full_name| on your local computer
is::

    snakemake --use-conda --cores N

where ``N`` is the maximum number of cores you want to allow for the workflow.
Snakemake will automatically reduce the number of cores available to individual
steps to this limit.


Running on cluster resources
****************************
In order to run |full_name| on a cluster, you need a special cluster
configuration file.  |full_name| ships with a pre-made configuration file for
use on UPPMAX's Rackham cluster.  Find all available cluster configuration
files in the ``cluster_configs`` directory in the repository. The cluster
configuration files specify the cluster scheduler account to use (e.g. Slurm
project account), as well as the processor, time, and memory requirements of
each individual step. Snakemake uses this information when submitting jobs to
the cluster scheduler.

To run |full_name| on e.g. Rackham, run the following command when standing in
the workflow repository directory::

    snakemake --use-conda --jobs 999 --cluster-config cluster_configs/rackham.json --cluster "sbatch -A {cluster.account} -p {cluster.partition} -n {cluster.n} -t {cluster.time}"

This will make Snakemake submit each workflow step as a separate cluster job
using the processor and time requirements specified in ``rackham.json``. The
``--jobs 999`` argument is to tell Snakemake to submit up to 999 jobs to the
cluster scheduler at once. In most situations, this will never happen as it is
unlikely that the workflow you are running will ever have 999 independent steps
ready to run at the same time. The above command of course assumes you are
using the default ``config.yaml`` configuration file. If you are using a custom
configuration file, just add ``--configfile <name_of_your_config_file>`` to
the command line.

