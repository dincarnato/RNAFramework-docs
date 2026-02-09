RF Eval allows evaluating the agreement between a given (set of) secondary structure(s) and a (set of) XML reactivity files.<br/>Reference structures can be provided either in Vienna format (dot-bracket notation), or in CT format. A single file containing the structure for multiple transcripts can be provided:```
# Vienna format
>Transcript#1AAAAAAAAAAAAAAAAAAAAUUUUUUUUUUUUUUUUUUUUU.((((((((((((((((((....))))))))))))))))))>Transcript#2CCCCCCCCCCCCCCCCCGGGGGGGGGGGGGGGGGGGG(((((((((((((((((...)))))))))))))))))>Transcript#3GCUAGCUAGCUAGCUAGCUAGUCAAGACGAGUCGAUGCU(((((((((....))))))))).................
```!!! note "Important"
    The IDs of the provided structures __must__ match the file name of the reactivity XML file (e.g. "Transcript#1" expects an XML file named "Transcript#1.xml")<br/>
## Metrics
RF Eval computes 3 metrics of agreement between reactivity data and structure. All 3 metrics yield values comprised between 0 and 1, with __0__ representing __0%__ agreement and __1__ representing __100%__ agreement.<br/><br/>

__[1] Unpaired coefficient__<br/><br/>
This is the simplest metric and it measures the fraction of highly reactive bases (bases whose reactivity exceeds a user-defined threshold *t*) that are unpaired in the secondary structure:<br/><br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><mi>C</mi><mo>=</mo><munderover accent='false' accentunder='false'><mo>&#x2211;</mo><mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>k</mi></munderover><mfenced open="{" close=""><mtable columnalign="left"><mtr><mtd><mn>1</mn></mtd><mtd><mi>i</mi><mi>f</mi><mo>&#xA0;</mo><mi>i</mi><mo>&#x2208;</mo><mi>u</mi></mtd></mtr><mtr><mtd><mn>0</mn></mtd><mtd><mi>i</mi><mi>f</mi><mo>&#xA0;</mo><mi>i</mi><mo>&#x2208;</mo><mi>p</mi></mtd></mtr></mtable></mfenced></math>
<br/>
where *k* is the set of bases having reactivity &gt; *t*, while *u* and *p* are respectively the sets of unpaired and paired bases in the structure.
<br/><br/>

__[2] Data-Structure Correlation Index (DSCI)__<br/><br/>
This metric was originally proposed by Lan *et al*., 2021 (doi: [10.1101/2020.06.29.178343](https://doi.org/10.1101/2020.06.29.178343)) and it is closely related to the Mann-Whitney *U* statistic. The DSCI is defined as the probability that a randomly chosen unpaired base will have greater reactivity than a randomly chosen paired base:<br/><br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><mi>D</mi><mi>C</mi><mi>S</mi><mi>I</mi><mo>=</mo><mfrac><mn>1</mn><mrow><mi>m</mi><mi>n</mi></mrow></mfrac><munderover accent='false' accentunder='false'><mo>&#x2211;</mo><mrow><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>m</mi></munderover><munderover accent='false' accentunder='false'><mo>&#x2211;</mo><mrow><mi>j</mi><mo>=</mo><mn>1</mn></mrow><mi>n</mi></munderover><mfenced open="{" close=""><mtable columnalign="left"><mtr><mtd><mn>1</mn></mtd><mtd><mi>i</mi><mi>f</mi><mo>&#xA0;</mo><msub><mi>p</mi><mi>i</mi></msub><mo>&lt;</mo><msub><mi>u</mi><mi>j</mi></msub></mtd></mtr><mtr><mtd><mn>0</mn></mtd><mtd><mi>i</mi><mi>f</mi><mo>&#xA0;</mo><msub><mi>p</mi><mi>i</mi></msub><mo>&#x2265;</mo><msub><mi>u</mi><mi>j</mi></msub></mtd></mtr></mtable></mfenced></math>
<br/>
where *p* is the set of reactivities for all *m* paired bases, while *u* is the set of reactivities for all *n* unpaired bases.
<br/><br/>

__[3] Area Under the Receiver Operating Characteristic Curve (AUROC)__<br/><br/>
This metric is typically employed to assess the performance of a binary classifier model at varying threshold values.<br/>
Briefly, the reactivity threshold *t* is slowly increased from 0 to 1, in 0.005 increments. At each threshold, the __True Positive Rate__ (TPR) is calculated as:<br/><br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><mi>T</mi><mi>P</mi><mi>R</mi><mo>=</mo><mfrac><mrow><mi>T</mi><mi>P</mi></mrow><mi>P</mi></mfrac></math>
<br/>
where *TP* is the number of unpaired bases whose reactivity &ge; *t*, and *P* is the total number of unpaired bases in the structure.<br/>
The __True Negative Rate__ (TNR) is instead calculated as:<br/><br/>

<math display="block" xmlns="http://www.w3.org/1998/Math/MathML"><mi>T</mi><mi>N</mi><mi>R</mi><mo>=</mo><mfrac><mrow><mi>T</mi><mi>N</mi></mrow><mi>N</mi></mfrac></math>
<br/>
where *TN* is the number of paired bases whose reactivity &ge; *t*, and *N* is the total number of paired bases in the structure.<br/><br/>
The AUROC is then defined as the area underlying the curve described by the set of FPR-TPR value pairs at each value of *t*.
<br/><br/>
Since version __2.9.1__, RF Eval can use __R__ to generate three types of plots to help visualize the three metrics:<br/>

__a.__ Distribution of reactivities on paired vs. unpaired bases<br/>
__b.__ Receiver Operating Characteristic (ROC) curves<br/>
__c.__ Histogram of scores
<br/><br/>
![AUROC](http://www.incarnatolab.com/images/docs/RNAframework/rf-eval_metrics.png)
<br/><br/>

# Usage

```bash
$ rf-eval [options] -s db_dir/ -r XML_dir/
$ rf-eval [option] -s rna.db -r rna.xml
```

To list all available parameters, simply type:

```bash
$ rf-eval -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-s__ *or* __--structures__ | string | Path to a (folder of) structure file(s)
__-r__ *or* __--reactivities__ | string | Path to a (folder of) XML reactivity file(s)
__-o__ *or* __--output__ | string | Output folder (Default: __rf_eval/__)
__-g__ *or* __--img__ | | Generates plots for the various metrics (requires R)
__-ow__ *or* __--overwrite__ | | Overwrites output file (if the specified file already exists)
__-p__ *or* __--processors__ | int | Number of processors to use (&ge;1, Default: __1__)
__-tu__ *or* __--terminal-as-unpaired__ | | Treats terminal base-pairs as if they were unpaired<br/>__Note:__ this parameter and ``-it`` are mutually exclusive
__-it__ *or* __--ignore_terminal__ | | Terminal base-pairs are excluded from calculations<br/>__Note:__ this parameter and ``-tu`` are mutually exclusive
__-kl__ *or* __--keep-lonelypairs__ | | Lonely base-pairs (helices of 1 bp) are retained
__-kp__ *or* __--keep-pseudoknots__ | | Pseudoknotted base-pairs are retained
__-c__ *or* __--reactivity-cutoff__ | | Cutoff for considering a base highly-reactive when computing the unpaired coefficient (&gt;0, Default: __0.7__)
__-no__ *or* __--no-overall__ | | Disables overall stats computation
__-R__ *or* __--R-path__ | string | Path to R executable (Default: assumes R is in PATH)<br/>__Note:__ also check `$RF_RPATH` under [Environment variables](https://rnaframework-docs.readthedocs.io/en/latest/envvars/#rf_rpath)



