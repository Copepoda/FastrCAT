------------------------------------------------------------------------

output: md\_document: variant: markdown\_github

------------------------------------------------------------------------

**Abstract** The `FastrCAT` package makes it easy to use temperature and
salinity data collected by a SeaBrid FastCAT. The package provides
functions to read in .up files and bind them into a single dataframe,
provide QA/QC feedback, plot depth by temperature/salinity for each
cast, and maps of these… indicies.

Introduction
============

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

``` r
library(FastrCAT)
```

Functions in FastrCAT
---------------------

`make_dataframe_fc()`

`plot_ts_fc()`

`fill_missing_stations()`

make\_dataframe\_fc()
---------------------

This function writes a single data frame in .csv format to file
containing oceanographic data collected by the FastCat during a cruise.
The format and column naming conventions are specific to the needs of
EcoDAAT. This is the primary function of the FastrCAT package and must
be run prior to all other functions. All other functions depend on the
data frame generated. Other outputs of the function are two text files.
This first is a cruise summary. The summary contains basic information
about the data and some summary statistics. The second is a warnings
file, which tells users if data or information is missing incase
reprocessing is necessary.

**Usage**

``` r
 make_dataframe_fc("path/ to directory/ of .up files")
```

**MasterCOD Acquired Information**

    All header information is hand entered into MasterCOD from a COD form for each consecutive FastCAT deployment. Header information acquired by the PERL script has not been consistent across all years. Since 1997 theses have been the various headers acquired from MasterCOD: latitude, longitude, date, time, cruise name, station name, haul name, Foci grid name, bottom depth, and instrument type. The function as built handles all types of header information listed above. Latitude and longitude are converted from degrees minutes seconds to decimal degrees. Date and time are converted into a date-time format set with a UTC time zone. One common error is that no header information is found. This error is most commonly due to the FastCat data being processed prior to station information being entered into MasterCOD. The function FastCat_compile.R within the package FastrCAT will generate a txt file with a warning. 

**Example Header Warning**

> “1” “This file G:/SeaCatData/Processed/1DY10/CAL001.up has no header
> info needs to be reprocessed.”

This warning is telling the user that all header information is missing
and that it needs to be reprocessed after the information has been
entered into MasterCOD. Once this has been done the function
make\_dataframe\_fc() can be run again. Human errors that are common are
handled with exceptions within the function. When a user does not enter
a field into MasterCOD the named header line is blank. The function will
enter an NA.

**Example Header line in .up file:**

> @ Cruise: DY12-05 (“DY12-05” in the dateframe)
>
> @ Cruise:" " (NA in the dataframe)

Another error is when the Foci-grid designation or the BON number is
entered into the station\_name. The function will find this type of
error and place the Foci\_grid name into the correct column and leave
the Station name blank. In other cases the Station name has been
station.haul. The function will check to see if this is the case and
separate the two and place station in station and haul in haul. When one
of these three are missing it will require some corrections. The
solution at sea would be to make sure that all these fields are entered
into MasterCOD and then re-run the function. For older files this would
require generating a table from EcoDAAT of Cruise, Haul, and FOCi\_Grid
to look-up and replace all missing values.  
Issues that can’t be resolved by the make\_dataframe\_fc() are when
Cruise names or Foci\_grid names are misspelled, which can be an issue
when filtering or manipulating the resulting data. These issues would
need to be QAQC’s after the fact and corrected. Sometimes, time, which
is entered into MasterCOD as a four digit string is a nonsense time such
as 0860. When this is the case, the function tries to convert it into a
date-time object and cannot which results in an NA. Date can also be
entered in incorrectly, if the format is correct the conversion to a
date-time object will be fine, but the date might be incorrect. This
would have to be QAQC’d after the data has been compiled by the
function.

**Seasoft Binned Data**

In each .up file the numerical data is found in tab delineated columnar
format after \*END\*. Each row in the .up file represents a single
integer of pressure. The SeaBird FastCAT takes readings 16 times a
second. For all values within an integer of pressure, one can select an
average or an interpolated average. Over the years there have been as
few as four columns of data up to nine. First the function looks to see
if there is any data. If there is no data then a text file will be
output.

**No data warning example:**

> 1" “This file G:/SeaCatData/Processed/DY17-05/BON055.up has no data
> associated with it.”

When this occurs then header information is still gathered so there is a
record of it but it will have blanks for all the numerical data columns.
The function can handle the inconsistent column amounts. It looks for
the name of each column and renames it in a standard format so that all
data generated can be easily bound together. The data columns selected
were pressure, temperature, conductivity, salinity, sigma\_T, and flag.
When there are more than these they will be dropped from the dataframe.
Depth is added as a calculated value from pressure. The calculation is
based on the AN69: Conversion of Pressure to Depth documented on the
SeaBird website. This conversion requires latitude, so if latitude is
missing depth will be an NA. Finally there are sometimes bad spikes in
the data, a common cause being a bad termination. There is a filter in
the function to remove temperatures less than -3 and greater than 20. As
well as salinity values less than 0.05 and greater than 38.

**Cruise Summary .txt**

    Provides a short paragraph of number of tows, salinity, temperature, and location information. Summary statistics: minimum, median, mean, max, and number of NA’s are also provided. This provides the user a paragraph ready for a cruise report and can be used as a quick data check.  If data seems incorrect then it is advised that values be corrected in MasterCOD and then to re-run the perl script and make_dataframe_fc(). Then a new summary will be generated. 

**Example of summary paragraph, excludes summary statistics after bottom
depth:**

