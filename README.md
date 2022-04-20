---
title: 'Weekly Exercises #6'
author: "Rylan Mueller"
output: 
  html_document:
    keep_md: TRUE
    toc: TRUE
    toc_float: TRUE
    df_print: paged
    code_download: true
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, error=TRUE, message=FALSE, warning=FALSE)
```

```{r libraries}
library(tidyverse)     # for data cleaning and plotting
library(gardenR)       # for Lisa's garden data
library(lubridate)     # for date manipulation
library(openintro)     # for the abbr2state() function
library(palmerpenguins)# for Palmer penguin data
library(maps)          # for map data
library(ggmap)         # for mapping points on maps
library(gplots)        # for col2hex() function
library(RColorBrewer)  # for color palettes
library(sf)            # for working with spatial data
library(leaflet)       # for highly customizable mapping
library(ggthemes)      # for more themes (including theme_map())
library(plotly)        # for the ggplotly() - basic interactivity
library(gganimate)     # for adding animation layers to ggplots
library(gifski)        # for creating the gif (don't need to load this library every time,but need it installed)
library(transformr)    # for "tweening" (gganimate)
library(shiny)         # for creating interactive apps
library(patchwork)     # for nicely combining ggplot2 graphs  
library(gt)            # for creating nice tables
library(rvest)         # for scraping data
library(robotstxt)     # for checking if you can scrape data
theme_set(theme_minimal())
```

```{r data}
# Lisa's garden data
data("garden_harvest")

#COVID-19 data from the New York Times
covid19 <- read_csv("https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv")

```

## Put your homework on GitHub!

Go [here](https://github.com/llendway/github_for_collaboration/blob/master/github_for_collaboration.md) or to previous homework to remind yourself how to get set up. 

Once your repository is created, you should always open your **project** rather than just opening an .Rmd file. You can do that by either clicking on the .Rproj file in your repository folder on your computer. Or, by going to the upper right hand corner in R Studio and clicking the arrow next to where it says Project: (None). You should see your project come up in that list if you've used it recently. You could also go to File --> Open Project and navigate to your .Rproj file. 

## Instructions

* Put your name at the top of the document. 

* **For ALL graphs, you should include appropriate labels.** 

* Feel free to change the default theme, which I currently have set to `theme_minimal()`. 

* Use good coding practice. Read the short sections on good code with [pipes](https://style.tidyverse.org/pipes.html) and [ggplot2](https://style.tidyverse.org/ggplot2.html). **This is part of your grade!**

* **NEW!!** With animated graphs, add `eval=FALSE` to the code chunk that creates the animation and saves it using `anim_save()`. Add another code chunk to reread the gif back into the file. See the [tutorial](https://animation-and-interactivity-in-r.netlify.app/) for help. 

* When you are finished with ALL the exercises, uncomment the options at the top so your document looks nicer. Don't do it before then, or else you might miss some important warnings and messages.


## Warm-up exercises from tutorial

1. Read in the fake garden harvest data. Find the data [here](https://github.com/llendway/scraping_etc/blob/main/2020_harvest.csv) and click on the `Raw` button to get a direct link to the data. After reading in the data, do one of the quick checks mentioned in the tutorial.

```{r}
fake_harvest <- read_csv("https://raw.githubusercontent.com/llendway/scraping_etc/main/2020_harvest.csv", 
    col_types = cols(`This is my awesome data!` = col_skip(), 
        weight = col_integer()), na = "null", 
    skip = 2)

fake_harvest %>% 
mutate(across(where(is.character), as.factor)) %>% 
  summary()

```


2. Read in this [data](https://www.kaggle.com/heeraldedhia/groceries-dataset) from the kaggle website. You will need to download the data first. Save it to your project/repo folder. Do some quick checks of the data to assure it has been read in appropriately.

```{r}
groceries <- read_csv("Groceries_dataset.csv")

groceries %>% 
  mutate(across(where(is.character), as.factor)) %>% 
  summary()

groceries %>% 
  group_by(itemDescription) %>% 
  summarize(item_sum = n())
```


3. Create a table using `gt` with data from your project or from the `garden_harvest` data if your project data aren't ready. Use at least 3 `gt()` functions.

```{r}
garden_harvest %>% 
  filter(vegetable == "lettuce") %>% 
  gt(rowname_col = "variety",) %>% 
  fmt_date(columns = vars(date),
           date_style = 5) %>% 
  tab_header(title = "Lettuce Harvest (in grams)") %>% 
  tab_row_group(label = "Reseed", 
                rows = contains("reseed")) %>% 
  tab_row_group(label = "Farmer's Market Blend", 
                rows = contains("Farmer's")) %>% 
  tab_row_group(label = "Tatsoi", 
                rows = contains("Tatsoi")) %>%
  tab_row_group(label = "Lettuce Mixture", 
                rows = contains("Mixture")) %>% 
  tab_row_group(label = "Mustard Greens",
                rows = contains("Mustard")) %>% 
    cols_hide(c(variety, vegetable, units))
