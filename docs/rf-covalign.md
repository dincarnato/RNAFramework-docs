The RF CovAlign module implements the `cm-builder` approach to iteratively build and refine structure-informed alignments of homologous sequences, starting from one or more RNA structure motifs (e.g., substructures extracted by [RF StructExtract](https://rnaframework-docs.readthedocs.io/en/latest/rf-structextract/)), and evaluates the statistical support of the base-pairs of each motif by covariation analysis.<br/>
Starting from the motif's sequence and structure, a covariance model (CM) is built with [Infernal]() and used to search a database of potential homologs. The matches passing the selection criteria are realigned to the model, the model is rebuilt from the resulting alignment, and the procedure is repeated for multiple rounds. At each iteration, the resulting alignment is evaluated with [R-scape](https://eddylab.org/infernal/), to measure how many base-pairs of the original structure significantly covary.
<br/><br/>

!!! note "Note"
    RF CovAlign requires the [__Infernal__](http://eddylab.org/infernal/) suite (``cmbuild``, ``cmcalibrate``, ``cmsearch``, ``cmalign`` and ``cmpress``) and [__R-scape__](http://eddylab.org/R-scape/) (unless the ``-r`` (or ``--skipRscape``) parameter is specified).


# Usage

```bash
$ rf-covalign [options] -m motifs.db -s sequence.fasta -d homologs.fasta
```

To list all available parameters, simply type:

```bash
$ rf-covalign -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors to use for Infernal processes (Default: __1__)
__-o__ *or* __--output__ | string | Path to the output directory (Default: __./rf_covalign__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory (if it already exists)
__-m__ *or* __--motifFile__ | string | Path to the RNA structure motif's file (in Vienna format)
__-d__ *or* __--dbFile__ | string | Path to the DB file of potential homologs (in FASTA format)
__-s__ *or* __--sequenceFile__ | string | Path to the original sequence file, from which the motif has been extracted (in FASTA format)
__-gm__ *or* __--groupMotifs__ | | Motifs sharing the same sequence will be grouped and simultaneously used for alignment construction, retaining at each iteration only the sequences matched by all of them
__-g__ *or* __--onGenome__ | | Assumes that the covariance model is built by using a database of genomic sequences, hence matches on both genomic strands must be retained
__-b__ *or* __--minBpFraction__ | float | Fraction of base-pairs that should be present in each sequence to retain it (Default: __0.5__ [50%])
__-mc__ *or* __--minQueryCov__ | float | Minimum query motif coverage, in case of truncated motifs (Default: __0.75__ [75%])
__-ng__ *or* __--noGapsInCov__ | | Gaps in the match between the CM and the target sequence are subtracted when calculating the query coverage
__-t__ *or* __--posTolerance__ | float | Maximum fractional tolerance for shift in position of the motif (Default: __1__ [100%])<br/>__Note:__ when working on genomic sequences (``-g``), it is advisable not to change this
__-l__ *or* __--minLen__ | int | If the provided motif is shorter than this length, it is first enlarged up to this length before performing the search, to maximize the chances of finding matches (Default: __100__)
__-em__ *or* __--enlargeMotif__ | int | Enlarges the motif by this number of bases in both directions before performing the search (Default: __0__)<br/>__Note:__ if ``--minLen`` is greater than the motif enlarged by ``--enlargeMotif`` in both directions, this parameter is ignored
__-i__ *or* __--iterations__ | int | Number of iterations to perform to refine the covariance model (Default: __10__)
__-f__ *or* __--stopIfFirstZero__ | | If 0 covarying base-pairs are found at the first iteration, the analysis of the motif stops immediately, instead of going to the next iteration
__-P__ *or* __--pathToInfernal__ | string | Path to the folder containing the Infernal executables (Default: assumes all executables are in PATH)
__-kl__ *or* __--keepLogs__ | | Keeps the output of the external programs (``cmbuild``, ``cmsearch``, ``cmcalibrate``, ``cmalign``, ``cmpress``, ``R-scape``) under the ``logs/`` subfolder of the output directory, one subfolder per model
__-ca__ *or* __--coloredAln__ | | Colored Stockholm alignments in HTML format, with alignment columns colored by helix, are written under the ``html/`` subfolder of the output directory
__-da__ *or* __--darkAln__ | | Uses a dark coloring scheme for the colored Stockholm alignments (requires ``-ca``)
 | | __Covariation analysis options__
__-r__ *or* __--skipRscape__ | | Skips the alignment evaluation step with R-scape
__-R__ *or* __--pathToRscape__ | string | Path to the R-scape executable (Default: assumes ``R-scape`` is in PATH)
__-e__ *or* __--rscapeEvalue__ | float | Threshold E-value for considering a base-pair to significantly covary (Default: __0.1__)
__-a__ *or* __--useHelixAggregatedEvalue__ | | At each iteration, the number of significantly covarying helices is evaluated, instead of the number of significantly covarying base-pairs (requires R-scape v2.0.0.q or greater)
__-cp__ *or* __--includeCompatiblePairs__ | | When calculating the number of significantly covarying base-pairs, also includes the base-pairs that are compatible with (but absent in) the provided structure
 | | __Bit score-based model building (default, faster)__
__-T__ *or* __--bitScore__ | float | Bit score threshold for including a match (Default: __20__)
__-I__ *or* __--iterativeBitScoreIncrease__ | float | The bit score threshold (``-T``) will be increased at each iteration by this value (Default: __0__)
__-N__ *or* __--noiseEstimate__ | | A "decoy" database is generated by reversing a subset of the sequences in the DB file (controlled by ``-ds``), and used to estimate a noise threshold for the bit score. The bit score at each iteration is then rounded up to the nearest multiple of 5<br/>__Note:__ the estimated bit score overrides the bit score threshold (``-T``)
__-ds__ *or* __--decoySize__ | float | Fraction of sequences of the database to be reversed and used to build the decoy database (0.1-1, Default: __0.1__)
__-md__ *or* __--minDecoySize__ | int | Minimum number of sequences to be included in the decoy database (&gt;0, Default: __1000__)<br/>__Note:__ if this number is larger than the number of sequences in the original database, the whole database will be reversed and used as a decoy
 | | __E-value-based model building (slow, requires model calibration)__
__-U__ *or* __--useEvalue__ | | Selects matches by E-value instead of bit score (parameters ``-T`` and ``-I`` will be ignored)<br/>__Note:__ bit scores are the default. E-value mode additionally requires the covariance model to be calibrated with ``cmcalibrate`` at every iteration, which is by far the slowest step of a run
__-E__ *or* __--evalue__ | float | Threshold E-value for including a match (Default: __10__)
__-D__ *or* __--iterativeEvalueDecrease__ | float | The threshold E-value (``-E``) will be decreased at each iteration by this fold (for example, with ``-E 10`` and ``-D 2``, the E-value threshold will become 5 at the second iteration, 2.5 at the third iteration, and so on) (&ge;2, Default: __0__)

<br/>

## Understanding the algorithm
Each motif is refined independently. Before the iterations begin, the sequence database is indexed (via ``samtools faidx`` if SAMTools is available, otherwise the index is built by RF CovAlign itself), so that any sequence can later be pulled out by ID without walking the whole file, and the motif is prepared as follows:
 
1. The motif is located within the reference sequence provided via ``-s`` (or ``--sequenceFile``). When ``-g`` (or ``--onGenome``) is specified, the reverse complement is searched as well. Motifs that cannot be found are skipped
2. If the motif is shorter than ``-l`` (or ``--minLen``), or if ``-em`` (or ``--enlargeMotif``) has been specified, it is extended on both sides with the flanking sequence taken from the reference, and its structures are padded accordingly. A short motif carries too little information for the covariance model to discriminate real homologs, hence this step maximizes the chances of finding meaningful matches
3. An initial single-sequence Stockholm alignment is generated from the (possibly enlarged) motif and its structure

<br/>
Each iteration then performs the following steps:

1. A covariance model is built from the current alignment with ``cmbuild``
2. The model is calibrated. In E-value mode (``-U``) this is done with ``cmcalibrate``, which is by far the slowest step of a run; in the default bit score mode, calibration is skipped and only the model's ECM fields are filled in, as E-values are not needed to select the matches
3. If ``-N`` (or ``--noiseEstimate``) is specified, the model is first searched against the decoy database (generated by reversing a subset of the sequences in the database of candidate homologs). Every decoy match that passes the filters of step 4 raises the bit score threshold to the next multiple of 5 above its own score, so that the threshold ends up just above the best score achievable on sequences that cannot be real homologs
4. The model is searched against the sequence database with ``cmsearch``, and the best-scoring match is retained only if:

	- in E-value mode, its E-value does not exceed ``-E`` (or ``--evalue``)
	- the fraction of the model covered by the match is at least ``-mc`` (or ``--minQueryCov``). When ``-ng`` (or ``--noGapsInCov``) is specified, gaps in the match are subtracted from the covered length
	- its relative position within the target sequence falls within ``-t`` (or ``--posTolerance``) of the motif's relative position within the reference sequence. This filter is disabled at the default value of 1 (100%), and is mostly useful when the motif is expected at a conserved position, such as a given distance from the 5'-end of a viral genome
	- in bit score mode, its bit score reaches the current threshold<br/>
	
5. The matched subsequences are extracted from the database (reverse-complemented when ``-g`` is specified and the match lies on the minus strand) and aligned to the model with ``cmalign``
6. The resulting alignment is filtered. First, the whole alignment is discarded if its consensus structure retains less than ``-b`` (or ``--minBpFraction``) of the base-pairs of the original motif; this guards against the model drifting towards a different structure. Then, each individual sequence is discarded unless it can form at least ``-b`` of those base-pairs as canonical pairs
7. When a motif carries more than one structure (see ["__Motifs with multiple conformations__"](https://rnaframework-docs.readthedocs.io/en/latest/rf-covalign/#motifs-with-multiple-conformations) below), only the sequences retained across *all* of its structures are carried forward, and each alignment is reduced to that common set. Sequences that became identical after trimming are collapsed to a single representative, so that covariation is not inflated by duplicates
8. Unless ``-r`` (or ``--skipRscape``) is specified, each alignment is evaluated with R-scape, and the number of significantly covarying base-pairs (or helices, with ``-a``) is counted
9. The bit score threshold is increased by ``-I``, or the E-value threshold divided by ``-D``, and the next iteration starts from the alignment just produced

<br/>

The procedure stops before reaching ``-i`` iterations when the alignment stops improving, namely when the number of covarying base-pairs (or helices) of *any* of the motif's structures is lower than it was at the previous iteration. A count that stays the same is not considered a worsening, and the refinement continues. The run also stops if no covariation at all is detected at an iteration past the first, or, with ``-f`` (or ``--stopIfFirstZero``), already at the first one. The alignments written to the output are those of the last iteration that completed successfully, i.e. the last ones before the counts started decreasing.
 
<br/>

### Motifs with multiple conformations
A motif can be associated with more than one structure. This happens naturally when the motifs come from an experiment resolving alternative conformations (e.g., ``rf-json2rc`` appends a conformation ID (``_c0``, ``_c1``, ...) to the transcript name, and the same region can therefore appear with two or more distinct structures).<br/>
When ``-gm`` (or ``--groupMotifs``) is specified, the conformation ID is stripped, and all the entries sharing the same sequence are grouped into a single motif carrying several structures. Each structure is then searched and aligned independently, but, as detailed above, only the sequences matched by *all* of them are retained at each iteration. The resulting alignments therefore describe the same set of homologs under each of the alternative conformations, which makes their covariation support directly comparable, and the summary table reports one row per structure.

<br/>

## Choosing between bit score and E-value

Matches can be selected either by bit score (the default) or by E-value (``-U``, or ``--useEvalue``). The two modes differ substantially in runtime: E-value mode requires the covariance model to be calibrated with ``cmcalibrate`` at every iteration, which is by far the slowest step of a run. Bit score mode skips calibration entirely, and is therefore the recommended choice for most analyses.<br/>
In both modes, the threshold can be made progressively more stringent across iterations, via ``-I`` (or ``--iterativeBitScoreIncrease``) and ``-D`` (or ``--iterativeEvalueDecrease``) respectively. This is useful to let the model first gather a broad set of candidate homologs, and then progressively discard the weakest matches as the model gets more informative.<br/>
As an alternative to picking a bit score threshold manually, the ``-N`` (or ``--noiseEstimate``) parameter lets RF CovAlign determine it from the data. A decoy database is built by reversing a fraction of the sequences of the DB file (controlled by ``-ds`` and ``-md``), the model is searched against it, and the resulting score distribution is used to set a threshold above which matches are unlikely to be noise.

<br/>

## Output files

Output file/folder | Description
-----------------: |:------------
``alignments/`` | One Stockholm alignment per motif, containing the sequences retained at the last iteration, with the consensus structure annotation (``SS_cons``)
``images/`` | One PDF per motif, containing the R-scape consensus structure, with the significantly covarying base-pairs highlighted (not generated if ``-r`` is set)
``html/`` | One HTML rendering of each Stockholm alignment, with the columns colored by helix (requires ``-ca``)
``logs/`` | The raw output of the Infernal/R-scape analyses, one subfolder per model (requires ``-kl``)

<br/>
At the end of the run, a summary table is displayed, reporting for each motif the number of significantly covarying base-pairs and helices (__Note:__ only motifs with at least one significant pair/helix are listed).