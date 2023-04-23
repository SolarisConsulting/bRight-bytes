# bRight bytes 2 - graphing labelled data

## <img src="img/core.png" alt="element" width="20"/>  learning goals

In this bRight byte, we will tackle the following questions: 
 * How can a little pre-cleaning planning improve our data cleaning process?
 * How do we create some test data to try our process on?
 * How do we label numeric variables?
 * How do we label character variables?
 * How do we recode labelled variables?
 * How do we save our data set?
 
We will primarily be using the tidyverse function `mutate()` to transform variables and `across()` to apply those transformations to multiple variables.  We will use a few selection helpers such as `contains()`, `starts_with()`, and `ends_with` in conjunction with the `.names` argument of `across()` to apply naming conventions and select groups of variables. `labelled()` and `rec()` will assist in labeling our variables and re-coding our labelled variables.

 
## <img src="img/core.png" alt="element" width="20"/>  a note about naming conventions

Planning our cleaning process and analysis plan is an important but often overlooked step in the data processing pipeline. Creating naming conventions that are easily used allows you to select and transform variables quickly and easily. In survey research, this could be a validated scale or simply questions with the same response options. These conventions do not have to be permanent, but may be dropped after cleaning.  Thinking deeply about your data structure can help avoid misunderstandings and better connect you with your data. Here are a few examples of naming conventions:
 * _r (recoded) 
 * _daX (X point disagree - agree scale)
 * _catX (X categories)
 * _asc (categories ascending) or _dsc (categories descending)
 * _rev (reversed)

 
## <img src="img/core.png" alt="element" width="20"/>  create test data

Since we're using R, lets install and load the required packages
 * tidyverse for data wrangling
 * haven, labelled, and sjmisc for labelled data transformation

```{r}
install.packages(c("tidyverse", "haven", "labelled", "sjmisc"))

library(tidyverse)
library(haven)
library(labelled)
library(sjmisc)
```

First, we will specify the number of cases for our test data set.  In this case, we've defined the object `n_sample` the number 100 cases, which we use in the creation of our data set.  We create the data set using `tibble()` to work with tidyverse package. First, we use `seq()` to create a unique, sequential id variable, the length of the sample `n_sample`. The next variables are created using `sample.int()` that samples integers that will represent questionnaire responses to a Likert scale.  We are not dealing with missing data in this tutorial. Remember to specify `replace = TRUE` in order to sample with replacement. Our grouping variable, prepost, is created as a character variable using `sample()` to demonstrate how to create text variables and to show the differences in cleaning various types of data.  

```{r}

```

 
## <img src="img/core.png" alt="element" width="20"/>  defining scales

Now we define our Likert scales for transformation. At the end of the document are some examples of common Likert scales you can use in your transformations along with a template for creating your own. By saving these as objects we only need to define them once.

```{r}

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


