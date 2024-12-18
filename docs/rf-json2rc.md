The RF JSON2RC module performs post-processing of JSON files generated by [DRACO](https://draco-docs.readthedocs.io/en/latest/draco/), into RC files. These can be further processed via the ``rf-norm`` module to obtain normalized reactivity profiles for structure prediction with the ``rf-fold`` module.<br /><br />
![Overview](http://www.incarnatolab.com/images/docs/RNAframework/rf-json2rc.png)
<br />

# Usage
To list the required parameters, simply type:

```bash
$ rf-json2rc -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-o__ *or* __--output-dir__ | string | Output directory for writing counts in RC (RNA Count) format (Default: __rf_json2rc/__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory if already exists
__-j__ *or* __--json__ | string | A comma-separated list of DRACO JSON files from replicate experiments
__-r__ *or* __--rc__ | string | A comma-separated list of RC files from replicate experiments<br/>__Note:__ the RC files must follow the same order of the JSON files
__-rci__ *or* __--rc-index__ | string | A comma-separated list of RCI index files<br/>__Note #1:__ the RCI indexes must follow the same order of the RC files.<br/>__Note #2:__ if a single RCI index is provided, it will be used for all the RC files.
__-ep__ *or* __--median-pre-cov__ | int | Windows with median *preCoverage* (see [DRACO](https://draco-docs.readthedocs.io/en/latest/draco/#output-json-files) docs for more information) below this threshold, will be discarded (Default: __1000__)
__-ec__ *or* __--median-cov__ | int | Windows with a mediam cumulative coverage (the sum of the coverage across all the conformations for that window) below this threshold, will be discarded (Default: __5000__)
__-sz__ *or* __--skip-zero-cluster-wins__ | | Skips windows for which DRACO failed to identify the number of conformations
__-nc__ *or* __--min-confs__ | int | Windows forming less than this number of conformations will be discarded (Default: __2__)
__-xc__ *or* __--max-confs__ | int | Windows forming more than this number of conformations will be discarded (Default: __no limit__)
__-nm__ *or* __--no-merge-overlapping__ | | Disables merging of intra-replicate concordant overlapping windows
__-mom__ *or* __--min-overlap-merge__ | float | Minimum fractional overlap between two concordant overlapping windows to be merged (0-1, Default: __0.5__)
__-mcm__ *or* __--min-corr-merge__ | float | Minimum average correlation between corresponding conformations for concordant overlapping windows to be merged (0-1, Default: __0.7__)
__-e__ *or* __--extend__ | int |  Windows are extended by these many bases upstream and downstream (Default: __off__)<br/>__Note:__ these bases will be assigned a coverage and mutation count of 0
__-sr__ *or* __--surround-to-rc__ | | Instead of getting coverage and mutation count of 0, bases in up/downstream extensions will be assigned the same coverage and mutation count they have in the input RC files (requires ``-e``)
__-i__ *or* __--ignore-terminal__ | float | Coverage and mutation counts for this fraction of bases at window termini will be set to 0 (0-0.2, Default: __0.05__)
__-ki__ *or* __--keep-ignored__ | | Bases ignored during correlation calculation, will be kept in the output RC files<br/>__Note:__ by default, both counts and coverage for these bases is set to 0
__-mor__ *or* __--min-overlap-reps__ | float | Minimum fractional overlap between windows across replicates to be merged (0-1, Default: __0.75__)
__-mcr__ *or* __--min-corr-reps__ | float | Minimum correlation between corresponding conformations for matched windows across replicates, to be reported (0-1, Default: __0.7__)
__-s__ *or* __--spearman__ | | Spearman will be used instead of Pearson for correlation analysis
__-cf__ *or* __--cap-mut-freqs__ | float | Mutation frequencies will be capped to this value for correlation calculation (>0-1, Default: __1__ (no cap))
__-ca__ *or* __--corr-all__ | | Each pairwise comparison between each conformation across every replicate needs to exceed the minimum correlation threshold (Default: only the average of all pairwise comparisons needs to exceed the threshold)
__-cm__ *or* __--corr-by-majority__ | | If two matching windows form *N* conformations, and at least N-1 conformations exceed the minimum correlation threshold, the last conformation is also accepted even if the correlation does not exceed the threshold (requires `-ca`)


<br/>
## Understanding the algorithm
Windows are pre-filtered based on a number of criteria:<br/>

1. The median preCoverage (the coverage calculated only on the reads used for the spectral analysis) must be &ge; ``--median-pre-cov``
2. The median cumulative coverage (the sum of the coverage across all the conformations for a given window) must be &ge; ``--median-cov``
3. If ``--skip-zero-cluster-wins`` has been specified, windows for which DRACO failed to identify the number of conformations will be discarded, otherwise they will be assumed to form a single conformation
4. Windows forming &lt; ``--min-confs`` conformations will be discarded
5. Windows forming &gt; ``--max-confs`` conformations will be discarded

DRACO analysis is performed in sliding windows. When two consecutive windows are found to form the same number of conformations, they are automatically merged. However, in certain cases, it might be possible for two non-consecutive overlapping windows forming the same number of conformations, to be interspersed among windows forming a different number of conformations.<br/>Let's consider the following example:
<br/><br/>
![Merging windows](http://www.incarnatolab.com/images/docs/RNAframework/rf-json2rc_mergewins.png)
<br/><br/>
In this case, four windows overlap. Of these, three form 2 conformations, while one forms 3 conformations. As long as the ``--no-merge-overlapping`` parameter has not been specified, the first step of the analysis will consist of merging overlapping windows forming a concordant number of conformations. The minimum overlap between two windows must be &ge; ``--min-overlap-merge`` &times; the length of the smaller window. In the above example, *Win #3* overlaps by 70% of its length with *Win #1*. Similarly, 90% of *Win #4* overlaps with *Win #3*, therefore all three windows could in principle be merged (with default parameters).<br/>
Before being merged, however, the overlapping segments of the conformations making up the two overlapping windows need to be *matched*.<br/>
<br/><br/>
![Window correlation](http://www.incarnatolab.com/images/docs/RNAframework/rf-json2rc_correlation.png)
<br/><br/>
To this end, the pairwise correlation between the reactivity profile for each conformation of the two windows is calculated at the level of the overlap. Any possible combination is evaluated, and the combination yielding the highest average correlation coefficient is selected. If `--corr-all` is enabled, all pairwise comparisons need to exceed the ``--min-corr-merge`` threshold. So, for example, for two overlapping windows, each one forming 3 conformations, to be merged, the pairwise correlation coefficient for all 3 conformations must exceed the threshold. However, if `--corr-by-majority` is also enabled, it is sufficient for 2 out of 3 comparisons to exceed  the correlation threshold, and the last conformation will be matched even if its correlation is lower. Alternatively, if `--corr-all` is not enabled, it is sufficient that the average of the correlation coefficients across all conformations exceeds the ``--min-corr-merge`` threshold, to enable merging of the windows. If none of the above conditions is met, the two windows are not merged. Pearson correlation is used by default; alternatively, Spearman correlation can be used, by specifying the ``--spearman`` parameter. The stoichiometries of the different conformations for the merged windows are averaged.<br/>
If a single JSON file has been provided, the resulting windows are then directly reported in the RC file. If multiple JSON files have been provided, instead, the algorithm will first look for windows common to all replicates.<br/>
<br/><br/>
![Window correlation](http://www.incarnatolab.com/images/docs/RNAframework/rf-json2rc_mergereps.png)
<br/><br/>
Windows overlapping across all the replicates, coherently forming the same number of conformations, are merged if the minimum overlap is &ge; ``--min-overlap-reps`` &times; the length of the smaller window. Only windows common to all replicates will be reported. In the above example *Win #1* and *Win #2* from *Replicate #1* overlap with their counterpart from *Replicate #2*, while *Win #3* does not; therefore, only *Win #1* and *Win 2* could in principle be reported. Analogously to what happens when merging overlapping windows within the same replicate, also in this case the pairwise correlation between the reactivity profile for each conformation of the windows is calculated, and the one yielding the highest average correlation coefficient is selected. Merging of windows across experiments follows the same rules as described above for merging of overlapping windows within the same experiment.<br/>
An RC file will be generated for each replicate being analyzed. Naming and sorting of the windows is consistent across replicates. For each window, different conformations are marked by the __\_c<*n*>__ suffix, where __*n*__ is an arbitrary number assigned to a specific conformation. When reporting windows in the output RC file, these can be enlarged both upstream and downstream by ``--extend`` bases, to account for the possibility that the structure(s) formed by a given window might involve extra bases outside of the window's boundaries. By default, these extra bases are assigned both mutation count and coverage of 0. If ``--surround-to-rc`` has been specified, however, the mutation count and coverage for these bases will be directly extracted from the corresponding RC file provided via ``--rc``. This file is supposed to be the RC file generated by ``rf-count`` alongside the MM file that has been analyzed with DRACO. Furthermore, to account for the lower reliability of bases closer to window boundaries, up to 20% of the terminal bases in a window can be masked by specifying the ``--ignore-terminal`` parameter; when doing so, mutation counts and coverage for these bases will be set to 0.<br/>
Alongside with the RC files, the __stoichiometries.txt__ file will be generated, with the following structure:

```text
Transcript      Start   End     extStart    extEnd  Replicate_1         Replicate_2
Transcript_1    704     1022    654         1072    0.562;0.438         0.541;0.459
Transcript_1    1024    1358    974         1408    0.537;0.463         0.537;0.463
...
Transcript_n    27984   28294   27934       28344   0.570;0.430         0.510;0.490
Transcript_n    29184   29358   29134       29408   0.380;0.314;0.307   0.344;0.299;357
```
where __Transcript__ is the transcript ID, __start__ and __end__ are the coordinates (0-based) of the window, and __extStart__ and __extEnd__ are the coordinates of the window after being extended by ``--extend`` bases. Following these columns, a column will be present for each replicate having been analyzed, reporting the relative stoichiometries of the conformations for that window.
