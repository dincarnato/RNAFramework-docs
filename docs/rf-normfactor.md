The RF NormFactor module takes an arbitrary number of RC files (for example corrisponding to multiple replicates or different conditions of the same experiment) and derives transcriptome-wide (and experiment-wide) normalization factors for use with ``rf-norm``.<br/>For details on the different scoring and normalization methods, please refer to the [RF Norm](https://rnaframework-docs.readthedocs.io/en/latest/rf-norm/) page.
<br/><br/>

# Usage

```bash
$ rf-normfactor [options] -t treated.rc
$ rf-normfactor [options] -t treated.rc -u untreated.rc
$ rf-normfactor [options] -t treated.rc -u untreated.rc -d denatured.rc
```

To list all available parameters, simply type:

```bash
$ rf-normfactor -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-u__ *or* __--untreated__ | string[,string] | A comma-separated list of untreated sample RC files (required by Ding/Siegfried scoring methods)<br/>__Note:__ if a single file is passed, this will be used for all treated files
__-t__ *or* __--treated__ | string[,string] | A comma-separated list of treated sample RC files
__-d__ *or* __--denatured__ | string[,string] | A comma-separated list of denatured sample RC files (optional for Siegfried scoring methods)<br/>__Note:__ if a single file is passed, this will be used for all treated files
__-i__ *or* __--index__ | string |  RCI index file
__-wl__ *or* __--whitelist__ | string | A text file containing a list of IDs of transcripts to be used for calculating normalization factors (Default: __use all transcripts__)
__-p__ *or* __--processors__ | int | Number of processors (threads) to use (Default: __1__)
__-o__ *or* __--output__ | string | Output file normalization factors will be reported to (Default: __norm_factors.txt__)
__-ow__ *or* __--overwrite__ | | Overwrites the output file (if it already exists)
__-sm__ *or* __--scoring-method__ | int | Method for score calculation (1-4, Default: __1__):<br/>__1.__ Ding *et al.*, 2014 <br/>__2.__ Rouskin *et al.*, 2014 <br/>__3.__ Siegfried *et al.*, 2014<br/>__4.__ Zubradt *et al.*, 2016
__-nm__ *or* __--norm-method__ | int | Method for signal normalization (1-3, Default: __1__):<br/>__1.__ 2-8% Normalization <br/>__2.__ 90% Winsorizing <br/>__3.__ Box-plot Normalization
__-rb__ *or* __--reactive-bases__ | string | Reactive bases to consider for signal normalization (Default: __all__ [ACGT])<br/>__Note:__ This parameter accepts any IUPAC code, or their combination (e.g. ``-rb M``, or ``-rb AC``)
__-mc__ *or* __--min-coverage__ | int | Discards any base with coverage below this threshold (&ge;1, Default: __10__)
__-ec__ *or* __--median-coverage__ | float | Discards transcripts having median coverage below this threshold (&ge; 0, Default: __0__)
__-rn__ *or* __--run-norm__ | | Automatically runs `rf-norm` with the derived normalization factors on the input files<br/>__Note:__ the default output folder name of rf-norm will be used, with appended `_normfactor`  suffix. Any pre-existing folder with the same name wil be overwritten
__-rf__ *or* __--rf-norm__ | | Path to rf-norm executable (Default: assumes `rf-norm` is in PATH) 
 | | __Scoring method #1 options (Ding *et al*., 2014)__
__-pc__ *or* __--pseudocount__ | float | Pseudocount added to reactivities to avoid division by 0 (&gt;0, Default: __1__)
__-s__ *or* __--max-score__ | float | Score threshold for capping raw reactivities (&gt;0, Default: __10__)
 | | __Scoring method #3 options (Siegfried *et al*., 2014)__
__-mu__ *or* __--max-untreated-mut__ | float | Maximum per-base mutation rate in untreated sample (&le;1, Default: __0.05__ [5%])
 | | __Scoring methods #1 and #3 options (Ding et al., 2014 & Siegfried et al., 2014)__
__-il__ *or* __--ignore-lower-than-untreated__ | | Bases having raw reactivity in the treated sample lower than the untreated control, will be ignored (not used during reactivity normalization)
 | | __Scoring methods #3 and #4 options (mutational profiling)__
__-mm__ *or* __--max-mutation-rate__ | float | Maximum per-base mutation rate (&le;1, Default: __1__ [100%])