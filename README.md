---
editor_options: 
  markdown: 
    wrap: 72
---

This application is a US Mortgage-Backed Securities (MBS) query tool. It
is today a $7.3 trillion market of 1M pools of US Prime Real State
loans.

The scope is

-   [Fannie Mae Single-Family
    MBS](https://capitalmarkets.fanniemae.com/mortgage-backed-securities/single-family/single-family-disclosure-information-center)

-   [Freddie Mac Single-Family MBS](http://www.freddiemac.com/mbs/)

-   [Ginnie Mae Single-Family
    MBS](https://www.ginniemae.gov/issuers/program_guidelines/Pages/mbs_guide.aspx)

## Main Sections

The upper panel selects the agency as above and the reporting month. For
demonstration, a random sample pool is loaded each time. The lower panel
has two main tabs:

### Pool View

For individual pools, containing the subtabs:

#### Input

For entering a single and displaying main characteristics of a single
pool, including geographical dispersion.

![](www/Pool_View_Input.jpg)

#### Detail

For displaying all information disclosed by the source plus some
calculated fields like prepayment speeds.

![](www/Pool_View_Detail.jpg)

#### ARM

Adjustable Rate Mortgage (ARM)-specific information in case the pool is
an ARM.

![](www/Pool_View_ARM.jpg)

#### Stratification

Supplemental information about the pool

![](www/Pool_View_Stratifications.jpg)

#### Loan Level

Displays loan level information for the pool when available.

![](www/Pool_View_Loan_Level.jpg)

## Aggregations

Analytics on the cohorts

### By Prefix

![](www/Pool_View_Aggregations_FN.jpg)

### By Prefix and Coupon

![](www/FNCL.jpg)

![](www/Pool_View_Aggregations_GNSF.jpg)

## Filtering Aggregations

![](www/Pool_View_Aggregations_GN.jpg)

## Implementation

The publicly available data files provided monthly by the agencies are
stored in Amazon Web Services (AWS) S3 file hosting platform.

For example, the first 2 Fannie Mae monthly factor files

    ## Bucket: fnma-mbs-sf-singleclass-datadir 
    ## 
    ## $Contents
    ## Key:            FNM_MF_201910.zip 
    ## LastModified:   2021-04-20T16:43:00.000Z 
    ## ETag:           "d7cfe52c7a07971021f5de1f74e6dcfb-4" 
    ## Size (B):       31689689 
    ## Owner:          3ed8938a6ec6ccbf8e5544fed9c6be5f74559d6d28ddeda9375f52176205d37b 
    ## Storage class:  STANDARD 
    ## 
    ## $Contents
    ## Key:            FNM_MF_201911.zip 
    ## LastModified:   2021-04-20T16:43:00.000Z 
    ## ETag:           "e82357d08212aa90c934e006d3f99f5d-4" 
    ## Size (B):       31702227 
    ## Owner:          3ed8938a6ec6ccbf8e5544fed9c6be5f74559d6d28ddeda9375f52176205d37b 
    ## Storage class:  STANDARD

The input files are parsed and stored in binary format in AWS S3

    ## Bucket: fnma-mbs-sf-singleclass 
    ## 
    ## $Contents
    ## Key:            FNM_MF_201910.fst 
    ## LastModified:   2020-10-27T06:05:25.000Z 
    ## ETag:           "ff8565f0ec8dea21ec118c0bb72433c1-4" 
    ## Size (B):       32205844 
    ## Owner:          3ed8938a6ec6ccbf8e5544fed9c6be5f74559d6d28ddeda9375f52176205d37b 
    ## Storage class:  STANDARD 
    ## 
    ## $Contents
    ## Key:            FNM_MF_201910.rds 
    ## LastModified:   2020-11-13T06:20:01.000Z 
    ## ETag:           "29651fef7406d7493e2cacbda7e9be39-4" 
    ## Size (B):       26424005 
    ## Owner:          3ed8938a6ec6ccbf8e5544fed9c6be5f74559d6d28ddeda9375f52176205d37b 
    ## Storage class:  STANDARD

[1] "2021-04-01" "2021-03-01"

    ## 
    ## Attaching package: 'magrittr'

    ## The following objects are masked from 'package:testthat':
    ## 
    ##     equals, is_less_than, not

|                | Prefix | Pool Count | Loan Count | Issuance Investor Security UPB |
|:--------------:|:------:|:----------:|:----------:|:------------------------------:|
| **2021-04-01** |   CL   |  332,217   | 11,482,384 |       9,262,679,611,404        |
| **2021-03-01** |   CL   |  331,316   | 11,478,699 |       9,165,122,220,810        |

FN CL (continued below)

|                | Current Investor Security UPB | Average Mortgage Loan Amount | Prior Month Investor Security UPB | Delinquent Loans Purchased Loan Count |
|:--------------:|:-----------------------------:|:----------------------------:|:---------------------------------:|:-------------------------------------:|
| **2021-04-01** |       2,373,122,267,045       |           227,702            |         2,360,337,363,211         |                 2,334                 |
| **2021-03-01** |       2,360,337,358,404       |           226,782            |         2,352,451,405,018         |                 2,164                 |

Table continues below

|                | Delinquent Loans Purchased Prior Month UPB | UPB of Delinquent Loans Purchased as % of Prior Month UPB | SMM  | Vol SMM | CPR1 | Vol CPR1 |
|:--------------:|:------------------------------------------:|:---------------------------------------------------------:|:----:|:-------:|:----:|:--------:|
| **2021-04-01** |                528,817,507                 |                             0                             | 0.04 |  0.04   | 0.37 |   0.36   |
| **2021-03-01** |                486,798,960                 |                             0                             | 0.03 |  0.03   | 0.33 |   0.33   |

Table continues below

|                | WA Net Interest Rate | WA Issuance Interest Rate | WA Current Interest Rate | WA Loan Term | WA Issuance Remaining Months to Maturity |
|:--------------:|:--------------------:|:-------------------------:|:------------------------:|:------------:|:----------------------------------------:|
| **2021-04-01** |         2.99         |           3.74            |           3.74           |    358.2     |                  357.4                   |
| **2021-03-01** |         3.05         |           3.79            |           3.79           |    358.2     |                  357.4                   |

Table continues below

|                | WA Current Remaining Months to Maturity | WA Loan Age | WA Mortgage Loan Amount | WA Loan To Value LTV | WA Combined Loan To Value CLTV |
|:--------------:|:---------------------------------------:|:-----------:|:-----------------------:|:--------------------:|:------------------------------:|
| **2021-04-01** |                   315                   |    38.36    |         301,499         |        74.65         |             75.59              |
| **2021-03-01** |                  313.9                  |    39.36    |         299,909         |        74.91         |             75.88              |

Table continues below

|                | WA Debt To Income DTI | WA Borrower Credit Score |
|:--------------:|:---------------------:|:------------------------:|
| **2021-04-01** |         34.79         |           751            |
| **2021-03-01** |         34.83         |          750.8           |
