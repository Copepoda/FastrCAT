<!-- README.md is generated from README.Rmd. Please edit that file -->
FastrCAT
--------

Oceanographic data generated from SeaBird FastCAT CTD’s have been
collected since 1995 under EcoFOCI. The file type which we can view and
collate the data from is a SeaBird proprietary file type called an .up
file. The .up file is a post process file generated from the SeaSoft
software and only includes the data from the upward cast of the device.
Since 1997 there have been various versions of a Perl script which will
append important station header information to the .up file. The Perl
script acts as an interface between the SeaSoft software and the
MasterCOD .db3 file to extract the header information based on the
BON/CALVET/TUCK number. Each station from a cruise has its’ own .up
file. This format isn’t in a traditional tabular format and is not
easily accessed by users. It has been determined that it would be useful
to build an R package for the FastCAT data that starts with a function
to aggregate each cruises .up files into a tabular format. The function
can then be used to generate tabular data and as a QAQC process for the
.up files. Then the tabular data be easily used to create plots and
calculate indices. This R package would be a final step in processing
.up files while out at sea. The steps would be: enter COD form data into
MasterCOD, run Perl script and then the R functions from the command
line.

documentation
-------------

Full documentation of the process is available in vignettes

install the package
-------------------

There are two options for installing the package.

1.  download/clone the project from GitLab and then install to R from
    source
2.  install with the `devtools` package

### install via `devtools`

You’ll need to setup ssh on GitLab and add your ssh key. Once that’s
done, the following R code should install the package

``` r
library(devtools)

creds = git2r::cred_ssh_key("C:\\Users\\MYSELF\\.ssh\\id_rsa.pub",
                            "C:\\Users\\MYSELF\\.ssh\\id_rsa")
devtools::install_git("git@gitlab.afsc.noaa.gov:Nissa.Ferm/FastrCAT.git",
                      credentials = creds)
```
