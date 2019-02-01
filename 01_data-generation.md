
``` r
library(here)
```

    ## here() starts at /Users/wickhamc/Documents/Projects/or-house-vis/gender

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.2.5
    ## ✔ tibble  2.0.1     ✔ dplyr   0.7.8
    ## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## Warning: package 'tibble' was built under R version 3.5.2

    ## ── Conflicts ───────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(glue)
```

    ## 
    ## Attaching package: 'glue'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     collapse

``` r
# devtools::install_github("or-house-vis/history")
library(history) 
```

Gender overall
==============

Compute proportions of each gender:

``` r
gender_overall <- house_reps_regular %>% 
  group_by(session_year, gender) %>% 
  count() %>%
  ungroup() %>% 
  complete(session_year, gender, fill = list(n = 0)) %>% 
  group_by(session_year) %>% 
  mutate(prop = n/sum(n),
    gender = gender %>% factor(levels = c("Male", "Female"))
    ) %>% 
  ungroup() 
```

Add captions

``` r
overall_captions <- gender_overall %>% 
  filter(gender == "Female") %>% 
  select(session_year, n, prop) %>% 
  mutate(caption = 
      glue('<strong>{ session_year }</strong><br />{ round(prop, 2) * 100 }% ({ n } seats) held by females')
    ) %>% 
  select(session_year, caption)
```

Add captions and make sure `session_year` is a date

``` r
gender_overall <- gender_overall %>%  
  left_join(overall_captions) %>% 
  mutate(
    session_year = ISOdate(year = session_year, 
      month = 1, day = 1, tz = "utc")
  )
```

    ## Joining, by = "session_year"

Add some rows for annotation

``` r
anno <- tribble(
  ~ gender, ~ session_year, ~ prop,
  "Male"  ,           1865,    0.95,
  "Female",           2013,    0.05
) %>% 
  mutate(
    session_year = ISOdate(year = session_year, 
      month = 1, day = 1, tz = "utc")
  )
anno$data = "annotation"
```

``` r
gender_overall_captioned <- 
  gender_overall %>% 
  mutate(data = "data") %>% 
  bind_rows(anno)
```

    ## Warning in bind_rows_(x, .id): Vectorizing 'glue' elements may not preserve
    ## their attributes

    ## Warning in bind_rows_(x, .id): binding factor and character vector,
    ## coercing into character vector

    ## Warning in bind_rows_(x, .id): binding character and factor vector,
    ## coercing into character vector

Examine changes

``` r
library(daff)
old <- read_csv(here("docs", "data", "vega-gender-overall.csv"))
diffs <- diff_data(gender_overall_captioned, old, ordered = FALSE)
render_diff(diffs)
```

``` r
gender_overall_captioned %>% 
  write_csv(here("docs", "data", "vega-gender-overall.csv"))
```

Gender by party
===============

``` r
gender_by_party <- house_reps_regular %>% 
  group_by(session_year, party, gender) %>% 
  count() %>%
  ungroup() %>% 
  complete(session_year, party, gender, fill = list(n = 0)) %>% 
  group_by(session_year, party) %>% 
  mutate(
    total = sum(n),
    prop = n/total
    ) %>% 
  filter(party %in% c("Democrat", "Republican")) %>% 
  filter(gender == "Female") %>% 
  ungroup() 
```

Add captions

``` r
party_captions <- gender_by_party %>% 
  select(session_year, n, total, prop, party) %>% 
  gather("type", "value", -session_year, -party) %>% 
  unite("party_type", party, type) %>% 
  spread(party_type, value) %>% 
  mutate(caption = glue(
    "<strong>{ session_year }</strong><br />{round(Democrat_prop, 2)*100}%  of Democrats ({ Democrat_n }/{ Democrat_total } seats) and <br />{ round(Republican_prop, 2)*100 }%  of Republicans ({ Republican_n }/{ Republican_total } seats) are female.")
  ) %>% 
  select(session_year, caption)
```

Weird captions

``` r
party_captions <- party_captions %>% 
  mutate(
    caption = ifelse(session_year %in% c(1887, 1889, 1891),
      glue("<strong>{session_year}</strong> Party affiliations unknown."),
      caption
    )
  )
```

Party annotations

``` r
anno_party <- tribble(
  ~ session_year, ~ party, ~ prop,
            2018,     "Democrat",    0.58,
            2018,     "Republican",    0.05
) %>% 
  mutate(
    data = "annotation"
  )
```

``` r
gender_by_party_captioned <- gender_by_party %>% 
  mutate(data = "data") %>% 
  left_join(party_captions) %>% 
  bind_rows(anno_party) %>% 
  mutate(
    session_year = ISOdate(year = session_year, 
      month = 1, day = 1, tz = "utc")
  )
```

    ## Joining, by = "session_year"

Examine changes

``` r
library(daff)
old <- read_csv(here("docs", "data", "vega-gender-by-party.csv"))
diffs <- diff_data(gender_by_party_captioned, old, ordered = FALSE)
render_diff(diffs)
```

``` r
gender_by_party_captioned %>% 
  write_csv(here("docs", "data", "vega-gender-by-party.csv"))
```
