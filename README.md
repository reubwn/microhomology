# microhomology
scripts for extracting alignments from MCScanX results and testing for microhomology

## add_kaks_to_MCScanX.pl

### Synopsis
Calculates Ka and Ks for pairs of genes from an MCScanX collinearity file.

Aligns proteins using Clustal-Omega before back-translation into nucleotides and calculation of Ka and Ks using the method of Nei-Gojobori (using BioPerl).

Will detect CDS which are non-multiples of 3 (eg due to fragmentation in draft genomes), test for the correct frame, and trim from 5' or 3' end accordingly (script which ships with MCScanX does not do this and can lead to downstream errors in Ka/Ks calculation).

### Options
Run with no options or `-h` to see options:
```
-i|--in      [FILE] : MCScanX collinearity file
-p|--prot    [FILE] : fasta file of protein sequences
-c|--cds     [FILE] : fasta file of corresponding CDS (nucleotide)
-t|--threads [INT]  : number of aligner threads (default = 1)
-h|--help           : this message
```

### Outputs
Annotated MCScanX collinearity file, with Ka and Ks as final 2 columns for every gene pair. Will write '-2' for cases where Ka/Ks cannot be calculated.

## get_alignments_from_MCScanX.pl

### Synopsis
Generates alignments for pairs of genes with a given minimum Ks from an MCScanX collinearity file (annotated with Ka/Ks).

If protein and CDS files (`-p` & `-c`) are provided, will do protein alignment followed by backtranslation; if gff and genome files (`-g` & `-f`) are provided, will do alignment of whole gene region.

Aligner (clustalo for proteins, mafft for nucleotides) must be executable and in `$PATH`.

Assumes Ks values are in the **final column** of the collinearity file (default).

### Options
Run with no options or `-h` to see options:
```
  -i|--in      [FILE]  : MCScanX collinearity file, annotated with Ka/Ks values
  -p|--prot    [FILE]  : fasta file of protein sequences
  -c|--cds     [FILE]  : fasta file of corresponding CDS (nucleotide)
  -f|--fasta   [FILE]  : fasta file of genome
  -g|--gff     [FILE]  : gff file of gene regions
  -o|--out     [STR]   : outfile prefix (default = 'ALN'); alignments will be written to ALN.<NUM>.fasta
  -d|--outdir  [DIR]   : dirname to save alignments (default = 'alignments')
  -k|--minks   [FLOAT] : minimum Ks between genes (default >= 0.5)
  -t|--threads [INT]   : number of aligner threads (default = 1)
  -h|--help            : this message
```

### Usage
1. To generate codon alignments for CDS using Clustal-Omega:
```
>> get_alignments_from_MCScanX.pl -i <Xy.collinearity> -p </path/to/proteins/> -c </path/to/cds/> -d <dirname> -k <minKs> -t <threads>
```
2. To generate whole gene (ie exonts + introns) alignments using MAFFT:
```
>> get_alignments_from_MCScanX.pl -i <Xy.collinearity> -f </path/to/genome/fasta/> -c </path/to/gff/> -d <dirname> -k <minKs> -t <threads>
```

### Outputs
A bunch of pairwise alignments in the directory specified by `-d`.

## microhomology.pl

### Synopsis
Calculates the number of exactly identical windows of length `-w <INT>` that exist between two prealigned nucleotide sequences.

Uses a sliding window incrementing by 1, and scales number of identical windows by the length of each alignment.

### Options
Run with no options or `-h` to see options:
```
  -d|--dir    [DIR]  : dirname of directory containing alignments (fasta format)
  -o|--out    [FILE] : outfile prefix (default = mhom.table)
  -w|--window [INT]  : window size range, comma delim, defined as: \"<SIZE>,<END>,<STEP>\" (default = 10,10,1)
  -h|--help          : this message
```
**NOTE:** `STEP` above only affect the step size *between* windows, ie. how much window size will increase between runs. Within a single run, the window will always slide along the alignment by 1 bp each iteration.

### Usage
1. To calculate number of idenical windows from window size 1 to 40, increasing window size by 2 each round:
```
>> microhomology.pl -d <alignments/> -w \"1,40,2\" -o out
```

### Outputs
A tab-delim text file where rows are window sizes and columns are alignments.

The value in each cell is defined as the number of identical regions of size [ROW] in alignment [COLUMN] per base, ie. divided by the length of that alignment. To get a value per kb, just multiply the matrix by 1000.
