# *Anopheles gambiae* 1000 Genomes (Ag1000G) Phase 2 SNP callset

The Ag1000G phase 2 callset contains analysis-ready SNP calls from
whole-genome Illumina sequencing of 1142 wild-caught Anopheles gambiae
mosquitoes.

## Data access

### Variant calls

Variation data are available in VCF format from the public FTP
site:

* ftp://ngs.sanger.ac.uk/production/ag1000g/phase2/AR1/variation/main/vcf/all/

These data have also been converted to Zarr format and deployed to a
Google Cloud Storage public bucket at the following location:

* gs://ag1000g-release/phase2.AR1/variation/main/zarr/all/ag1000g.phase2.ar1

These data can be accessed via the following Python code, assuming the
Zarr and gcsfs packages are installed:

```python
import zarr
import gcsfs

store_path = 'ag1000g-release/phase2.AR1/variation/main/zarr/all/ag1000g.phase2.ar1'
gcs = gcsfs.GCSFileSystem(token='anon')
store = gcs.get_mapper(store_path)
callset = zarr.open_consolidated(store=store)
```


## References

* The Anopheles gambiae 1000 Genomes Consortium (2020) Genome
  variation and population structure among 1,142 mosquitoes of the
  African malaria vector species Anopheles gambiae and Anopheles
  coluzzii. https://www.biorxiv.org/content/10.1101/864314v2
