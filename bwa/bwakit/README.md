## Introduction

Bwakit is a self-consistent installation-free package of scripts and precompiled
binaries, providing an end-to-end solution to read mapping. In addition to the
basic mapping functionality implemented in bwa, bwakit is able to generate
proper human reference genome and to take advantage of ALT contigs, if present,
to improve read mapping and to perform HLA typing for high-coverage human data.
It can remap name- or coordinate-sorted BAM with read group and barcode
information retained. Bwakit also *optionally* trims adapters (via
[trimadap][ta]), marks duplicates (via [samblaster][sb]) and sorts the final
alignment (via [samtools][smtl]).

Bwakit has two entry scripts: `run-gen-ref` which downloads and generates human
reference genomes, and `run-bwamem` which prints mapping command lines on the
standard output that can be piped to `sh` to execute. The two scripts will call
other programs or use data in `bwa.kit`. The following shows an example about
how to use bwakit:

```sh
# Download bwakit (or from <http://sourceforge.net/projects/bio-bwa/files/bwakit/> manually)
wget -O- http://sourceforge.net/projects/bio-bwa/files/bwakit/bwakit-0.7.11_x64-linux.tar.bz2/download \
  | gzip -dc | tar xf -
# Generate the GRCh38+ALT+decoy+HLA and create the BWA index
bwa.kit/run-gen-ref hs38d6   # download GRCh38 and write hs38d6.fa
bwa.kit/bwa index hs38d6.fa  # create BWA index
# mapping
bwa.kit/run-bwamem -o out hs38d6.fa read1.fq read2.fq | sh
```

The last mapping command line will generate the following files:

* `out.aln.bam`: unsorted alignments with ALT-aware mapping quality. In this
  file, one read may be placed on multiple overlapping ALT contigs at the same
  time even if the read is mapped better to some contigs than others. This makes
  it possible to analyze each contig independent of others.

* `out.hla.top`: best genotypes for HLA-A, -B, -C, -DQA1, -DQB1 and -DRB1 genes.

* `out.hla.all`: other possible genotypes on the six HLA genes.

* `out.log.*`: bwa-mem, samblaster and HLA typing log files.

Bwakit can be [downloaded here][res]. It is only available to x86_64-linux. The
scripts in the package are available in the [bwa/bwakit][kit] directory.
Packaging is done manually for now.

## Contents
```
bwa.kit
|-- README.md                  This README file.
|-- run-bwamem                 *Entry script* for the entire mapping pipeline.
|-- bwa                        *BWA binary*
|-- k8                         Interpreter for *.js scripts.
|-- bwa-postalt.js             Post-process alignments to ALT contigs/decoys/HLA genes.
|-- htsbox                     Used by run-bwamem for shuffling BAMs and BAM=>FASTQ.
|-- samblaster                 MarkDuplicates for reads from the same library. v0.1.20
|-- samtools                   SAMtools for sorting and SAM=>BAM conversion. v1.1
|-- seqtk                      For FASTQ manipulation.
|-- trimadap                   Trim Illumina PE sequencing adapters.
|
|-- run-gen-ref                *Entry script* for generating human reference genomes.
|-- resource-GRCh38            Resources for generating GRCh38
|   |-- hs38d6-decoy.nt.anno   Top decoy-to-nt hits. Not used by any scripts.
|   |-- hs38d6-decoy.rm.out    RepeatMasker report. Not used.
|   |-- hs38d6-extra.fa        Decoy and HLA gene sequences. Used by run-gen-ref.
|   `-- hs38d6.fa.alt          ALT-to-GRCh38 alignment. Used by run-gen-ref.
|
|-- run-HLA                    HLA typing for sequences extracted by bwa-postalt.js.
|-- typeHLA.sh                 Type one HLA-gene. Called by run-HLA.
|-- typeHLA.js                 HLA typing from exon-to-contig alignment. Used by typeHLA.sh.
|-- typeHLA-selctg.js          Select contigs overlapping HLA exons. Used by typeHLA.sh.
|-- fermi2.pl                  Fermi2 wrapper. Used by typeHLA.sh for de novo assembly.
|-- fermi2                     Fermi2 binary. Used by fermi2.pl.
|-- ropebwt2                   RopeBWT2 binary. Used by fermi2.pl.
|-- resource-human-HLA         Resources for HLA typing
|   |-- HLA-ALT-exons.bed      Exonic regions of HLA ALT contigs. Used by typeHLA.sh.
|   |-- HLA-CDS.fa             CDS of HLA-{A,B,C,DQA1,DQB1,DRB1} genes from IMGT/HLA-3.18.0.
|   |-- HLA-ALT-type.txt       HLA types for each HLA ALT contig. Not used.
|   `-- HLA-ALT-idx            BWA indices of each HLA ALT contig. Used by typeHLA.sh
|       `-- (...)
|
`-- doc                        BWA documentations
    |-- bwa.1                  Manpage
    |-- NEWS.md                Release Notes
    |-- README.md              GitHub README page
    `-- README-alt.md          Documentation for ALT mapping
```

[res]: https://sourceforge.net/projects/bio-bwa/files/bwakit
[sb]: https://github.com/GregoryFaust/samblaster
[ta]: https://github.com/lh3/seqtk/blob/master/trimadap.c
[smtl]: http://www.htslib.org
[kit]: https://github.com/lh3/bwa/tree/master/bwakit
