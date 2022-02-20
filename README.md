
# ggtranscript <img src="man/figures/ggtranscript_logo_cropped.svg" align="right" height="139" />

<!-- badges: start -->

[![GitHub
issues](https://img.shields.io/github/issues/dzhang32/ggtranscript)](https://github.com/dzhang32/ggtranscript/issues)
[![GitHub
pulls](https://img.shields.io/github/issues-pr/dzhang32/ggtranscript)](https://github.com/dzhang32/ggtranscript/pulls)
[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![R-CMD-check-bioc](https://github.com/dzhang32/ggtranscript/workflows/R-CMD-check-bioc/badge.svg)](https://github.com/dzhang32/ggtranscript/actions)
[![Codecov test
coverage](https://codecov.io/gh/dzhang32/ggtranscript/branch/main/graph/badge.svg)](https://app.codecov.io/gh/dzhang32/ggtranscript?branch=main)
<!-- badges: end -->

## Overview

`ggtranscript` is a `ggplot2` extension that makes it easy visualize
transcript structure and annotation.

## Installation

``` r
# you can install the development version of ggtranscript from GitHub:
# install.packages("devtools")
devtools::install_github("dzhang32/ggtranscript")
```

## Usage

`ggtranscript` introduces 5 new `geom`s (`geom_range()`,
`geom_half_range()`, `geom_intron()`, `geom_junction()` and
`geom_junction_label_repel()`) and several helper functions designed to
visualize transcript structure and annotation.

`geom_range()` and `geom_intron()` enable the plotting of exons and
introns, the core components of transcript annotation. `ggtranscript`
also provides `to_intron()`, which convert exon co-ordinates to the
corresponding introns. Together, `ggtranscript` enables users to plot
transcript structures with only exons as the required input and a few
lines of code.

``` r
library(magrittr)
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
library(ggplot2)
library(ggtranscript)

# to illustrate the package's functionality
# ggtranscript includes example transcript annotation
sod1_annotation %>% head()
#> # A tibble: 6 × 8
#>   seqnames  start    end strand type  gene_name transcript_name transcript_biot…
#>   <fct>     <int>  <int> <fct>  <fct> <chr>     <chr>           <chr>           
#> 1 21       3.17e7 3.17e7 +      gene  SOD1      <NA>            <NA>            
#> 2 21       3.17e7 3.17e7 +      tran… SOD1      SOD1-202        protein_coding  
#> 3 21       3.17e7 3.17e7 +      exon  SOD1      SOD1-202        protein_coding  
#> 4 21       3.17e7 3.17e7 +      CDS   SOD1      SOD1-202        protein_coding  
#> 5 21       3.17e7 3.17e7 +      star… SOD1      SOD1-202        protein_coding  
#> 6 21       3.17e7 3.17e7 +      exon  SOD1      SOD1-202        protein_coding

# extract exons
sod1_exons <- sod1_annotation %>% dplyr::filter(type == "exon")

sod1_exons %>%
    ggplot(aes(
        xstart = start,
        xend = end,
        y = transcript_name
    )) +
    geom_range(
        aes(fill = transcript_biotype)
    ) +
    geom_intron(
        data = to_intron(sod1_exons, "transcript_name"),
        aes(strand = strand),
        arrow.min.intron.length = 500,
    )
```

<img src="man/figures/README-geom-range-intron-1.png" width="100%" />

`geom_range()` is designed to be used for any range-based genomic
annotation. For example, when plotting protein-coding transcripts, this
can be used to visually distinguish the coding segments from UTRs.

``` r
# filter for exons from protein coding transcripts
sod1_exons_prot_cod <- sod1_exons %>%
    dplyr::filter(transcript_biotype == "protein_coding")

# obtain cds
sod1_cds <- sod1_annotation %>% dplyr::filter(type == "CDS")

sod1_exons_prot_cod %>%
    ggplot(aes(
        xstart = start,
        xend = end,
        y = transcript_name
    )) +
    geom_range(
        fill = "white",
        height = 0.25
    ) +
    geom_range(
        data = sod1_cds
    ) +
    geom_intron(
        data = to_intron(sod1_exons_prot_cod, "transcript_name"),
        aes(strand = strand),
        arrow.min.intron.length = 500,
    )
```

<img src="man/figures/README-geom-range-intron-w-cds-1.png" width="100%" />

`geom_half_range()` takes advantage of the vertical symmetry of
transcript annotation by plotting only half of a range on the top or
bottom of a transcript structure. This can be useful for visualizing the
differences between transcript structure.

``` r
# geom_half_range() can be useful for comparing between two transcripts
# enabling visualization of one transcript on the top, other on the bottom
sod1_201_exons <- sod1_exons %>% dplyr::filter(transcript_name == "SOD1-201")
sod1_201_cds <- sod1_cds %>% dplyr::filter(transcript_name == "SOD1-201")
sod1_202_exons <- sod1_exons %>% dplyr::filter(transcript_name == "SOD1-202")
sod1_202_cds <- sod1_cds %>% dplyr::filter(transcript_name == "SOD1-202")

sod1_201_202_plot <- sod1_201_exons %>%
    ggplot(aes(
        xstart = start,
        xend = end,
        y = "SOD1-201/202"
    )) +
    geom_half_range(
        fill = "white",
        height = 0.125
    ) +
    geom_half_range(
        data = sod1_201_cds
    ) +
    geom_intron(
        data = to_intron(sod1_201_exons, "transcript_name")
    ) +
    geom_half_range(
        data = sod1_202_exons,
        range.orientation = "top",
        fill = "white",
        height = 0.125
    ) +
    geom_half_range(
        data = sod1_202_cds,
        range.orientation = "top",
        fill = "purple"
    ) +
    geom_intron(
        data = to_intron(sod1_202_exons, "transcript_name")
    )

sod1_201_202_plot
```

<img src="man/figures/README-geom-half-range-1.png" width="100%" />

As a `ggplot2` extension, `ggtranscript` geoms inherit the the
functionality of `ggplot2`. For instance, by leveraging
`coord_cartesian()`, it can be useful to zoom in on areas of interest.

``` r
sod1_201_202_plot + coord_cartesian(xlim = c(31659500, 31660000))
```

<img src="man/figures/README-geom-half-range-zoomed-1.png" width="100%" />

`geom_junction()` enables to plotting of junction curves, which can be
useful to overlay across transcript structures. `geom_junction_repel()`
labels junction curves, for example with their counts.

``` r
# ggtranscript includes a set of example (unannotated) junctions
# originating from GTEx and downloaded via the Bioconductor package snapcount
sod1_junctions
#> # A tibble: 5 × 5
#>   seqnames    start      end strand mean_count
#>   <fct>       <int>    <int> <fct>       <dbl>
#> 1 chr21    31659787 31666448 +           0.463
#> 2 chr21    31659842 31660554 +           0.831
#> 3 chr21    31659842 31663794 +           0.316
#> 4 chr21    31659842 31667257 +           4.35 
#> 5 chr21    31660351 31663789 +           0.324

# add transcript_name to junctions for plotting
sod1_junctions <- sod1_junctions %>%
    dplyr::mutate(transcript_name = "SOD1-201")

sod1_201_exons %>%
  ggplot(aes(
    xstart = start,
    xend = end,
    y = transcript_name
  )) +
  geom_range(
    fill = "white", 
    height = 0.25
  ) +
  geom_range(
    data = sod1_201_cds
  ) + 
  geom_intron(
    data = to_intron(sod1_201_exons, "transcript_name")
  ) + 
  geom_junction(
    data = sod1_junctions,
    junction.y.max = 0.5
  ) +
  geom_junction_label_repel(
    data = sod1_junctions,
    aes(label = round(mean_count, 2)),
    junction.y.max = 0.5
  )
```

<img src="man/figures/README-geom-junction-1.png" width="100%" />

Alternatively, users may prefer to map the count to the thickness of the
junction line. As a `ggplot2` extension, this can be done intuitively as
well as the modification of scales and themes which together, allow
users to easily create informative, publication-ready plots.

``` r
sod1_201_exons %>%
  ggplot(aes(
    xstart = start,
    xend = end,
    y = transcript_name
  )) +
  geom_range(
    fill = "white", 
    height = 0.25
  ) +
  geom_range(
    data = sod1_201_cds
  ) + 
  geom_intron(
    data = to_intron(sod1_201_exons, "transcript_name")
  ) + 
  geom_junction(
    data = sod1_junctions,
    aes(size = mean_count),
    junction.y.max = 0.5, 
    ncp = 30, 
    colour = "purple"
  ) + 
  scale_size_continuous(range = c(0.1, 1), guide = "none") + 
  xlab("Genomic position (chr21)") + 
  ylab("Transcript name") + 
  theme_bw()
```

<img src="man/figures/README-geom-junction-pub-1.png" width="100%" />

## Code of Conduct

Please note that the `ggtranscript` package is released with a
[Contributor Code of
Conduct](http://bioconductor.org/about/code-of-conduct/). By
contributing to this project, you agree to abide by its terms.

## Development tools

-   Continuous code testing is possible thanks to [GitHub
    actions](https://www.tidyverse.org/blog/2020/04/usethis-1-6-0/)
    through *[usethis](https://CRAN.R-project.org/package=usethis)*,
    *[remotes](https://CRAN.R-project.org/package=remotes)*, and
    *[rcmdcheck](https://CRAN.R-project.org/package=rcmdcheck)*
    customized to use [Bioconductor’s docker
    containers](https://www.bioconductor.org/help/docker/) and
    *[BiocCheck](https://bioconductor.org/packages/3.14/BiocCheck)*.
-   Code coverage assessment is possible thanks to
    [codecov](https://codecov.io/gh) and
    *[covr](https://CRAN.R-project.org/package=covr)*.
-   The [documentation website](http://dzhang32.github.io/ggtranscript)
    is automatically updated thanks to
    *[pkgdown](https://CRAN.R-project.org/package=pkgdown)*.
-   The code is styled automatically thanks to
    *[styler](https://CRAN.R-project.org/package=styler)*.
-   The documentation is formatted thanks to
    *[devtools](https://CRAN.R-project.org/package=devtools)* and
    *[roxygen2](https://CRAN.R-project.org/package=roxygen2)*.

This package was developed using
*[biocthis](https://bioconductor.org/packages/3.14/biocthis)*.
