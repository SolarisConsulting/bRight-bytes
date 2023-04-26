# bRight bytes 2 - tables for labelled data

## <img src="img/core.png" alt="element" width="20"/>  learning goals

In this bRight byte, we will tackle the following questions: 
 * How do we read in our saved data set?
 * How do we format our data for table creation?
 * How do we create usable tables?
 * How do we edit table options?
 * How do we combine multiple questions into one table?
 * How do we export our tables?
 
We will primarily be using the tidyverse function `mutate()` to transform variables and `across()` to apply those transformations to multiple variables.  We will use a few selection helpers such as `contains()`, `starts_with()`, and `ends_with` in conjunction with the `.names` argument of `across()` to apply naming conventions and select groups of variables. `labelled()` and `rec()` will assist in labeling our variables and re-coding our labelled variables.

 
## <img src="img/core.png" alt="element" width="20"/>  reading in our data

First, lets install and load the required packages
 * tidyverse for data wrangling
 * haven, labelled, and sjmisc for labelled data transformation
 * gt and gtsummary for summaries and initial table creation
 * flextable for table formatting and editing

```{r}
install.packages(c("tidyverse", "haven", "sjmisc", "labelled"))

library(tidyverse)
library(haven)
library(sjmisc)
library(labelled)

install.packages(c("gt", "gtsummary", "flextable", "sysfonts"))

library(gt)
library(gtsummary)
library(flextable)
library(sysfonts)

library(tinytex)
```

Let's create some test data. We will create both continuous and categorical variables. We've defined the object `n_sample` as the number of cases, 100, in our data set. We create the data set using `tibble()` to work with tidyverse package. First, we use `seq()` to create a unique, sequential id variable, the length of the sample `n_sample`. The continuous variables are created using `sample.int()` that samples integers that will represent questionnaire responses to a Likert scale. Our character variables are created using `sample()`. Remember to specify `replace = TRUE` in order to sample with replacement. For a deeper explanation of these transformations, see [bRight byte 1 - Working with Labelled Data](bRight-bytes-1_labelleddata.md).

```{r}
n_sample <- 100

dat <- tibble(
  id = seq(1, n_sample, 1),
  q1 = sample.int(5, n_sample, replace = TRUE),
  q2 = sample.int(5, n_sample, replace = TRUE),
  q3 = sample.int(5, n_sample, replace = TRUE),
  group = sample.int(4, n_sample, replace = TRUE),
  sa_1 = sample(c("Selected", "Not selected"), n_sample, replace = TRUE),
  sa_2 = sample(c("Selected", "Not selected"), n_sample, replace = TRUE),
  cont = sample.int(100, n_sample, replace = TRUE),
  prepost = sample(c("Pre","Post"), n_sample, replace = TRUE)
)

# create scale variable from sub-scale items
dat <- dat %>%
  # rowMeans(.[,2:4], ) specifies the 2nd through 4th column in the dataset
  # round sets the created variable to a single decimal point
  mutate(scale = round(rowMeans(.[,2:4], na.rm = TRUE), digits = 1))

# specify the labels for your Likert scales
da5_labels <- c("Strongly disagree" = 1,
                "Somewhat disagree" = 2,
                "Neither disagree nor agree" = 3,
                "Somewhat agree" = 4,
                "Strongly agree" = 5)

sa_labels = c("Not selected" = 1,
             "Selected" = 2)

group_labels <- c("Group 1" = 1,
                  "Group 2" = 2,
                  "Group 3" = 3,
                  "Group 4" = 4)

dat_labelled <- dat %>%
  mutate(across(contains("q"), ~labelled(.x, labels = da5_labels))) %>%
  mutate(group = labelled(group, labels = group_labels)) %>%
  mutate(across(contains("sa"), ~factor(.x, levels = c("Not selected", "Selected")))) %>%
  mutate(across(contains("sa"), ~labelled(.x, labels = sa_labels))) %>%
  mutate(prepost = factor(prepost, levels = c("Pre", "Post"))) %>%
  mutate(across("prepost", ~labelled(.x, labels = c(Pre = 1, Post = 2))))
```

 
## <img src="img/core.png" alt="element" width="20"/>  factor transformation

The `gtsummary` package works particularly well with factor variables. We will transform our categorical variables to factors using `mutate()` to apply `labelled::to_factor()` to the factor variables with `across(!all_of(nonfactors),)`, using our defined variable labels as the factor levels. The use of `!all_of()` within `across()` means we want the transformations applied to none of the columns (`!` within `all_of()`) indicated by `nonfactors`.

```{r}
# define non-factor variables
nonfactors <- c("scale", "cont")

# change variables to factor to more easily work with tables
dat_factor <- dat_labelled %>% 
  mutate(across(!all_of(nonfactors), ~labelled::to_factor(., levels = "labels", sort_levels = "none")))
```

 
## <img src="img/core.png" alt="element" width="20"/>  categorical variable frequency table

First we will look at a frequency table for a single categorical variable, `group`. The table produced has a header that provides a title and the number of cases, the variable's name and its response categories, frequencies and percentages of the responses, and a footer providing further detail about the statistic column. 

```{r}
dat_factor %>%
  select(group) %>%
  tbl_summary()
```

`tbl_summary()` has a number of options that allow us to edit or specify particular parts of the table. The code below does the following: 
 * `statistic = list(all_categorical() ~ "{p}% ({n}")` allows us to specify the statistics produced for all the categorical variables
 * `missing = "no"` indicates we do not want missing values displayed
 * `sort = list(all_categorical() ~ "frequency")` indicates that we want to sort each categorical variable by response frequencies (`sort = NULL does no sorting`, and `sort = list(all_categorical() ~ "alphanumeric")` sorts alphanumerically)
 * `percent = "column"` is the default, however you can specify if you want to display "row" or "cell" percentages

```{r}
# specifying table options
ft_group <- tbl_summary(group_table, 
                       # set categorical variable statistics {percent}% and ({number}) that show in the summary column
                       statistic = list(all_categorical() ~ "{p}% ({n})"),
                       # do not show missing values
                       missing = "no",
                       # sort the table by frequency
                       sort = list(all_categorical() ~ "frequency")) %>%
  # set the header labels
  modify_header(label ~ "", stat_0 ~ "% (N)") %>% 
  # remove the footnote
  modify_footnote(update = everything() ~ NA) %>%
  # use flextable::as_flex_table() to turn the table to a flextable object which can be edited there for display
  as_flex_table()

# edit the group table
ft_group <- ft_group %>% 
  # reduce padding around the cells (default is 5)
  padding(padding.bottom = 3.5, padding.top = 3.5, part = "body") %>% 
  # auto fit the table
  set_table_properties(layout = "autofit") %>%
  # set font
  font(fontname = "Montserrat", part = "all")

# display group in the report - table 1
ft_group
```


`gtsummary()` includes a number of pre-made themes as well as the ability to create and edit your own themes.

```{r}
# set significant digits for table percents
mytheme <- list(
  "pkgwide-fn:pvalue_fun" = function(x) style_pvalue(x, digits = 2),
  "pkgwide-fn:prependpvalue_fun" = function(x) style_pvalue(x, digits = 2, prepend_p = TRUE),
  "tbl_summary-fn:percent_fun" = function(x) sprintf("%.0f", 100 * x)
)

# alter your theme using set_gtsummary_theme() or 
set_gtsummary_theme(mytheme)

# use reset_gtsummary_theme() to reset the themes to their defaults
reset_gtsummary_theme()
```

```{r}

```



> 2023 Solaris Consulting Group, LLC. info@solarisconsultinggroup.com


