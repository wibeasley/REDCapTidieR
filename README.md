
<!-- README.md is generated from README.Rmd. Please edit that file -->

# REDCapTidieR

<p align="center">
<img src="man/figures/REDCapTidieR.png" alt="drawing" width="150" align ="right"/>
</p>
<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![R-CMD-check](https://github.com/CHOP-CGTDataOps/REDCapTidieR/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/CHOP-CGTDataOps/REDCapTidieR/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

[REDCap](https://www.project-redcap.org/) is a powerful database
solution used by many research institutions:

> “REDCap is a secure web application for building and managing online
> surveys and databases. While REDCap can be used to collect virtually
> any type of data in any environment (including compliance with 21 CFR
> Part 11, FISMA, HIPAA, and GDPR), it is specifically geared to support
> online and offline data capture for research studies and operations.”

The goal of `REDCapTidieR` is to provide users with a simple API for
extracting REDCap databases and turning them into tidy `tibble`s. This
makes for easily joinable tables with only the data necessary for
specific REDCap forms, instead of the singular complex table typically
received from a single export.

`REDCapTidieR` will be of great use to any databases using:

-   Repeating Instruments
-   Longitudinal Events
-   Longitudinal Arms

`REDCapTidieR` is meant to be used in conjunction with the
[`REDCapR`](https://ouhscbbmc.github.io/REDCapR/) package. It is built
to be reliant on data extraction from underlying functions including
`redcap_read_oneshot()`, `redcap_metadata_read()`, and
`redcap_event_instruments()`. You may have even noticed the name for
this package is just a “tidy”-fied addition to `REDCapR`! Therefore, we
owe much credit and acknowledgement to the developers of `REDCapR`.

## Installation

You can install the development version of `REDCapTidieR` like so:

``` r
devtools::install_github("CHOP-CGTDataOps/REDCapTidieR")
```

## Examples

### Exporting REDCap Databases to Tidy `tibbles`

Using the main function is simple and only requires a REDCap URI and an
API token with read/write privileges:

``` r
redcap_export <- read_redcap_tidy(redcap_uri, token)
```

Where a sample output containing repeating instruments may look like:

``` r
redcap_export
```

``` bash
# A tibble: 3 × 3
  redcap_form_name redcap_data  structure   
  <chr>            <list>       <chr>       
1 repeated         <df [9 × 6]> repeating   
2 nonrepeated      <df [8 × 5]> nonrepeating
3 nonrepeated2     <df [3 × 5]> nonrepeating
```

Where the contents of `redcap_data` contain only the data associated
with the form:

``` r
redcap_export$redcap_data %>% pluck(1)
```

``` bash
  record_id redcap_repeat_instance redcap_event redcap_arm repeat_1 repeat_2
1         1                      1      event_1          1        1        2
2         1                      2      event_1          1        3        4
3         1                      3      event_1          1        5        6
4         1                      1      event_2          1        A        B
5         1                      2      event_2          1        C        D
6         3                      1      event_1          1        C        D
7         3                      1      event_2          1        E        F
8         3                      2      event_2          1        G        H
9         4                      1      event_3          2       R1       R2
```

### Loading Tidy `tibbles` to Specified Environments

As an additional feature to `REDCapTidieR`, the `bind_tables()` function
serves to allow users to assign the tidy data tables under
`$redcap_data` to loaded data elements in a specified environment.

By default, when using `bind_tables()` all elements will load as
individual dataframes into the global environment. Users have the option
to specify a different environment object to assign dataframes to, as
well as specify specific `redcap_form_name`s and `structure`s if the
they do not wish to load all data elements.

**Load all elements to global environment:**

``` r
read_redcap_tidy(redcap_uri, token) %>% 
  bind_tables()
```

**Load selected elements to specified environment:**

``` r
sample_environment = rlang::new_environment()

read_redcap_tidy(redcap_uri, token) %>% 
  bind_tables(environment = sample_environment, 
              redcap_form_name = c("nonrepeated", "repeated"))
```
