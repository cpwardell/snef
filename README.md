<p align="center">
  <img src="/www/fings_logo_20190809_color_banner.png">
</p>

# <span style="color:#8a0707">F</span></span><span style="color:#b2b2b2">***i***</span>NGS: <span style="color:#8a0707">***F***</span><span style="color:#b2b2b2">***i***</span>lters for ***N***ext ***G***eneration ***S***equencing

## Introduction
Somatic variant callers produce variant calls based on the differences between matched tumor-normal pairs aligned to a reference genome. These variant calls can be extremely rich in false positives due to confounding factors such as purity of the samples, artifacts introduced by sequencing chemistry, the alignment algorithm and the incomplete and repetitive nature of reference genomes.  
It has become common practice for sequencing studies to attempt to ameliorate these effects using a variety of filtering techniques, including taking the intersect of results from multiple variant callers and employing some post-calling filtering. This ad-hoc filtering varies greatly between laboratories.  Attempts have been made to standardize the methodology of this filtering, with recommendations produced by the International Cancer Genome Consortium (ICGC) [(Alioto et al., 2015)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4682041/).  
We have developed Filters for Next Generation Sequencing (FiNGS), software written specifically to address these filtering issues. FiNGS can be used in addition to the default filters of somatic variant callers to substantially increase the precision of results, providing high quality variants for further analysis. 

## Key features
- Calculates metrics based on BAM files and provides filtering not possible with other tools
- Fully user-configurable filtering (including which filters to use and their thresholds)
- Option to use filters identical to ICGC recommendations

