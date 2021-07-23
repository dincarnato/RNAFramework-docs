The RF Duplex module performs the analysis of direct RNA-RNA interaction mapping experiments, such as PARIS, SPLASH and COMRADES. It enables clustering chimeric reads and estimation of base-pairing probabilities. <br />
!!! warning "Important"
    It is strongly advised to use [STAR](https://github.com/alexdobin/STAR) for read mapping. Currently, mapping has to be performed by directly calling STAR. In future releases, support for STAR will be added to the ``rf-index`` and ``rf-map`` modules.
<br />
# Usage
RF Duplex accepts as input a comma-separated list of two files: a BAM file and a Chimeric.out.junction file. If an aligner other than STAR is being used, the Chimeric.out.junction file can be omitted.

```bash
$ rf-duplex file1.bam,file1_Chimeric.out.junction ... filen.bam,filen_Chimeric.out.junction
```

To list the required parameters, simply type:

```bash
$ rf-duplex -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors to use (Default: __1__)
__-st__ *or* __--single-transcript__ | | Enables multithreading optimization for single trancript analysis<br/>__Note:__ when active, a single transcript at a time will be analyzed, and multiple processors will be used to speed-up time-comsuming operations such as read clustering and base-pair analysis. This is useful, for example, for the analysis of COMRADES experiments, in which a single target transcript is analyzed at a high sequencing depth
__-o__ *or* __--output-dir__ | string | Output directory (Default: __rf_duplex/__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory if already exists
__-f__ *or* __--fasta__ | string | Path to a FASTA file containing the reference transcripts<br>__Note:__ Transcripts in this file must match transcripts in SAM/BAM file headers
__-wl__ *or* __--whitelist__ | string | A whitelist containing transcript IDs (one per each row) to restrict the analysis to (optional)
__-mg__ *or* __--min-gap__ | int | Minimum gap length (in nt) to consider a chimeric read (>0, Default: __1__)
__-mh__ *or* __--min-half__ | int | Minimum length (in nt) of each half of a chimeric read to consider it (>0, Default: __20__)
__-xr__ *or* __--max-reads__ | int | Maximum number of chimeric reads to analyze for each transcript (>0, Default: __100000__)<br/>__Note:__ all chimeric reads are first imported, then a random sample is extracted
__-c__ *or* __--constraint__ | | Generates a dot-bracket constraint file containing the inferred base-pairs whose probability exceeds a given threshold (controlled by ``-ct`` or ``--constraint-theshold``)
__-ct__ *or* __--constraint-threshold__ | float | Sets the probability threshold to include a base-pair in the constraint (0.5-1, Default: __0.5__)
 | | __Base-pairs inference options__
__-nv__ *or* __--no-use-vienna__ | | A modified Smith-Waterman local alignment algorithm is used instead of the ViennaRNA package to infer base-pairs from chimeric reads
__-mr__ *or* __--min-reads__ | int | Minimum number of chimeric reads to retain a cluster (>0, Default: __2__)
__-mpf__ *or* __--min-paired-frac__ | float | Minimum fraction of paired bases in a chimeric read to retain it (>0-1, Default: __0.25__)
__-mp__ *or* __--min-prob__ | float | Minimum probability to retain an inferred base-pair (>0-1, Default: __0.01__)
__-rmc__ *or* __--require-min-chimera__ | | Instead of being inferred from the cluster's centroid, base-pairs are inferred from every single chimeric read belonging to the cluster, and only base-pairs common to a certain fraction of the reads in the cluster (controlled by ``-mcf`` or ``--min-chimera-frac``), are retained<br/>__Note:__ if no base-pair is supported by at least ``-mcf`` chimeric reads, then the cluster is discarded
__-mcf__ *or* __--min-chimera-frac__ | float | Sets the minimum fraction of chimeric reads supporting a base-pair (requires -rmc; 0.5-1, Default: __0.5__)
 | | __Plotting options__
__-g__ *or* __--img__ | | Enables the generation of graphical reports reporting the inferred base-pairing probabilities, and the Shannon Entropy
__-cs__ *or* __--color-scale__ | string | Allows specifying 4 cut-points for the color scale used to report base-pairing probabilities (Default: __0.05,0.1,0.4,0.7__ for white|grey|yellow|blue|green)
 | | __Clustering options__
__-cb__ *or* __--make-cluster-bam__ | | Generates a BAM file for each transcript being analyzed, with chimeric reads belonging to the same cluster grouped together via the XG tag
__-mss__ *or* __--min-sample-size__ | int | Minimum number of reads to be considered for each iteration of mini-batch clustering (>0, Default: __1000__)
__-mo__ *or* __--min-overlap-frac__ | float | Minimum fractional overlap between the two halves of two chimeric reads to trigger clustering (>0-1, Default: __0.05__)
__-mi__ *or* __--max-iterations__ | int | Maximum number of iterations for K-means (>0, Default __5__)

<br/>
## Understanding the algorithm
In its current implementation, the module can do the following:<br/>

1. Cluster chimeric reads (and generate a clustered BAM file for visualization)
2. Infer base-pairs and base-pairing probabilities from each cluster
3. Generate a structure constraint corresponding to the major conformation detected (estimated from base-pairing probabilities)

Analysis performed RF Duplex can be quite computational intensive. To this end, different multithreading optimizations are adopted to ensure reasonable execution times. By default, the module is optimized for the analysis of PARIS/SPLASH datasets; these experiments are usually performed transcriptome-wide, and they are characterized by a relatively low number of reads mapping to several different transcripts. In these cases, multiple transcripts are simultaneously analyzed on multiple threads. However, in the case of COMRADES experiments, a single transcript is mapped with a very high coverage. To deal with these cases, enabling the ``--single-transcript`` parameter will force a single transcript at a time to be analyzed, and multiple processors will be used to speed-up time-comsuming operations such as read clustering and base-pair analysis.
<br/><br/>
When analyzing large COMRADES datasets, clustering can become challenging. To this end, it is possible to limit the analysis to a randomly extracted sample of ``--max-reads`` chimeric reads. Clustering is performed by using a modified version of the K-means algorithm, that allows automatically determining the optimal number of clusters. To speed up the process, clustering is performed in mini-batches of ``--min-sample-size`` reads. The maximum number of K-means clustering iterations is controlled via the ``--max-iterations`` parameter.
<br/><br/>
![Centroid definition](http://www.rnaframework.com/images/rf-duplex_centroid.png)
<br/><br/>
As shown above, for each cluster, a *centroid* is defined by identifying the point of maximum coverage on both sides of the chimeras making up the cluster, and by enlarging it upstream and downstream by *L*/2, where *L* is the median length of the each half of the chimeras in the cluster. A new chimeric read is assigned to a given cluster if each half of the chimera overlap by at least ``--min-overlap-frac`` with the corresponding half of the centroid. Clusters supported by &lt; ``--min-reads`` chimeric reads are discarded.<br/>The results of clustering can be visualized by enabling the generation of clustered BAM files via the ``--make-cluster-bam`` parameter. These BAM files store the cluster each read has been assigned to in their __XG__ tag, and they can be easily visualized with __Integrative Genomics Viewer (IGV)__ (for additional details, please refer to the official <a href="http://software.broadinstitute.org/software/igv/">Broad Institute's IGV page</a>).
<br/><br/>
![IGV Clusters](http://www.rnaframework.com/images/rf-duplex_clusters.png)
<br/><br/>
Once clusters have been defined, the algorithm proceeds to identifying the possible base-pairs, and it estimates the base-pairing probabilities. By default, base-pairs are identified by taking the cluster centroid, and by co-folding the two halves of the centroid using the ViennaRNA package. Alternatively, if the ``--no-use-vienna`` parameter is provided, a modified Smith-Watherman algorithm is used to identify the candidate base-pairs. If &lt; ``--min-paired-frac`` of the centroid is base-paired, the cluster is discarded. For each cluster, a probability is calculated as the ratio of the number of reads belonging to the cluster, divided by the total number of reads in the cluster, plus the reads belonging to clusters that are incompatible with the cluster in analysis. This probability is then assigned to the base-pairs inferred from that cluster. It is theoretically possible, under certain extreme conditions, for two clusters to share one or more base-pairs; in such situations, the base-pair is assigned the sum of the probabilities of the two clusters.<br/>
As an alternative to estimating base-pairs from the cluster centroid, one might evaluate independently each of the chimeras making up a given cluster by enabling the ``--require-min-chimera`` parameter; in this case, only the base-pairs common to at least ``--min-chimera-frac`` &times; the chimeras of the cluster will be retained. Base-pairing probabilities are reported as dot-plot files, that can be readily imported and visualized with __IGV__. 
<br/><br/>
![IGV Dotplot](http://www.rnaframework.com/images/rf-duplex_dotplot.png)
<br/><br/>
Furthemore, base-pairing probabilities can be exported as arc-plots in SVG format by enabling the ``--img`` parameter.
<br/><br/>
![Graphics](http://www.rnaframework.com/images/rf-duplex_graphics.png)
<br/><br/>
In addition to computing base-pairing probabilities, if the ``--constraint`` parameter has been specified, the algorithm generates a constraint in dot-bracket (Vienna) format, containing only the inferred base-pairs whose probability exceedes ``--constraint-threshold``. This constraint, likely representing the base-pairs present in the predominant conformation for the RNA in analysis, can be further used for structure inference.

!!! note "Information"
    Starting from the next release, it will be possible to directly import constraint files into ``rf-fold``, hence enabling the integration of structure probing and direct RNA-RNA interaction capture data.