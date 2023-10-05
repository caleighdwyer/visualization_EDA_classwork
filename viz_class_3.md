viz_class_3
================
Caleigh Dwyer
2023-10-05

Exploratory analysis (EDA)!

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(ggridges)
library(lubridate)
```

``` r
weather_df <- 
  rnoaa::meteo_pull_monitors(
    c("USW00094728", "USW00022534", "USS0023B17S"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2022-01-01",
    date_max = "2023-12-31") %>%
  mutate(
    name = recode(
      id, 
      USW00094728 = "CentralPark_NY", 
      USW00022534 = "Molokai_HI",
      USS0023B17S = "Waterhole_WA"),
    tmin = tmin / 10,
    tmax = tmax / 10,
    month = lubridate::floor_date(date, unit = "month")) %>%
  select(name, id, everything())
```

    ## using cached file: /Users/caleighdwyer/Library/Caches/org.R-project.R/R/rnoaa/noaa_ghcnd/USW00094728.dly

    ## date created (size, mb): 2023-09-28 10:20:17.291852 (8.524)

    ## file min/max dates: 1869-01-01 / 2023-09-30

    ## using cached file: /Users/caleighdwyer/Library/Caches/org.R-project.R/R/rnoaa/noaa_ghcnd/USW00022534.dly

    ## date created (size, mb): 2023-09-28 10:20:26.444587 (3.83)

    ## file min/max dates: 1949-10-01 / 2023-09-30

    ## using cached file: /Users/caleighdwyer/Library/Caches/org.R-project.R/R/rnoaa/noaa_ghcnd/USS0023B17S.dly

    ## date created (size, mb): 2023-09-28 10:20:28.983341 (0.994)

    ## file min/max dates: 1999-09-01 / 2023-09-30

^chunk didn’t work for me

\##initial numeric work

``` r
weather_df |> 
  ggplot(aes(x=prcp)) +
  geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

    ## Warning: Removed 30 rows containing non-finite values (`stat_bin()`).

![](viz_class_3_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

here are the big outliers:

``` r
weather_df |> 
  filter(prcp > 1000)
```

    ## # A tibble: 2 × 7
    ##   name       id          date        prcp  tmax  tmin month     
    ##   <chr>      <chr>       <date>     <dbl> <dbl> <dbl> <date>    
    ## 1 Molokai_HI USW00022534 2022-12-18  1120  23.3  18.9 2022-12-01
    ## 2 Molokai_HI USW00022534 2023-01-28  1275  21.7  18.3 2023-01-01

``` r
weather_df |> 
  filter(tmax>= 20, tmax <= 30) |> 
  ggplot(aes(x=tmin, y= tmax, color = name)) +
  geom_point()
```

![](viz_class_3_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

\##grouping

``` r
weather_df |> 
  group_by(name, month) |> 
  summarize(n_obs = n())
```

    ## `summarise()` has grouped output by 'name'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 63 × 3
    ## # Groups:   name [3]
    ##    name           month      n_obs
    ##    <chr>          <date>     <int>
    ##  1 CentralPark_NY 2022-01-01    31
    ##  2 CentralPark_NY 2022-02-01    28
    ##  3 CentralPark_NY 2022-03-01    31
    ##  4 CentralPark_NY 2022-04-01    30
    ##  5 CentralPark_NY 2022-05-01    31
    ##  6 CentralPark_NY 2022-06-01    30
    ##  7 CentralPark_NY 2022-07-01    31
    ##  8 CentralPark_NY 2022-08-01    31
    ##  9 CentralPark_NY 2022-09-01    30
    ## 10 CentralPark_NY 2022-10-01    31
    ## # ℹ 53 more rows

``` r
weather_df |> 
  count(name, name = "n_obs")
```

    ## # A tibble: 3 × 2
    ##   name           n_obs
    ##   <chr>          <int>
    ## 1 CentralPark_NY   638
    ## 2 Molokai_HI       638
    ## 3 Waterhole_WA     638

``` r
weather_df |> 
  count(name, month) |> 
  pivot_wider(
    names_from = name,
    values_from = n
  )
```

    ## # A tibble: 21 × 4
    ##    month      CentralPark_NY Molokai_HI Waterhole_WA
    ##    <date>              <int>      <int>        <int>
    ##  1 2022-01-01             31         31           31
    ##  2 2022-02-01             28         28           28
    ##  3 2022-03-01             31         31           31
    ##  4 2022-04-01             30         30           30
    ##  5 2022-05-01             31         31           31
    ##  6 2022-06-01             30         30           30
    ##  7 2022-07-01             31         31           31
    ##  8 2022-08-01             31         31           31
    ##  9 2022-09-01             30         30           30
    ## 10 2022-10-01             31         31           31
    ## # ℹ 11 more rows

\##general summaries

``` r
weather_df |> 
  group_by (name) |> 
  summarize(
    mean_tmax = mean(tmax, na.rm = TRUE),
    sd_tmax = sd(tmax, na.rm = TRUE),
    median_tmax = median (tmax, na.rm = TRUE)
  ##take averages after removing missing values
  )
```

    ## # A tibble: 3 × 4
    ##   name           mean_tmax sd_tmax median_tmax
    ##   <chr>              <dbl>   <dbl>       <dbl>
    ## 1 CentralPark_NY     18.6     9.67        20  
    ## 2 Molokai_HI         28.5     1.85        28.9
    ## 3 Waterhole_WA        8.01    7.57         6.7

``` r
weather_df |> 
  group_by(name, month) |> 
  summarize(mean_tmax = mean(tmax, na.rm = TRUE)) |> 
  ggplot(aes(x=month, y=mean_tmax, color = name)) + 
  geom_point()+
  geom_line()
```

    ## `summarise()` has grouped output by 'name'. You can override using the
    ## `.groups` argument.

![](viz_class_3_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
weather_df |> 
  group_by(name,month) |> 
  summarize(mean_tmax = mean(tmax, na.rm = TRUE)) |> 
  pivot_wider(
    names_from = name,
    values_from = mean_tmax
  ) |> 
  knitr::kable(digits=2)
```

    ## `summarise()` has grouped output by 'name'. You can override using the
    ## `.groups` argument.

| month      | CentralPark_NY | Molokai_HI | Waterhole_WA |
|:-----------|---------------:|-----------:|-------------:|
| 2022-01-01 |           2.85 |      26.61 |         3.61 |
| 2022-02-01 |           7.65 |      26.83 |         2.99 |
| 2022-03-01 |          11.99 |      27.73 |         3.42 |
| 2022-04-01 |          15.81 |      27.72 |         2.46 |
| 2022-05-01 |          22.25 |      28.28 |         5.81 |
| 2022-06-01 |          26.09 |      29.16 |        11.13 |
| 2022-07-01 |          30.72 |      29.53 |        15.86 |
| 2022-08-01 |          30.50 |      30.70 |        18.83 |
| 2022-09-01 |          24.92 |      30.41 |        15.21 |
| 2022-10-01 |          17.43 |      29.22 |        11.88 |
| 2022-11-01 |          14.02 |      27.96 |         2.14 |
| 2022-12-01 |           6.76 |      27.35 |        -0.46 |
| 2023-01-01 |           9.29 |      26.41 |         0.74 |
| 2023-02-01 |           9.21 |      26.37 |        -0.23 |
| 2023-03-01 |          10.83 |      27.00 |         1.24 |
| 2023-04-01 |          19.03 |      27.72 |         3.97 |
| 2023-05-01 |          22.10 |      28.17 |        11.89 |
| 2023-06-01 |          25.42 |      29.66 |        13.50 |
| 2023-07-01 |          30.20 |      30.05 |        16.34 |
| 2023-08-01 |          27.62 |      30.60 |        17.50 |
| 2023-09-01 |          25.62 |      30.89 |        13.29 |
