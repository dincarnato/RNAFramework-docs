The RF JackKnife takes one or more XML reactivity files, and a set of reference RNA structures in dotbracket notation, and iteratively calls ``rf-fold`` by tuning the slope and intercept folding parameters. This is useful to calibrate the folding parameters for a specific probing reagent or experiment type.<br/>
It produces a CSV table the FMI (Fowlkes-Mallows index, the geometric mean of PPV and sensitivity), or the mFMI (modified FMI, for additional details check the [Metrics](https://rnaframework-docs.readthedocs.io/en/latest/rf-compare/#metrics) section of RF Compare) for each slope/intercept pair:
<br/>
![PPV Sensitivity table](http://www.incarnatolab.com/images/docs/RNAframework/rf-jackknife_PPV_Sensitivity.png)
<br/>

## Combining multiple experiments
Since version __2.9.0__ it is possible to identify a single slope/intercept pair that yields the best prediction across multiple experiments.<br/>

In the follwing example:

```bash
$ rf-jackknife -x -r reference.db experiment_1/ experiment_2/ experiment_3/
```

slope and intercept will be optimized on the reference transcripts present in `reference.db`. All transcripts, including those that are present only in a subset of the experiments, will by default be used for parameter optimization. If parameter `-oc` is enabled, however, only transcripts for which reactivity data is available across all experiments will be used.

For additional details on how multiple replicates are combined into a single prediction, please refer to the "[Combining multiple experiments](https://rnaframework-docs.readthedocs.io/en/latest/rf-fold/#combining-multiple-experiments)" paragraph of the RF Fold's documentation page.
<br/>

# Usage
To list the required parameters, simply type:

```bash
$ rf-jackknife -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-r__ *or* __--reference__ | string | A file containing reference structures in Vienna format (dotbracket notation)
__-oc__ *or* __--only-common__ | | In case of replicates, only transcripts covered across all experiments will be used to derive the optimal slope/intercept pair
__-p__ *or* __--processors__ | int | Number of processors to use (Default: __1__)
__-o__ *or* __--output-dir__ | string | Output directory (Default: __rf_jackknife/__)
__-ow__ *or* __--overwrite__ | | Overwrites output directory (if the specified path already exists)
__-g__ *or* __--img__ | | Generates heatmap of grid search results (requires R)
__-sl__ *or* __--slope__ | float,float | Range of slope values to test (Default: __0,5__)
__-in__ *or* __--intercept__ | float,float | Range of intercept values to test (Default: __-3,0__)
__-ss__ *or* __--slope-step__ | float | Step for testing slope values (Default: __0.2__)
__-is__ *or* __--intercept-step__ | float | Step for testing intercept values (Default: __0.2__)
__-x__ *or* __--relaxed__ | | Uses relaxed criteria (Deigan *et al.*, 2009) to calculate the FMI
__-m__ *or* __--mFMI__ | | Uses modified FMI (mFMI, Lan *et al.*, 2022; for additional details check the [Metrics](https://rnaframework-docs.readthedocs.io/en/latest/rf-compare/#metrics) section of RF Compare) instead of standard FMI to quantify the agreement between predicted and reference structure
__-kp__ *or* __--keep-pseudoknots__ | | Keeps pseudoknotted basepairs in reference structure
__-kl__ *or* __--keep-lonelypairs__ | | Keeps lonely basepairs (helices of length 1 bp) in reference structure
__-i__ *or* __--ignore-sequence__ | | Ignores sequence differences (e.g. SNVs) between the compared structures
__-e__ *or* __--median__ | | The FMI across multiple reference structures is aggregated by median<br/>__Note:__ by default, FMI values are aggregated by geometric mean
__-am__ *or* __--arithmetic-mean__ | | The FMI across multiple reference structures is aggregated by arithmetic mean<br/>__Note:__ by default, FMI values are aggregated by geometric mean
__-rf__ *or* __--rf-fold__ | string | Path to ``rf-fold`` executable (Default: assumes ``rf-fold`` is in PATH)
__-rp__ *or* __--rf-fold-params__ | string | Manually specify additional RF Fold parameters (e.g. -rp "-md 500 -m 2")
__-R__ *or* __--R-path__ | string | Path to R executable (Default: assumes R is in PATH)<br/>__Note:__ also check `$RF_RPATH` under [Environment variables](https://rnaframework-docs.readthedocs.io/en/latest/envvars/#rf_rpath)

<br/>
## Output CSV files
RF JackKnife produces a CSV file reporting the FMI (or mFMI) for each intercept (x-axis) and slope (y-axis) value pair
