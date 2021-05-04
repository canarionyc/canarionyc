# Applications and libraries stored in Github

# MBS Viewer

US Mortgage-Backed Securities (MBS0 is a $7.3 trillion market of 1M pools of Prime Real State loans. This shiny app hosted at [MBS Viewer](https://tgonzale.shinyapps.io/FNMA_PoolView/) is a US MBS query tool. It is a

The scope is:

-   [Fannie Mae Single-Family MBS](https://capitalmarkets.fanniemae.com/mortgage-backed-securities/single-family/single-family-disclosure-information-center)

-   [Freddie Mac Single-Family MBS](http://www.freddiemac.com/mbs/)

-   [Ginnie Mae Single-Family MBS](https://www.ginniemae.gov/issuers/program_guidelines/Pages/mbs_guide.aspx)

## Main Panels

The upper panel selects the agency as above and the reporting month. For demonstration, a random sample pool is loaded each time. The lower panel has two main tabs:

### Pool Viewer

For individual pools, containing the subtabs:

#### Security Identifier Input and Pool Summary

For entering a single pool and displaying it's main characteristics, including it's geographical dispersion.

![](www/Pool_View_Input.jpg)

#### Pool Detailed Information

For displaying all information disclosed by the source plus some calculated fields like prepayment speeds.

![](www/Pool_View_Detail.jpg)

#### Adjustable Rate Mortgage (ARM) Specific

ARM-specific information in case the pool is an ARM.

![](www/Pool_View_ARM.jpg)

#### Stratifications

Supplemental information about the pool

![](www/Pool_View_Stratifications.jpg)

#### Loan Level Information

Displays loan level information for the pool when available.

![](www/Pool_View_Loan_Level.jpg)

## Cohort Viewer

Displays aggregations and analytics (prepayment rates) on the cohorts

### Aggregations by Prefix

![](www/Pool_View_Aggregations_FN.jpg)

### Aggregations by Prefix and Coupon

![](www/FNCL.jpg)

![](www/Pool_View_Aggregations_GNSF.jpg)

### Aggregations on filtered data

![](www/Pool_View_Aggregations_GN.jpg)

## Implementation

### Data in AWS S3

#### Input data files

The publicly available data files monthly provided by the agencies are stored in Amazon Web Services (AWS) S3 file hosting platform. The list of buckets created is:

    ##                                  Bucket             CreationDate
    ## 1          fhlmc-mbs-sf-arm-singleclass 2021-01-26T18:23:19.000Z
    ## 2  fhlmc-mbs-sf-arm-singleclass-datadir 2021-04-20T16:38:38.000Z
    ## 3              fhlmc-mbs-sf-singleclass 2021-01-09T11:40:48.000Z
    ## 4      fhlmc-mbs-sf-singleclass-datadir 2021-04-20T16:39:09.000Z
    ## 5                       fnma-llp-2020q3 2021-02-22T07:19:50.000Z
    ## 6                       fnma-llp-2020q4 2021-04-28T16:11:13.000Z
    ## 7               fnma-mbs-sf-singleclass 2020-10-27T05:57:35.000Z
    ## 8       fnma-mbs-sf-singleclass-datadir 2021-04-20T16:37:45.000Z
    ## 9                             gnma-hmbs 2021-01-09T11:51:03.000Z
    ## 10                    gnma-hmbs-datadir 2021-04-18T12:35:03.000Z
    ## 11              gnma-mbs-sf-singleclass 2020-12-25T05:57:33.000Z
    ## 12      gnma-mbs-sf-singleclass-datadir 2021-04-15T18:07:20.000Z
    ## 13                              test-tg 2021-02-09T18:27:34.000Z

For example, the first Fannie Mae monthly factor files is:

    ## Bucket: fnma-mbs-sf-singleclass-datadir 
    ## 
    ## $Contents
    ## Key:            FNM_MF_201910.zip 
    ## LastModified:   2021-04-20T16:43:00.000Z 
    ## ETag:           "d7cfe52c7a07971021f5de1f74e6dcfb-4" 
    ## Size (B):       31689689 
    ## Owner:          3ed8938a6ec6ccbf8e5544fed9c6be5f74559d6d28ddeda9375f52176205d37b 
    ## Storage class:  STANDARD

#### Processed data files

The input files are parsed and stored in binary format in AWS S3:

    ## Bucket: fnma-mbs-sf-singleclass 
    ## 
    ## $Contents
    ## Key:            FNM_MF_201910.fst 
    ## LastModified:   2020-10-27T06:05:25.000Z 
    ## ETag:           "ff8565f0ec8dea21ec118c0bb72433c1-4" 
    ## Size (B):       32205844 
    ## Owner:          3ed8938a6ec6ccbf8e5544fed9c6be5f74559d6d28ddeda9375f52176205d37b 
    ## Storage class:  STANDARD

## R packages developed

Package [loanroll](https://github.com/canarionyc/loanroll) for Fannie Mae and Freddie Mac pools and package [gnmar](https://github.com/canarionyc/gnmar "R pacackge on Github") for Ginnie Mae pools. Examples of their use are:

### FNMA Aggregations

The `MonthlyFactorDataset` object is loaded from **AWS S3**

``` r
readRenviron("~/Finance/FNMA/.Renviron")
remotes::install_github("canarionyc/loanroll",
#                        dependencies = FALSE,
                        force = FALSE)
library(loanroll)
# devtools::load_all("~/Finance/FNMA/loanroll", reset = TRUE, recompile = FALSE, export_all = FALSE)

Factor_Date <- "2021-04-01"
args.lst <- list(Factor_Date = Factor_Date, bucket_name=fn_mbs_sf_bucket, verbose = FALSE)

MF <- tryCatch( 
  do.call(MonthlyFactorDataset, args.lst)
  , error = function(e) e
)
if(inherits(MF, "error")) {
  stop(conditionMessage(MF))
}
```

Then monthly aggregations are made. For example, for **FNCL**:

``` r
FNCL <- subset(MF, subset = quote(Prefix=="CL" & Seller_Name != "SCR" & WA_Net_Interest_Rate %in% seq(1,15,0.5) & Security_Factor_Date == Factor_Date))
FNCL.stats <- aggregate(FNCL, by.vars=c('Prefix'))
saveRDS(FNCL.stats, "FNCL_stats.Rds")

FNCL_Coupon.stats <- aggregate(FNCL, by.vars=c('Prefix', 'WA_Net_Interest_Rate'))
saveRDS(FNCL_Coupon.stats, "FNCL_Coupon_stats.Rds")
```

#### FNMA by Prefix

| Security Factor Date | Prefix | Pool Count | Loan Count | Issuance Investor Security UPB | Current Investor Security UPB | Average Mortgage Loan Amount | Prior Month Investor Security UPB | Delinquent Loans Purchased Loan Count | Delinquent Loans Purchased Prior Month UPB | UPB of Delinquent Loans Purchased as % of Prior Month UPB |  SMM | Vol SMM | CPR1 | Vol CPR1 | WA Net Interest Rate | WA Issuance Interest Rate | WA Current Interest Rate | WA Loan Term | WA Issuance Remaining Months to Maturity | WA Current Remaining Months to Maturity | WA Loan Age | WA Mortgage Loan Amount | WA Loan To Value LTV | WA Combined Loan To Value CLTV | WA Debt To Income DTI | WA Borrower Credit Score |
|---------------------:|-------:|-----------:|-----------:|-------------------------------:|------------------------------:|-----------------------------:|----------------------------------:|--------------------------------------:|-------------------------------------------:|----------------------------------------------------------:|-----:|--------:|-----:|---------:|---------------------:|--------------------------:|-------------------------:|-------------:|-----------------------------------------:|----------------------------------------:|------------:|------------------------:|---------------------:|-------------------------------:|----------------------:|-------------------------:|
|           2021-04-01 |     CL |    325,457 | 11,448,610 |              9,228,990,826,702 |             2,369,072,946,198 |                      227,968 |                 2,356,210,765,245 |                                 2,321 |                                526,912,427 |                                                         0 | 0.04 |    0.04 | 0.37 |     0.36 |                 2.99 |                      3.74 |                     3.73 |        358.2 |                                    357.4 |                                   315.1 |       38.34 |                 301,724 |                74.62 |                          75.55 |                 34.78 |                    751.1 |
|           2021-03-01 |     CL |    324,554 | 11,444,363 |              9,131,366,726,631 |             2,356,210,760,438 |                      227,050 |                 2,348,268,402,356 |                                 2,161 |                                486,305,646 |                                                         0 | 0.03 |    0.03 | 0.33 |     0.33 |                 3.04 |                      3.79 |                     3.79 |        358.2 |                                    357.4 |                                   313.9 |       39.33 |                 300,137 |                74.88 |                          75.84 |                 34.83 |                    750.8 |

FN CL

#### FNMA by Prefix and Coupon

| Security Factor Date | Prefix | WA Net Interest Rate | Pool Count | Loan Count | Issuance Investor Security UPB | Current Investor Security UPB | Average Mortgage Loan Amount | Prior Month Investor Security UPB | Delinquent Loans Purchased Loan Count | Delinquent Loans Purchased Prior Month UPB | UPB of Delinquent Loans Purchased as % of Prior Month UPB | SMM  | Vol SMM | CPR1 | Vol CPR1 | WA Issuance Interest Rate | WA Current Interest Rate | WA Loan Term | WA Issuance Remaining Months to Maturity | WA Current Remaining Months to Maturity | WA Loan Age | WA Mortgage Loan Amount | WA Loan To Value LTV | WA Combined Loan To Value CLTV | WA Debt To Income DTI | WA Borrower Credit Score |
|:--------------------:|:------:|:--------------------:|:----------:|:----------:|:------------------------------:|:-----------------------------:|:----------------------------:|:---------------------------------:|:-------------------------------------:|:------------------------------------------:|:---------------------------------------------------------:|:----:|:-------:|:----:|:--------:|:-------------------------:|:------------------------:|:------------:|:----------------------------------------:|:---------------------------------------:|:-----------:|:-----------------------:|:--------------------:|:------------------------------:|:---------------------:|:------------------------:|
|      2021-04-01      |   CL   |          1           |     1      |     87     |           34,604,768           |          34,445,922           |           398,264            |            34,528,078             |                   0                   |                     0                      |                             0                             |  0   |    0    |  0   |    0     |             2             |            2             |     360      |                   359                    |                   357                   |      3      |         432,206         |          63          |               63               |          30           |           775            |
|      2021-04-01      |   CL   |         1.5          |    465     |  225,281   |         82,751,797,677         |        81,364,106,436         |           365,185            |          71,258,978,497           |                   2                   |                  742,661                   |                             0                             |  0   |    0    | 0.04 |   0.04   |           2.52            |           2.52           |    358.8     |                  358.5                   |                  353.9                  |    3.98     |         403,357         |        69.18         |             69.58              |         32.07         |          773.5           |
|      2021-04-01      |   CL   |          2           |   5,956    | 1,831,107  |        579,368,821,095         |        557,758,385,267        |           308,470            |          500,989,257,819          |                  27                   |                 8,216,632                  |                             0                             | 0.01 |  0.01   | 0.1  |   0.1    |           2.89            |           2.89           |    357.4     |                  357.2                   |                  351.3                  |    4.96     |         360,596         |        72.05         |              72.6              |         33.56         |           764            |
|      2021-04-01      |   CL   |         2.5          |   9,178    | 1,529,252  |        506,246,625,693         |        407,940,349,751        |           273,036            |          394,745,420,130          |                  110                  |                 31,607,978                 |                             0                             | 0.03 |  0.03   | 0.33 |   0.33   |           3.35            |           3.35           |    357.2     |                  356.9                   |                  345.7                  |    9.89     |         332,380         |        74.33         |             74.85              |         34.57         |           754            |
|      2021-04-01      |   CL   |          3           |   25,085   | 2,111,124  |       1,044,285,130,877        |        451,917,143,327        |           241,686            |          471,944,765,150          |                  310                  |                 90,060,054                 |                             0                             | 0.05 |  0.05   | 0.48 |   0.48   |           3.73            |           3.72           |    358.7     |                  357.9                   |                  304.3                  |    47.71    |         294,570         |        74.94         |             76.02              |         34.51         |          755.5           |
|      2021-04-01      |   CL   |         3.5          |   45,879   | 2,086,749  |       1,241,467,006,709        |        377,578,197,058        |           208,760            |          400,207,448,652          |                  502                  |                121,499,615                 |                             0                             | 0.06 |  0.05   | 0.49 |   0.49   |           4.12            |           4.12           |    358.9     |                  358.1                   |                  290.6                  |    60.23    |         264,637         |        76.49         |             78.08              |         35.57         |           748            |
|      2021-04-01      |   CL   |          4           |   49,918   | 1,760,002  |       1,216,921,152,905        |        282,895,269,956        |           185,135            |          298,731,071,619          |                  607                  |                141,540,164                 |                             0                             | 0.05 |  0.05   | 0.47 |   0.47   |           4.58            |           4.58           |    359.1     |                   358                    |                  288.9                  |    62.24    |         244,357         |        77.33         |              79.4              |         36.94         |          735.6           |
|      2021-04-01      |   CL   |         4.5          |   34,825   |  891,257   |        834,080,624,248         |        123,372,525,106        |           163,530            |          129,379,349,527          |                  411                  |                 86,121,464                 |                             0                             | 0.04 |  0.04   | 0.42 |   0.42   |           5.04            |           5.04           |    359.1     |                  357.7                   |                  278.8                  |    72.13    |         223,095         |        77.07         |             80.61              |         38.22         |          725.1           |
|      2021-04-01      |   CL   |          5           |   28,900   |  387,901   |        779,367,195,516         |        42,848,026,035         |           147,877            |          44,466,595,368           |                  181                  |                 29,707,598                 |                             0                             | 0.03 |  0.03   | 0.34 |   0.33   |           5.56            |           5.56           |    358.8     |                  354.8                   |                  236.7                  |    113.1    |         202,812         |        76.27         |             81.87              |         38.79         |          716.3           |
|      2021-04-01      |   CL   |         5.5          |   35,312   |  272,606   |       1,066,006,752,408        |        22,025,748,993         |           128,215            |          22,627,622,498           |                  82                   |                 10,514,941                 |                             0                             | 0.02 |  0.02   | 0.24 |   0.24   |           6.01            |           6.01           |    358.8     |                  351.9                   |                  169.4                  |    178.8    |         172,219         |         73.1         |             86.86              |         39.38         |          712.4           |
|      2021-04-01      |   CL   |          6           |   34,652   |  188,291   |        856,179,658,061         |        13,556,054,628         |           113,173            |          13,871,067,620           |                  50                   |                 4,494,521                  |                             0                             | 0.02 |  0.02   | 0.2  |   0.2    |           6.54            |           6.54           |    359.1     |                  350.6                   |                  163.3                  |    184.8    |         156,624         |        75.46         |             87.51              |         39.37         |          702.4           |
|      2021-04-01      |   CL   |         6.5          |   26,191   |   96,130   |        531,392,923,391         |         5,256,495,488         |            92,909            |           5,369,178,411           |                  26                   |                 1,716,714                  |                             0                             | 0.02 |  0.02   | 0.18 |   0.17   |           7.03            |           7.02           |    359.2     |                  350.8                   |                  150.9                  |    197.3    |         130,051         |        77.97         |             76.65              |         37.08         |          692.8           |
|      2021-04-01      |   CL   |          7           |   15,407   |   44,599   |        253,421,440,867         |         1,870,369,923         |            79,380            |           1,912,310,358           |                   9                   |                  515,923                   |                             0                             | 0.02 |  0.02   | 0.17 |   0.17   |           7.59            |           7.59           |    359.3     |                  344.6                   |                  136.9                  |    211.5    |         112,154         |         79.5         |             80.56              |         37.97         |          677.7           |
|      2021-04-01      |   CL   |         7.5          |   7,373    |   13,676   |        123,297,882,258         |          417,840,326          |            73,219            |            428,361,125            |                   2                   |                   20,163                   |                             0                             | 0.01 |  0.01   | 0.16 |   0.16   |           8.09            |           8.09           |    359.7     |                  352.3                   |                  105.6                  |    243.8    |         99,847          |         79.9         |               NA               |          NA           |          675.4           |
|      2021-04-01      |   CL   |          8           |   4,090    |   7,182    |         70,711,436,813         |          175,137,375          |            67,501            |            180,002,156            |                   1                   |                  107,872                   |                             0                             | 0.01 |  0.01   | 0.16 |   0.15   |           8.57            |           8.57           |    359.7     |                   348                    |                  88.08                  |    261.7    |         90,657          |        80.36         |               NA               |          NA           |          674.1           |
|      2021-04-01      |   CL   |         8.5          |   1,532    |   2,423    |         27,780,574,596         |          47,796,322           |            63,106            |            49,226,045             |                   0                   |                     0                      |                             0                             | 0.01 |  0.01   | 0.14 |   0.14   |           9.06            |           9.05           |    359.6     |                  346.4                   |                  75.85                  |     274     |         82,209          |        80.51         |               NA               |          NA           |          662.9           |
|      2021-04-01      |   CL   |          9           |    533     |    731     |         13,512,984,873         |          11,681,396           |            58,741            |            12,121,410             |                   1                   |                   46,125                   |                             0                             | 0.02 |  0.01   | 0.17 |   0.13   |           9.61            |           9.6            |     360      |                  339.8                   |                  68.65                  |    283.2    |         77,184          |        77.27         |               NA               |          NA           |          660.3           |
|      2021-04-01      |   CL   |         9.5          |    119     |    157     |         1,940,464,361          |           2,000,986           |            60,268            |             2,070,682             |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.04 |   0.04   |           10.03           |          10.02           |     360      |                  313.4                   |                  50.43                  |    303.1    |         69,854          |         71.8         |               NA               |          NA           |          646.8           |
|      2021-04-01      |   CL   |          10          |     33     |     40     |          215,668,649           |           1,053,010           |            58,650            |             1,067,732             |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.03 |   0.03   |            11             |          10.91           |     360      |                  251.8                   |                  84.88                  |    263.5    |         82,957          |        78.52         |               NA               |          NA           |          648.3           |
|      2021-04-01      |   CL   |         10.5         |     6      |     10     |           5,477,700            |            221,202            |            42,200            |              223,302              |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.01 |   0.01   |           11.49           |          11.62           |    358.4     |                  293.2                   |                  88.65                  |    264.3    |         49,094          |        75.29         |               NA               |          NA           |          615.6           |
|      2021-04-01      |   CL   |         11.5         |     2      |     5      |           2,603,237            |            97,693             |            44,800            |              99,065               |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.04 |   0.04   |           12.52           |          12.97           |     360      |                  198.9                   |                  80.87                  |    276.1    |         77,686          |        68.02         |               NA               |          NA           |           638            |
|      2021-03-01      |   CL   |          1           |     1      |     87     |           34,604,768           |          34,528,078           |           398,264            |                 0                 |                   0                   |                     0                      |                            NA                             |  NA  |   NA    |  NA  |    NA    |             2             |            2             |     360      |                   359                    |                   358                   |      2      |         432,187         |          63          |               63               |          30           |           775            |
|      2021-03-01      |   CL   |         1.5          |    399     |  196,451   |         72,245,207,648         |        71,258,978,497         |           366,163            |          60,971,782,978           |                   2                   |                  704,896                   |                             0                             |  0   |    0    | 0.03 |   0.03   |           2.52            |           2.52           |    358.6     |                  358.4                   |                  354.4                  |     3.4     |         402,484         |        69.42         |             69.87              |         32.07         |          773.5           |
|      2021-03-01      |   CL   |          2           |   5,180    | 1,633,407  |        517,334,382,825         |        500,989,257,819        |           310,086            |          442,904,782,729          |                  24                   |                 7,772,415                  |                             0                             | 0.01 |  0.01   | 0.09 |   0.09   |            2.9            |           2.9            |    357.2     |                  357.1                   |                  351.7                  |    4.47     |         361,162         |        72.29         |              72.7              |         33.52         |          764.2           |
|      2021-03-01      |   CL   |         2.5          |   8,452    | 1,455,790  |        479,533,896,894         |        394,745,420,130        |           277,613            |          394,707,634,856          |                  100                  |                 31,054,679                 |                             0                             | 0.03 |  0.03   | 0.31 |   0.31   |           3.37            |           3.37           |    357.1     |                  356.9                   |                   346                   |    9.58     |         335,358         |        74.44         |             75.01              |         34.51         |          754.7           |
|      2021-03-01      |   CL   |          3           |   24,839   | 2,185,551  |       1,038,367,591,745        |        471,944,765,150        |           243,609            |          492,940,854,201          |                  315                  |                 87,594,196                 |                             0                             | 0.05 |  0.05   | 0.43 |   0.43   |           3.73            |           3.72           |    358.7     |                  357.9                   |                  304.9                  |    47.2     |         296,410         |        75.01         |             76.11              |         34.46         |          756.1           |
|      2021-03-01      |   CL   |         3.5          |   45,889   | 2,175,955  |       1,241,408,026,878        |        400,207,448,652        |           210,231            |          419,435,932,664          |                  440                  |                105,654,450                 |                             0                             | 0.04 |  0.04   | 0.42 |   0.42   |           4.13            |           4.12           |     359      |                  358.1                   |                   292                   |    59.05    |         266,489         |        76.56         |             78.16              |         35.57         |          748.2           |
|      2021-03-01      |   CL   |          4           |   49,963   | 1,837,177  |       1,216,961,513,388        |        298,731,071,619        |           186,606            |          311,977,422,443          |                  583                  |                135,319,481                 |                             0                             | 0.04 |  0.04   | 0.4  |   0.39   |           4.58            |           4.58           |    359.1     |                   358                    |                  290.4                  |    60.95    |         245,929         |        77.43         |             79.47              |         36.92         |          735.9           |
|      2021-03-01      |   CL   |         4.5          |   34,895   |  924,072   |        834,293,884,456         |        129,379,349,527        |           164,623            |          134,198,116,693          |                  345                  |                 70,009,676                 |                             0                             | 0.03 |  0.03   | 0.34 |   0.34   |           5.05            |           5.04           |    359.1     |                  357.7                   |                  280.4                  |    70.72    |         224,502         |        77.15         |             80.67              |         38.2          |          725.3           |
|      2021-03-01      |   CL   |          5           |   28,992   |  398,703   |        779,815,937,573         |        44,466,595,368         |           148,463            |          45,790,389,785           |                  156                  |                 27,475,776                 |                             0                             | 0.03 |  0.03   | 0.27 |   0.27   |           5.56            |           5.56           |    358.8     |                  354.8                   |                  238.4                  |    111.5    |         203,711         |        76.33         |             81.86              |         38.78         |          716.4           |
|      2021-03-01      |   CL   |         5.5          |   35,438   |  277,931   |       1,066,998,752,059        |        22,627,622,498         |           128,451            |          23,105,112,737           |                  81                   |                 9,118,552                  |                             0                             | 0.02 |  0.02   | 0.18 |   0.18   |           6.01            |           6.01           |    358.8     |                  351.9                   |                  170.8                  |    177.5    |         172,625         |        73.14         |             86.96              |         39.37         |          712.4           |
|      2021-03-01      |   CL   |          6           |   34,801   |  191,518   |        857,616,866,370         |        13,871,067,620         |           113,310            |          14,133,362,119           |                  72                   |                 8,166,296                  |                             0                             | 0.01 |  0.01   | 0.16 |   0.16   |           6.54            |           6.54           |    359.1     |                  350.6                   |                  164.4                  |    183.8    |         156,842         |        75.48         |             87.72              |         39.33         |          702.5           |
|      2021-03-01      |   CL   |         6.5          |   26,356   |   97,653   |        533,356,973,879         |         5,369,178,411         |            92,960            |           5,467,783,035           |                  24                   |                 2,218,074                  |                             0                             | 0.01 |  0.01   | 0.15 |   0.14   |           7.03            |           7.02           |    359.2     |                  350.8                   |                  151.9                  |    196.3    |         130,126         |        77.96         |             76.83              |         37.08         |          692.9           |
|      2021-03-01      |   CL   |          7           |   15,513   |   45,348   |        254,338,923,598         |         1,912,308,337         |            79,421            |           1,945,792,101           |                  17                   |                 1,164,549                  |                             0                             | 0.01 |  0.01   | 0.12 |   0.12   |           7.59            |           7.59           |    359.3     |                  344.6                   |                  137.8                  |    210.6    |         112,469         |        79.51         |             80.59              |         37.96         |          677.8           |
|      2021-03-01      |   CL   |         7.5          |   7,436    |   13,907   |        123,967,193,792         |          428,361,125          |            73,233            |            438,554,464            |                   1                   |                   13,354                   |                             0                             | 0.01 |  0.01   | 0.15 |   0.15   |           8.09            |           8.09           |    359.7     |                  352.2                   |                  106.2                  |    243.1    |         100,126         |        79.84         |               NA               |          NA           |          675.5           |
|      2021-03-01      |   CL   |          8           |   4,123    |   7,339    |         70,978,726,577         |          180,001,151          |            67,455            |            184,371,004            |                   1                   |                   39,252                   |                             0                             | 0.01 |  0.01   | 0.12 |   0.12   |           8.57            |           8.57           |    359.7     |                   348                    |                  88.44                  |    261.3    |         90,601          |        80.38         |               NA               |          NA           |          674.2           |
|      2021-03-01      |   CL   |         8.5          |   1,557    |   2,489    |         27,907,513,764         |          49,226,045           |            63,033            |            50,462,615             |                   0                   |                     0                      |                             0                             | 0.01 |  0.01   | 0.09 |   0.09   |           9.06            |           9.05           |    359.6     |                  346.2                   |                  76.06                  |    273.7    |         82,024          |        80.44         |               NA               |          NA           |          663.2           |
|      2021-03-01      |   CL   |          9           |    557     |    767     |         14,404,259,988         |          12,120,921           |            58,747            |            12,495,938             |                   0                   |                     0                      |                             0                             | 0.01 |  0.01   | 0.1  |   0.1    |            9.6            |           9.6            |     360      |                  340.1                   |                  69.2                   |    282.7    |         76,955          |        77.36         |               NA               |          NA           |          659.7           |
|      2021-03-01      |   CL   |         9.5          |    123     |    161     |         1,661,869,650          |           2,069,800           |            61,068            |             2,141,834             |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.04 |   0.04   |           10.03           |          10.02           |     360      |                  313.9                   |                  50.58                  |     303     |         70,326          |        71.71         |               NA               |          NA           |          647.4           |
|      2021-03-01      |   CL   |          10          |     32     |     42     |          132,519,842           |           1,067,322           |            60,548            |             1,084,349             |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.03 |   0.03   |            11             |          10.91           |     360      |                   252                    |                  85.54                  |    262.8    |         82,901          |         78.5         |               NA               |          NA           |          648.4           |
|      2021-03-01      |   CL   |         10.5         |     6      |     10     |           5,477,700            |            223,302            |            42,200            |              225,386              |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.01 |   0.01   |           11.49           |          11.62           |    358.4     |                   293                    |                  89.49                  |    263.4    |         49,103          |        75.31         |               NA               |          NA           |          615.5           |
|      2021-03-01      |   CL   |         11.5         |     2      |     5      |           2,603,237            |            99,065             |            44,800            |              100,423              |                   0                   |                     0                      |                             0                             |  0   |    0    | 0.03 |   0.03   |           12.52           |          12.96           |     360      |                  198.8                   |                  81.53                  |    275.5    |         77,332          |        68.06         |               NA               |          NA           |          637.8           |

FN CL by Coupon

### GNMA Aggregations

A `GinnieMBS` object is loaded from **AWS S3**

``` r
options(verbose = FALSE)
library(aws.s3)
readRenviron("~/Finance/GNMA/.Renviron")
# devtools::load_all("~/Finance/GNMA/gnmar", reset = TRUE, recompile = FALSE, export_all = FALSE)
# library(gnmar)
remotes::install_github("canarionyc/gnmar",
#                        dependencies = FALSE,
                        force = FALSE)
library(gnmar)

As_of_Date <- as.Date("2021-03-01")

args.lst <- list(
  As_of_Date = As_of_Date
 #                , mf_zip=mf_zip
  #              ,  bucket_name = gnma_mbs_sf_bucket
  , overwrite = TRUE
  , verbose = FALSE)

ginnieMBS <- do.call(GinnieMBS, args.lst)
# show(ginnieMBS)
```

#### GNMA by Prefix

``` r
GNSF <- subset(ginnieMBS, subset=quote(Pool_Indicator=="X" & Pool_Type=="SF" &  Issuer_Number!=9999 & Security_Interest_Rate %in% seq(0.5, 11, by=0.5)))
# print(summary(GNSF))
GNSF_stats <- aggregate(GNSF, xvar=NULL,  by.vars=c('Pool_Indicator', 'Pool_Type' )
                        , verbose=FALSE
                          )
```

| grouping | Pool Indicator | Pool Type | Pool Count | Loan Count | Original Aggregate Amount | Remaining Security RPB | WA Interest Rate WAC | WA Remaining Months to Maturity WARM | WA Loan Age WALA | WA Original Loan Term WAOLT | WA Loan to Value LTV | WA Combined Loan to Value CLTV | Average Original Loan Size AOLS | WA Original Loan Size | SMM  | CPR  |
|:--------:|:--------------:|:---------:|:----------:|:----------:|:-------------------------:|:----------------------:|:--------------------:|:------------------------------------:|:----------------:|:---------------------------:|:--------------------:|:------------------------------:|:-------------------------------:|:---------------------:|:----:|:----:|
|    0     |       X        |    SF     |  110,852   |  908,889   |     1,487,953,599,954     |     84,506,015,250     |         4.66         |                225.4                 |      118.1       |            351.7            |        93.66         |             93.77              |             129,528             |        174,202        | 0.03 | 0.27 |

### GNMA by Prefix and Coupon

``` r
# GNSF_Coupon <- subset(GNSF, subset=quote(Pool_Indicator=="X" & Pool_Type=="SF" &  Issuer_Number!=9999 & Security_Interest_Rate %in% seq(0.5, 11, by=0.5)))
# print(summary(GNSF))
GNSF_stats.by_Coupon <- aggregate(GNSF, xvar=NULL
                        , by.vars=c('Pool_Indicator', 'Pool_Type', 'Security_Interest_Rate' )
                        , verbose=FALSE
                          )
```

| grouping | Pool Indicator | Pool Type | Security Interest Rate | Pool Count | Loan Count | Original Aggregate Amount | Remaining Security RPB | WA Interest Rate WAC | WA Remaining Months to Maturity WARM | WA Loan Age WALA | WA Original Loan Term WAOLT | WA Loan to Value LTV | WA Combined Loan to Value CLTV | Average Original Loan Size AOLS | WA Original Loan Size | SMM  | CPR  |
|:--------:|:--------------:|:---------:|:----------------------:|:----------:|:----------:|:-------------------------:|:----------------------:|:--------------------:|:------------------------------------:|:----------------:|:---------------------------:|:--------------------:|:------------------------------:|:-------------------------------:|:---------------------:|:----:|:----:|
|    0     |       X        |    SF     |          0.5           |     14     |     21     |         4,532,671         |       2,288,173        |          1           |                 260                  |      97.91       |             360             |         97.7         |              97.7              |             145,542             |        167,299        |  NA  |  NA  |
|    0     |       X        |    SF     |          1.5           |     10     |     11     |         3,441,367         |       1,406,007        |          2           |                275.9                 |      82.79       |             360             |        97.01         |             97.01              |             157,027             |        170,971        | 0.08 | 0.65 |
|    0     |       X        |    SF     |           2            |     84     |    849     |        199,937,563        |       80,060,513       |         2.5          |                250.1                 |      64.21       |            319.4            |        94.37         |             94.76              |             132,669             |        159,954        | 0.01 | 0.16 |
|    0     |       X        |    SF     |          2.5           |    962     |   21,189   |       7,528,787,878       |     2,373,650,035      |          3           |                230.2                 |      75.72       |            312.6            |        92.18         |             92.34              |             164,191             |        216,192        | 0.02 | 0.22 |
|    0     |       X        |    SF     |           3            |   5,438    |  125,394   |      59,323,555,859       |     15,687,611,756     |         3.5          |                255.7                 |      83.87       |            346.8            |        93.92         |             94.04              |             161,782             |        210,662        | 0.03 | 0.31 |
|    0     |       X        |    SF     |          3.5           |   7,186    |  121,756   |      69,278,090,890       |     13,648,931,327     |          4           |                 253                  |      89.99       |            350.3            |        93.64         |             93.78              |             143,957             |        184,564        | 0.03 | 0.3  |
|    0     |       X        |    SF     |           4            |   9,068    |  146,968   |      114,699,350,895      |     15,270,926,354     |         4.5          |                237.9                 |      104.5       |            349.9            |        93.68         |             93.83              |             140,846             |        180,225        | 0.03 | 0.32 |
|    0     |       X        |    SF     |          4.5           |   9,696    |  153,188   |      223,713,195,025      |     15,932,445,144     |          5           |                220.7                 |      127.4       |            357.1            |         93.6         |             93.65              |             139,987             |        174,231        | 0.03 | 0.29 |
|    0     |       X        |    SF     |           5            |   11,185   |  126,348   |      197,959,700,477      |     10,655,398,760     |         5.5          |                 205                  |      143.6       |            358.1            |        93.65         |             93.81              |             118,454             |        149,070        | 0.02 | 0.24 |
|    0     |       X        |    SF     |          5.5           |   11,930   |   73,420   |      189,769,115,441      |     4,816,564,503      |          6           |                168.2                 |       179        |            357.4            |        93.52         |             93.53              |             103,863             |        126,099        | 0.02 | 0.18 |
|    0     |       X        |    SF     |           6            |   13,878   |   60,434   |      180,168,214,680      |     3,562,473,736      |         6.5          |                163.1                 |      184.1       |            357.9            |        93.54         |             93.54              |             95,312              |        117,379        | 0.01 | 0.15 |
|    0     |       X        |    SF     |          6.5           |   12,968   |   32,368   |      151,657,306,854      |     1,354,904,021      |          7           |                135.9                 |      212.6       |            358.8            |        94.29         |             94.29              |             82,142              |        99,476         | 0.01 | 0.09 |
|    0     |       X        |    SF     |           7            |   12,382   |   23,727   |      131,582,962,190      |      675,420,154       |         7.5          |                103.6                 |      246.3       |            359.5            |        94.17         |             94.17              |             72,200              |        88,181         |  0   | 0.05 |
|    0     |       X        |    SF     |          7.5           |   7,540    |   11,506   |      76,427,395,516       |      239,624,173       |          8           |                74.34                 |      277.4       |            359.8            |        94.81         |             94.81              |             65,774              |        79,851         |  NA  |  NA  |
|    0     |       X        |    SF     |           8            |   5,622    |   8,242    |      59,952,927,565       |      154,687,565       |         8.5          |                71.19                 |      280.4       |            359.9            |        94.78         |             94.78              |             60,917              |        74,506         |  NA  |  NA  |
|    0     |       X        |    SF     |          8.5           |   2,000    |   2,484    |      17,735,357,600       |       39,068,138       |          9           |                67.16                 |      284.6       |            359.9            |        94.47         |             94.47              |             55,047              |        72,143         |  NA  |  NA  |
|    0     |       X        |    SF     |           9            |    745     |    842     |       7,157,365,908       |       9,254,595        |         9.5          |                 53.1                 |      299.9       |             360             |          94          |               94               |             52,856              |        73,888         |  NA  |  NA  |
|    0     |       X        |    SF     |          9.5           |    127     |    126     |        703,623,183        |       1,250,457        |          10          |                52.64                 |      299.6       |            359.9            |         93.7         |              93.7              |             49,751              |        89,361         |  NA  |  NA  |
|    0     |       X        |    SF     |           10           |     16     |     15     |        85,069,873         |         46,970         |         10.5         |                 22.8                 |       326        |            355.6            |         89.9         |              89.9              |             50,540              |        46,943         |  NA  |  NA  |
|    0     |       X        |    SF     |          10.5          |     1      |     1      |         3,668,519         |         2,869          |          11          |                  3                   |       355        |             360             |          67          |               67               |             171,250             |        171,250        |  NA  |  NA  |
