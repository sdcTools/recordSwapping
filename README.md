# recordSwapping

Package **recordSwapping** has been moved to [sdcMicro](https://github.com/sdcTools/sdcMicro)!
Please use package `sdcMicro` and see `?sdcMicro::recordSwap()` or `vignette('recrodSwapping')` for details and examples.\n
CRAN: https://github.com/sdcTools/sdcMicro/packages/sdcMicro/index.html\n
Development: https://github.com/sdcTools/sdcMicro

**recordSwapping** was an R-package for record swapping. Its aim was to develop a core library using pure `C++` (11) code to implement targeted record swapping (TRS). Since version `1.0.1`, the functionality has been included in [sdcMicro](https://github.com/sdcTools/sdcMicro) (versions >= `5.7.1`) where also any future development will take place.

The current files of the "core" library can be found in  [github.com/sdcTools/sdcMicro/tree/master/src/recordSwap](https://github.com/sdcTools/sdcMicro/tree/master/src/recordSwap). 
This folder contains the pure `C++` (11) code `recordSwap.cpp` and `recordSwap.h` which should be used if the functionality should be included in other software tools. The `C++` code is not directly embedded using `Rcpp` so that it can be more easily implemented into other projects which do not depend on `R` libraries.

Within `sdcMicro`, the library is called by an `Rcpp` wrapper ([src/recordSwap_R.cpp](https://github.com/sdcTools/sdcMicro/blob/master/src/recordSwap_R.cpp)) which again is called from the top level `R`-function `recordSwap()`.  

## Versions and Updates

Below are the changes between versions shown; Versions `> 1.0.1` will be documented in [sdcMicro](https://github.com/sdcTools/sdcMicro).

### 1.0.1

* fix bug when producing the log file.
* If a household has no suitable donor household once, then this household is discarded from swapping for all subsequent swaps in all other geographic hierarchies. Otherwise households which are `at-risk` and have no suitable donor at a specific geographic hierarchy can be swapped in lower geographic hierarchies. This leads to the household being swapped but the swap does not account for the `at-risk` issue.
* Improved inputs checks for `recordSwap()`, thanks @Kyoshido for PR [67093f7](https://github.com/sdcTools/recordSwapping/pull/8/commits/67093f7361748c0b8d72cbf2ac92111d6ec8c512)
* Corrected spelling on vignette and documentation, thanks @Kyoshido for PR [4dfb068](https://github.com/sdcTools/recordSwapping/pull/9/commits/4dfb068e3b6539b752fd015e231a87c2f895337a) 

### 1.0.0

* included parameters `int &count_swapped_records` and `int &count_swapped_hid` `std::string log_file_name` to the cpp-function `recordSwap()` which count the number of swapped records and swapped households. These parameters are convenience parameters for mu-Argus.
* included parameter `std::string log_file_name` and `log_file_name` in the cpp-function `recordSwap()` and R-function `recordSwap()` respectively. Contains path for writing a log file. The log file contains a list of household IDs (`hid`) which could not have been swapped and is only created if any such households exist.
* Changed definition of parameter `k_anonymity`: now a household is treated as `high-risk-household` if for at least 1 person in the household `counts < k_anonymity`, was previously `counts <= k_anonymity`. This definition is now consistent with parameter `k` of function `sdcMicro::localSuppression()`.

### 0.4.0

* Made `recordSwap()` an S3 Method, which can also accept an `sdcMicro`-object as input.
* included unit tests for `recordSwap()` when using `sdcMicro`-objects
* included unit tests for `infoLoss()`
* cleaned up code and unit tests such that `R CMD check` runs without notes, warnings and errors
* fixed warnings when compiling c++ code like `comparing unsigned integer ...` or `unused variable ...`
* fixed minor typos in README, help pages and vignettes

### 0.3.3

* included function `infoLoss()` to calculated various information loss measures after `recordSwap()` was applied. Function `infoLoss()` is heavily inspired by function `cellKey::ck_cnt_measures()` but accepts the micro data as input instead of frequency or magnitude counts.
* changed variable names and content of dummy data created by `createDat`.
* updated documentation and fixed some typos.

### 0.3.2

* extended unit tests
* Argument `data` for R-function `recordSwap()` does no longer need to have only integer values. Only variables needed for underlying C++ function defined through parameter `hid`, `hierarchy`, `risk_variables`, `similar`, `carry_along` need to be integer type.


### 0.3.1

* Fixed bug where less households than indicated through the swaprate were drawn
* added version number to header file
* developed different method for distributing number of draws over all geographic hierarchies

### 0.3.0

* Changed parameter order in `R` and `C++` function `recordSwap()` to have a more consistent documentation. Function call to `C++` function changed to 

```r
std::vector< std::vector<int> > recordSwap(std::vector< std::vector<int> > data, int hid,
                                           std::vector<int> hierarchy, 
                                           std::vector< std::vector<int> > similar,
                                           double swaprate,
                                           std::vector< std::vector<double> > risk, double risk_threshold,
                                           int k_anonymity, std::vector<int> risk_variables,  
                                           std::vector<int> carry_along,
                                           int seed = 123456)
```

* Improved interface of `R` function `recordSwap()`. Now, parameters `hid`, `hierarchy`, `similar` and `risk_variables` can be used with column indices or column names of `data`. Please note that indices in `R` start with 1 but in `C++` they start with `0`. The `R` function `recordSwap()` which is basically just a wrapper for the `C++` function converts column names or indices into the correct format for the `C++` function. So the call for `recordSwap()` from `R` expects indices starting from 1.

* added parameter `std::vector<int> carry_along` to `C++` function `recordSwap()`. Like the paramter `hierarchy`, `carry_along` expects column indices of `data` and swaps these values in addition to the ones defined in `hierarchy`. These variables do however not interfere with risk calculation, sampling or finding a donor.

* added parameter `carry_along` to `R` function `recordSwap()`, expects either column indices or column names

* added parameter `return_swapped_id` to `R` function `recordSwap()`. If `return_swapped_id` is `TRUE` an additional column will be returned which holds the `hid` with which each record was swapped with. If this new column has the same value as `hid` the record was not swapped.

* Improved documentation and vignette

### 0.2.0

* Some parameter changes to the `C++` function `recordSwap()`:

    + `similar` is now a vector of vectors allowing for multiple similarity profiles
    + changed `std::vector<int> risk` to `std::vector<int> risk_variables` for a more descriptive name
    + changed `int th` to `int k_anonymity` for a more descriptive name
    + added parameter `double risk_threshold` which can be used to set a custom risk threshold. household whith a risk **greater or equal** `risk_threshold` is set as risky household (**not yet supported by `R` wrapper**)
    + added parameter `std::vector<std::vector<double>> risk` which can be used as custom risks for each household in each hierarchy level (**not yet supported by `R` wrapper**)

```r
recordSwap(std::vector< std::vector<int> > data, std::vector<std::vector<int>> similar,
                                           std::vector<int> hierarchy, std::vector<int> risk_variables, int hid, 
                                           int k_anonymity, double swaprate, double risk_threshold,
                                           std::vector<std::vector<double>> risk, int seed = 123456)
```
                                           
* Fixed some bugs concerning the application of `swaprate`
* `data` as well as `risk` are used internally such that `data[0]` or `risk[0]` contain the micro data or risk over all hierarchies for the first individual,
`data[1]` or `risk[1]` contain the micro data or risk, over all hierarchies, for the second individual, and so on.
* Documentation and parameter descriptions have been updated in the corresponding help files.

### 0.1.0

* First prototype version of record swapping 
* Contains the function `recordSwap()` as the main function of this package
* `recordSwap()` is an `R`-wrapper which calls an underlying `Rcpp` function where at the bottom a call to the **C++** function `recordSwap()` is made.

```r
recordSwap(std::vector< std::vector<int>> data, std::vector<int> similar,
                                           std::vector<int> hierarchy, std::vector<int> risk, int hid, 
                                           int th, double swaprate, int seed = 123456)
```

* The parameter `data` contains the household data and is used internally such that `data[0]` contains values for each individual for the first column of the data,
`data[1]` contains the values for each individual for the second column of the data. This also implies that `data[0][0]` addresses the first value of the first column and so on.
* The procedure expects the data to be **ordered** by household ID (column `hid`). Ordering inside each household is irrelevant. 
* Various internal help functions are included. These are however not intended for flexible use and are only called from withing `recordSwap()`                                           

## Installation and Application

The functionality can be used by installing `sdcMicro` and using `recordSwap()`
```r
install.packages("sdcMicro")
```

The application of TRS using the record-swapping library is shown in a package vignette

```r
vignette("recordSwapping", package = "sdcMicro")
```

which is also available [online](https://sdctools.github.io/sdcMicro/articles/recordSwapping.html). Further information is available in the help-pages (`?sdcMicro::recordSwap`) 