```




4. CHALLENGE (not graded): Write code to replicate the table shown below (open the .html file to see it) created from the `garden_harvest` data as best as you can. When you get to coloring the cells, I used the following line of code for the `colors` argument:
  
```{r, eval=FALSE}
colors = scales::col_numeric(
      palette = paletteer::paletteer_d(
        palette = "RColorBrewer::YlGn"
      ) %>% as.character()
```

  
5. Use `patchwork` operators and functions to combine at least two graphs using your project data or `garden_harvest` data if your project data aren't ready.

```{r}
plot1 <- garden_harvest %>% 
  filter(vegetable %in% c("tomatoes", "squash", "raspberries")) %>% 
  group_by(vegetable) %>% 
  ggplot(aes(y = vegetable)) + 
  geom_bar() + 
  labs(x = "Count", y = "", title = "Total Harvests")

plot2 <- garden_harvest %>% 
  filter(vegetable %in% c("tomatoes", "squash", "raspberries")) %>% 
  group_by(vegetable, date) %>% 
  summarize(daily_harvest_lbs = sum(weight * 0.00220462)) %>% 
  mutate(cum_sum_harvest = cumsum(daily_harvest_lbs)) %>% 
  ggplot(aes(x = date, y = cum_sum_harvest)) +
  geom_line(aes(color = vegetable)) +
  labs(x = "", y = "", title = "Cumulative Harvest (lbs)")

plot1 + plot2 +
  plot_annotation(title = "Tomatoes, Squash, and Raspberries")
```

  
## Webscraping exercise (also from tutorial)

Use the data from the [Macalester Registrar's Fall 2017 Class Schedule](https://www.macalester.edu/registrar/schedules/2017fall/class-schedule/#crs10008) to complete all these exercises.

6. Find the correct selectors for the following fields. Make sure that each matches 762 results:

  * Course Number
  * Course Name
  * Day
  * Time
  * Room
  * Instructor
  * Avail. / Max
  * General Education Requirements (make sure you only match 762; beware of the Mac copyright banner at the bottom of the page!)
  * Description

Then, put all this information into one dataset (tibble or data.frame) Do not include any extraneous information like "Instructor: ".
  
```{r}
fall2017 <- read_html("https://www.macalester.edu/registrar/schedules/2017fall/class-schedule/#crs10008")

course_number <- fall2017 %>% 
  html_elements(".class-schedule-course-number") %>%
  html_text2()

course_title <- fall2017 %>% 
  html_elements(".class-schedule-course-title") %>%
  html_text2()

course_day <- fall2017 %>% 
  html_elements(".class-schedule-course-title+ .class-schedule-label") %>%
  html_text2() %>% 
  str_remove("Days: ")

course_time <- fall2017 %>% 
  html_elements(".class-schedule-label:nth-child(4)") %>% 
  html_text2() %>% 
  str_remove("Time: ")

course_room <- fall2017 %>% 
  html_elements(".class-schedule-label:nth-child(5)") %>% 
  html_text2()%>% 
  str_remove("Room: ")

course_instructor <- fall2017 %>% 
  html_elements(".class-schedule-label:nth-child(6)") %>% 
  html_text2()%>% 
  str_remove("Instructor: ")

course_avail <- fall2017 %>% 
  html_elements(".class-schedule-label:nth-child(7)") %>% 
  html_text2()%>% 
  str_remove("Avail./Max.: ")

course_general <- fall2017 %>% 
  html_elements("#content p:nth-child(2)") %>% 
  html_text2() %>% 
  str_remove("General Education Requirements:")

course_desc <- fall2017 %>% 
  html_elements(".collapsed p:nth-child(1)") %>% 
  html_text2()
```



7. Create a graph that shows the number of sections offered per department. Hint: The department is a substring of the course number - there are `str_XXX()` functions that can help. Yes, COMP and MATH are the same department, but for this exercise you can just show the results by four letter department code, e.g., with COMP and MATH separate.


```{r, fig.height = 6, fig.width = 8}
course_letters <- gsub("[^a-zA-Z]", "", course_number)

course_depts <- tibble(
  Department = substr(course_letters, 1, 4)
)

course_depts %>% 
group_by(Department) %>% 
  ggplot(aes(y = Department)) +
  geom_bar() +
  labs(x = "Count", y = "", title = "Number of Sections offered by each Macalester Department in Fall Semester 2017")
```



8. Analyze the typical length of course names by department. To do so, create a new data table based on your courses data table, with the following changes:
  
  * New columns for the length of the title of a course and the length of the description of the course. Hint: `str_length`.  
  * Remove departments that have fewer than 10 sections of courses. To do so, group by department, then remove observations in groups with fewer than 10 sections (Hint: use filter with n()). Then `ungroup()` the data.  
  * Create a visualization of the differences across groups in lengths of course names or course descriptions. Think carefully about the visualization you should be using!
  
```{r}
course_depts_changed <- course_depts %>% 
  mutate(Title_Length = str_length(course_title)) %>% 
  mutate(Desc_Length = str_length(course_desc)) %>% 
  group_by(Department) %>% 
  filter(n() >= 10) %>% 
  ungroup() 
  
  title_plot <- course_depts_changed %>% 
  ggplot(aes(x = Title_Length, y = Department)) +
  geom_boxplot()

  desc_plot <- course_depts_changed %>% 
    ggplot(aes(x = Desc_Length, y = Department)) +
    geom_boxplot()
  
  title_plot + desc_plot
```
