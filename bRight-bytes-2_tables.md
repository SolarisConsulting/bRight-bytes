# bRight bytes 2 - tables for labelled data

## <img src="img/core.png" alt="element" width="20"/>  learning goals

In this bRight byte, we will tackle the following questions: 
 * How do we read in our data?
 * How do we format our data for table creation?
 * How do we create usable tables?
 * How do we edit table options?
 * How do we combine multiple questions into one table?
 * How do we export our table for use?
 
We will primarily be using the `tidyverse`, `haven`, `labelled`, and `sjmisc` packages to create and transform our data set and the `gtsummary` and `flextable` packages to create and format our tables.  These tables can then be used in RMarkdown, Markdown, Windows Office, and various other formats. What's presented here is an introduction to a deep set of tools and further investigation is encouraged.  A resources section is included at the end. 

 
## <img src="img/core.png" alt="element" width="20"/>  creating our data set

First, lets install and load the required packages
 * tidyverse for data wrangling
 * haven, labelled, and sjmisc for labelled data transformation
 * gt and gtsummary for summaries and initial table creation
 * flextable for table formatting and editing

```{r}
# install.packages(c("tidyverse", "haven", "sjmisc", "labelled"))

library(tidyverse)
library(haven)
library(sjmisc)
library(labelled)

# install.packages(c("gt", "gtsummary", "flextable", "sysfonts"))

library(gt)
library(gtsummary)
library(flextable)
library(sysfonts)

library(tinytex)
```

Let's create some test data. We will create both continuous and categorical variables. We've defined the object `n_sample` as the number of cases, 100, in our data set. We create the data set using `tibble()` to work with tidyverse package. First, we use `seq()` to create a unique, sequential id variable, the length of the sample `n_sample`. The continuous variables are created using `sample.int()` that samples integers that will represent questionnaire responses to a Likert scale or a range question (1-100). Our character variables are created using `sample()`. Remember to specify `replace = TRUE` in both cases in order to sample with replacement. For a deeper explanation of these transformations, see [bRight byte 1 - Working with Labelled Data](bRight-bytes-1_labelleddata.md).

<details>
    <summary> Click here for the dataset creation and cleaning syntax. </summary>

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
    
</details>

Alternatively, if you completed bRight bytes 1, you can read in your saved data by modifying the following code.

```{r}
dat_labelled <- haven::read_sav("path/to/data/filename.sav", user_na = TRUE)
dat_labelled <- read_rds("path/to/data/filename.rds")
```

 
## <img src="img/core.png" alt="element" width="20"/>  factor transformation

The `gtsummary` package works particularly well with factor variables, but will work with your already labelled data as well. As an example, we will transform our categorical variables to factors. We will apply `labelled::to_factor()` to the factor variables using the now familiar `mutate(across())`, using our defined variable labels as the factor levels. The use of `!all_of()` within `across()` means we want the transformations applied to none of the columns (`!` within `all_of()`) indicated by the object `nonfactors`. 

```{r}
# define non-factor variables
nonfactors <- c("scale", "cont")

# change variables to factor to more easily work with tables
dat_factor <- dat_labelled %>% 
  mutate(across(!all_of(nonfactors), ~labelled::to_factor(., levels = "labels", sort_levels = "none")))
```

 
## <img src="img/core.png" alt="element" width="20"/>  single categorical variable frequency table

First we will look at a frequency table for a single categorical variable, `group`. The table produced has a header that provides a title and the number of cases, the variable's name and its response categories, frequencies and percentages of the responses, and a footer providing further detail about the statistic column. 

```{r}
dat_factor %>%
  select(group) %>%
  tbl_summary()
```

While this table is well formatted and usable, `tbl_summary()` has a number of options that allow us to edit or specify particular parts of the table that expand its flexibility exponentially. The code below does the following: 
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
                       sort = list(all_categorical() ~ "frequency")) 

ft_group
                       
```


## <img src="img/core.png" alt="element" width="20"/>  further formatting and themes

Now that our table contains what we want it to, let's look at updating the headers and footers. Specifically here, we use `modify_header()` to remove the general header and update the header for the statistics column to match our specifications and `modify_footnote()` to remove the footnote that duplicates what's in the header.

```{r}
ft_group <- ft_group %>%
  # set the header labels
  modify_header(label ~ "", stat_0 ~ "% (N)") %>% 
  # remove the footnote
  modify_footnote(update = everything() ~ NA)
