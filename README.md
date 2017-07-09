# mlst

Scan contig files against traditional PubMLST typing schemes

## Quick Start

```
% mlst contigs.fa
contigs.fa  neisseria  11149  abcZ(672) adk(3) aroE(4) fumC(3) gdh(8) pdhC(4) pgm(6)

% mlst genome.gbk.gz
genome.gbk.gz  sepidermidis  184  arcC(16) aroE(1) gtr(2) mutS(1) pyrR(2) tpiA(1) yqiL(1)
```

## Installation

### Brew
If you are using the [OSX Brew](http://brew.sh/) or [LinuxBrew](http://brew.sh/linuxbrew/) packaging system:

```
% brew tap homebrew/science
% brew update
% brew install mlst
```

Or if you already have the old version installed:

```
% brew update
% brew upgrade mlst
```

### Source

```
% cd $HOME
% git clone https://github.com/tseemann/mlst.git
```   
 
### Dependencies

* [NCBI BLAST+ blastn](https://www.ncbi.nlm.nih.gov/books/NBK279671/) 
  * You probably have `blastn` already installed already.
  * If you use Brew, this will install the `blast` package for you.
* Perl modules *Moo* and *List::MoreUtils*
  * Debian: `sudo apt-get install libmoo-perl liblist-moreutils-perl`
  * Redhat: `sudo apt-get install perl-Moo perl-List-MoreUtils`
  * Most Unix: `sudo cpan Moo List::MoreUtils`
* Standard Unix commands
  * `file` (for file format detection)
  * `gzip` (for decompressing .gz files)

## Usage

Simply just give it a genome file in FASTA or GenBank format, optionally compressed with gzip!

```
% mlst contigs.fa
contigs.fa  neisseria  11149  abcZ(672) adk(3) aroE(4) fumC(3) gdh(8) pdhC(4) pgm(6)
```

It returns a tab-separated line containing
* the filename
* the matching PubMLST scheme name
* the ST (sequence type)
* the allele IDs

You can give it multiple files at once, and they can be in FASTA or GenBank format, and even compressed with gzip!

```
% mlst genomes/*
genomes/6008.fna        saureus         239  arcc(2)   aroe(3)   glpf(1)   gmk_(1)   pta_(4)   tpi_(4)   yqil(3)
genomes/strep.fasta.gz  ssuis             1  aroA(1)   cpn60(1)  dpr(1)    gki(1)    mutS(1)   recA(1)   thrA(1)
genomes/NC_002973.gbk   lmonocytogenes    1  abcZ(3)   bglA(1)   cat(1)    dapE(1)   dat(3)    ldh(1)    lhkA(3)
genomes/L550.gbk.gz     leptospira      152  glmU(26)  pntA(30)  sucA(28)  tpiA(35)  pfkB(39)  mreA(29)  caiB(29)
```

## Without auto-detection

You can force a particular scheme (useful for reporting systems):

```
% mlst --scheme neisseria NM*
FILE       SCHEME     ST    abcZ       adk     aroE      fumC       gdh      pdhC     pgm
NM003.fa   neisseria  4821  abcZ(222)  adk(3)  aroE(58)  fumC(275)  gdh(30)  pdhC(5)  pgm(255)
NM005.gbk  neisseria  177   abcZ(7)    adk(8)  aroE(10)  fumC(38)   gdh(10)  pdhC(1)  pgm(20)
NM011.fa   neisseria  11    abcZ(2)    adk(3)  aroE(4)   fumC(3)    gdh(8)   pdhC(4)  pgm(6)
NMC.gbk.gz neisseria  8     abcZ(2)    adk(3)  aroE(7)   fumC(2)    gdh(8)   pdhC(5)  pgm(2)
```

You can make `mlst` behave like older version before auto-detection existed
by  providing the `--legacy` parameter with the  `--scheme` parameter. In that case
it will print a fixed tabular output with a heading containing allele names specific to that scheme:

```
% mlst --legacy --scheme neisseria *.fa
FILE      SCHEME     ST    abcZ  adk  aroE  fumC  gdh  pdhC  pgm
NM003.fa  neisseria  11    2     3    4     3       8     4    6
NM009.fa  neisseria  11149 672   3    4     3       8     4    6
MN043.fa  neisseria  11    2     3    4     3       8     4    6
NM051.fa  neisseria  11    2     3    4     3       8     4    6
NM099.fa  neisseria  1287  2     3    4    17       8     4    6
NM110.fa  neisseria  11    2     3    4     3       8     4    6
```

## Available schemes

To see which PubMLST schemes are supported:

```
% mlst --list

abaumannii achromobacter aeromonas afumigatus cdifficile efaecium
hcinaedi hparasuis hpylori kpneumoniae leptospira
saureus xfastidiosa yersinia ypseudotuberculosis yruckeri
```

The above list is shortened. You can get more details using `mlst --longlist`.

## Missing data

MLST 2.0 does not just look for exact matches to full length alleles. 
It attempts to tell you as much as possible about what it found using the
notation below:

Symbol | Meaning | Length | Identity
---   | --- | --- | ---
`n`   | exact intact allele                   | 100%            | 100%
`~n`  | novel full length allele similar to n | 100%            | &ge; `--minid`
`n?`  | partial match to known allele         | &ge; `--mincov` | &ge; `--minid`
`-`   | allele missing                        | &lt; `--mincov` | &lt; `--minid`
`n,m` | multiple alleles                      | &nbsp;          | &nbsp;

### Tweaking the output

The output is TSV (tab-separated values). This makes it easy to parse 
and manipulate with Unix utilities like cut and sort etc. For example, 
if you only want the filename and ST you can do the following:
```
% mlst --scheme abaumanii AB*.fasta | cut -f1,3 > ST.tsv
```    
If you prefer CSV because it loads more smoothly into MS Excel, use the `--csv` option:
```
% mlst --csv Peptobismol.fna.gz > mlst.csv
```

## Mapping to genus/species

Included is a file called `db/scheme_species_map.tab` which has 3
tab-separated columns as follows:

```
#SCHEME GENUS   SPECIES
abaumannii      Acinetobacter   baumannii
abaumannii_2    Acinetobacter   baumannii
achromobacter   Achromobacter
aeromonas       Aeromonas
afumigatus      Aspergillus     afumigatus
arcobacter      Arcobacter
bburgdorferi    Borrelia        burgdorferi
bhampsonii      Brachyspira     hampsonii
bhenselae       Bartonella      henselae
borrelia        Borrelia
bpilosicoli     Brachyspira     pilosicoli
...
```

Note that that some schemes are species specific, and others are genus
specific, so the `SPECIES` column is empty.  Note that the same
species/genus can apply to multiple schemes, see `abaumanii` above.

## Updating the database

The `mlst` software comes bundled with the traditional MLST databases;
namely those schemes with less than 10 genes. I strive to make regular
releases with updated databases, but if this is not frequent enough you
can update the databases yourself using some tools included in the `scripts`
folder as follows:

```
# Figure out where mlst is installed
% which mlst
/home/user/sw/mlst

# Go into the scripts folder (you need to have write access!)
% cd /home/user/sw/mlst/scripts

# Run the downloader script (you need 'wget' installed)
% ./mlst-download_pub_mlst | bash

# Check it downloaded everything ok
% find pubmlst | less

# Save the old database folder
% mv ../db/pubmlst ../db/pubmlst.old

# Put the new folder there
% mv ./pubmlst ../db/

# Regenerate the BLAST database
% ./mlst-make_blast_db

# Check schemes are installed
% ../bin/mlst --list
```

## Bugs

Please submit via the [Github Issues page](https://github.com/tseemann/mlst/issues)

## Licence

[GPL v2](https://raw.githubusercontent.com/tseemann/mlst/master/LICENSE)

## Author

* Torsten Seemann
* Web: https://tseemann.github.io/
* Twitter: [@torstenseemann](https://twitter.com/torstenseemann)
* Blog: [The Genome Factory](https://thegenomefactory.blogspot.com/)

