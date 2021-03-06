Infant Mortality
================
Lucy
2018-05-18

-   [Racial disparities in infant mortality](#racial-disparities-in-infant-mortality)
-   [Part 1: Examine racial disparities in longitudinal trends](#part-1-examine-racial-disparities-in-longitudinal-trends)
    -   [Download and read the CDC Linked Birth / Infant Death records](#download-and-read-the-cdc-linked-birth-infant-death-records)
    -   [Create new variables](#create-new-variables)
    -   [Examine longitudinal trends in infant mortality](#examine-longitudinal-trends-in-infant-mortality)
    -   [Identify riskiest U.S. regions for Non-Hispanic Black infants and mothers](#identify-riskiest-u.s.-regions-for-non-hispanic-black-infants-and-mothers)
-   [Part 2: Examine infant mortality at the U.S. state-level](#part-2-examine-infant-mortality-at-the-u.s.-state-level)
    -   [Download and tidy infant mortality data at the U.S. state-level](#download-and-tidy-infant-mortality-data-at-the-u.s.-state-level)
    -   [Map infant mortality data at the U.S. state-level](#map-infant-mortality-data-at-the-u.s.-state-level)
-   [Answers](#answers)

``` r
# Libraries
library(tidyverse)
library(sf)
library(compare)

# Parameters
# solution-begin

## Files 
file_answers <- "~/answers.rds"

infant_mortality_file_path <- "~/Desktop/datalab_data/c01_data/census_region_level/ten_regions/"

infant_mortality_files <-
  list.files(
    path = infant_mortality_file_path,
    pattern = "^Linked Birth  Infant Death Records, \\d{4}-\\d{4}.txt$",
    full.names = TRUE
  )

## data across 2007-2015 grouped by state, hispanic origin
state_infant_mortality_file_path <-
  "~/Desktop/datalab_data/c01_data/state_level/Linked Birth  Infant Death Records, 2007-2015_allyears.txt"

##data across 2010-2015 grouped by state, hispanic origin
state_10_15_infant_mortality_file_path <-
  "~/Desktop/datalab_data/c01_data/state_level/Linked Birth  Infant Death Records, 2010-2015.txt"

##data across 1999-2000 grouped by state, hispanic origin
state_early_years_infant_mortality_file_path <-
  "~/Desktop/datalab_data/c01_data/state_level/early_years/"

##early years (1995-2000) grouped by state, hispanic origin
early_years_infant_mortality_files <-
  list.files(
    path = state_early_years_infant_mortality_file_path,
    pattern = "^Linked Birth  Infant Death Records, \\d{4}-\\d{4}.txt$",
    full.names = TRUE
  )
  
# Albers projection for 48 contiguous US states
US_ALBERS <- "+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96 +x_0=0 +y_0=0 +datum=WGS84 +no_defs"

#US county and state boundaries in sf format

file_state_boundaries <- "~/Desktop/datalab_data/cb_2015_us_state_20m_sf.rds"

##Labels, colors, and values for regions and states

#region labels 
region_labels = c(
  "1: CT, ME, MA, NH, RI, VT",
  "2: NJ, NY",
  "3: DE, DC, MD, PA, VA, WV",
  "4: AL, FL, GA, KY, MS, NC, SC, TN",
  "5: IL, IN, MI, MN, OH, WI",
  "6: AR, LA, NM, OK, TX",
  "7: IA, KS, MO, NE",
  "8: CO, MT, ND, SD, UT, WY",
  "9: AZ, CA, HI, NV",
  "10: AK, ID, OR, WA"
)

#region values
region_order = c(
  "HHS Region #1  CT, ME, MA, NH, RI, VT",
  "HHS Region #2  NJ, NY",
  "HHS Region #3  DE, DC, MD, PA, VA, WV" ,
  "HHS Region #4  AL, FL, GA, KY, MS, NC, SC, TN",
  "HHS Region #5  IL, IN, MI, MN, OH, WI",
  "HHS Region #6  AR, LA, NM, OK, TX",  
  "HHS Region #7  IA, KS, MO, NE", 
  "HHS Region #8  CO, MT, ND, SD, UT, WY", 
  "HHS Region #9  AZ, CA, HI, NV",
  "HHS Region #10  AK, ID, OR, WA" 
)



# Colors for single time point rates
rate_colors_black <- c(
  "#e9b777",
  "#d27952",
  "#b63132",
  "#740023"
)

rate_colors_black <- c(
  "#e9b777", 
  "#d27952", 
  "#740023",
  "black"
)

rate_colors_white <- c(
  "#004364",
  "#55889e",
  "#fdf6a3",
  "#e9b777"
)

## Mapping colors and values

# Values corresponding to single time point rate and disparity colors
rate_values_white <- 
  c(2.76, 3.22, 6.87, 7.04) %>%  #minimum, .025, .975, maximum
  scales::rescale()

rate_values_black <-
  c(6.83, 7.42, 13.62, 13.84) %>% 
  scales::rescale()

disparity_values <-
  c(1.24, 1.37, 2.18,  3.19, 4.14) %>% #minimum, ,.025, median, .975,  max)
  scales::rescale()


# Colors for change in rates
rate_change_colors <- c(
  "#006837",
  "#31a354",
  "#78c679",
  "#addd8e",
  "#d9f0a3",
  "#ffffcc"
)

#Colors for single time point disparity
disparity_colors <- c(
  "#8c510a", 
  "#d8b365", 
  "#c7eae5",
  "#67a9cf", 
  "darkslateblue"
)

na_color <- "#d9d9d9"


# solution-end

#===============================================================================

  # Save answers
SAVE_ANSWERS <- TRUE

# task-begin
  # File for downloaded answers
file_answers <- "~/Desktop/datalab_data/c01_data/census_region_level/ten_regions/answers.rds"

# Read in answers
if (str_length(file_answers) > 0) {
  answers <- read_rds(file_answers) 
}
# task-end
```

Racial disparities in infant mortality
--------------------------------------

A recent New York Times article described the striking racial disparities in infant mortality in U.S. (<https://www.nytimes.com/2018/04/11/magazine/black-mothers-babies-death-maternal-mortality.html>) through the lens of one black mother living in New Orleans who received inadequate medical care and, consequently, lost her baby. The article cites this alarming statistic, “Black infants in America are now more than twice as likely to die as white infants — 11.3 per 1,000 black babies, compared with 4.9 per 1,000 white babies, according to the most recent government data — a racial disparity that is actually wider than in 1850.”

The disparity in infant mortality does not appear to be explained by racial disparities in socioeconomic factors. In fact, a 1992 study (<https://www.nejm.org/doi/full/10.1056/NEJM199206043262303>) concluded that infants born to college-educated black parents were twice as likely to die as infants born to similarly educated white parents, leading to the conclusion that toxic stress and inadequate medical care associated with systemic racism may be the culprit.

In this challenge, you will explore data the from the Centers for Disease Control documenting rates of infant mortality in the U.S. First, you will investigate racial disparities in longitudinal trends in infant mortality from 1995-2015 in 10 different regions of the U.S. Second, you will create U.S. maps to examine current rates of infant mortality, change in infant mortality, and racial disparities in infant mortality by state. By the end of this challenge, you will gain an understanding of which places in the U.S. show the most problematic trends in infant mortality.

The CDC defines infant mortality as follows:

"Infant mortality is the death of an infant before his or her first birthday. The infant mortality rate is the number of infant deaths for every 1,000 live births."

Part 1: Examine racial disparities in longitudinal trends
---------------------------------------------------------

### Download and read the CDC Linked Birth / Infant Death records

**q1**

First, go to <https://wonder.cdc.gov/lbd.html> and download all Linked Birth / Infant Death records for each time period. We will examine *yearly* estimates of infant mortality, focusing on Non-Hispanic White, Non-Hispanic Black, or Hispanic (of any origin) mothers. We will examine these yearly estimates across the 10 HHS (Health and Human Services) regions of the U.S. Save the files somewhere other than your git repository.

Second, use a `map()` function to read in the files to a single dataframe, and retain the following variables (save the dataframe to the tibble `q1`):

HHS Region as `region`

Hispanic Origin as `hisp_origin`

Births as `births`

Deaths as `deaths`

Year of Death as `year`

Hint: `View()` the dataframe and scroll to the bottom to determine any issues.

``` r
# solution-begin
q1 <-
  infant_mortality_files %>%
  map_dfr(read_tsv) %>%
  filter(
    is.na(Notes)
    ) %>%
  select(
    region = `HHS Region`,
    hisp_origin = `Hispanic Origin`,
    births = Births,
    deaths = Deaths,
    year = `Year of Death`
  )
```

    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   `HHS Region` = col_character(),
    ##   `HHS Region Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_character(),
    ##   `Year of Death` = col_integer(),
    ##   `Year of Death Code` = col_integer(),
    ##   Deaths = col_integer(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   `HHS Region` = col_character(),
    ##   `HHS Region Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_character(),
    ##   `Year of Death` = col_integer(),
    ##   `Year of Death Code` = col_integer(),
    ##   Deaths = col_integer(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   `HHS Region` = col_character(),
    ##   `HHS Region Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_character(),
    ##   `Year of Death` = col_integer(),
    ##   `Year of Death Code` = col_integer(),
    ##   Deaths = col_integer(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   `HHS Region` = col_character(),
    ##   `HHS Region Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_character(),
    ##   `Year of Death` = col_integer(),
    ##   `Year of Death Code` = col_integer(),
    ##   Deaths = col_integer(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )

``` r
# solution-end

# task-begin
# Print results
if (exists("q1")) q1
```

    ## # A tibble: 946 x 5
    ##    region                                hisp_origin   births deaths  year
    ##    <chr>                                 <chr>          <int>  <int> <int>
    ##  1 HHS Region #1  CT, ME, MA, NH, RI, VT Puerto Rican    8483     80  1995
    ##  2 HHS Region #1  CT, ME, MA, NH, RI, VT Puerto Rican    8540     73  1996
    ##  3 HHS Region #1  CT, ME, MA, NH, RI, VT Puerto Rican    8945     85  1997
    ##  4 HHS Region #1  CT, ME, MA, NH, RI, VT Puerto Rican    9373     86  1998
    ##  5 HHS Region #1  CT, ME, MA, NH, RI, VT Central or S…   5095     30  1995
    ##  6 HHS Region #1  CT, ME, MA, NH, RI, VT Central or S…   5073     24  1996
    ##  7 HHS Region #1  CT, ME, MA, NH, RI, VT Central or S…   5254     40  1997
    ##  8 HHS Region #1  CT, ME, MA, NH, RI, VT Central or S…   5693     38  1998
    ##  9 HHS Region #1  CT, ME, MA, NH, RI, VT Non-Hispanic… 135365    687  1995
    ## 10 HHS Region #1  CT, ME, MA, NH, RI, VT Non-Hispanic… 133836    590  1996
    ## # ... with 936 more rows

``` r
# Compare result with answer
if (exists("q1")) compare(answers$q1, q1)
```

    ## TRUE

``` r
# task-end
```

### Create new variables

**q2**

We will focus only on Non-Hispanic White, Non-Hispanic Black, and Hispanic (of any origin) mothers/infants. Create a new variable `race_eth` representing whether the mother was Non-Hispanic White, Non-Hispanic Black, or Hispanic (of any origin).

Save to the new tibble `q2`.

``` r
# solution-begin
q2 <-
  q1 %>%
  mutate(
    race_eth = if_else(
      hisp_origin != "Non-Hispanic Black" & hisp_origin != "Non-Hispanic White",
      "Hispanic", hisp_origin
    )
  ) %>%
  select(-hisp_origin)
# solution-end

# task-begin
# Print results
if (exists("q2")) q2
```

    ## # A tibble: 946 x 5
    ##    region                                births deaths  year race_eth     
    ##    <chr>                                  <int>  <int> <int> <chr>        
    ##  1 HHS Region #1  CT, ME, MA, NH, RI, VT   8483     80  1995 Hispanic     
    ##  2 HHS Region #1  CT, ME, MA, NH, RI, VT   8540     73  1996 Hispanic     
    ##  3 HHS Region #1  CT, ME, MA, NH, RI, VT   8945     85  1997 Hispanic     
    ##  4 HHS Region #1  CT, ME, MA, NH, RI, VT   9373     86  1998 Hispanic     
    ##  5 HHS Region #1  CT, ME, MA, NH, RI, VT   5095     30  1995 Hispanic     
    ##  6 HHS Region #1  CT, ME, MA, NH, RI, VT   5073     24  1996 Hispanic     
    ##  7 HHS Region #1  CT, ME, MA, NH, RI, VT   5254     40  1997 Hispanic     
    ##  8 HHS Region #1  CT, ME, MA, NH, RI, VT   5693     38  1998 Hispanic     
    ##  9 HHS Region #1  CT, ME, MA, NH, RI, VT 135365    687  1995 Non-Hispanic…
    ## 10 HHS Region #1  CT, ME, MA, NH, RI, VT 133836    590  1996 Non-Hispanic…
    ## # ... with 936 more rows

``` r
# Compare result with answer
if (exists("q2")) compare(answers$q2, q2)
```

    ## TRUE

``` r
# task-end
```

**q3.1**

Calculate the infant death rate per 1000 births for each year within each region and race. Save this variable to the new tibble `q3` as `death_rate`.

``` r
# solution-begin
q3 <- 
  q2 %>%
  group_by(region, year, race_eth) %>%
  summarise(
    death_rate = (sum(deaths) / sum(births)) * 1000
  ) %>%
  ungroup()
# solution-end

# task-begin
# Print results
if (exists("q3")) q3
```

    ## # A tibble: 630 x 4
    ##    region                                 year race_eth         death_rate
    ##    <chr>                                 <int> <chr>                 <dbl>
    ##  1 HHS Region #1  CT, ME, MA, NH, RI, VT  1995 Hispanic               8.10
    ##  2 HHS Region #1  CT, ME, MA, NH, RI, VT  1995 Non-Hispanic Bl…      10.7 
    ##  3 HHS Region #1  CT, ME, MA, NH, RI, VT  1995 Non-Hispanic Wh…       5.08
    ##  4 HHS Region #1  CT, ME, MA, NH, RI, VT  1996 Hispanic               7.13
    ##  5 HHS Region #1  CT, ME, MA, NH, RI, VT  1996 Non-Hispanic Bl…      12.8 
    ##  6 HHS Region #1  CT, ME, MA, NH, RI, VT  1996 Non-Hispanic Wh…       4.41
    ##  7 HHS Region #1  CT, ME, MA, NH, RI, VT  1997 Hispanic               8.80
    ##  8 HHS Region #1  CT, ME, MA, NH, RI, VT  1997 Non-Hispanic Bl…      11.1 
    ##  9 HHS Region #1  CT, ME, MA, NH, RI, VT  1997 Non-Hispanic Wh…       4.83
    ## 10 HHS Region #1  CT, ME, MA, NH, RI, VT  1998 Hispanic               8.23
    ## # ... with 620 more rows

``` r
# Compare result with answer
if (exists("q3")) compare(answers$q3, q3)
```

    ## TRUE

``` r
# task-end
```

**q3.2**

Visualize the infant mortality rate for each race/ethnicity in 2015, averaging across all regions. What can you conclude?

``` r
# solution-begin
q3 %>%
  filter(year == "2015") %>%
  group_by(race_eth) %>%
  summarise(
    mean_death_rate = mean(death_rate)
  ) %>%
  mutate(
    race_eth = ordered(
      race_eth,
      levels = c("Non-Hispanic Black", "Hispanic", "Non-Hispanic White"),
      labels = c("Non-Hispanic Black", "Hispanic", "Non-Hispanic White")
    )
  ) %>%
  ggplot(aes(race_eth, mean_death_rate)) +
  geom_hline(aes(yintercept = mean(mean_death_rate)), color = "red") +
  geom_col() +
  annotate(
    "text",
    x = "Non-Hispanic White",
    y = 7.2,
    label = "National mean"
  ) +
  theme_minimal() +
  labs(
    x = NULL,
    y = "Infant mortality rate\n(deaths per 1000 live births)" 
  )
```

![](master_files/figure-markdown_github/unnamed-chunk-6-1.png)

``` r
# solution-end
```

<!-- solution-begin -->
Averaging across the U.S., infant mortality is strikingly higher among Non-Hispanic Black families than among Hispanic or Non-Hispanic White families. Infant mortality in Hispanic and Non-Hispanic White families falls below the national mean across the three race/ethnicities, suggesting that markedly higher rates of infant mortality among Non-Hispanic Black families drive the high national mean. <!-- solution-end -->

**q3.3**

Collapsing across all regions, calculate the proportions of all infant births and infant deaths for each race/ethnicity in 2015. What can you conclude?

``` r
# solution-begin
q3.3 <-
  q2 %>%
  filter(year == "2015") %>%
  group_by(race_eth) %>%
  summarise(
    n_births = sum(births),
    n_deaths = sum(deaths)
  ) %>%
  mutate(
    prop_births = n_births / sum(n_births),
    prop_deaths = n_deaths / sum(n_deaths)
  )

q3.3
```

    ## # A tibble: 3 x 5
    ##   race_eth           n_births n_deaths prop_births prop_deaths
    ##   <chr>                 <int>    <int>       <dbl>       <dbl>
    ## 1 Hispanic             772276     3722       0.221       0.179
    ## 2 Non-Hispanic Black   589047     6623       0.169       0.319
    ## 3 Non-Hispanic White  2130279    10445       0.610       0.502

``` r
# solution-end 

# task-begin
# Print results
if (exists("q3.3")) q3.3
```

    ## # A tibble: 3 x 5
    ##   race_eth           n_births n_deaths prop_births prop_deaths
    ##   <chr>                 <int>    <int>       <dbl>       <dbl>
    ## 1 Hispanic             772276     3722       0.221       0.179
    ## 2 Non-Hispanic Black   589047     6623       0.169       0.319
    ## 3 Non-Hispanic White  2130279    10445       0.610       0.502

``` r
# Compare result with answer
if (exists("q3.3")) compare(answers$q3.3, q3.3)
```

    ## TRUE

``` r
# task-end
```

<!-- solution-begin -->
Non-Hispanic Black mothers gave birth to only 16% of the infants born in 2015; however, their infants made up 31% of the infants who died in 2015. Thus, the infants of Non-Hispanic Black died at highly disproportionate rates. In comparison, the infants of Hispanic and Non-Hispanic White accounted for 25% and 59% of the births in 2015 and 21% and 48% of the infant deaths, respectively. <!-- solution-end -->

### Examine longitudinal trends in infant mortality

**q4**

Create a plot showing the longitudinal trends in infant mortality for each race across the U.S. from 1995-2015 for each region. What can you conclude?

``` r
# solution-begin
q3 %>%
  mutate(
    region = ordered(
      region, 
      labels = region_labels,
      levels = region_order
    ),
    race_eth = ordered(
      race_eth,
      levels = c("Non-Hispanic Black", "Hispanic", "Non-Hispanic White"),
      labels = c("Non-Hispanic Black", "Hispanic", "Non-Hispanic White")
    )
  ) %>%
  ggplot(aes(year, death_rate, group = race_eth)) +
  geom_smooth(method = "loess", color = "grey", se = FALSE) +
  geom_line(aes(color = race_eth)) +
  scale_y_continuous(breaks = seq.int(0, 20, 2)) +
  scale_color_hue() +
  theme(
    legend.title = element_blank()
  ) +
  facet_wrap(~region) +
  theme_minimal() +
  labs(
    color = "Maternal race/ethnicity",
    title = "Black infants die at consistently higher rates",
    subtitle = "Black infant mortality is decreasing in some regions but not in others",
    x = "Year of death",
    y = "Infant mortality rate\n(deaths per 1000 live births)"
  )
```

<img src="master_files/figure-markdown_github/unnamed-chunk-8-1.png" width="100%" />

``` r
# solution-end
```

<!-- solution-begin -->
Overall, mortality is higher for Non-Hispanic Black infants compared to Hispanic and White infants in every region of the country. In regions of the country with small Non-Hispanic Black populations, the longitudinal trends are noisy.

-   However, in several regions, Non-Hispanic Black infant mortality is clearly decreasing. These regions include New York/New Jersey; Delaware, D.C., Maryland, Pennsylvania, Virginia, and West Virginia; Arizona, California, Hawaii, and Nevada.

-   In other regions, Non-Hispanic Black infant mortality does not show clear decreases across the last 20 years, most notably in Southern regions (AL, FL, GA, KY, MS, NC, SC, TN and AK, LA, NM, OK, TX).

-   There are some nonlinear longitudinal patterns that could be accounted for changes in social/healthcare policy:

    -   In the region encompassing AK, ID, OR, and WA there were steep decreases in Non-Hispanic Black infant mortality from 1995-2000 followed by a plateau.

    -   In the region encompassing IA, KS, MO, NE, Non-Hispanic Black infant mortality did not begin decreasing until the year 2000.

<!-- solution-end -->
### Identify riskiest U.S. regions for Non-Hispanic Black infants and mothers

**q5**

Given the evidence that the infants of Non-Hispanic Black mothers are at pronounced risk across U.S. regions and across time, we will now focus on only these infants. Create a new tibble `q5` with data from `Non-Hispanic Black` mothers/infants for years 1995 and 2015 only.

``` r
# solution-begin-
q5 <-
  q3 %>%
  filter(
    race_eth == "Non-Hispanic Black",
    year %in% c(1995, 2015)
  )
# solution-end
```

**q6**

The riskiest regions for Non-Hispanic Black infant mothers may be those that show the least change in infant mortality from 1995 to 2015 (i.e., minimal decreases in the death rate over these 20 years), and where infant mortality remained high in 2015. Using the tibble you created in q5, visualize the change in infant mortality between 1995 and 2015 for each region. What can you conclude?

``` r
# solution-begin-
#pull mean infant mortality in 2015 for reference line
mean_2015 <-
  q5 %>%
  filter(year == "2015") %>%
  summarise(mean_2015 = mean(death_rate)) %>%
  pull(mean_2015)

#plot the data 
q5 %>%
  mutate(
    region = ordered(
      region, 
      labels = region_labels,
      levels = region_order
    ),
    race_eth = ordered(
      race_eth,
      levels = c("Non-Hispanic Black", "Hispanic", "Non-Hispanic White"),
      labels = c("Non-Hispanic Black", "Hispanic", "Non-Hispanic White")
    ),
    region_number = if_else(
      year == 2015,
      str_extract(region, "\\d+"),
      NULL
    )
  ) %>%
  ggplot(aes(year, death_rate, group = region)) +
  geom_hline(yintercept = mean_2015, linetype = "dashed") +
  geom_line(aes(color = region), size = 2, alpha = 1/2) +
  ggrepel::geom_text_repel(aes(label = region_number)) +
  theme_minimal() +
  scale_color_hue() +
  scale_y_continuous(breaks = seq.int(0, 20, 1)) +
  labs(
    color = "U.S Region",
    title = "Change in Non-Hispanic Black infant mortality from 1995-2015",
    x = "Year of death",
    y = "Infant mortality rate\n(deaths per 1000 live births)"
  )
```

    ## Warning: Removed 10 rows containing missing values (geom_text_repel).

<img src="master_files/figure-markdown_github/unnamed-chunk-10-1.png" width="100%" />

``` r
# solution-end
```

<!-- solution-begin -->
Based on showing minimal decreases in infant mortality between 1995 and 2015, the riskiest regions include the regions encompassing the south (4: AL, FL, GA, KY, MS, NC, SC and 6: AR, LA, NM, OK, TX). Both southern regions show flatter negative slopes from 1995 to 2015 and are above the 2015 mean infant mortality for Non-Hispanic Black families. Similarly, the "heartland" region of the Midwest, encompassing IA, KS, MO, NE, fits this risky profile. Rates of infant mortality in 2015 remained high in other regions, including region 5 (Midwest) and region 3 (Mid Atlantic), but these regions have shown larger decreases in Non-Hispanic Black infant mortality since 1995, suggesting the situation is improving.

To note, infant mortality in Non-Hispanic Black families is a critical public health problem across **all** regions, but some regions show particularly concerning trends. Given that the "riskiest regions" also have larger Black populations, public health initiatives to prevent racial disparities in infant mortality should prioritize these regions.

<!-- solution-end -->
Part 2: Examine infant mortality at the U.S. state-level
--------------------------------------------------------

Now that we have gained an understanding of longitudinal trends in infant mortality by U.S. region, we will examine state-level data in order to identify the states that show the most troubling statistics.

### Download and tidy infant mortality data at the U.S. state-level

Because the CDC suppresses sub-national data representing zero to nine (0-9) deaths or births, investigating single years of data at the state-level is not possible. Instead, we will focus on totals during the first five years for which the CDC has data (1995-2000) and the latest five year years (2010-2015).

Return to <https://wonder.cdc.gov/lbd.html> and download the following datasets, grouping by `State` and `Hispanic Origin`. Select only Non-Hispanic Black and Non-Hispanic White. *Do not* group by year.

1.  Infant mortality data from 1995-1998.
2.  Infant mortality data from 1999-2000.
3.  Infant mortality data from 2010-2015.

In addition, read in the file with the state boundaries (`file_state_boundaries` available on Box as cb\_2015\_us\_state\_20m\_sf.rds"). Use `st_transform` to apply the Albers equal area projection coordinate reference system.

``` r
# solution-begin-
states <-
  read_rds(file_state_boundaries) %>%
  filter(NAME != "ALaska", NAME != "Hawaii") %>%
  st_transform(US_ALBERS) %>%
  mutate(
    state_fips = as.integer(STATEFP)
  )
# solution-end
```

**q7.1**

Read in the CDC wonder files spanning 1995-2000 into a single dataset using a `map()` function. Read in the 2010-2015 file separately.

``` r
# solution-begin-

##read in state data from 1995-2000
state_early_years <- 
  early_years_infant_mortality_files %>% 
  map_dfr(read_tsv) %>% 
  filter(
    is.na(Notes)
    ) %>%
  select(
    state = State,
    state_fips = `State Code`,
    hisp_origin = `Hispanic Origin`,
    births = Births,
    deaths = Deaths
  )
```

    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   State = col_character(),
    ##   `State Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_integer(),
    ##   Deaths = col_character(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )
    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   State = col_character(),
    ##   `State Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_integer(),
    ##   Deaths = col_character(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )

``` r
##read in state data from 2010-2015
state_latest_years <- 
  read_tsv(
    state_10_15_infant_mortality_file_path
    ) %>%
  filter(
    is.na(Notes)
    ) %>%
  select(
    state = State,
    state_fips = `State Code`,
    hisp_origin = `Hispanic Origin`,
    births = Births,
    deaths = Deaths
  ) 
```

    ## Parsed with column specification:
    ## cols(
    ##   Notes = col_character(),
    ##   State = col_character(),
    ##   `State Code` = col_character(),
    ##   `Hispanic Origin` = col_character(),
    ##   `Hispanic Origin Code` = col_integer(),
    ##   Deaths = col_character(),
    ##   Births = col_integer(),
    ##   `Death Rate` = col_character()
    ## )

``` r
# solution-end
```

**q7.2**

For each dataset, how many death and birth cells are suppressed for Non-Hispanic Black mothers?

``` r
# solution-begin-
state_early_years %>%
  group_by(hisp_origin) %>%
  mutate(
    births_suppresed = if_else(
      births == "Suppressed",
      1, 0
    ),
    deaths_suppresed = if_else(
      deaths == "Suppressed",
      1, 0
    )
  ) %>%
  summarise(
    n_births_suppressed = sum(births_suppresed),
    n_deaths_suppressed = sum(deaths_suppresed)
  )
```

    ## # A tibble: 2 x 3
    ##   hisp_origin        n_births_suppressed n_deaths_suppressed
    ##   <chr>                            <dbl>               <dbl>
    ## 1 Non-Hispanic Black                   0                  18
    ## 2 Non-Hispanic White                   0                   0

``` r
# solution-end
```

``` r
# solution-begin-
state_latest_years %>%
  group_by(hisp_origin) %>%
  mutate(
    births_suppresed = if_else(
      births == "Suppressed",
      1, 0
    ),
    deaths_suppresed = if_else(
      deaths == "Suppressed",
      1, 0
    )
  ) %>%
  summarise(
    n_births_suppressed = sum(births_suppresed),
    n_deaths_suppressed = sum(deaths_suppresed)
  )
```

    ## # A tibble: 2 x 3
    ##   hisp_origin        n_births_suppressed n_deaths_suppressed
    ##   <chr>                            <dbl>               <dbl>
    ## 1 Non-Hispanic Black                   0                   4
    ## 2 Non-Hispanic White                   0                   0

``` r
# solution-end
```

<!-- solution-begin -->
There are 18 and 4 cells counting deaths suppressed for Non-Hispanic Black mothers in the data spanning 1995-2000 and 2005-2010, respectively. This amount of missing data is small enough that it is appropriate to proceed. <!-- solution-end -->

**q7.3**

Tidy the state-level infant mortality data as follows:

-   Filter rows containing “Suppressed" data from both datasets.
-   Change columns containing numbers to integers.
-   Compute the death rate for each racial group and state.

``` r
# solution-begin-
state_early_years <-
  state_early_years %>%
  filter(deaths != "Suppressed") %>%
  mutate(
    deaths = as.integer(deaths),
    births = as.integer(births),
    state_fips = as.integer(state_fips)
  ) %>% 
  group_by(state, state_fips, hisp_origin) %>%
  summarise(
    death_rate = (sum(deaths) / sum(births)) * 1000
  ) 

state_latest_years <-
  state_latest_years %>%
  filter(deaths != "Suppressed") %>%
  mutate(
    deaths = as.integer(deaths),
    births = as.integer(births),
    state_fips = as.integer(state_fips)
  ) %>% 
  group_by(state, state_fips, hisp_origin) %>%
  summarise(
    death_rate = (sum(deaths) / sum(births)) * 1000
  ) 
# solution-end
```

**q7.4**

Next, calculate a "disparity" variable for each dataset. To do so, for each state, calculate the proportional difference between the death rate for infants of Non-Hispanic Black versus infants of Non-Hispanic White mothers as follows:

    disparity_early = black_death_rate_early / white_death_rate_early 

    disparity_latest = black_death_rate_latest / white_death_rate_latest

These values indicate how many times higher the infant mortality rate is for infants of Black mothers relative to infants of White mothers.

Hint: `spread()` is useful here

``` r
# solution-begin-
state_early_years_wf <-
  state_early_years %>% 
  spread(hisp_origin, death_rate) %>%
  rename(
    "black_death_rate_early" = `Non-Hispanic Black`,
    "white_death_rate_early" = `Non-Hispanic White`
  ) %>%
  mutate(
    disparity_early = black_death_rate_early / white_death_rate_early
  ) %>% 
  ungroup()

state_latest_years_wf <-
  state_latest_years %>% 
  spread(hisp_origin, death_rate) %>%
  rename(
    "black_death_rate_latest" = `Non-Hispanic Black`,
    "white_death_rate_latest" = `Non-Hispanic White`
  ) %>%
  mutate(
    disparity_latest = black_death_rate_latest  / white_death_rate_latest
  ) %>% 
  ungroup()
# solution-end
```

**q7.5**

Finally, join the two state-level infant mortality datasets as well as the dataset with the state boundaries. Remove Alaska and Hawaii. Calculate the difference in infant mortality for Non-Hispanic Black mothers between 1995-2000 and 2010-2015:

    black_diff_rate = black_death_rate_latest - black_death_rate_early

``` r
# solution-begin-
state_infant_mortality <-
  state_early_years_wf %>% 
  left_join(
    state_latest_years_wf, by = "state_fips"
  ) %>% 
  left_join(states, state_infant_mortality, by = "state_fips") %>% 
  filter(state.x != "Alaska", state.x != "Hawaii") %>% 
  mutate(
    black_diff_rate = black_death_rate_latest - black_death_rate_early
  ) 
# solution-end
```

### Map infant mortality data at the U.S. state-level

**q8.1**

We will now examine racial disparities at the state-level. Specifically, we will produce several maps to clarify which states show especially problematic trends in infant mortality based on the risk indices we have calculated.

First, create two separate maps showing the infant mortality rate by state in 2010-2015 for:

1.  Non-Hispanic White mothers
2.  Non-Hispanic Black mothers

What can you conclude?

``` r
# solution-begin-

#map white infant mortality in 2010-2015

white_infant_mortality_map <- 
  state_infant_mortality %>%
  ggplot() +
  geom_sf(aes(fill = white_death_rate_latest), size = 0.0) +
  geom_sf(data = states, color = "white", fill = NA, size = 0.4) +
  scale_fill_gradientn(
   breaks = c(3, 4, 5, 6, 7),
   colors = rate_colors_white,
   values = rate_values_white,
   na.value = na_color
  ) +
  coord_sf(
    datum = NA, 
    xlim = c(-2500000, 2500000),
    ylim = c(-1200000, 1750000)
    ) +
  guides(
    fill = 
      guide_colorbar(
        title = NULL,
        barwidth = 9,
        barheight = 0.25,
        nbin = 5,
        raster = FALSE,
        ticks = FALSE,
        direction = "horizontal"
      )
  ) +
  theme_void() +
  theme(
    legend.position = "bottom",
    legend.text = element_text(size = 6),
    plot.title = element_text(hjust = .5),
    plot.subtitle = element_text(hjust = .5)
  ) +
  labs(
    title = "Non-Hispanic White Infant Mortality Rates during 2010-2015",
    subtitle = "(deaths per 1000 born)"
  )
# solution-end
```

``` r
# solution-begin-
#map black infant mortality in 2010-2015

black_infant_mortality_map <- 
  state_infant_mortality %>%
  ggplot() +
  geom_sf(aes(fill = black_death_rate_latest), size = 0.0) +
  geom_sf(data = states, color = "white", fill = NA, size = 0.4) +
  scale_fill_gradientn(
    limits = c(7, 15),
    breaks = c(7, 9, 11, 13, 15),
    labels = c("7", "9", "11", "13", ">13"),
    colors = rate_colors_black,
    values = rate_values_black,
    na.value = na_color
  ) +
  coord_sf(
    datum = NA, 
    xlim = c(-2500000, 2500000),
    ylim = c(-1200000, 1750000)
    ) +
  guides(
    fill = 
      guide_colorbar(
        title = NULL,
        barwidth = 9,
        barheight = 0.25,
        nbin = 5,
        raster = FALSE,
        ticks = FALSE,
        direction = "horizontal"
      )
  ) +
  theme_void() +
  theme(
    legend.position = "bottom",
    legend.text = element_text(size = 6),
    plot.title = element_text(hjust = .5),
    plot.subtitle = element_text(hjust = .5)
  ) +
  labs(
    title = "Non-Hispanic Black Infant Mortality Rates during 2010-2015",
    subtitle = "(deaths per 1000 born)"
  )

# solution-end
```

``` r
# solution-begin-
white_infant_mortality_map
```

![](master_files/figure-markdown_github/unnamed-chunk-20-1.png)

``` r
black_infant_mortality_map
```

![](master_files/figure-markdown_github/unnamed-chunk-20-2.png)

``` r
# solution-end
```

<!-- solution-begin -->
As was evident in Part I, the infant mortality rate is considerably lower for Non-Hispanic White families than for Non-Hispanic Black families in every state. In fact, the distributions for this variable barely overlap for the two races.

For White Non-Hispanic families, the infant mortality rate is lowest in most of New England (MA, CT, NH), New York and New Jersey, Minnesota, California, and Washington. Riskier states for White families include Maine, Indiana, Oklahoma, and much of the South, including Kentucky, Mississippi, Alabama, and Arkansas. However, the riskiest place for White families in terms of infant mortality is West Virginia, with ~7 deaths out of every 1000 births.

For Black Non-Hispanic families, North Dakota has the lowest infant mortality rate, with ~7 deaths out of every 1000 births. Thus, the lowest state-level rate for Black families is equivalent to the highest state-level rate of White families. For Black families, riskier states include much of the South (Alabama, Mississippi, Louisiana, South Carolina), the Industrial Midwest (Ohio, Indiana, Michigan, Illinois), the "Heartland" states of Missouri, Oklahoma, and Kansas. The riskiest region is Wisconsin, where &gt;13 infants die for every 1000 that are born.

<!-- solution-end -->
**q8.2**

Next, create a map showing the *change* in infant mortality for Non-Hispanic Black mothers between the two time periods (i.e., fill with the `black_diff_rate` variable). What can you conclude?

``` r
# solution-begin-
state_infant_mortality %>%
  ggplot() +
  geom_sf(aes(fill = black_diff_rate), size = 0.0) +
  geom_sf(data = states, color = "white", fill = NA, size = 0.4) +
  scale_fill_gradientn(
   breaks = c(-7, -5, -3, -1, 1),
   colors = rate_change_colors,
   na.value = na_color
  ) +
  coord_sf(
    datum = NA, 
    xlim = c(-2500000, 2500000),
    ylim = c(-1200000, 1750000)
    ) +
  guides(
    fill = 
      guide_colorbar(
        title = NULL,
        barwidth = 9,
        barheight = 0.25,
        nbin = 5,
        raster = FALSE,
        ticks = FALSE,
        direction = "horizontal"
      )
  ) +
  theme_void() +
  theme(
    legend.position = "bottom",
    legend.text = element_text(size = 6),
    plot.title = element_text(hjust = .5),
    plot.subtitle = element_text(hjust = .5)
  )  +
  labs(
    title = "Change in Infant Mortality rate for Non-Hispanic Black Infants",
    subtitle = "(deaths per 1000 born)"
  )
```

![](master_files/figure-markdown_github/unnamed-chunk-21-1.png)

``` r
# solution-end
```

<!-- solution-begin -->
In terms of change in infant mortality among Non-Hispanic Black families, most states show decreases from the period of 1995-2000 to 2010-2015. Areas of greatest decrease include some of the areas that show the highest 2010-2015 rates, suggesting that these areas, although risky, are improving. Such states include Illinois and Michigan where 3-5 fewer infants die per every 1000 that are born. Unfortunately, as was evident in Part I, there are some places that have decreased minimally since 1995-2000 and remain high. These states include much of the South (MS, AL, LA) and other areas of the Industrial Midwest (WI, OH), as well as areas of the Heartland (KS, OC). In one state, Utah, Black infant mortality has actually increased since 1995-2000, such that ~1 more infant dies per every 1000 born.

<!-- solution-end -->
**q8.3**

Next, create a map showing the disparity in infant mortality for Non-Hispanic Black mothers versus Non-Hispanic white mothers during 2005-2015 (i.e., fill with the `disparity_latest` variable).

What can you conclude?

``` r
# solution-begin-
state_infant_mortality %>%
  ggplot() +
  geom_sf(aes(fill = disparity_latest), size = 0.0) +
  geom_sf(data = states, color = "white", fill = NA, size = 0.4) +
  scale_fill_gradientn(
    limits = c(1, 5),
    labels = c("1x", "2x", "3x", "4x", ">4x"),
    breaks = c(1, 2, 3, 4, 5),
    colors = disparity_colors,
    values = disparity_values,
    na.value = na_color
  ) +
  coord_sf(
    datum = NA, 
    xlim = c(-2500000, 2500000),
    ylim = c(-1200000, 1750000)
    ) +
  guides(
    fill = 
      guide_colorbar(
        title = NULL,
        barwidth = 9,
        barheight = 0.25,
        nbin = 5,
        raster = FALSE,
        ticks = FALSE,
        direction = "horizontal"
      )
  ) +
  theme_void() +
  theme(
    legend.position = "bottom",
    legend.text = element_text(size = 6),
    plot.title = element_text(hjust = .5),
    plot.subtitle = element_text(hjust = .5)
  ) +
  labs(
    title = "Black-to-White Disparity in Infant Mortality during 2010-2015",
    subtitle = "(proportional difference in mortality rate)"
  )
```

![](master_files/figure-markdown_github/unnamed-chunk-22-1.png)

``` r
# solution-end
```

<!-- solution-begin -->
Some of the states that evidence the highest Black infant mortality rate in 2010-2015 also show some of the largest disparities between the black and white infant mortality rates, including much of the Industrial Midwest (WI, IL, MI). For example, in Wisconsin, the Black infant mortality rate is 3-4x higher than the White infant mortality rate.

However, there are other states that do not rank among the highest in terms of Black infant mortality but evidence some of the highest disparities in Black-to-White infant mortality. These states include much of the Northeast (NJ, CT, NH, MD,VA) as well as Utah and California. This is somewhat surprising given that most of these states are known for having more progressive policies that may mitigate the systemic racial bias that may cause disparities. Of course, these ratios depend on the level of White infant mortality. Thus, areas with greater disparities include those where White infants do especially well.

Taken together, these state-level analyses indicate that the Industrial Midwest shows the most the troubling trends in Black infant mortality. In summary, states in this region rank among the highest in terms of recent Black infant mortality rates, show minimal decreases in Black infant mortality over the past 15-20 years, and have high racial disparities in black infant mortality.

<!-- solution-end -->
Answers
-------

Save answers.

``` r
if (SAVE_ANSWERS) {
  ls(pattern = "^q[1-9][0-9]*(\\.[1-9][0-9]*)*$") %>%
  str_sort(numeric = TRUE) %>% 
  set_names() %>% 
  map(get) %>%
  discard(is.ggplot) %>%
  write_rds(file_answers)
}
```

<!-- solution-end -->
