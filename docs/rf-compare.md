RF Compare allows comparing RF Fold-inferred secondary structures, with a reference of known secondary structures, reporting for each comparison 4 metrics: the PPV, the sensitivity, the FMI (Fowlkes-Mallows index) and the mFMI (modified FMI). For additional details, check the [Metrics](https://rnaframework-docs.readthedocs.io/en/latest/rf-compare/#metrics) section below.<br/>
Reference structures can be provided both in Vienna (dot-bracket), or in CT format. Since version 2.8.8, reference structures can be provided both as a single file containing multiple structures, or as a folder of individual structure files.<br/>The sequence ID of the reference structures __must__ match the compared file's name (e.g. "Transcript#1" expects a file named "Transcript#1.ct" or "Transcript#1.db").<br/>RF Compare can be invoked both on a single structure, or on an entire folder of RF Fold-predicted structure files. Structures can be provided either in CT or Vienna (dot-bracket) format.<br/>
RF Compare can further generates vector graphical reports (SVG format) for each structure, reporting the reference structure and the compared structure, with base-pairs colored according to their presence in both structures:<br/><br/>
![RF Compare plot](http://www.incarnatolab.com/images/docs/RNAframework/rf-compare.png)
<br/><br/>

## Metrics
Given a reference and a predicted structure as input, RF Compare calculates 4 metrics. Each metric ranges between 0 (more dissimilar structures) and 1 (more similar structures):<br/>

### Positive Predictive Value (PPV)
The fraction of base-pairs present in the predicted structure that are also present in the reference structure

### Sensitivity
The fraction of base-pairs present in the reference structure that are also present in the predicted structure

### Fowlkes-Mallows index (FMI)
The geometric mean of PPV and sensitivity (introduced by Deigan *et al.*, 2009, PMID:[19109441](https://pubmed.ncbi.nlm.nih.gov/19109441/)):<br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><mi>F</mi><mi>M</mi><mi>I</mi><mo>&#xa0;</mo><mo>=</mo><mo>&#xa0;</mo><mfrac><mrow><msub><mi>P</mi><mrow><mi>b</mi><mi>o</mi><mi>t</mi><mi>h</mi></mrow></msub></mrow><msqrt><mrow><mfenced><mrow><msub><mi>P</mi><mrow><mi>b</mi><mi>o</mi><mi>t</mi><mi>h</mi></mrow></msub><mo>+</mo><msub><mi>P</mi><mrow><mi>r</mi><mi>e</mi><mi>f</mi></mrow></msub></mrow></mfenced><mfenced><mrow><msub><mi>P</mi><mrow><mi>b</mi><mi>o</mi><mi>t</mi><mi>h</mi></mrow></msub><mo>+</mo><msub><mi>P</mi><mrow><mi>p</mi><mi>r</mi><mi>e</mi><mi>d</mi></mrow></msub></mrow></mfenced></mrow></msqrt></mfrac></math>

<br/>
where *P<sub>both</sub>* is the number of base-pairs common to both reference and predicted structure, while *P<sub>ref</sub>* and *P<sub>pred</sub>* are the numbers of base-pairs respectively unique to reference and predicted structures.

### Modified Fowlkes-Mallows index (mFMI)
A variant of the FMI (introduced by Lan *et al.*, 2022, PMID:[35236847](https://pubmed.ncbi.nlm.nih.gov/35236847/)), which also rewards bases that are unpaired both in the reference and in the predicted structure:<br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><mi>m</mi><mi>F</mi><mi>M</mi><mi>I</mi><mo>&#xA0;</mo><mo>=</mo><mo>&#xA0;</mo><mi>u</mi><mo>+</mo><mfenced><mrow><mn>1</mn><mo>-</mo><mi>u</mi></mrow></mfenced><mo>&#xd7;</mo><mi>F</mi><mi>M</mi><mi>I</mi></math>

<br/>
where *u* is the number of unpaired bases common to both reference and predicted structure.


# Usage
To list the required parameters, simply type:

```bash
$ rf-compare -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors to use (&ge; 1; Default: __1__)
__-r__ *or* __--reference__ | string | Path to a file containing reference structures in Vienna format (dot-bracket)
__-g__ *or* __--img__ | | Enables generation of graphical comparison images
__-o__ *or* __--output-dir__ | string | Images output directory (Default: __rf_compare/__, requires ``-g``)
__-ow__ *or* __--overwrite__ | | Overwrites output directory (if the specified path already exists)
__-x__ *or* __--relaxed__ | | Uses relaxed criteria (described in Deigan *et al.*, 2009) to calculate PPV and sensitivity
__-kp__ *or* __--keep-pseudoknots__ | | Keeps pseudoknotted basepairs in reference structure
__-kl__ *or* __--keep-lonelypairs__ | | Keeps isolated base-pairs (helices of length 1 bp) in reference structure
__-i__ *or* __--ignore-sequence__ | | Ignores sequence differences (e.g. SNVs) between the compared structures

!!! note "Note"
    When parameter ``--relaxed`` is specified, a basepair i-j is considered to be present in the reference structure if any of the following pairs exist: i/j; i-1/j; i+1/j; i/j-1; i/j+1. For additional details, please refer to Deigan *et al*., 2009 (PMID: [19109441](https://www.ncbi.nlm.nih.gov/pubmed/19109441))