
<!-- README.md is generated from README.Rmd. Please edit that file -->

# indexing

<!-- badges: start -->
<!-- badges: end -->

The `indexing` package provides a convenient function, `indexing()`, to
create an index from a variable in a tidy dataset. This index is
generated using specified observations as reference points.

## Installation

You can install the development version of indexing from
[GitHub](https://github.com/stat545ubc-2023/indexing) with:

``` r
# install.packages("devtools")
devtools::install_github("stat545ubc-2023/indexing")
```

## Demonstrated usage

### Example 1:

In this example, the function constructs a time series and establishes
an index utilizing the `values` column, with the base observation set at
`2023-01-01`.

``` r
# Generate a toy dataset
set.seed(545)
df <- data.frame(date = seq(as.Date("2023-01-01"), 
                            as.Date("2025-12-01"),
                            "month"),
                 values = runif(12*3, 50, 150) * (2 + cumsum(runif(12*3, 0, 0.15))))

# Display the first few rows of the dataset
head(df)
#>         date   values
#> 1 2023-01-01 252.4081
#> 2 2023-02-01 318.1164
#> 3 2023-03-01 285.1015
#> 4 2023-04-01 230.2832
#> 5 2023-05-01 183.6369
#> 6 2023-06-01 322.0151

# Generate an index
df2 <- df %>% 
  indexing(values,
           date == as.Date("2023-01-01"))

# Display the first few rows of the indexed dataset
head(df2)
#>         date   values     index
#> 1 2023-01-01 252.4081 100.00000
#> 2 2023-02-01 318.1164 126.03257
#> 3 2023-03-01 285.1015 112.95261
#> 4 2023-04-01 230.2832  91.23447
#> 5 2023-05-01 183.6369  72.75395
#> 6 2023-06-01 322.0151 127.57717

# Compare the original variable vs. the indexed variable
df2 %>% 
  ggplot(aes(x = date)) +
  geom_hline(yintercept = 100,
             linetype = "dashed") +
  geom_line(aes(y = values,
                color = "values")) +
  geom_line(aes(y = index,
                color = "index")) +
  theme_minimal()
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />

### Example 2:

Using the toy dataset from earlier, this code chunk generates an indexed
variable by averaging values across multiple observations. Specifically,
it calculates the average for the year 2023, with a warning issued if
this operation was unintended.

``` r
# Generate an index based on the year 2023

df2 <- df %>% 
  indexing(values,
           year(date) == 2023)
#> Warning in indexing(., values, year(date) == 2023): More than one row is being
#> used as a reference.

# Display the first few rows of the indexed dataset
head(df2)
#>         date   values     index
#> 1 2023-01-01 252.4081  93.41062
#> 2 2023-02-01 318.1164 117.72781
#> 3 2023-03-01 285.1015 105.50974
#> 4 2023-04-01 230.2832  85.22269
#> 5 2023-05-01 183.6369  67.95992
#> 6 2023-06-01 322.0151 119.17063

# Compare the original variable vs. the indexed variable
df2 %>% 
  ggplot(aes(x = date)) +
  geom_hline(yintercept = 100,
             linetype = "dashed") +
  geom_line(aes(y = values,
                color = "values")) +
  geom_line(aes(y = index,
                color = "index")) +
  theme_minimal()
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />

### Example 3:

In this example, we demonstrate indexing the GDP per capita for Oceania
countries, utilizing the year 1952 as the reference point.

``` r
# Load the gapminder library
library(gapminder)

# Select Oceania countries from the gapminder dataset
oc_gapminder <- gapminder %>% 
  filter(continent == "Oceania") 

# Display the first few rows of the Oceania dataset
oc_gapminder %>% 
  arrange(year) %>% 
  head()
#> # A tibble: 6 × 6
#>   country     continent  year lifeExp      pop gdpPercap
#>   <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
#> 1 Australia   Oceania    1952    69.1  8691212    10040.
#> 2 New Zealand Oceania    1952    69.4  1994794    10557.
#> 3 Australia   Oceania    1957    70.3  9712569    10950.
#> 4 New Zealand Oceania    1957    70.3  2229407    12247.
#> 5 Australia   Oceania    1962    70.9 10794968    12217.
#> 6 New Zealand Oceania    1962    71.2  2488550    13176.

# Create a country-wise index based on GDP per capita for the year 1952
idx_gap <- oc_gapminder %>% 
  group_by(country) %>% 
  indexing(gdpPercap,
           year == 1952) 

# Display the first few rows of the indexed dataset
idx_gap %>% 
  arrange(year) %>% 
  head()
#> # A tibble: 6 × 7
#> # Groups:   country [2]
#>   country     continent  year lifeExp      pop gdpPercap index
#>   <fct>       <fct>     <int>   <dbl>    <int>     <dbl> <dbl>
#> 1 Australia   Oceania    1952    69.1  8691212    10040.  100 
#> 2 New Zealand Oceania    1952    69.4  1994794    10557.  100 
#> 3 Australia   Oceania    1957    70.3  9712569    10950.  109.
#> 4 New Zealand Oceania    1957    70.3  2229407    12247.  116.
#> 5 Australia   Oceania    1962    70.9 10794968    12217.  122.
#> 6 New Zealand Oceania    1962    71.2  2488550    13176.  125.

# Compare indexes across countries using a line plot
idx_gap %>% 
  ggplot(aes(x = year, 
             y = index,
             color = country)) +
  geom_line() +
  theme_minimal()
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />
