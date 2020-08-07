# clinicaltrialr

<div align="justify">

## Overview

[ClinicalTrials.gov](https://www.clinicaltrials.gov/) provides a fantastic resource of data on all registered clinical trials. However, even though it also provides [API access](https://www.clinicaltrials.gov/ct2/resources/download), it is not possible to easily create a readily analysable database with all study fields of interest. This package aims to make using data from ClinicalTrials.gov as easy as it could be, by providing the functions required to extract all data for all studies of interest. It does so by creating an interface to the old ClinicalTrials.gov API, which can be found [here](https://www.clinicaltrials.gov/ct2/resources/download); more in the alternative sources [below](#Alternatives).


## Usage

1. Install package.

    ```r
    devtools::install_github("serghiou/clinicaltrialr")
    ```

2. Build a query using [Advanced Search](https://www.clinicaltrials.gov/ct2/search/advanced?cond=&term=&cntry=&state=&city=&dist=) and copy the URL, e.g. http://www.clinicaltrials.gov/ct2/results?cond=Heart+Failure.

3. Download the results table corresponding to the copied link.

    ```r
    library(clinicaltrialr)
    results <- ct_read_results("http://www.clinicaltrials.gov/ct2/results?cond=Heart+Failure")
    ```

4. Download all records and construct a dataframe with variables of interest in parallel (for the non-parallelized version, remove the `cl` argument).

    ```r
    library(pbapply)
    trials_list <- pbapply::pblapply(results$`NCT Number`, ct_read_trial, cl = parallel::detectCores() - 1)
    ```

5. (Optional) If an error occurred, re-extract values for which the algorithm was not allowed acccess to the website in parallel (for the non-parallelized version, remove the `cl` argument).

    ```r
    missing_index <- grep("Error in open", trials_list)
    missing_nct <- paed$`NCT Number`[missing_index]
    missing_df <- pblapply(missing_nct, ct_read_trial, cl = parallel::detectCores() - 1)
    trials_list[missing_index] <- missing_df
    ```

6. Create the dataframe.

    ```r
    trials <- dplyr::bind_rows(trials)
    ```

7. Save as CSV in a folder called "Output".

    ```r
    write_csv(trials, "../Output/pediatric-trial-records.csv")
    ```


## Alternatives

* ClinicalTrials.gov has its own [API interface](https://clinicaltrials.gov/api/gui) to their new API (this package at the moment uses the old API). This can be used to create an XML file of [all records about studies of interest](https://clinicaltrials.gov/api/gui/demo/simple_full_study), a CSV file with [specific fields from all studies of interest](https://clinicaltrials.gov/api/gui/demo/simple_study_fields) or a CSV file with [just one field of interest for all studies of interest](https://clinicaltrials.gov/api/gui/demo/simple_field_values). These allow the retrieval of at most 100, 1000 or all records, but using the fields `min_rnk` and `max_rnk` it is possible to, in chunks, download all records of interest (in XML for the former, in XML/CSV for the latter two). However, these records can only contain a maximum of 20 variables, which means that multiple requests would have to be done to fulfill searches of more than 20 records.

* There is an [rclinicaltrials](https://github.com/sachsmc/rclinicaltrials) package. However, (a) it does not allow for the complicated kind of queries that I would like to use and for which I needed to use the Advanced Search function of the website and (b) it creates dataframes that are not easy to analyze and share with others possibly using other platforms (e.g. dataframes that contain lists).

* A list of clinincal trial registry scrapers can be found in the [opentrials/registers GitHub repository](https://github.com/opentrials/registers), however most of these are old and poorly maintained.


## Acknowledgements

* This package was built using information provided by ClinicalTrials.gov [here](https://www.clinicaltrials.gov/ct2/resources/download).

* This package would not have been possible without the [xml2 package](https://github.com/r-lib/xml2).

* All I know about building packages I owe to Hadley Wickham and Jennifer Bryan's [book](https://r-pkgs.org/)!


## TODO

This release only contains the basic functions to download and extract popular fields of studies on ClinicalTrials.gov. The following functions would also be great to have and contributions by anyone with the time and interest to enhance this package are more than welcome!

1. Migrate the current functions to the [new](https://clinicaltrials.gov/api/gui/home) API, rather than the [old](https://www.clinicaltrials.gov/ct2/resources/download) one.
2. A function to download all records of interest in bulk and process that XML file, rather than downloading one at a time.
3. A function to download all of ClinicalTrials.gov.
4. A function to extract even more fields from each study record.
5. Tidy up complex fields, such as primary and secondary outcomes, which can be placed within lists.
6. Functions to help with data post-processing (e.g. identify cluster trials, overall survival outcomes, etc.)
7. Additional functions (e.g. should this have already been published, etc.)

</div>