```  


`gtsummary` includes a number of pre-made themes as well as the ability to create and edit your own themes. You can also apply themes sequentially.  One example could be applying the JAMA theme with `theme_gtsummary_journal(journal = "jama")` and then remove extra space with `theme_gtsummary_compact()`. You can also modify theme elements individually into a list and edit your theme using `set_gtsummary_theme()`. There are a large number of options so review the documentation carefully! If you make a mistake you can always use `reset_gtsummary_theme()` to reset the theme to its default.

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


## <img src="img/core.png" alt="element" width="20"/>  advanced formatting through flextable

While flexible, some table aspects cannot be adjusted through a gtsummary theme. For these options, we use the `flextable` package. First we convert the gtsummary table to a flextable using `as_flex_table()`. `flextable` is a deep package and we suggest consulting package author David Gohel's [Using the flextable R package](https://ardata-fr.github.io/flextable-book/) for specifics. `flextable` is useful for incorporating tables into Microsoft Office products through RMarkdown documents. This will be covered in a future bRight byte.
  
```{r}  
ft_group2 <- ft_group  %>%
  # use flextable::as_flex_table() to turn the table to a flextable object which can be edited there for display
  as_flex_table() %>%
  # reduce padding around the cells (default is 5)
  padding(padding.bottom = 3.5, padding.top = 3.5, part = "body") %>% 
  # auto fit the table
  set_table_properties(layout = "autofit") %>%
  # set font
  font(fontname = "Montserrat", part = "all") %>%
  # highlight a particular row or rows (i indicates the row, bg indicates the color)
  bg(i = 3, bg = "#EFEFEF", part = "body")

ft_group2
```


## <img src="img/core.png" alt="element" width="20"/>  multiple variables in a table

We can use `tbl_summary()` to combine multiple variable types into a single table and specify options for each variable type independently. The table below includes dichotomous and continuous variables, as opposed to the categorical variables in the tables above.

```{r}
# create demo table
demo_table <- dat_factor %>% 
  # select demographic variables
  select(sa_1, sa_2, cont)

# create table summary of demographics
ft_demo <- tbl_summary(demo_table, 
                       # define or change variable labels
                       label = list(sa_1 ~ "Select All Item A", 
                                    sa_2 ~ "Select All Item B", 
                                    cont ~ "Continuous Variable"),
                       # set variable statistics that are computed and displayed in the column for each type
                       statistic = list(all_continuous() ~ "{mean} {median} ({min}, {max})"),
                       # explicitly set variable type
                       type = list(sa_1 ~ "dichotomous", 
                                   sa_2 ~ "dichotomous", 
                                   cont ~ "continuous"),
                       # set the value displayed for dichotomous variable (in this case "Selected")
                       value = list(sa_1 = "Selected",
                                    sa_2 = "Selected"),
                       # do not display missing data
                       missing = "no") %>%
  # set header labels
  modify_header(label ~ "Variable", stat_0 ~ "N (%) Selected") %>%
  modify_footnote(all_stat_cols() ~ "N (%) Selected, mean median (min, max)") %>%
  # set as flextable object (flextable::as_flex_table())
  as_flex_table() %>% 
  padding(padding.bottom = 3.5, padding.top = 3.5, part = c("body")) %>% 
  set_table_properties(layout = "autofit") %>%
  font(fontname = "Montserrat", part = "all") %>%
  # highlight a line in the table (highlighting different variable types, headers, etc.)
  bg(i = 3, bg = "#EFEFEF", part = "body")

ft_demo
```


## <img src="img/core.png" alt="element" width="20"/>  exporting tables

If we didn't use the options available in the `flextable` package, the `gt` package has a handy function called `gtsave()` which can output our table in multiple formats including .png, .pdf, .html, or LaTeX. We will first need to convert our table to a gt table using `as_gt()`.

```{r}
gtsave(as_gt(ft_demo), file = "path/to/data/filename.type")
```

As could be expected, the `flextable` package also has a large number of formats into which we can export our tables. Many of these have specific functions associated with them such as `save_as_image()`, `save_as_html()`, `save_as_pptx()`, `save_as_docx()`, and `save_as_rtf()`.

```{r}
save_as_image(ft_demo, path = "path/to/data/filename.type")
```


## <img src="img/core.png" alt="element" width="20"/>  resources

[gtsummary vignettes](https://www.danieldsjoberg.com/gtsummary/articles/)

[gtsummary gallery](https://www.danieldsjoberg.com/gtsummary/articles/gallery.html)

[Using the flextable R package](https://ardata-fr.github.io/flextable-book/)

[flextable gallery](https://ardata.fr/en/flextable-gallery/)

[Publication-focused introduction to both gtsummary and flextable](https://www.cincibrainlab.com/post/stats_with_gtsummary/)


> 2023 Solaris Consulting Group, LLC. info@solarisconsultinggroup.com


