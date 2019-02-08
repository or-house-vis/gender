
``` r
library(history)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.3.0
    ## ✔ tibble  2.0.1     ✔ dplyr   0.7.8
    ## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
    ## ✔ readr   1.3.1     ✔ forcats 0.3.0

    ## Warning: package 'tibble' was built under R version 3.5.2

    ## Warning: package 'purrr' was built under R version 3.5.2

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(jsonlite)
```

    ## 
    ## Attaching package: 'jsonlite'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     flatten

``` r
library(here)
```

    ## here() starts at /Users/wickhamc/Documents/Projects/or-house-vis/gender

``` r
end_year <- 2019
```

Gender Overall
--------------

``` r
spec_overall <- list(
  `$schema` = "https://vega.github.io/schema/vega-lite/v3.json", 
  data = list(url = "data/vega-gender-overall.csv"), 
  description = "Gender breakdown in house over time",
  config = list(style = list(cell = list(stroke = "transparent"))))
```

``` r
gender_plot <- list(
  encoding = list(
    x = list(
      field = "session_year",
      type = "temporal",
      timeUnit = "year",
      axis = list(
        title = "",
        values = c(1860, seq(1875, end_year, by = 25), end_year),
        grid = FALSE
      )
    ),
    y = list(
      field = "prop",
      type = "quantitative",
      scale = list(
        domain = list(0, 1)
      ),
      axis = list(
        values = seq(0, 1, .25),
        format = "%",
        title = "Percentage of house seats",
        ticks = FALSE,
        labelPadding = 5
      )
    ),
    color = list(
      field = "gender",
      type = "nominal",
      scale = list(
        domain = c("Female", "Male"), 
        range = c("#6F50A1", "#AAC8BD")
      ),
      legend = NULL
    ),
    opacity = list(
      value = 0.8
    ),
    order = list(
      field = "gender_order",
      type = "nominal"
    ),
    tooltip = list(
      value = NULL
    )
  ),
  layer = list(
    list(
      transform = list(
        list(
          filter = list(field = "data", equal = "data")
        ),
        list(
          calculate = "if(datum.gender == 'Female', 0, 1)",
          as = "gender_order"
        )
      ),
      mark = "area"
    ),
    list(
      transform = list(
        list(
          filter = list(field = "data", equal = "data")
        ),
        list(
          calculate = "if(datum.gender == 'Female', 0, 1)",
          as = "gender_order"
        )
      ),
      mark = "area",
      encoding = list(
        opacity = list(
          value = 0
        ),
        tooltip = list(
          field = "caption",
          type = "nominal"
        )
      )
    ),
    list(
      transform = list(
        list(
          filter = list(field = "data", equal = "annotation")
        ),
        list(
          filter = list(field = "gender", equal = "Male")
        )
      ),
      mark = list(
        type = "text",
        size = 20,
        align = "left",
        baseline = "top"
      ),
      encoding = list(
        text = list(
          value = "Male"
        ),
        color = list(
          value = "black"
        ),
        opacity = list(
          value = 0.5
        )
      )
    ),
    list(
      transform = list(
        list(
          filter = list(field = "data", equal = "annotation")
        ),
        list(
          filter = list(field = "gender", equal = "Female")
        )
      ),
      mark = list(
        type = "text",
        size = 20,
        align = "right",
        baseline = "bottom"
      ),
      encoding = list(
        text = list(
          value = "Female"
        ),
        color = list(
          value = "white"
        ),
        opacity = list(
          value = 0.5
        )
      )
    )
  )
)  


spec_overall %>% c(list(autosize = list(type = "fit")), gender_plot) %>% 
  write_json(here("docs", "spec-overall.json"), 
  auto_unbox = TRUE, pretty = TRUE, null = "null")
```

Gender by party
---------------

``` r
spec_party <- list(
    `$schema` = "https://vega.github.io/schema/vega-lite/v3.json", 
    data = list(url = "data/vega-gender-by-party.csv"), 
    description = "Gender breakdown by party in house over time",
    config = list(style = list(cell = list(stroke = "transparent"))))
```

``` r
gender_by_party_plot <- list(
  encoding = list(
    x = list(
      field = "session_year",
      type = "temporal",
      timeUnit = "year",
      axis = list(
        title = "",
        values = c(1860, seq(1875, end_year, by = 25), end_year),
        grid = FALSE
      )
    ),
    y = list(
      field = "prop",
      type = "quantitative",
      scale = list(
        domain = list(0, 1)
      ),
      axis = list(
        values = seq(0, 1, .25),
        format = "%",
        title = "Percentage of party seats",
        ticks = FALSE,
        labelPadding = 5
      )
    ),
    color = list(
      field = "party",
      type = "nominal",
      scale = list(
        domain = list("Democrat", "Republican"),
        range = list("#377EB8", "#E41A1C")
      ),
      legend = NULL
    ),
    tooltip = list(
      value = NULL
    )
  ),
  layer = list(
    list(
      transform = list(
        list(filter = list(
          field = "data", equal = "data"
        ))
      ),
      mark = "line"
    ),
    list(
      transform = list(
        list(filter = list(
          field = "data", equal = "data"
        )),
        list(filter = list(
          field = "party", equal = "Democrat"
        )),
        list(
          calculate = "1",
          as = "one"
        )
      ),
      mark = list(
        type = "bar"
      ),
      encoding = list(
        y = list(
          field = "one",
          type = "quantitative"
        ),
        opacity = list(
          value = 0
        ),
        tooltip = list(
          field = "caption"
        )
      )
    ),
    list(
      transform = list(
        list(
          filter = list(
            field = "data",
            equal = "annotation"
          )
        )
      ),
      mark = list(
        type = "text",
        align = "right",
        dx = 0
      ),
      encoding = list(
        text = list(
          field = "party",
          type = "nominal"
        ),
        y = list(
          stack = NULL,
          field = "prop",
          type = "quantitative"
        )
      )
    )
  )
)  

spec_party %>% 
  c(list(autosize = list(type = "fit")),
    gender_by_party_plot) %>% 
  write_json(here("docs", "spec-by-party.json"), 
  auto_unbox = TRUE, pretty = TRUE, null = "null")
```
