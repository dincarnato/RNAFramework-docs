The RF MotifDiscovery module allows identifying significantly enriched sequence motifs, such as RNA-binding protein or RNA post-transcriptional modification consensus sequences, from peaks identified in RNA immunoprecipitation experiments via [RF PeakCall](https://rnaframework-docs.readthedocs.io/en/latest/rf-peakcall/).
<br/><br/>

!!! warning "Important"
    This is a beta version. As such, it might be subjected to changes/improvements.


# Usage
To list the required parameters, simply type:

```bash
$ rf-motifdiscovery -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors (threads) to use for shuffling (Default: __1__)<br/>__Note:__ this parameter has no effect when specified without ``-s`` (or ``--shuffle``)
__-b__ *or* __--peaks__ | string | Peaks BED file (mandatory)
__-nb__ *or* __--negative-peaks__ | string | A BED file containing negative peak sequences (optional)<br/>__Note:__ when no negative peaks file is specified, a set of negative sequences will be generated by ``-ns`` (or ``--neg-samplings``) rounds of random sampling from reference transcripts, or random shuffling if ``-s`` (or ``--shuffle``) has been specified
__f__ *or* __--fasta__ | string | A FASTA file containing the reference transcript sequences (mandatory)
__-o__ *or* __--output-dir__ | string | Output directory for writing counts in RC (RNA Count) format (Default: __rf_motifdiscovery/__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory if already exists
__-w__ *or* __--window__ | int | Size of the window, centered on the center of each peak, in which motif discovery should be performed (&ge;3, Default: __50__)
__-np__ *or* __--neg-samplings__ | int | Number of negative sequences to generate/sample for each peak (Default: __20__)
__-s__ *or* __--shuffle__ | | Negative sequences will be generated by random shuffling peak sequences<br/>__Note:__ default is to sample ``--neg-samplings`` random windows from reference transcripts, for each peak in the dataset
__-ns__ *or** __--nuc-shuffling__ | | Performs random shuffling of nucleotides without preserving dinucleotide frequencies
__-k__ *or* __--kmer__ | int | K-mer size (&ge;4, Default: __5__)
__-v__ *or* __--pvalue__ | float | P-value threshold to consider an enrichment significant (0-1, Default: __1e-3__)
__-nm__ *or* __--n-motifs__ | int | Maximum number of motifs to report (&ge;1, Default: __3__)
__-ops__ *or* __--one-per-seq__ | | K-mers are counted only once per peak
__-t__ *or* __--tollerance__ | float | Fractional tollerance to consider a position degenerate (0-1, Default: __0.2__)
__-sk__ *or* __--save-kmer-table__ | | Saves the list of k-mers, and their associated p-values

<br/>
## Understanding the algorithm
The algorithm starts by identifying significantly enriched k-mers of length ``--kmer`` in peak sequences. By default, analysis is limited to a ``--window`` nt-long window centered on the center of the peak. Significance is assessed using a Fisher test, by comparing the number of occurrences of each k-mer within peak sequences, as compared to a set of negative sequences. By default, all the occurrences of the same k-mer within each peak are counted, unless the ``--one-per-seq`` parameter has been specified; in such a case, each k-mer is counted only once per peak. An user-provided list of negative peaks can be provided in BED format via the ``--negative-peaks`` parameter. If no negative peaks file is provided, by default, for each peak in the dataset, up to ``--neg-samplings`` negative sequences will be randomly sampled from the reference transcripts. If the ``--shuffle`` parameter is specified, however, negative sequences will be generated by ``--neg-samplings`` random shufflings of the original peak sequences. By default, shuffling is performed in such a way that original dinucleotide frequencies are preserved; this can be overriden via the ``--nuc-shuffling`` parameter, that enables fully-random shuffling.<br/>
Once significantly enriched k-mers have been identified, motifs are built as follows:

1. The most significant k-mer is selected (in case of multiple k-mers having the same p-value, the one with the highest enrichment is selected)
2. Significant k-mers within a maximum Hamming distance of 1 are identified
3. A *core motif* is built by using this initial set of k-mers
4. The core motif is extended by identifying significant k-mers overlapping by at least 75% with the core motif
5. This expanded set of sequences is then used to build a Position Frequency Matrix for the motif
<br/><br/>
![Motif discovery](http://www.incarnatolab.com/images/docs/RNAframework/rf-motifdiscovery.png)
<br/><br/>
Significantly enriched motifs will be reported as Position Frequency Matrices (PFMs), in [TRANSFAC format](https://meme-suite.org/meme/doc/transfac-format.html):

```text
ID Motif_DRACW
BF RF_MotifDiscovery
P0 A C G U
1 34 9 23 34 W
2 35 0 36 29 D
3 49 0 51 0 R
4 100 0 0 0 A
5 0 65 0 35 C
6 35 24 0 40 W
7 21 20 32 28 K
XX
//
```
Currently, RF MotifDiscovery does not generate sequence logos. However, output PFMs can be directly used as input for [__WebLogo 3__](http://weblogo.threeplusone.com), either via the web interface, or locally:

```bash
weblogo -a RNA -s large -c classic --format PDF < motif_DRACW.mat > motif_DRACW.pdf
```