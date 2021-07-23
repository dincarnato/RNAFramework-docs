RF Combine allows combining XML files from multiple experiments into a single profile.<br/>
For example, this can be useful when performing CIRS-seq experiments, to combine into a single profile both the reactivity of A/C residues probed with DMS, and of G/U residues probed with CMCT.<br/>
Alternatively, RF Combine is able to combine into a single profile multiple replicates of the same probing experiment. In these cases, the resulting XML files may contain optional “-error” tags, in which the per-base standard deviation of the measure from each experiment is reported.<br/>
RF Combine further allows retaining only transcripts whose Pearson correlation coefficient exceeds a user-defined threshold.

!!! warning "Important"
    RF Combine does not allow combining RF Norm XML files generated using different scoring/normalization methods, since this will produce inconsistent data.
    
!!! note "Note"
    In XML files generated using RF Combine, the ``combined`` attribute of the ``transcript`` tag is set to ``TRUE``.

# Usage
To list the required parameters, simply type:

```bash
$ rf-combine -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors (threads) to use (Default: __1__)
__-o__ *or* __--output-dir__ | string | Output directory for writing combined data in XML format (Default: __combined/__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory if already exists
__-s__ *or* __--stdev__ | | When combining multiple replicates, an optional "-error" tag will be reported in the output XML files, containing the per-base standard deviation of the measure
__-d__ *or* __--decimals__ | int | Number of decimals for reporting reactivities (1-10, Default: __3__)
__-m__ *or* __--min-values__ | float | Minimum number of values to calculate correlation (Default: __off__)<br/>__Note:__ if a value between 0 and 1 is provided, this is interpreted as a fraction of the transcript's length 
__-c__ *or* __--min-correlation__ | float | Minimum correlation to report a combined profile (-1&lt;r&lt;1, Default: __off__)<br/>__Note:__ if more than two replicates are being combined, RF Combine requires this threshold to be satisfied all pairwise comparisons
__-S__ *or* __--spearman__ | | Uses Spearman instead of Pearson to calculate correlation
__-l__ *or* __--log-transform__ | | Log transforms reactivity values before averaging

!!! note "Note"
    When ``--min-values`` specified value is interpreted as a fraction of the transcript's length, only reactive bases (specified by the XML ``reactive`` attribute; for additional details, please refer to the [RF Norm documentation](https://rnaframework-docs.readthedocs.io/en/latest/rf-norm/)) are considered. For example, if a transcript containing 25% of each base has been modified with DMS (than only modifies A/C residues), setting ``--min-values`` to 0.5 will cause RF Combine to skip the transcript if more than 50% of the A/C residues are NaNs.