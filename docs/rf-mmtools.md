The RF MMtools module enables easy visualization/manipulation of MM files.<br />
This tool is particularly useful when the same sample is sequenced more than one time to increase its coverage. Now, instead of merging the BAM files and re-calling the `rf-count` on the whole dataset (that is very time-consuming), each sample can be processed independently and simply merged to the MM file from the previous analysis.<br/>

# Usage
Available tools are: __index__, __view__, __merge__, __extract__ and __stats__.

Tool      |  Description
--------: | :------------
view | Dumps to screen the content of the provided MM file         
merge | Combines multiple MM files
extract | Generates a new MM file by extracting/filtering the reads according to a set of user-defined rules
toRC | Converts MM files to RC format
index | Indexes MM files
stats | Prints length and mutation distributions

!!! note "Note"
    MM files need to be indexed (via `rf-mmtools index`) before any of the above commands can be executed

To list the required parameters, simply type:

```bash
$ rf-mmtools [tool] -h
```

Parameter         | Tool | Type | Description
----------------: | :--: | :--: | :------------
__-o__ *or* __--output__ | __merge__, __extract__,  *or* __toRC__ | string | Output MM filename (Default: __merge.mm__ for `merge`, __&lt;input&gt;.extracted.rc__ for `extract`, *or* __&lt;input&gt;.rc__ for `toRC`)
__-ow__ *or* __--overwrite__ | __merge__, __extract__,  *or* __toRC__ | | Overwrites output file (if the specified file already exists)
__-kb__ *or* __--keepBases__ | __extract__ | string | Only retains mutations on specified bases (Default: __ACGT__)<br/>__Note:__ IUPAC codes are allowed
__-mpr__ *or* __--minMutPerRead__ | __extract__ | int | Reads with &lt; than this number of mutations are discarded (Default: __1__)
__-mrl__ *or* __--minReadLen__ | __extract__ | int | Reads shorter than this length are discarded (&gt;0, Default: __1__)
__-rs__ *or* __--randomSubsample__ | __extract__ | int | Randomly subsamples this fraction of reads (Default: __keep all reads__)<br/>__Note:__ for example, if `-rs 2`, 1/2 of the reads will be subsampled
__-a__ *or* __--annotation__ | __extract__ | string | Path to a list of regions (in BED format) to extract from the MM file<br/>__Note:__ only the portion of the read falling within the boundaries of the provided BED intervals will be retained and subjected to the other filtering steps
__-wl__ *or* __--whitelist__ | __extract__ | string | Path to a file containing a list (one per line) of transcripts to be extracted from the MM file

<br/>
## MMtools "view" output
The `view` command produces an output structured as follows:<br/>

```
Transcript#1
ATTTGCGAGCTAGCGATCGAGTCGATGC...GATGCGTACGTAGTCGTAGTC
0	119   23,35,73,99
0	88    80
0	119   58,69,74,96
0	102   35
0	95    32,64
0	103   75
0	93    38,71
0	107   32,59,62,71,101
0	119   23,63,83,102
0	119   70,102,105,113
0	113   53
0	92    21
0	119   73
0	108   21,48,72
0	59    5,51
0	107   52
0	88    35,38
0	92    35
0	120   35,51,53,72,102,115
0	119   22,29,93
```
in which, for each transcript, the first two rows correspond to the transcript's ID and sequence, followed by the reads, one per row. Each read row contains:

- Start mapping position (0-based)
- End mapping position (0-based)
- Comma-separated list of indexes (0-based) of mutated bases (with respect to  transcript's start)

Consecutive entries are separated by a newline.<br/>
If a comma (or semicolon) separated list of transcript IDs is provided, only those transcripts will be shown in the output (e.g. `rf-mmtools view input.mm 'Transcript_2'`).<br/>

## Merging MM files
The `merge` command allows merging multiple MM files. Files to be merged do not have to contain the same set of transcripts, therefore, MM files generated using different references can be combined. An important caveat here is that, if two transcripts with different sequences share the same ID, an exception will be thrown.
<br/>

## Subsetting and filtering reads
Starting from an input MM file, the `extract` command generates an output MM file by applying a number of user-defined filters.<br/>
If a [BED](https://genome.ucsc.edu/FAQ/FAQformat.html#format1) annotation file is provided, the portion of reads falling within the user-defined regions will be extracted. 

!!! note "Notes"
    1. BED files will be interpreted as BED3, therefore only the start (0-based) and end (1-based) coordinates (2nd and 3rd field) will be considered). <br/>2. A single BED entry per transcript is allowed

Reads can be filtered by mutated base (e.g., `-kb AC` or `-kb M` will only retain mutations on A/C bases), length (e.g., `-mrl 100` will only retain reads &ge; 100 bp), or minimum number of mutations per base (e.g., `-mpr 2` will only retain reads with &ge; 2 mutations. 

Filters are applied in the following order:

1. Whitelisted transcripts
2. Subsetting of regions in BED file
3. Filtering by read length
4. Filtering by base
5. Filtering by number of mutations per read

In the following example:<br/>

```
0         10        20        30        40        50
|         |         |         |         |         |
GCATCGTGAGCGTATCGATGCGATGCTAGTCGAGCATCGAGCGACTGATGACTACG  Transcript
          CGTAgCGcTGgGATaCTA                              read#1
              TCGcTGCGgTGCTAGTtGAGCATCG                   read#2
```
*read#1* starts at position 10, ends at position 27 (length: 18), and carries mutations at positions 14-17-20-24, while *read#2* starts at position 14, ends at position 38 (length: 25), and carries mutations at positions 17-22-30.<br/><br/>
Let's suppose the following BED file has been provided:<br/>

```
Transcript    15     35
```
This will result in transcript and reads being subsetted as it follows:<br/>

```
# Before subsetting:

0         10        20        30        40        50
|         |         |         |         |         |
GCATCGTGAGCGTATCGATGCGATGCTAGTCGAGCATCGAGCGACTGATGACTACG  Transcript
          CGTAgCGcTGgGATaCTA                              read#1
              TCGcTGCGgTGCTAGTtGAGCATCG                   read#2
               ====================                       BED region
               
# After subsetting:

15                 34
|                  |
CGATGCGATGCTAGTCGAGC                                      Transcript
CGcTGgGATaCTA                                             read#1
CGcTGCGgTGCTAGTtGAGC                                      read#2
```
Let's now suppose that the `-kb AC -mpr 2 -mrl 20` filters have been specified, which would cause only reads at least 20 bp-long, carrying at least 2 mutations on A/C bases to be retained. *#read1*, which was originally 18 bp-long, has become 13 bp-long after being subsetted due to the region specified in the BED file, therefore it won't pass the `-mrl 20` filter. On the other hand, *#read2* is 20-bp long after subsetting, and it carries 3 mutations on A/C bases, therefore it will pass all filters and it will be retained.
