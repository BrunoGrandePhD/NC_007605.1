# NC_007605.1

## Disclosure

These files have been manually edited and thus should not be used as reference data. They cannot be readily recreated. They should only be used for rough analyses. Otherwise, please use the gene annotations available at [NCBI](https://www.ncbi.nlm.nih.gov/nucleotide/NC_007605). However, take note of the issues with the NCBI annotations listed below. 

## Motivation

I found several issues with the NCBI gene annotations for EBV ([NC_007605.1](https://www.ncbi.nlm.nih.gov/nucleotide/NC_007605)):
* Some transcripts use multiple gene names (_e.g._ EBNA-1 and BKRF1).
* Some genes are lumped together despite the utility of them being considered separately (_e.g._ LMP2A and LMP2B). 
* Several transcripts are less than 30 bases long.

## Changes

Here is a summary of the changes I made:
1. I filtered for `gene`, `exon` and `CDS` features. I also manually kept EBER1/2, which were annotated as `sequence_feature`.
2. I replaced the `NC_007605.1` with `chrEBV`, because that's the reference name used in the GRCh38 genome I use. 
3. I removed all attributes except for `ID`, `Parent`, `gene`, `product` and `gene_biotype`. 
4. I trimmed any extra text in the `product` attribute value. For example, I replace `EBNA3A nuclear protein` with just `EBNA3A`. 
5. I renamed the `product` tag with `gene_name` for it to be picked up by `gffread`. 
6. I ordered the features by gene and ensured that each gene has a `gene` feature and at least one `exon` feature or one `CDS` feature. `gffread` takes care of creating an exon if a `CDS` exists without a corresponding exon. 
    * This is where I split certain genes based on whether I wanted to measure them separately. In general, I erred on the side of splitting rather than not. Notably, LMP2A and LMP2B are separate genes from LMP2
    * An unintended consequence of this approach is that some genes are left without any coding sequence because the genes that were split away have them instead. This is the case for LMP2 and EBNA, among others. **I will look into this and determine whether I should remove these "orphan" genes.** My fear is that they overlap the CDS regions of the genes that were split away and will complicate transcript expression quantification. 
    * This lead to some genes (_e.g._ LMP2A and LMP2B) spanning most of the genome because they have exons before and after the origin of replication (_i.e._ genome coordinate 0). 
7. The GFF3 file was validated and simplified using `gffread`. See command below. 
8. The uninformative IDs (_e.g._ `gene0`) were replaced with actual gene names. 
9. A transcriptome FASTA file was generated using `gffread`. 

## Commands

```
# My manual edits come first. 

# Then, here are the commands I ran to produce the final files.

# Simplify the GFF3 file to the minimum required. 
# This also has the benefit of highlighting any issues with
# the input file thanks to the `-E` option. 
gffread -o- -E NC_007605.1.genes.simplified.custom.gff3 > NC_007605.1.genes.simplified.custom.gffread.gff3  

# Create a sed script for replacing the uninformative IDs (_e.g._ `gene0`)
# with actual gene names. I confirmed that the gene names are unique.
grep "ID=" NC_007605.1.genes.simplified.custom.gffread.gff3 \
    | sed -r \
        -e 's/.*ID=(.*)/\1/' \
        -e 's/;gene_name=/\\b:/' \
        -e 's/^/s:\\b/' \
        -e 's/$/:g/' \
    > replace.txt

# Use the sed script to replace all instances of the IDs with the
# corresponding gene name. 
sed -r -f replace.txt < NC_007605.1.genes.simplified.custom.gffread.gff3 > NC_007605.1.genes.simplified.custom.gffread.fixed_ids.gff3

# Finally, generate the transcriptome FASTA file using gffread. 
# I used my GRCh38 reference genome. You can also leave the `NC_007605.1`
# chromosome name intact and use the GenBank FASTA file as input instead.
gffread -E -w /dev/stdout -g genome.fa NC_007605.1.genes.simplified.custom.gffread.fixed_ids.gff3 > NC_007605.1.genes.simplified.custom.gffread.fixed_ids.fasta
```
