RF Correlate allows calculating pairwise correlations of structure probing experiments. It can be invoked either on individual transcripts or on whole XML folders, as well as RC files.<br/>
Overall, as well as per-transcript correlations are reported in TSV format.

!!! note "Note"
    When directly comparing two XML files, no check is made on the transcript ID, hence allowing the direct correlation of any two XML files (provided that they are of the same length).
<br/>

# Usage
To list the required parameters, simply type:

```bash
$ rf-correlate -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors to use (Default: __1__)
__-o__ *or* __--output__ | string | Output CSV file (Default: __rf_correlate.txt__)
__-ow__ *or* __--overwrite__ | | Overwrites output file (if the specified file already exists)
__-m__ *or* __--min-values__ | float | Minimum number of values to calculate correlation (Default: __off__)<br/>__Note:__ if a value between 0 and 1 is provided, this is interpreted as a fraction of the transcript's length 
__-cr__ *or* __--cap-react__ | float | Maximum reactivity value to cap reactivities to (&gt; 0, Default: __1e9__)<br/>__Note:__ if processing RC files, this parameter only applies to ratios (`-r`)
__-mr__ *or* __--max-react__ | float |  Reactivity values above this threshold will be excluded from correlation calculation (&gt; 0, Default: __none__)<br/>__Note:__ if processing RC files, this parameter only applies to ratios (`-r`)
__-s__ *or* __--skip-overall__ | | Skips overall experiment correlation calculation (faster)
__-i__ *or* __--ingore-sequence__ | | Ignores sequence differences (e.g. SNVs) between the compared transcripts
__-S__ *or* __--spearman__ | | Uses Spearman instead of Pearson to calculate correlation
 | | __RC file-specific options__
__--kb__ *or* __--keep-bases__ | string | Bases on which correlation should be calculated (Default: __all__)<br/>__Note:__ this option has effect only on RC files. For XML files, reactive bases are automatically identified from the ``reactive`` attribute
__-mc__ *or* __--min-coverage__ | int | Restricts the correlation analysis to bases exceeding this coverage
__-ec__ *or* __--median-coverage__ | float | Restricts the correlation analysis to transcripts with median coverage above this threshold (&ge; 0, Default: __0__)
__-c__ *or* __--coverage__ | | Correlation is calculated on the coverage, rather than on the raw RT stop/mutation counts
__-r__ *or* __--ratio__ | | Correlation is calculated on the ratio between, the RT stop/mutation counts and the coverage, rather than on the raw RT stop/mutation counts

!!! note "Note"
    When ``--min-values`` specified value is interpreted as a fraction of the transcript's length, only reactive bases (specified by the ``reactive`` [attribute](https://rnaframework-docs.readthedocs.io/en/latest/rf-norm/) for XML files, or via the ``--keep-bases`` parameter for RC files) are considered. For example, if a transcript containing 25% of each base has been modified with DMS (than only modifies A/C residues), setting ``--min-values`` to 0.5 will cause RF Correlate to skip the transcript if more than 50% of the A/C residues are NaNs (or do not exceed the ``--min-coverage`` threshold for RC files).
    
<br/>
## Output file
RF Correlate generates a TSV file containing 3 fields:

1. Transcript ID
2. Correlation coefficient
3. P-value