> “1” "Quick Cruise FastCat Summary.
>
> During the cruise, MF08-02, there were 32 tows with depth,
> temperature, and salinity collected. Tows were taken during the time
> period from 2008-02-18 to 2008-02-26. Temperature ranged from 2.11 to
> 4.13 with a mean of 3.57C. Salinity ranged from 4.11 to 34.11 with a
> mean of 33.32 PSU. The deepest tow was 596 meters. The Southern most
> bongo tow was 53.65 and the furthest North was 55.328 Latitude. The
> Western most bongo tow was -168.374 and the Eastern most was -164.88
> Longitude.
>
> If any of this looks suspect, check for the values in the .csv file
> and then correct in MasterCOD. After this is done, re-run the Perl
> script and then re-run FastrCAT.
>
> SUMMARY CRUISE\_NAMES MF08-02, MF08=02, MF0802
>
> STATION\_HAUL\_NAMES NA.NA
>
> FOCI\_GRID\_NAMES BD07, BG07, BJ07, BM07, BP07, BS04, BV04, BV07,
> BS07, BS10, AX07, BP10, BM10, BJ10, BS01, BV01, AX04, BP19, BM16,
> BP16, BM19, BS16, BS13, BP13, BM13, BA04, BJ0710, BD04, BG04, BJ04,
> BM04, BP04
>
> BOTTOM\_DEPTHS Min. : 75.0  
> Median : 661.0  
> Mean : 835.8  
> Max. :2150.0  
> NA’s :1003

Notice that this cruise has multiple cruise names, these would need to
be corrected in MasterCOD. Instead of Station and haul names there are
only FOCI grid names. Make sure that this is what is wanted for the
cruise and correct in MasterCOD. In this case the cruise station haul
info was in the .up file name, this is an artifact of older cruises.
Bottom depth also had a lot of NA’s. This cruise had several tows in
very deep water, this mean there were a couple of tows where bottom
depth was not recorded in MasterCOD. You can discover which files didn’t
have bottom depth recorded by filtering for NA’s in the BOTTOM\_DEPTHS
column in the .csv file and looking at the DIRECTORY column for the cast
number. From this we discover that it was two casts, ST14H1.up and
ST15.H1.up or FOCI grids BP07 and BS04. Then go back and correct in
MasterCOD if necessary.

plot\_ts\_fc()
--------------

    Once make_dataframe_fc() has been run, then plot_ts_fc() can be used. This function creates a depth by salinity and temperature plot for each station. 

``` r
 plot_ts_fc("path/ to directory/ of fastrcat dataframe")
```

These are all .png files which will be located in the plot folder within
the current folder. It only needs to be run once to generate a plot for
each station. Each dot is a data point. Check the profile for each
station/haul.

<img src="AE10-01_Station_2_2.png" alt="Correct Output" width="100%" />
<p class="caption">
Correct Output
</p>

For instance in the second plot below, it seems that two tows had the
same Station and haul designation. If this occurs check the COD forms
and MasterCOD and correct. This occurred because Station ID was
station.haul and haul was not a value grabbed from MasterCOD. This
allowed two consecutive casts to have the same station.haul information,
where the second should have been Station 41, haul 3.

<img src="AE10-01_Station_41_2.png" alt="Incorrect Output" width="100%" />
<p class="caption">
Incorrect Output
</p>

fill\_missing\_stations()
-------------------------

Like the name implies, this function will fill in the missing station
name, foci grid name, and haul number in the dataframe which is
generated by make\_dataframe\_fc(). This is a function which is used in
conjuction with a dataframe in .csv format of haul records queried from
EcoDAAT. This requires that the cruise haul information associated with
the .up files has been uploaded into EcoDAAT. This function assits in
data completness after a cruise has been completed.

**Usage**

``` r
fill_missing_stations("path/ to directory/ of fastrcat dataframe",
                      "path/ to directory/ of haul records dataframe")
```

Using FastrCAT at Sea
---------------------

Since people are already familiar with using the command line to run the
Perl script out at sea it seemed logical to have people do the same for
the R package functions. Requirements for running the package out at
sea.

1.The most current version of R, R (32-bit) version 3.5.1 – “Feather
Spray” must be installed on the at sea laptops.

2.All libraries and packages required to run functions must be
installed. “list of dependencies”

3.The FastrCAT package

Example of Using FastrCAT in the command line:

**1. Change directory to where the R program files are.**

> C:&gt; cd C:/”Program Files”/R/R-3.5.1/bin

**2. Start an R session.**

> C:Files-3.5.1&gt; R
>
> *You will see a bunch info about R pop up. You can ignore this.*

**3. Get the FastrCat package.**

> &gt; library(“FastrCAT”)

**4. Run the function to make the data frame and cruise summary.**

> &gt; make\_dataframe\_fc(“path to current folder”)

**5. Run the function make all the temperature/salinity plots.**

> &gt; plot\_ts\_fc(“path to current folder”)

**6. Quit the R session.**

> &gt; q()

**7. It will ask if you want to save the workspace image. You don’t need
to.**

> &gt; q()
>
> Save workspace image? \[y/n/c\]: n

**8. Now you are back into the command prompt. If you need to re-run the
Perl script to make the .up files remember to change the working
directory. Refer to the directions for setting up MasterCOD. **

Comments on historical .up files
--------------------------------

    In order to re-run older files due to missing header information it will require finding the appropriate MasterCOD.db3 files and the folder with the .XMLCON, .hdr, and .hex files. The greatest amount of time will be spent retrieving the .db3 files for each cruise. An estimate for finding one file is 30 minutes. But once found it will only take a minute to reprocess. When evaluating the Processed folder it seems that 11 cruises will need to be reprocessed. There are most likely more cruises as the processed folder in incomplete. Some folders only have an excel file, this is the case for all Oshoro Maru cruises. This will require editing the excel file into the format requested for EcoDAAT. 

<http://www.seabird.com/document/an69-conversion-pressure-depth>