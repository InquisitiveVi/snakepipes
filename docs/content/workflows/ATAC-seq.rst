.. _ATAC-seq:

ATAC-seq
========

What it does
------------

The ATAC-seq pipeline takes one or more BAM files and attempts to find accessible regions. If multiple samples and a sample sheet are provided, then CSAW is additionally used to find differentially accessible regions. Prior to finding open/accessible regions, the BAM files are filtered to include only properly paired reads with appropriate fragment sizes (<150 bases by default). These filtered fragments are then used for the remainder of the pipeline.

.. image:: ../images/ATACseq_pipeline.png

Note that the `CSAW` step will be skipped if there is no sample sheet.

Input requirements
------------------

The DNA mapping pipeline generates output that is fully compatible with the ATAC-seq pipeline input requirements!
When running the ATAC-seq pipeline, please specify the output directory of DNA-mapping pipeline as the working directory (-w).

If you need to provide files **NOT** generated by the DNA-mapping pipeline, then you must provide a directory with the following structure::

    .
    ├── deepTools_qc
    │   └── bamPEFragmentSize
    │       ├── fragmentSize.metric.tsv
    │       └── fragmentSizes.png
    ├── filtered_bam
    │   ├── sample1.filtered.bam
    │   ├── sample1.filtered.bam.bai
    │   ├── sample2.filtered.bam
    │   └── sample2.filtered.bam.bai
    ├── Picard_qc
    │   └── AlignmentSummaryMetrics
    │       ├── sample1.alignment_summary_metrics.txt
    │       └── sample2.alignment_summary_metrics.txt
    └── sampleSheet.tsv

The `filtered_bam` directory contains either filtered or unfiltered BAM file, however you prefer. The other two directories contain the output of `bamPEFragmentSize` from deepTools and `AlignmentSummaryMetrics` from picard, respectively.

`sampleSheet.tsv` is only needed for differential accessibility.

Sample sheet
~~~~~~~~~~~~

The (optional) sample sheet is a tab-separated file with two columns, named `name` and `condition`. An example is below::

    name    condition
    sample1      eworo
    sample2      eworo
    SRR7013047      eworo
    SRR7013048      OreR
    SRR7013049      OreR
    SRR7013050      OreR

The condition associated with the first line is used as the base level for comparisons.

Configuration file
~~~~~~~~~~~~~~~~~~

There is a configuration file in `snakePipes/workflows/ATACseq/defaults.yaml`::

    pipeline: ATAC-seq
    configfile:
    cluster_configfile:
    local: false
    max_jobs: 5
    ## workingdir need to be required DNA-mapping output dir, 'outdir' is set to workingdir internally
    workingdir:
    ## preconfigured target genomes (mm9,mm10,dm3,...) , see /path/to/snakemake_workflows/shared/organisms/
    ## Value can be also path to your own genome config file!
    genome:
    ## Bin size of output files in bigWig format
    bw_binsize: 25
    fragmentSize_cutoff: 150
    verbose: false
    # sampleInfo_DB
    sample_info:
    # window_size
    window_size: 20
    fragmentCount_cutoff: 1

The only parameters that are useful to change are `bw_binsize`, `fragmentSize_cutoff`, and `window_size`.
Note, however that those can be more conveniently changed on the command line. `fragmentCount_cutoff` is
introduced to avoid errors in the peak calling step and should only be changed if MACS2 fails.

Output structure
----------------

Assuming a sample sheet is used, the following will be **added** to the working directory::

    .
    ├── CSAW
    │   ├── CSAW.log
    │   ├── CSAW.session_info.txt
    │   ├── DiffBinding_allregions.bed
    │   ├── DiffBinding_analysis.Rdata
    │   ├── DiffBinding_modelfit.pdf
    │   ├── DiffBinding_scores.txt
    │   ├── DiffBinding_significant.bed
    │   ├── QCplots_first_sample.pdf
    │   ├── QCplots_last_sample.pdf
    │   └── TMM_normalizedCounts.pdf
    ├── deepTools_ATAC
    │   └── plotFingerprint
    │       ├── plotFingerprint.metrics.txt
    │       └── plotFingerprint.png
    ├── MACS2
    │   ├── sample1.filtered.BAM_control_lambda.bdg
    │   ├── sample1.filtered.BAM_peaks.narrowPeak
    │   ├── sample1.filtered.BAM_peaks.xls
    │   ├── sample1.filtered.BAM_summits.bed
    │   ├── sample1.filtered.BAM_treat_pileup.bdg
    │   ├── sample1.short.metrics
    │   ├── sample2.filtered.BAM_control_lambda.bdg
    │   ├── sample2.filtered.BAM_peaks.narrowPeak
    │   ├── sample2.filtered.BAM_peaks.xls
    │   ├── sample2.filtered.BAM_summits.bed
    │   ├── sample2.filtered.BAM_treat_pileup.bdg
    │   └── sample2.short.metrics
    └── MACS2_QC
        ├── sample1.filtered.BAM_peaks.qc.txt
        └── sample2.filtered.BAM_peaks.qc.txt

There are additionally log files in most of the directories. The various outputs are documented in the CSAW and MACS2 documentation. The `MACS2_QC` folder contains a number of QC metrics that we find useful, namely the number of peaks, fraction of reads in peaks (FRiP) and percentage of the genome covered by peaks.

Command line options
--------------------

.. argparse::
    :func: parse_args
    :filename: ../snakePipes/workflows/ATAC-seq/ATAC-seq
    :prog: ATAC-seq
    :nodefault:
