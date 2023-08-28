The RF Count Genome module is an extension of the RF Count module, introduced in version __2.8.0__ to process genome-level alignments. It can process any number of genome-level SAM/BAM files to calculate per-base RT-stops/mutations and read coverage. For any information on the RC file format, or on the RF Count algorithm, please refer to the [manual page](https://rnaframework-docs.readthedocs.io/en/latest/rf-count/) of `rf-count`.<br /><br />

# Usage
To list the required parameters, simply type:

```bash
$ rf-count-genome -h
```

Parameter         | Type | Description
----------------: | :--: |:------------
__-p__ *or* __--processors__ | int | Number of processors (threads) to use (Default: __1__)
__-wt__ *or* __--working-threads__ | int | Number of working threads to use for each instance of SAMTools/Bowtie (Default: __1__).<br/>__Note:__ RT Counter executes 1 instance of SAMTools for each processor specified by ``-p``.  At least ``-p <processors>`` * ``-wt <threads>`` processors are required.
__-o__ *or* __--output-dir__ | string | Output directory for writing counts in RC (RNA Count) format (Default: __rf_count_genome/__)
__-ow__ *or* __--overwrite__ | | Overwrites the output directory if already exists
__-t__ *or* __--tmp-dir__ | string | Path to a directory for temporary files creation (Default: __<output-dir>/tmp__)<br/>__Note:__ If the provided directory does not exist, it will be created
__-s__ *or* __--samtools__ | string | Path to ``samtools`` executable (Default: assumes ``samtools`` is in PATH)
__-r__ *or* __--sorted__ | | In case SAM/BAM files are passed, assumes that they are already sorted lexicographically by transcript ID, and numerically by position
__-t5__ *or* __--trim-5prime__ | int[,int] | Comma separated list (no spaces) of values indicating the number of bases trimmed from the 5'-end of reads in the respective sample SAM/BAM files (Default: __0__)<br/>__Note #1:__ Values must be provided in the same order as the input files (e.g. rf-count -t5 0,5 file1.bam file2.bam, will consider 0 bases trimmed from file1 reads, and 5 bases trimmed from file2 reads)<br/>__Note #2:__ If a single value is specified along with multiple SAM/BAM files, it will be used for all files
__-fh__ *or* __--from-header__ | | Instead of providing the number of bases trimmed from 5'-end of reads through the ``-t5`` (or ``--trim-5prime``) parameter, RF Count will try to guess it automatically from the header of the provided SAM/BAM files
__-f__ *or* __--fasta__ | string | Path to a FASTA file containing the reference transcripts<br/>__Note:__ Transcripts in this file must match transcripts in SAM/BAM file headers
__-mf__ *or* __--mask-file__ | string | Path to a mask file
__-ndd__ *or* __--no-discard-duplicates__ | | Reads marked as PCR/optical duplicates, discarded by default, will be also considered
__-pn__ *or* __--primary-only__ | | Considers only primary alignments (SAM bitwise flag != 256)
__-po__ *or* __--paired-only__ | | When processing SAM/BAM files from paired-end experiments, only those reads for which both mates are mapped will be considered
__-pp__ *or* __--properly-paired__ | | When processing SAM/BAM files from paired-end experiments, only those reads mapped in a proper pair will be considered
__-mq__ *or* __--map-quality__ | int | Minimum mapping quality to consider a read (Default: __0__)
__-co__ *or* __--coverage-only__ | | Only calculates per-base coverage (disables RT-stops/mutations count)
__-m__ *or* __--count-mutations__ | | Enables mutations count instead of RT-stops count (for SHAPE-MaP/DMS-MaPseq)
 | | __Mutation count mode options__
__-om__ *or* __--only-mut__ | string | Only the specified mutations will be counted<br/>__Note #1:__ mutations must be provided in the form [original]2[mutated]. For example, "A2T" (or "A>T", or "A:T") will only count mutation events in which a reference A base has been sequenced as a T. IUPAC codes are also accepted. Multiple mutations must be provided as a comma (or semi-colon) separated list (e.g. A2T;C:N,G>A)<br/>__Note #2:__ when specified, this parameter automatically disables insertion and deletion count
__-ds__ *or* __--discard-shorter__ | int | Discards reads shorter than this length (excluding clipped bases, Default: __1__)
__-q__ *or* __--min-quality__ | int | Minimum quality score value to consider a mutation (Phred+33, requires ``-m``, Default: __20__)
__-es__ *or* __--eval-surrounding__ | | When considering a mutation/indel, also evaluates the quality of surrounding bases (&#177;1 nt)<br/>__Note:__ the quality score threshold set by ``-q`` (or ``--min-quality``) also applies to these bases
__-nd__ *or* __--no-deletions__ | | Ignores deletions
__-ni__ *or* __--no-insertions__ | | Ignores insertions
__-na__ *or* __--no-ambiguous__ | | Ignores ambiguously mapped deletions<br/>__Note:__ the default behavior is to re-align them to their right-most valid position (or to their left-most valid position if ``-la`` has been specified)
__-la__ *or* __--left-align__ | | Re-aligns ambiguously mapped deletions to their left-most valid position
__-rd__ *or* __--right-deletion__ | | Only the right-most base in a deletion is marked as mutated
__-ld__ *or* __--left-deletion__ | | Only the left-most base in a deletion is marked as mutated
__-md__ *or* __--max-deletion-len__ | int | Ignores deletions longer than this number of nucleotides (Default: __10__)
__-me__ *or* __--max-edit-distance__ | float | Discards reads with editing distance frequency higher than this threshold (0<m&le;1, Default: __0.15__ [15%])
__-eq__ *or* __--median-quality__ | int | Median quality score threshold for discarding low-quality reads (Phred+33, Default: __20__)
__-dc__ *or* __--discard-consecutive__ | int | Discards consecutive mutations within this distance from eachothers
__-cc__ *or* __--collapse-consecutive__ | | Collapses consecutive mutations/indels toward the 3'-most one (recommended for SHAPE-MaP experiments)
__-mc__ *or* __--max-collapse-distance__ | int | Maximum distance between consecutive mutations/indels to allow collapsing (requires ``-cc``, &ge;0, Default_ __2__)

<br/>
## Strandedness of genome-level alignments
An important difference with transcriptome-level analyses is that, at the level of the genome, the structure signal can be originated by either of the two DNA strands, dependening on where the gene resides. The strandedness depends on how the library has been generated. Specifying the proper strandedness of the library is __crucial__ as it determines the way ``rf-count-genome`` will assign the RT-stop/mutation counts.<br/>
The strandedness of each sample can be specified by appending one of the following strings to the path to the SAM/BAM files to be processed:


String     | Strandedness
-----------| :------------
__:u__ *or* __:unstranded__ | The information on the genomic strand that originated the transcript is not preserved
__:f__ *or* __:first__ *or* __:first-strand__ | R1 aligns to the strand complementary to the one that originated the transcript
__:s__ *or* __:second__ *or* __:second-strand__ | R1 aligns to the same strand that originated the transcript

<br/>
__Second-strand__ is the default (and only accepted) mode for the analysis of RT-stop-based RNA structure mapping experiments. When mutation count (``-m``) or coverage-only (``-co``) modes are enabled, if the strandedness of the samples is not specified, samples are assumed to be __unstranded__.<br/>
For instance, in the example below:

```bash
$ rf-count-genome -m -f reference.fasta -r sample1.bam:s sample2.bam:f sample3.bam
```

*sample1.bam* is generated using a second-strand directional library prep strategy, *sample1.bam* is generated using a first-strand directional library prep strategy and *sample3.bam* is assumed to have been generated using a non-directional library prep strategy.<br/>

When processing __unstranded__ experiments, ``rf-count-genome`` will generate a single output RC file, named after the sample, having the ``.plus.rc`` suffix. When processing __stranded__ experiments, two output RC files will be generated, named after the sample, respectively having the ``.plus.rc`` and ``.minus.rc`` suffixes and corresponding to the plus and minus strands of the genome.<br/>
Transcriptome-level RC files can be generated starting from genome-level RC files using the ``extract`` tool of the ``rf-rctools`` module. For additional information, please refer to the [manual page](https://rnaframework-docs.readthedocs.io/en/latest/rf-rctools/).