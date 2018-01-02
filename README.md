[![Travis-CI Build Status](https://travis-ci.org/arendsee/rhmmer.svg?branch=master)](https://travis-ci.org/arendsee/rhmmer)
[![Coverage Status](https://img.shields.io/codecov/c/github/arendsee/rhmmer/master.svg)](https://codecov.io/github/arendsee/rhmmer?branch=master)
[![CRAN_Status_Badge](http://www.r-pkg.org/badges/version/rhmmer)](https://cran.r-project.org/package=rhmmer)

# rhmmer

[HMMER](hmmer.ord) is a powerful package for profile HMM analysis. If you want
to interface with the web server through R, for example to search for domains
in a small number of proteins, consider using the Bio3D package.`rhmmer` is
specifically designed for working with the standalone HMMer tool.

## Installation

To install the github version


```r
library(devtools)
install_github('arendsee/rhmmer')
```

## Examples

`rhmmer` currently exports exactly two functions: `read_domtblout` and
`read_tblout`. These read the `hmmscan` outputs specified by the `--domtblout`
and `--tblout` arguments, respectively.

The `domtblout` files have the format:

```
#                                                                            --- full sequence --- -------------- this domain -------------   hmm coord   ali coord   env coord
# target name        accession   tlen query name           accession   qlen   E-value  score  bias   #  of  c-Evalue  i-Evalue  score  bias  from    to  from    to  from    to  acc description of target
#------------------- ---------- ----- -------------------- ---------- ----- --------- ------ ----- --- --- --------- --------- ------ ----- ----- ----- ----- ----- ----- ----- ---- ---------------------
Rer1                 PF03248.12   171 AT2G18240.1          -            221   2.3e-61  206.4  10.4   1   2   4.3e-65   7.1e-61  204.8  11.2     4   168    24   181    21   183 0.96 Rer1 family
Rer1                 PF03248.12   171 AT2G18240.1          -            221   2.3e-61  206.4  10.4   2   2      0.43   7.2e+03   -3.6   0.0    55    67   187   199   184   217 0.69 Rer1 family
DUF4220              PF13968.5    352 AT5G45530.1          -            798   4.6e-78  263.0   0.0   1   1   1.5e-81   1.3e-77  261.5   0.0     1   344    51   396    51   427 0.78 Domain of unknown function (DUF4220)
DUF594               PF04578.12    55 AT5G45530.1          -            798   4.7e-24   83.6   1.5   1   1   1.3e-27   1.1e-23   82.4   1.5     3    55   726   778   724   778 0.96 Protein of unknown function, DUF594
DEAD                 PF00270.28   176 AT1G27880.2          -            890   6.7e-20   71.5   0.1   1   1   3.4e-23   1.9e-19   70.0   0.1     2   171   272   433   271   438 0.83 DEAD/DEAH box helicase
...
#
# Program:         hmmscan
# Version:         3.1b2 (February 2015)
# Pipeline mode:   SCAN
# Query file:      five.faa
# Target file:     /home/z/db/Pfam-A.hmm
# Option settings: hmmscan --tblout x.tblout --domtblout x.domtblout --pfamtblout x.pfamtblout --noali /home/z/db/Pfam-A.hmm five.faa 
# Current dir:     /home/z/src/git/rhmmer/tests/testthat/sample-data
# Date:            Fri Dec 15 02:09:00 2017
# [ok]
```

This is tricky to parse. It is mostly space delimited, but spaces appear freely
in the `description of target` column. The column names, as given, cannot be
directly used since they 1) contain illegal characters and 2) are not unique
unless information from two rows is considered (e.g. `ali_from` versus
`env_from`). The metadata at the end of the file I do not currently extract,
though I will likely add handling for this in the future.


```r
library(rhmmer)
domtblout <- system.file('extdata', 'example.domtblout.txt', package='rhmmer')
read_domtblout(domtblout)
```

```
## # A tibble: 70 x 23
##    domai… doma… doma… quer… quer…  qlen sequence… sequ… seque… doma… doma…
##  * <chr>  <chr> <int> <chr> <chr> <int>     <dbl> <dbl>  <dbl> <int> <int>
##  1 Rer1   PF03…   171 AT2G… -       221  2.30e⁻⁶¹ 206   10.4       1     2
##  2 Rer1   PF03…   171 AT2G… -       221  2.30e⁻⁶¹ 206   10.4       2     2
##  3 DUF42… PF13…   352 AT5G… -       798  4.60e⁻⁷⁸ 263    0         1     1
##  4 DUF594 PF04…    55 AT5G… -       798  4.70e⁻²⁴  83.6  1.50      1     1
##  5 DEAD   PF00…   176 AT1G… -       890  6.70e⁻²⁰  71.5  0.100     1     1
##  6 Helic… PF00…   111 AT1G… -       890  3.20e⁻¹⁸  66.0  0         1     2
##  7 Helic… PF00…   111 AT1G… -       890  3.20e⁻¹⁸  66.0  0         2     2
##  8 ResIII PF04…   171 AT1G… -       890  1.20e⁻ ³  18.8  0.100     1     2
##  9 ResIII PF04…   171 AT1G… -       890  1.20e⁻ ³  18.8  0.100     2     2
## 10 TIR    PF01…   176 AT1G… -       897  7.30e⁻⁴⁶ 156    0         1     1
## # ... with 60 more rows, and 12 more variables: domain_cevalue <dbl>,
## #   domain_ievalue <dbl>, domain_score <dbl>, domain_bias <dbl>,
## #   hmm_from <int>, hmm_to <int>, ali_from <int>, ali_to <int>,
## #   env_from <int>, env_to <int>, acc <dbl>, description <chr>
```

The `tblout` output is fairly similar and presents the same parsing difficulties:

```
#                                                               --- full sequence ---- --- best 1 domain ---- --- domain number estimation ----
# target name        accession  query name           accession    E-value  score  bias   E-value  score  bias   exp reg clu  ov env dom rep inc description of target
#------------------- ---------- -------------------- ---------- --------- ------ ----- --------- ------ -----   --- --- --- --- --- --- --- --- ---------------------
Rer1                 PF03248.12 AT2G18240.1          -            2.3e-61  206.4  10.4   7.1e-61  204.8  11.2   1.3   2   0   0   2   2   2   1 Rer1 family
DUF4220              PF13968.5  AT5G45530.1          -            4.6e-78  263.0   0.0   1.3e-77  261.5   0.0   1.7   1   1   0   1   1   1   1 Domain of unknown function (DUF4220)
DUF594               PF04578.12 AT5G45530.1          -            4.7e-24   83.6   1.5   1.1e-23   82.4   1.5   1.7   1   0   0   1   1   1   1 Protein of unknown function, DUF594
DEAD                 PF00270.28 AT1G27880.2          -            6.7e-20   71.5   0.1   1.9e-19   70.0   0.1   1.8   1   0   0   1   1   1   1 DEAD/DEAH box helicase
Helicase_C           PF00271.30 AT1G27880.2          -            3.2e-18   66.0   0.0     3e-17   62.9   0.0   2.5   2   0   0   2   2   2   1 Helicase conserved C-terminal domain
ResIII               PF04851.14 AT1G27880.2          -             0.0012   18.8   0.1    0.0084   16.0   0.0   2.4   2   1   0   2   2   2   1 Type III restriction enzyme, res subunit
...
#
# Program:         hmmscan
# Version:         3.1b2 (February 2015)
# Pipeline mode:   SCAN
# Query file:      five.faa
# Target file:     /home/z/db/Pfam-A.hmm
# Option settings: hmmscan --tblout x.tblout --domtblout x.domtblout --pfamtblout x.pfamtblout --noali /home/z/db/Pfam-A.hmm five.faa 
# Current dir:     /home/z/src/git/rhmmer/tests/testthat/sample-data
# Date:            Fri Dec 15 02:09:00 2017
# [ok]
```


```r
tblout <- system.file('extdata', 'example.tblout.txt', package='rhmmer')
read_tblout(tblout)
```

```
## # A tibble: 37 x 19
##    domai… domai… quer… quer… sequence… sequ… seque… best_dom… best… best_…
##  * <chr>  <chr>  <chr> <chr>     <dbl> <dbl>  <dbl>     <dbl> <dbl>  <dbl>
##  1 Rer1   PF032… AT2G… -      2.30e⁻⁶¹ 206   10.4    7.10e⁻⁶¹ 205   11.2  
##  2 DUF42… PF139… AT5G… -      4.60e⁻⁷⁸ 263    0      1.30e⁻⁷⁷ 262    0    
##  3 DUF594 PF045… AT5G… -      4.70e⁻²⁴  83.6  1.50   1.10e⁻²³  82.4  1.50 
##  4 DEAD   PF002… AT1G… -      6.70e⁻²⁰  71.5  0.100  1.90e⁻¹⁹  70.0  0.100
##  5 Helic… PF002… AT1G… -      3.20e⁻¹⁸  66.0  0      3.00e⁻¹⁷  62.9  0    
##  6 ResIII PF048… AT1G… -      1.20e⁻ ³  18.8  0.100  8.40e⁻ ³  16.0  0    
##  7 TIR    PF015… AT1G… -      7.30e⁻⁴⁶ 156    0      1.20e⁻⁴⁵ 155    0    
##  8 NB-ARC PF009… AT1G… -      1.90e⁻¹⁷  63.1  0      3.50e⁻¹⁷  62.3  0    
##  9 LRR_3  PF077… AT1G… -      1.00e⁻ ⁷  31.4  1.10   1.00e⁻ ⁷  31.4  1.10 
## 10 LRR_8  PF138… AT1G… -      2.30e⁻ ⁷  30.4  7.00   4.80e⁻ ³  16.5  0    
## # ... with 27 more rows, and 9 more variables: domain_number_exp <dbl>,
## #   domain_number_reg <int>, domain_number_clu <int>,
## #   domain_number_ov <int>, domain_number_env <int>,
## #   domain_number_dom <int>, domain_number_rep <int>,
## #   domain_number_inc <chr>, description <chr>
```


## What `rhmmer` does and doesn't

What `rhmmer` currently does

 * parse HMMER output (tblout and domtblout) into tidy data frames

What `rhmmer` may do in the future

 * provide visualization or analysis of HMMER results 

 * provide simple wrappers for calling some of the HMMER functions

 * parse metadata from the comments in HMMER output files

 * read pfamtblout and the raw output (with alignments)

 * read the HMM models

 * download databases for you (e.g. PFAM)

 * access precompiled PFAM results

What `rhmmer` will never do

 * interface with the HMMER web server (use Bio3D)

 * duplicate the whole HMMER api (just use the CLI tool)


## TODO

 [ ] Rewrite the parser in C++

 [ ] Parse out the file metadata
