# microhomology
scripts for extracting alignments from MCScanX results and testing for microhomology

## add_kaks_to_MCScanX.pl

### SYNOPSIS
Calculates Ka and Ks for pairs of genes from an MCScanX collinearity file.

Aligns proteins using Clustal-Omega before back-translation into nucleotides and calculation of Ka and Ks using the method of Nei-Gojobori (using BioPerl).

Will detect CDS which are non-multiples of 3 (eg due to fragmentation in draft genomes), test for the correct frame, and trim from 5' or 3' end accordingly (script which ships with MCScanX does not do this and can lead to downstream errors in Ka/Ks calculation).

### OPTIONS
Run with no options or `-h` to see options:
```
-i|--in      [FILE] : MCScanX collinearity file
-p|--prot    [FILE] : fasta file of protein sequences
-c|--cds     [FILE] : fasta file of corresponding CDS (nucleotide)
-t|--threads [INT]  : number of aligner threads (default = 1)
-h|--help           : this message
```

### OUTPUTS
Annotated MCScanX collinearity file, with Ka and Ks as final 2 columns for every gene pair. Will write '-2' for cases where Ka/Ks cannot be calculated.
