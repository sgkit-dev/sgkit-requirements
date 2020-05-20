# Principal components analysis (PCA) to identify population structure


## Motivation

I'm a population geneticist studying a non-human diploid species.  I
have performed deep whole-genome sequencing on a set of individuals
collected from nature.  I have produced a set of genome-wide
analysis-ready SNP calls for those individuals.  I would like to
perform a principal components analysis (PCA) to explore evidence for
genetic population structure.

E.g., I'd like to know how many genetically distinct populations are
represented by the individuals I've sampled, and whether or not
genetic structure is associated with any other variables such as
location or date of sampling.


## Input data


### Analysis-ready variant calls

A set of genome-wide SNP calls generated from the sequence reads via a
variant calling pipeline such as GATK best practices. Can assume these
variant calls have been through sample QC and had variant filters
applied, i.e., are analysis-ready. Can also assume the variant calls
were originally generated in VCF format and have been converted to
Zarr format.


### Sample metadata

Data with any extra variables relating to the samples, such as gender,
date and location of sampling, etc. Assume these are available in a
simple text-based format, e.g., CSV or TSV.


## Analysis protocol


### 1. Select data variables

The following data variables will be needed for this analysis:

* Genotype calls: The genotype calls for each sample at each variant
  site.

* Variant chromosome and positions: The chromosome and position
  coordinates for each variant.

* Variant filters: For each variant, whether or not it passed all
  variant filters.


### 2. Select genome region

Subset the input data to retain only data within specified chromosome
or chromosomal region.

Notes: We may want to perform this analysis within a specific
chromosome or chromosomal region. For example, in *Anopheles gambiae*
we often use chromosome arm 3R from position 1-37 Mbp which excludes
heterochromatic regions. This is because some chromosome arms have
large inversions, and heterochromatic regions have low recombination
and tend to convey different patterns of population structure.


### 3. Apply variant filters

Subset the input data to retain only data for variants passing all
variant filters.

Notes: We remove variants which may have low quality data, using
previously created filter annotations included in the input data.


### 4. Perform an allele count

Perform an allele count across the whole cohort.

Notes: The allele count data is required for subsequent data selection
steps.


### 5. Remove multiallelic variants

Remove data for variants which have more than 2 alleles observed.

Notes: There are alternative analysis protocols which can handle
multiallelic variants, but this protocol is designed for biallelic
variants only.


### 6. Remove uninformative and weakly informative variants

Remove data for variants which are not segregating or are singletons
(minor allele count <= 1).

Optionally also remove low frequency variants (e.g., minor allele
frequency <1%) which are unlikely to carry much information about
population structure.

There is also an edge case where a variant may have a heterozygous
genotype for all samples. In this case the variant has a high minor
allele frequency but is not informative for PCA. Locate and remove any
such variants.


### 7. Locate unlinked variants and thin/prune

Locate a subset of variants that are not in strong linkage
disequilibrium (LD), then retain data only for this subset of
variants.

There are possible several methods for doing this. For the purposes of
this analysis, the following two methods are usually sufficient:

* Method A: Move a sliding window with a fixed number of variants
  along the data, compute LD between all pairs of variants within each
  window, excluding any variants with LD above a given threshold.

* Method B: Randomly downsample variants based on physical distance
  between them, e.g., remove more variants in regions of higher
  variant density.

Notes: Variants that occur nearby each other within the genome are
likely to be highly correlated due to lack of recombination between
them.


### 8. Downsample variants

Optionally, randomly downsample variants to a smaller number.

Notes: This is often done because a smaller number of variants may be
sufficient to identify population structure, and can reduce
computational resources needed (memory, CPU time).


### 9. Transform genotype calls into alternate allele dosages

Transform the genotype calls into the number of alternate allele calls
per genotype call. This turns each genotype call into a single
continuous variable.


### 10. Centre and scale

Centre and scale the genotype dosage data for each variant. There are
two possible methods.

* Method A: Standard scaling, subtract the mean and divide by the
  standard deviation.

* Method B: "Patterson" scaling, similar to standard scaling but
  assuming binomial variance.


### 11. Perform SVD

Perform a singular value decomposition on the scaled genotype dosage
matrix. There are (at least) two possible methods.

* Method A: Perform a standard (full) SVD.

* Method B: Perform a randomized (a.k.a. truncated) SVD.

Notes: The randomized SVD produces approximate results for a smaller
number of principal components, but is often sufficient for population
structure analysis where only up to ~20 components may be needed.


### 12. Visualise

Create scatter plots of samples using the transformed coordinates from
the SVD (e.g., PC1 versus PC2, etc.).

Often colour the samples using other variables, e.g., sampling
location.


## References

* Patterson et al. (2006) Population Structure and
  Eigenanalysis. https://doi.org/10.1371/journal.pgen.0020190
