Tool              | Description
----------------: | :------------
__rf-index__      | Automatically queries UCSC genome database and builds the transcriptome Bowtie reference index for the RF Map module
__rf-map__        | Performs reads pre-processing and mapping
__rf-count__      | Calculates per-base RT-stops/mutations and coverage from transcriptome-level SAM/BAM files, and generates mutation map files for processing with [DRACO](https://draco-docs.readthedocs.io/)
__rf-count-genome__      | Calculates per-base RT-stops/mutations and coverage from genome-level SAM/BAM files
__rf-norm__       | Performs whole-transcriptome normalization of structure probing data
__rf-fold__       | Produces secondary structures for the analyzed transcripts using structure probing data to guide folding
__rf-structextract__ | Extracts substructures based on certain criteria (e.g. median SHAPE reactivity, median Shannon entropy, stability higher than expected by chance, etc.)
__rf-duplex__       | Analyzes direct RNA-RNA interaction mapping experiments (i.e. COMRADES, SPASH, PARIS, etc.)
__rf-compare__    | Compares secondary structures inferred by ``rf-fold`` with a set of reference structures, and computes PPV and sensitivity
__rf-jackknife__     | Iteratively optimize slope and intercept parameters to maximize PPV and sensitivity using a set of reference structures
__rf-modcall__    | Performs analysis of &Psi;-seq/Pseudo-seq and 2OMe-seq data
__rf-peakcall__   | Performs peak calling of RNA immunoprecipitation (RIP) experiments
__rf-motifdiscovery__  | Discovers significantly enriched sequence motifs in RIP peaks
__rf-combine__    | Combines results of multiple experiments into a single profile
__rf-correlate__ | Calculates pairwise correlation of structure probing experiments
__rf-wiggle__    | Produces WIGGLE track files from RC or XML input files
__rf-rctools__   | Allows manipulating RC files
__rf-mutate__  | Designs structure disrupting (and compensatory) mutations
__rf-json2rc__  | Post-processes DRACO JSON output files into RC files

<br/>
![RNAFramework pipeline](http://www.rnaframework.com/images/overview.png)