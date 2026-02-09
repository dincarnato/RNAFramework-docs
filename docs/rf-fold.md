The RF Fold module is designed to allow transcriptome-wide reconstruction of RNA structures, starting from XML files generated using the RF Norm tool.This tool can process a single, or an entire directory of XML files, and produces the inferred secondary structures (either in dot-bracket notation, or CT format) and their graphical representation (either in Postscript, or SVG format).<br/>Folding inference can be performed using 2 different algorithms:<br/><br/>1. __ViennaRNA__<br/>2. __RNAstructure__<br/><br/>
Prediction can be performed either on the whole transcript, or through a windowed approach (see next paragraph).
<br/><br/>
## Structure modelling    
The structure modelling approach is inspired by the original method described in Siegfried *et al*., 2014 (PMID: [25028896](https://www.ncbi.nlm.nih.gov/pubmed/25028896)). Since version __2.8.0__, the underlying logic of the windowed approach has been slightly changed, by performing the detection of pseudoknots as the last step. The procedure is outlined below:
<br/><br/>
![RNAFramework pipeline](http://www.incarnatolab.com/images/docs/RNAframework/rf-fold_windowed_folding.png)
<br/><br/>
In step I, a window is slid along the RNA (if the `-w` option is enabled, otherwise the entire RNA is used), and partition function is calculated. If provided, soft-constraints from structure probing are applied. Predicted base-pair probabilities are averaged across all windows in which they have appeared, and base-pairs with >99% probability are retained, and hard-constrained to be paired in step III.<br/>
In step II, a window is slid along the RNA, and MFE folding is performed, including (where present) soft-constraints from probing data, and base-pairs from step I. Predicted base-pairs are retained if they appear in >50% of analyzed windows.<br/>
In step III (optional), a window is slid along the RNA, and putative pseudoknots are detected using the same approach employed by the __ShapeKnots__ algorithm (Hajdin *et al.*, 2013 (PMID: [23503844](https://www.ncbi.nlm.nih.gov/pubmed/23503844))). Our implementation of the ShapeKnots algorithm relies on the __ViennaRNA package__ (instead of __RNAstructure__ as in the original implementation), thus it is __much__ faster:
<br/><br/>
![ShapeKnots/RNA Framework comparison](http://www.incarnatolab.com/images/docs/RNAframework/rnaframework_vs_shapeknots.png)
<br/><br/>
Nonetheless, both algorithms work in single thread. Alternatively, the multi-thread implementation ``ShapeKnots-smp`` shipped with the latest __RNAstructure__ version can be used.<br/> 
If constraints from structure probing experiments are provided, these are incorporated in the form of soft-constraints. Predicted pseudoknotted base-pairs are retained if they apper in >50% of analyzed windows and if they do not clash with the nested base-pairs indentified in step II. In case structure probing constraints are provided, pseudoknots are retained only if the average reactivity of bases on both sides of the pseudoknotted helix is below a certain reactivity cutoff.<br/>

!!! note "Note"
    At all stages, increased sampling is performed at the 5'/3'-ends to avoid end biases

Along with the predicted structure, the windowed method also produces a WIGGLE track file containing per-base Shannon entropies.<br/>Regions with higher Shannon entropies are likely to form alternative structures, while those with low Shannon entropies correspond to regions with well-defined RNA structures, or persistent single-strandedness (Siegfried *et al*., 2014).<br/>
Shannon entropy is calculated as: <br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><msub><mi>H</mi><mi>i</mi></msub><mo>=</mo><mo>-</mo> <munderover><mo>&sum;</mo><mrow><mi>j</mi><mo>=</mo><mn>1</mn></mrow><mi>J</mi></munderover><msub><mi>p</mi><mi>i,j&#xA0;</mi></msub><msub><mi>log</mi><mn>10&#xA0;</mn></msub><msub><mi>p</mi><mi>i,j</mi></msub></math><br/>
where *p<sub>i,j</sub>* is the probability of base *i* of being base-paired to base *j*, over all its potential J pairing partners.<br/>
Since version __2.9.1__, RF Fold can use __R__ to generate PDF graphical reports for each structure, reporting reactivity, MEA structure, Shannon entropy, and base-pairing probabilities:<br/><br/>
![Graphical report](http://www.incarnatolab.com/images/docs/RNAframework/rf-fold_graphical_report.png)
<br/><br/>

!!! note "Note"
    The calculation of Shannon entropy and base-pairing probabilities requires partition function to be computed. Since this is a *very slow* step, partition function folding is performed only in windowed mode, or if parameters ``-dp`` (or ``--dotplot``) or ``-sh`` (or ``--shannon``) are explicitly specified.


Additionally, since version __2.9.4__, RF Fold can use the __RNAplot__ tool of the __ViennaRNA package__ (v2.7.0 or greater) to generate SVG secondary structure plots with overlaid reactivities:<br/>
![Graphical report](http://www.incarnatolab.com/images/docs/RNAframework/rf-fold_secondary_structure_plot.png)
<br/>

## Combining multiple experiments
Since version __2.9.0__, RF Fold can combine multiple experiments into a single prediction. These can either be replicates prepared using the same chemical probe, or experiments performed by using different chemical probes.<br/>

The combination is achieved by iterating the folding process (as described in the "[Structure modelling](https://rnaframework-docs.readthedocs.io/en/latest/rf-fold/#structure-modelling)" paragraph above) over all experiments. Windows from all experiments are combined using a *majority voting* approach, so that only the pairs passing the above-detailed thresholds (i.e., pairs with >99% probability and pairs appearing in >50% windows) are retained. Therefore, if, for example, a certain base-pair appears in 100% of the windows for one replicate and in just 25% of the windows for another replicate, it will be included in the final model as, in total, it appears in 62.5% of the windows.<br/>

In the example below:

```bash
$ rf-fold -sl 2.4 -in -0.2 experiment_1/ experiment_2/
```

the structure of the transcripts common to both experiments will be modelled y combining both sets of reactivities, while transcripts unique to either experiments will be modelled using a single set of reactivities.<br/>
Single XML files and entire XML folders can also be combined:

```bash
$ rf-fold -sl 2.4 -in -0.2 experiment_1/ experiment_2/ transcript_1.xml
```

In the above example, __transcript_1__ will be modelled using data coming from both experiment #1 and #2 (if the file `transcript_1.xml` is present in the respective folders), as well as data from `transcript_1.xml`, while the all other transcripts potentially shared across experiments #1 and #2 will only be modelled using those two datasets.<br/>

Importantly, it is possible combining even experiments generated using different chemical probes. In the following example:

```bash
$ rf-fold -sl 2.4 -in -0.2 DMS_data/ CMCT_data/
```

two experiments, one generated using DMS and the other using CMCT, are combined into a single prediction that incorporates both sets of reactivities.<br/>

While in this case, the same slope and intercept values will be used for both experiments, this might not be ideal, as different reagents might require very different slope/intercept pairs. Therefore, RF Fold can also accept multiple slope/intercept pairs, as a comma-separated list, which must follow the same order as the provided experiments, for example:

```bash
$ rf-fold -sl 2.4,4.5 -in -0.2 DMS_data/ CMCT_data/
```

At this point, a slope of 2.4 will be used for the DMS dataset and a slope of 4.5 will be used for the CMCT datset, while the intercept will be -0.2 for both experiments.
<br/>

# Usage

```bash
$ rf-fold [options] XML_dir_1/ XML_dir_2/ .. XML_dir_N/
$ rf-fold [options] rna1.xml rna2.xml .. rnaN.xml
```

To list all available parameters, simply type:

```bash
$ rf-fold -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-o__ *or* __--output-dir__ | string | Output directory for writing inferred structures (Default: __rf\_fold/__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory if already exists
__-ct__ *or* __--connectivity-table__ | | Writes predicted structures in CT format (Default: __Dot-bracket notation__)
__-m__ *or* __--folding-method__ | int | Folding method (1-2, Default: __1__):<br/>__1.__ ViennaRNA <br/>__2.__ RNAstructure
__-p__ *or* __--processors__ | int | Number of processors (threads) to use (Default: __1__)
__-oc__ *or* __--only-common__ | int | In case of multiple experiments, only transcripts covered across at least this number of experiments will be folded
__-g__ *or* __--img__ | | Enables the generation of graphical reports (requires R and, optionally, RNAplot v2.7.0 or greater)
__-vrp__ *or* __--vienna-rnaplot__ | string | Path to ViennaRNA ``RNAplot`` v2.7.0 (or greater) executable (Default: assumes ``RNAplot`` is in PATH)
__-R__ *or* __--R-path__ | string | Path to R executable (Default: assumes R is in PATH)<br/>__Note:__ also check `$RF_RPATH` under [Environment variables](https://rnaframework-docs.readthedocs.io/en/latest/envvars/#rf_rpath)
__-t__ *or* __--temperature__ | float | Temperature in Celsius degrees (Default: __37.0__)
__-sl__ *or* __--slope__ | float | Sets the slope used with structure probing data restraints (Default: __1.8__ [kcal/mol])
__-in__ *or* __--intercept__ | float | Sets the intercept used with structure probing data restraints (Default: __-0.6__ [kcal/mol])
__-md__ *or* __--maximum-distance__ | int | Maximum pairing distance (in nt) between transcript's residues (Default: __0__ [no limit])
__-nlp__ *or* __--no-lonelypairs__ | | Disallows lonely base-pairs (1 bp helices) inside predicted structures
__-i__ *or* __--ignore-reactivity__ | | Ignores XML reactivity data when performing folding (MFE unconstrained prediction)
__-is__ *or* __--ignore-sequence__ | | In case of multiple experiments, nucleotide differences (e.g. SNVs) between XML files are ignored
__-hc__ *or* __--hard-constraint__ | | Besides performing soft-constraint folding, allows specifying a reactivity cutoff (specified by ``-f``) for hard-constraining a base to be single-stranded
__-c__ *or* __--constraints__ | string | Path to a directory containing [constraint files](https://rnaframework-docs.readthedocs.io/en/latest/rf-fold/#constraint-files) (in dot-bracket notation), that will be used to enforce specific base-pairs in the structure models
__-f__ *or* __--cutoff__ | float | Reactivity cutoff for constraining a position as unpaired (&gt;0, Default: __0.7__) 
__-w__ *or* __--windowed__ | | Enables windowed folding
__-pt__ *or* __--partition__ | string | Path to RNAstructure ``partition`` executable (Default: assumes ``partition`` is in PATH)<br/>__Note:__ by default, ``partition-smp`` will be used (if available)
__-pp__ *or* __--probabilityplot__ | string | Path to RNAstructure ``ProbabilityPlot`` executable (Default: assumes ``ProbabilityPlot`` is in PATH)
__-fw__ *or* __--fold-window__ | int | Window size (in nt) for performing MFE folding (>=50, Default: __600__)
__-fo__ *or* __--fold-offset__ | int | Offset (in nt) for MFE folding window sliding (Default: __200__)
__-pw__ *or* __--partition-window__ | int | Window size (in nt) for performing partition function (>=50, Default: __600__)
__-po__ *or* __--partition-offset__ | int | Offset (in nt) for partition function window sliding (Default: __200__)
__-wt__ *or* __--window-trim__ | int | Number of bases to trim from both ends of the partition windows to avoid end biases (Default: __100__)
__-dp__ *or* __--dotplot__ | | Enables generation of dot-plots of base-pairing probabilities
__-sh__ *or* __--shannon-entropy__ | | Enables generation of a WIGGLE track file with per-base Shannon entropies
__-pk__ *or* __--pseudoknots__ | | Enables detection of pseudoknots (computationally intensive)
__-ksl__ *or* __--pseudoknot-slope__ | float | Sets slope used for pseudoknots prediction (Default: same as ``-sl <slope>``)
__-kin__ *or* __--pseudoknot-intercept__ | float | Sets intercept used for pseudoknots prediction (Default: same as ``-in <intercept>``)
__-kp1__ *or* __--pseudoknot-penality1__ | float | Pseudoknot penality P1 (Default: __0.35__)
__-kp2__ *or* __--pseudoknot-penality2__ | float | Pseudoknot penality P2 (Default: __0.65__)
__-kt__ *or* __--pseudoknot-tollerance__ | float | Maximum tollerated deviation of suboptimal structures energy from MFE (>0-1, Default: __0.5__ [50%])
__-kh__ *or* __--pseudoknot-helices__ | int | Number of candidate pseudoknotted helices to evaluate (>0, Default: __100__)
__-kw__ *or* __--pseudoknot-window__ | int | Window size (in nt) for performing pseudoknots detection (>=50, Default: __600__)
__-ko__ *or* __--pseudoknot-offset__ | int | Offset (in nt) for pseudoknots detection window sliding (Default: __200__)
__-kc__ *or* __--pseudoknot-cutoff__ | float | Reactivity cutoff for retaining a pseudoknotted helix (0-1, Default: __0.5__)
__-km__ *or* __--pseudoknot-method__ | int | Algorithm for pseudoknots prediction (1-2, Default: __1__):<br/>__1.__ RNA Framework <br/>__2.__ ShapeKnots<br/>__Note:__ the chosen folding method (specified by ``-m``) affects the algorithm used by RNA Framework (pseudoknot detection method #1) to define the initial MFE structure
 | | __RNA Framework pseudoknots detection algorithm options__
__-vrs__ *or* __--vienna-rnasubopt__ | string | Path to ViennaRNA  ``RNAsubopt`` executable (Default: assumes ``RNAsubopt`` is in PATH)
__-ks__ *or* __--pseudoknot-suboptimal__ | int | Number of suboptimal structures to evaluate for pseudoknots prediction (>0, Default: __1000__)
__-nz__ *or* __--no-zuker__ | | Disables the inclusion of Zuker suboptimal structures (reduces the sampled folding space)
__-zs__ *or* __--zuker-suboptimal__ | | Number of Zuker suboptimal structures to include (>0, Default: __1000__)
 | | __ShapeKnots pseudoknots detection algorithm options__
__-sk__ *or* __--shapeknots__ | string | Path to ``ShapeKnots`` executable (Default: assumes ``ShapeKnots`` is in PATH)<br/>__Note:__ by default, ``ShapeKnots-smp`` will be used (if available)
 | | __Folding method #1 options (ViennaRNA)__
__-vrf__ *or* __--vienna-rnafold__ | string | Path to ViennaRNA ``RNAfold`` executable (Default: assumes ``RNAfold`` is in PATH)
__-ngu__ *or* __--no-closing-gu__ | | Disallows G:U wobbles at the end of helices
 | | __Folding method #2 options (RNAstructure)__
__-rs__ *or* __--rnastructure__ | string | Path to RNAstructure ``Fold`` executable (Default: assumes ``Fold`` is in PATH)<br/>__Note:__ by default, ``Fold-smp`` will be used (if available)
__-d__ *or* __--data-path__ | string | Path to RNAstructure data tables (Default: assumes __DATAPATH__ environment variable is already set)

!!! note "Information"
    For additional details relatively to ViennaRNA soft-constrained prediction, please refer to the [ViennaRNA documentation](http://www.tbi.univie.ac.at/RNA/documentation.html), or to Lorenz *et al*., 2016 (PMID: [26353838](https://www.ncbi.nlm.nih.gov/pubmed/26353838)).

!!! note "Information"
    For additional details relatively to ShapeKnots pseudoknots detection parameters, please refer to Hajdin *et al.*, 2013 (PMID: [23503844](https://www.ncbi.nlm.nih.gov/pubmed/23503844)).
<br/> 
<br/> 
## Constraint files
Constraint files allow forcing base-pairing of certain positions in the RNA. These files are standard dot-bracket files and they must be named after the transcript ID used in the corresponding XML files (for instance, if the XML file is named ``XYZ.xml``, the module will look for a ``XYZ.db`` file in the constraint folder):<br/>
  
```
>XYZ
UUUCGUACGUAGCGAGCGAGUAGCUGAUGCUGAUAGCGGCGAUGCUAGCUGAUCGUAGCGCGCGAUCGAUCGAUGC
..(((.............................................................))).......
```
In the above example, the constraint file instructs the module to force the base-pairing between positions 3-69, 4-68 and 5-67 of the XYZ transcript.

!!! note "Information"
    At present, only nested base-pairs are allowed. Pseudoknotted helices will be automatically discarded. 
<br/> 
<br/>
## Output dot-plot files
When option ``-dp`` is provided, RF Fold produces a dot-plot file for each transcript being analyzed, with the following structure:<br/>

```
1549                                   # RNA's length
i       j       -log10(Probability)	   # Header 
8       254     0.459355416499312
9       253     0.446335563943221
10      252     0.456738523239413
11      251     0.454733421725068
12      250     0.46965667808714
13      249     0.47837140333524
21      35      0.268192200569539
22      34      0.0183400615262171
23      33      0.0166665677814708
24      32      0.0128927546134575
25      31      0.0148601207296645
26      30      0.0252017532628297

-- cut --

1497    1510    0.0147874890078331
1498    1509    0.0102803152157546
1499    1508    0.0137510190884233
1500    1507    0.0402352346970943
```
where *i* and *j* are the positions (1-based) of the bases involved in a given base-pair, followed by the -log<sub>10</sub> of their base-pairing probability.<br/>These files can be easily viewed using the __Integrative Genomics Viewer (IGV)__ (for additional details, please refer to the official <a href="http://software.broadinstitute.org/software/igv/">Broad Institute's IGV page</a>).