## Availability
FiNGS is open source and released under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0.html). The latest source code is [freely available at GitHub](https://github.com/cpwardell/FiNGS).

## Dependencies
Python 3

## Quickstart guide, with example data and test
You have three options for running FiNGS; 
1. Download directly from GitHub
2. Download using Bioconda
3. Download using Docker

### Standalone installation via GitHub
Clone the git repository and run the included example script in the `exampledata` directory.
```
git clone https://github.com/cpwardell/FiNGS.git
cd FiNGS/exampledata
python3 ../FiNGS.py -n normal.bam -t tumor.bam -v s2.raw.vcf --PASSonlyin --PASSonlyout
```
You should see the progress printed to the screen. When filtering is complete, a `results` directory will have been created containing the output.

### Bioconda installation

### Docker installation
This guide assumes you have Docker installed and have some basic knowledge on how to use it. The Dockerfile builds an image based on Ubuntu 16.04. 
You need to get a copy of the Dockerfile in this repository; below we use "wget" on Linux to download it, but you could just as easily 
copy and paste the link in your web browser and "right click/save as" the file. The Docker build command works identically in both Bash on Linux and PowerShell on Windows
and assumes that you're in the same directory as the dockerfile named "Dockerfile".

```
# Download the Dockerfile from this address:
wget https://raw.githubusercontent.com/cpwardell/FiNGS/master/Dockerfile
# Build the image and call it "fings" (lowercase)
docker build -t fings .
```

You can test that your image works by running a container interactively:
```
docker run -it fings
cd /FiNGS/exampledata
python3 ../FiNGS.py -n normal.bam -t tumor.bam -v s2.raw.vcf --PASSonlyin --PASSonlyout
```

When you run it on your own data, you can mount the location of your files as below. This would output a results directory in the directory the command was executed in. Note that the `-u` argument ensures that files created by the 
docker container will be owned by the user invoking the docker.
```
docker run -v /path/to/tumorbamdir:/tumorbamdir -v /path/to/normalbamdir:/normalbamdir -v /path/to/vcfdir:/vcfdir -v $PWD:/local -w /local -u $UID:$UID fings /bin/bash -c "python3 /FiNGS/FiNGS.py -n /normalbamdir/normal.bam -t /tumorbamdir/tumor.bam -v /vcfdir/somatic.vcf --PASSonlyin --PASSonlyout"
```

## Suggested usage
+ Use default filtering thresholds
+ Use every available processor
+ Only consider variants with a PASS filter value from the variant caller
+ Only emit variants that PASS all filters  

```
python3 /path/to/FiNGS.py -n /path/to/normal.bam -t /path/to/tumor.bam -v /path/to/somaticvariants.vcf --PASSonlyin --PASSonlyout
```

## Advanced standalone installation guide
FiNGS depends on Python3 and several packages.

On Debian-based systems such as Ubuntu, you can install Python3 and pip, the recommended tool for installing Python packages using the following commands:
```
apt-get -y install python3
apt-get -y install python3-pip
```

Now Python3 and pip are installed, you may install the required Python packages:
```
pip3 install PyVCF
pip3 install pysam
pip3 install editdistance
pip3 install scipy
pip3 install joblib
```

## Basic usage and outputs
In the simplest case, provide FiNGS with paths to a VCF and the BAM files used to produce it.
```
python3 FiNGS.py -n /path/to/normal.bam -t /path/to/tumor.bam -v /path/to/somaticvariants.vcf
```

FiNGS will create a directory called "results" containing the following files, where "date" is the time of execution expressed as a string e.g. "20180420155129" is 2018/04/20 15:51:59 local time.

+ `filtered.results.date.vcf`  
A VCF containing the filtering results. Descriptions of the filters used and their threshold values are stored in the header, and PASS/fail status stored in the FILTER column of each record
+ `filtered.results.date.txt`  
A text file giving the raw data for every variant and TRUE/FALSE values for each filter. A dictionary explaining the contents of each column is below
+ `filter.plots.date.vcf`  
Plots for every filter applied in a single PDF. Data is plotted as a histogram with a kernel density plot overlaid, and a dashed vertical line demarcating the threshold used 
+ `log.txt`  
The log file for the run. Contains the command line arguments used and a complete log of the run
+ `normal.combined.txt`  
All intermediate chunks of the normal BAM file analysis are combined into this file. This and the matching combined tumor file are the input for the R filtering step
+ `tumor.combined.txt`  
All intermediate chunks of the tumor BAM file analysis are combined into this file.  This and the matching combined normal file are the input for the R filtering step

## Advanced usage

+ **-v** Required; path to the somatic variant VCF from any variant caller
+ **-t** Required; path to the tumor BAM file used to produce the VCF
+ **-n** Required; path to the normal BAM file used to produce the VCF
+ **-r** Optional; absolute path to faidx indexed reference genome; required if using 'repeats' filter 
+ **-d** Optional; path to output directory.  Default is to create a directory called "results" in the current working directory
+ **-p** Optional; path to file specifying filtering parameters. Details on filters and default values is provided below. Default is a file located at `FiNGS/R/filter_parameters.txt`.  To alter these parameters, copy and edit this file, and provide a path to it
+ **-c** Optional; chunk size, the number of records to process per chunk. Default is 100
+ **-m** Optional; maximum read depth. Reads beyond this depth will be ignored and a warning emitted to the log file. Default is 1000 
+ **-j** Optional; number of processors to use. -1 uses all available. Default is -1 
+ **--logging** Optional; change logging level. Default is INFO, can be DEBUG for more detail or NOTSET for silent
+ **--overwrite** Optional; allow overwriting of existing results if rerunning  
+ **--PASSonlyin** Optional; only consider variants with a PASS in the filter field of the input VCF  
+ **--PASSonlyout** Optional; only write variants that PASS all filters to the output VCF

## Getting help
Run FiNGS with no additional arguments to get the help file.  If there's something not adddressed here, or if you need further help, raise an issue on GitHub or find me online.

## Citing FiNGS
A paper is being prepared for submission shortly and will be referenced here when available.

## Description of filters
FiNGS assesses variants using any combination of 28 filters.  Below are descriptions of each filter and the default values used.
 
1. **Minimum depth in tumor**.  Default value is 10x.  
This filter excludes variants that do not have adequate sequencing depth in the tumor sample.
 
2. **Minimum depth in normal**.  Default value is 10x.  
This filter excludes variants that do not have adequate sequencing depth in the normal sample.

3. **Maximum depth in normal**.  Default value is 1000x.  
This filter excludes variants that are unusually deep.  Some variants have apparent depths of thousands of reads, particularly around the centromeres, telomeres and other repeat-rich regions.

4. **Maximum depth in tumor**.  Default value is 1000x.  
This filter excludes variants that are unusually deep.  Some variants have apparent depths of thousands of reads, particularly around the centromeres, telomeres and other repeat-rich regions.

5. **Minimum number of ALT reads in tumor**.  Default value is 3.  
At low depths, a variant might have VAF above the default threshold of 5%, but very few reads supporting it.  This default value ensures that such a 5% variant would have a minimum of 60x depth.

6. **Minimum median base quality of ALT reads in tumor**.  Default value is 30.  
Excludes variants where the supporting reads in the tumor have low base quality.  Taking the median of all reads would allow non-variant reads to weight the median base quality.

7. **Minimum median base quality of REF reads in tumor**.  Default value is 30.  
Excludes variants where REF reads in the tumor have low base quality.  Variants with no REF reads (i.e. homozygous ALT) pass this filter.

8. **Minimum median base quality of REF reads in normal**.  Default value is 30.  
Excludes variants where REF reads in the normal have low base quality.

9. **Maximum proportion of zero mapping quality reads in tumor**.  Default value is 0.05.  
A high proportion of zero mapping quality reads suggests a region with poorly mappability.

10. **Maximum proportion of zero mapping quality reads in normal**.  Default value is 0.05.  
A high proportion of zero mapping quality reads suggests a region with poorly mappability.

11. **Minimum median mapping quality of ALT reads in tumor**.  Default value is 50.  
Excludes variants where the supporting reads in the tumor have low mapping quality.  Taking the median of all reads would allow non-variant reads to weight the median mapping quality.

12. **Maximum difference between median mapping quality of ALT reads in tumor and REF reads in normal**.  Default value is 5.  
Excludes variants with a large difference in mapping quality between ALT reads in the tumor and REF reads in the matched normal sample.  Mismatches in the ALT reads will lower mapping quality, but this difference should be limited.

13. **Maximum median shortest distance to either aligned end in tumor**.  Default value is 10.  
Excludes variants that consistently occur only the ends of reads.

14. **Minimum MAD of ALT position in tumor**.  Default value is 3.  
Excludes variants that consistently occur in the same position within reads.

15. **Maximum edit distance of ALT reads in tumor**.  Default value is 4.  
Limits the number of mismatches possible within variant supporting reads in the tumor.  Default value allows for multiple mismatches, allowing SNPs and small indels to occur in addition to the variant of interest.

16. **Maximum edit distance of REF reads in tumor**.  Default value is 3.  
Limits the number of mismatches possible within reference reads in the normal.  Default value allows for multiple mismatches, allowing SNPs and small indels to occur in addition to the variant of interest.  Multiple mismatches in the reference reads in the normal would be indicative of a low quality region in the reference genome.

17. **Maximum VAF in normal**.  Default value is 0.03.  
Excludes variants with too many supporting reads in the normal sample.  Some support is allowed, which accounts for sequencing errors and tumor cross-contamination.

18. **Minimum VAF in tumor**.  Default value is 0.05.  
Excludes variants with too few supporting reads in the tumor sample.  It is difficult to differentiate between genuine low-frequency, subclonal variants, and sequencing errors or contamination.

19. **Maximum OAF in tumor**.  Default value is 0.04.  
Excludes variants where multiple alleles are present.  Other Allele Frequency (OAF) is the proportion of reads that support an allele other than the reference or alternate alleles.

20. **Maximum proportion of secondary alignments in tumor**.  Default value is 0.05.  
Excludes variants with a high proportion of secondary alignments.  Secondary alignments are indicative of poor mappability.

21. **Maximum proportion of inversion orientation reads in normal**.  Default value is 0.2.  
Excludes variants with a high proportion of reads in the normal sample that have inverted orientation.  This suggests a region of poor mappability.

22. **FoxoG artifact proportion**.  Default value is 0.9.  
C>A|G>T variants can be oxidation artifacts introduced during library preparation [(Costello et al, 2013)](http://doi.org/10.1093/nar/gks1443).
These "OxoG" artifacts have a telltale read orientation, with the majority of ALT reads in the artifact orientation.
All C>A|G>T variants are classified as being part of two binomial distributions, one centered at 0.5 (50% artifact orientation, 50% non-artifact orientation) and the other at 0.9 (90% artifact orientation reads).
C>A|G>T variants classified as OxoG are removed.

23. **Strand bias exclusion proportion**.  Default value is 0.1.  
Strand bias is defined below [(Guo et al., 2012)](https://bmcgenomics.biomedcentral.com/articles/10.1186/1471-2164-13-666) where a,c represent the forward and reverse strand allele counts of REF reads and b,d represent the forward and reverse strand allele counts of ALT reads.  The topmost proportion of biased variants is removed. Note that this is not the GATK strand bias, which is calculated differently.  
Strand bias: |b/(a+b)-d/(c+d)|/ ((b+d)/(a+b+c+d))  
GATK strand bias: max(((b/(a+b))&ast;(c/(c+d)))/((a+c)/(a+b+c+d)),((d/(c+d))&ast;(a/(a+b)))/((a+c)/(a+b+c+d)))

24. **Strandbias**.  Default is 0.02.  
Exclude variants with high strand bias.

25. **Repeats**.  Default is 12.  
Maximum length of 1/2/3/4mers surrounding the SNV. Lengths must be factors of repeats e.g. n=8 would only consider 1/2/4mers because 3 is not a factor of 8.  Repeated regions frequently result in false positives.  When using this filter, a reference genome must be supplied so FiNGS can find the flanking sequences.

26. **SNVcluster50**.  Default is 2.  
Maximum number of SNVs in the input VCF in a 50 bp window centered on the current SNV.  SNVs must have at least 2 reads supporting them and a VAF>=5%.

27. **SNVcluster100**.  Default is 4.  
Maximum number of SNVs in the input VCF in a 100 bp window centered on the current SNV.  SNVs must have at least 2 reads supporting them and a VAF>=5%.

## Dictionary of values reported in the metrics files
The `filtered.results.date.txt` output file is the `normal.combined.txt` and `tumor.combined.txt` files joined by column, with additional columns for each filter containing TRUE/FALSE for whether or not that variant passed that filter. Each row is a single variant.  
Column names are delimited by full stops (periods). The first half defines the data type and the second the source. For example, `depth.tumor` and `depth.normal` contain the depth of a variant in the tumor and normal BAM files respectively.  
Non-integer values are rounded to 3 decimal places. Not all of the values reported are used for filtering.


Column | Description | Example
--- | --- | ---
**UID** | Unique Identifier for variant | 1:14677:G:A
**CHR** | Chromosome | 1
**POS** | Position on chromsome | 14677
**REF** | Reference allele | G
**ALT** | Alternate allele | A
**refcount** | Count of REF alleles | 22
**altcount** | Count of ALT alleles | 7
**varianttype** | SNV or INDEL | SNV
**depth** | Depth of all reads | 29
**vaf** | Variant Allele Frequency (altcount/depth) | 0.241
**raf** | Reference Allele Frequency (refcount/depth) | 0.759
**oaf** | Other Allele Frequency ((depth-altcount-refcount)/depth) | 0
**medianbaseq** | Median base quality (all reads) | 36
**medianbaseqref** | Median base quality (REF reads only) | 36
**medianbaseqalt** | Median base quality (ALT reads only) | 36
**medianmapq** | Median mapping quality (all reads) | 3
**medianmapqref** | Median mapping quality (REF reads only) | 11.5
**medianmapqalt** | Median mapping quality (ALT reads only) | 3
**zeros** | Total number of zero mapping quality reads | 13
**zerospersite** | Proportion of reads that have zero mapping quality | 0.448
**softreadlengthsrefmean** | Mean length of REF reads after soft clipping | 75.864
**softreadlengthsaltmean** | Mean length of ALT reads after soft clipping | 76
**goodoffsetproportion** | Proportion of variants that occur within the first 2/3rds of the mean read length | 0.897
**distancetoend1median** | Median distance to lefthand soft-clipped read end (all reads) | 27
**mad1** | Median absolute deviation of distancetoend1median | 10
**distancetoend2median** | Median distance to righthand soft-clipped read end (all reads) | 48
**mad2** | Median absolute deviation of distancetoend2median | 10
**distancetoend1medianref** | Median distance to lefthand soft-clipped read end (REF reads only) | 26.5
**madref1** | Median absolute deviation of distancetoend1medianref | 7.5
**distancetoend2medianref** | Median distance to righthand soft-clipped read end (REF reads only) | 48
**madref2** | Median absolute deviation of distancetoend2medianref | 7.5
**distancetoend1medianalt** | Median distance to lefthand soft-clipped read end (ALT reads only) | 27
**madalt1** | Median absolute deviation of distancetoend1medianalt | 10
**distancetoend2medianalt** | Median distance to righthand soft-clipped read end (ALT reads only) | 48
**madalt2** | Median absolute deviation of distancetoend2medianalt | 10
**shortestdistancetoendmedian** | Median of shortest distance of distancetoend1alt and distancetoend2alt | 20
**madaltshort** | Median absolute deviation of shortestdistancetoendmedian | 12
**sb** | Strand bias, see definition above | 0.207
**gsb** | GATK strand bias, see definition above | 0.264
**fishp** | P value for Fisher's exact test for strand bias | 1
**FoxoG** | Oxoguanine artifact orientation proportion, only relevant for for C>A or G>T mutations, defined in [Costello et al, 2013](https://academic.oup.com/nar/article/41/6/e67/2902364) | NA
**refld** | Edit distance for REF reads | 0
**altld** | Edit distance for ALT reads | 2
**refsecondprop** | Proportion of REF reads that have secondary alignments | 0
**altsecondprop** | Proportion of ALT reads that have secondary alignments | 0
**refbadorientationprop** | Proportion of REF reads with an inverted orientation | 0.045
**altbadorientationprop** | Proportion of ALT reads with an inverted orientation | 0
**refmatecontigcount** | Number of contigs seen in REF reads | 1
**altmatecontigcount** | Number of contigs seen in ALT reads | 1
**phalf** | P value for this variant belonging to the non-OxoG-artifact distribution, only calculated for for C>A or G>T mutations | NA
**poxog** | P value for this variant belonging to the OxoG-artifact distribution, only calculated for for C>A or G>T mutations | NA





