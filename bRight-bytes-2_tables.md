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


```{r}
# select the variable to be summarized
group_table <- dat_factor %>% 
  select(group)

# basic table  
tbl_summary(group_table)

# specifying table options
ft_group <- tbl_summary(group_table, 
                       # set categorical variable statistics {percent}% and ({number}) that show in the summary column
                       statistic = list(all_categorical() ~ "{p}% ({n})"),
                       # do not show missing values
                       missing = "no",
                       # sort the table by frequency
                       sort = list(everything() ~ "frequency")) %>%
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

 
## <img src="img/core.png" alt="element" width="20"/>  labelling numeric variables

Now that we have our test data and have defined our Likert scales, lets apply those labels and save the transformed data into the new data set `dat_labelled`.  In this case we are using `mutate()` to transform our variables. By using `across(variables, ~labelled())` within `mutate()` we can apply a function, in this case `labelled()`, to multiple variables. `contains()` makes selecting by naming conventions easy, here we're selecting the variables whose names contains "q".  Finally, `~labelled(.x,)` applies the labels we defined above (the ~ and .x are necessary programming nonsense - just make sure they're there for now).

```{r}

```

We are overwriting the existing variables, so lets be sure to double check our transformations.  We will use `apply(data, 2, table)`, where 2 indicates that we're applying the function `table()` to each of the data set's columns that contain "q" in order to check frequencies (a 1 instead of a 2 would apply `table()` to rows).  Below this we use `lapply(data, str)` to apply `str()` to each variable that contains "q" and output the results in order to check the structure of the transformed variables. This will be used below as well. 

```{r}
# always double check transformations

```

 
## <img src="img/core.png" alt="element" width="20"/>  labelling character variables 

Here we use `factor()` within `mutate()` to overwrite our prepost with a factorized version of itself. We must remember to explicitly define the levels within `factor()` since the default numeric order is alphabetically which poses a problem for Pres and Posts. Now, rather than using pre-defined labels, we can define the labels directly in `labelled()` through the `labels = c()` option.  Again, be sure to check any transformation.

```{r}

```

 
## <img src="img/core.png" alt="element" width="20"/>  recoding variables

Let's create new variables with the agree and disagree categories collapsed and provide these new variables with an appended name. We will again use mutate(across(contains())) to apply `rec()` across the desired variables. Specifying the recode within `rec()` use the pattern `rec = "old=new;old,old=new"`, chaining grouped recodes together with `;`.  The `.names` option within `across()` appends a naming convention to the new transformed variables. In this case we are adding "_rda3" to indicate that these variables have been recoded to a three category disagree agree scale. `{col}` in this case keeps the original column name at the front.

```{r}

```

 
## <img src="img/core.png" alt="element" width="20"/>  saving data

We can  use haven's `write_sav()` to write our labelled data as an spss data file `.sav`, or use `write_rds()` to write the data as an r data file `.rds`.

```{r}
haven::write_sav(dat_labelled.2, "path/to/data/filename.sav")
write_rds(dat_labelled.2, "path/to/data/filename.rds")
```

> 2023 Solaris Consulting Group, LLC. info@solarisconsultinggroup.com


