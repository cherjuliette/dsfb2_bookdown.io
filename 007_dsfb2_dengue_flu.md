# Evaluating the global Dengue Fever and Flu cases in R

Google.org provides a number of datasets on Dengue Fever and flue cases. I downloaded two files that contained the number of dengue fever cases and flu cases which were already present in the GitHub repository of the course. These files provide cases from 2002 till 2015.

I began my analysis with the following steps: 

1. Manual download of the Dengue Fever dataset from the [dsfb2 repository](https://github.com/DataScienceILC/tlsc-dsfb26v-20_workflows).
2. Manual download of the Flu Fever dataset from the [dsfb2 repository](https://github.com/DataScienceILC/tlsc-dsfb26v-20_workflows).
3. Clean and tidy the data
4. Write the data away in the folder data
5. Make a SQL database of the data using RPostgreSQL
6. Data visualization
7. Conclusion

```{r load packages, include=FALSE, warning=FALSE, message=FALSE}
if (!require(rcolorUtrecht)) install.packages("rcolorUtrecht")  ## color palette
if (!require(RPostgreSQL)) install.packages("RPostgreSQL")  ## SQL functions
if (!require(gapminder)) install.packages("gapminder")      ## gapminder dataset
if (!require(tidyverse)) install.packages("tidyverse")
if (!require(RPostgres)) install.packages("RPostgres")      ## SQL functions
if (!require(gridExtra)) install.packages("gridExtra")          ## putting graphs side by side
if (!require(ggplot2)) install.packages("ggplot2")          ## data visualization
if (!require(ggtext)) install.packages("ggtext")            ## add value labels to plot
if (!require(plotly)) install.packages("plotly")            ## interactive graphs
if (!require(dslabs)) install.packages("dslabs")            ## data wrangling
if (!require(here)) install.packages("here")                ## working dir
if (!require(DBI)) install.packages("DBI")                  ## DataBase Interface
if (!require(DT)) install.packages("DT")                    ## interactive tables
```

**Data import of Dengue dataset**
```{r dengue data import, warning=FALSE, message=FALSE}
dengue <- read_csv(here::here("raw_data/007_dengue.csv"),
                   
                   ## ignores the rows containing metadata
               skip = 11)
```

**Data import of the Flu dataset**
```{r flu data import, warning=FALSE, message=FALSE}
flu <- read_csv(here::here("raw_data/007_flu.csv"),
                ## ignore rows with metadata
                skip = 11)
```

```{r gapminder import, message=FALSE, warning=FALSE}
gapminder <- read_builtin("gapminder")
```


```{r gapminder, message=FALSE, warning=FALSE, include=TRUE}
## inspect the datasets
dplyr::glimpse(gapminder)
dplyr::glimpse(dengue)
dplyr::glimpse(flu)
```

**Tidy data of the Flue dataset**
```{r tidy data, message=FALSE, warning=FALSE}
## tidy the dengue dataset
## Put all countries in 1 column, and the flu cases in 1 column
dengue_tidy <- dengue %>% 
  
  ## selecting all columns by hand is tedious
  ## Here i select column Argentia till Venezuela using :
  pivot_longer(
                cols = Argentina:Venezuela,
                        names_to = "Country",
                        values_to = "dengue_cases"
                )

## tidy the flu dataset
## Put all countries in 1 column and cases in another column
flu_tidy <- flu %>%
                   pivot_longer(
                                cols = Argentina:Uruguay,
                                names_to = "Country",
                                values_to = "flu_cases"
                                )

## DENGUE
## split the column Date to three columns named Year, Month and Day
dengue_tidy <- dengue_tidy %>% separate(Date,
                                        c("Year", "Month", "Day"))


## FLU
## split the column Date to three columns named Year, Month and Day
flu_tidy <- flu_tidy %>% separate(Date,
                                 c("Year", "Month", "Day"))

## correct data class for the flu dataset
flu_tidy$Year <- as.numeric(flu_tidy$Year)
flu_tidy$Month <- as.numeric(flu_tidy$Month)
flu_tidy$Day <- as.numeric(flu_tidy$Day)

## correct data class for the dengue dataset
dengue_tidy$Year <- as.numeric(dengue_tidy$Year)
dengue_tidy$Month <- as.numeric(dengue_tidy$Month)
dengue_tidy$Day <- as.numeric(dengue_tidy$Day)

```

**Save the clean and tidy data in a new CSV file using the write function**
```{r new save, warning=FALSE, warning=FALSE}
## write the tidy data into a new file
## the location of the files will be in folder data

## write away the tidy flu dataset
flu_tidy %>% write.csv(file = "./data/flu_tidy.csv")

## write away the tidy dengue dataset
dengue_tidy %>% write.csv(file = "./data/dengue_tidy.csv")

## write away the gapminder dataset
gapminder %>% write.csv(file = "./data/gapminder.csv")

## save the data in a RDS file also
saveRDS(gapminder, file = "./data/gapminder.rds")
saveRDS(dengue_tidy, file = "./data/dengue_tidy.rds")
saveRDS(flu_tidy, file = "./data/flu_tidy.rds")
```

```{r export to DBeaver, eval=FALSE, include=TRUE, warning=FALSE, message=FALSE}
## exporting to DBeaver requires a password which is unique for all users
## for safety reasons I will be omitting this information
## this analysis is still reproducible if you copy the code and insert your own password

## export the new files into DBeaver using RPostgres
con <- dbConnect(RPostgres::Postgres(),
                 dbname = "workflowsdb",
                 host = "localhost",
                 port = "5432",
                 user = "postgres",
                 password = "2022")
dbWriteTable(con, "flu_tidy", flu_tidy)
dbWriteTable(con, "dengue_tidy", dengue_tidy)
dbWriteTable(con, "gapminder", gapminder)
dbDisconnect(con)

## Inspect the files in DBeaver
## SELECT dengue_cases, country 
## FROM dengue tidy order 
## BY dengue_cases asc

## SELECT dengue_cases, country 
## FROM dengue tidy order 
## BY dengue_cases desc

## do this for all 3 files.
```


**After inspecting the data in DBeaver, import the files back into R**
```{r import again, eval=FALSE, include=TRUE}
## import the data into R again after inspecting in DBeaver
## using RPostgres
gapminder <- dbReadTable(con, "gapminder")
dengue <- dbReadTable(con, "dengue_tidy")
flu <- dbReadTable(con, "flu_tidy")
```


**Data visualization**
Is the flu and dengue fever more prevalent in certain areas? Let's check

```{r standard flu graph, message=FALSE, warning=FALSE}
## clean variables names
## default option is snake case
flu_tidy <- janitor::clean_names(flu_tidy)

flu_1 <- flu_tidy %>% 
  group_by(year) %>% 
  na.omit() %>% 
  ggplot(aes(x = year,
             y = flu_cases,
             fill = year)) +
  geom_col() +
  theme_minimal() +
  labs(title = "Evaluating the number of flu cases",
       subtitle = "Have flu cases risen over the years?",
       x = "Year",
       y = "Number of flu cases",
       caption = "Data source: Course instructors")


## Check per country
flu_1 + facet_wrap(~country)

flu_1
```

```{r standard dengue graph, message=FALSE, warning=FALSE}
## clean variables names
## default option is snake case
dengue_tidy <- janitor::clean_names(dengue_tidy)

dengue_1 <- dengue_tidy %>% 
  group_by(year) %>% 
  na.omit() %>% 
  ggplot(aes(x = year,
             y = dengue_cases,
             fill = year)) +
  geom_col() +
  theme_minimal() +
  labs(title = "Evaluating the number of flu cases",
       subtitle = "Have flu cases risen over the years?",
       x = "Year",
       y = "Number of flu cases",
       caption = "Data source: Course instructors")

## let's see the difference between countries
dengue_1 + facet_grid(~country)

```

**Conclusions on the flu dataset**
When considering the number of **flu** cases globally, we can see that the highest occurence of the flu is in Spain, followed by Canada and the US. In contrast, the lowest flu occurences are found in Sweden, followed by Chile and New Zealand.
The reason for this is unknown. To establish significance we could do statistical testing. 

## Conclusions on the dengue dataset
When looking at the graph that shows dengue occurrences, we can see that the highest number of cases are seen in Venezuele, followed by Indonesia and Brazil. Dengue fever develops after infection with the *Flaviviridae* virus carried by mosquitoes. These mosquitoes are more prevalent in countries with warmer climates, which might explain the high occurrences.

**Let's do some joins**
We can already tell a lot by standard visualization as displayed above, but we want to compare the two occurances.
```{r join, message=FALSE, warning=FALSE}
flu_dengue <- full_join(flu_tidy, 
                        dengue_tidy, 
                        by = c("country", "year"), 
                        suffix = c("_flu", "_dengue"))

## let's also join the data from the gapminder dataset with the flu and dengue dataset
gapminder_flu_dengue <- inner_join(flu_dengue, 
                                   gapminder, 
                                   by = c("country", 
                                          "year"))

```


**Visualize the cases of flu per country**
```{r}
# plot occurrences of flu and dengue per region
flu_country <- gapminder_flu_dengue %>% 
              ggplot() + 
              geom_col(aes(x = country, 
                           y = flu_cases,
                           fill = country)) +
  theme_minimal() +
  scale_fill_rcolorUtrecht(palette = "hu") +
  ## turn the x axis labels by a 45 degree angle
  theme(axis.text.x = element_text(angle = 45, 
                                   hjust = 1),
        ## legend is unnecessary
        legend.position = "none"
        ) + 
  ## Explain the graph
  labs(title = "Flu cases per country", 
       x = "",
       y = "Flu cases")

```

**Visualize the cases of dengue fever per country**
```{r}
dengue_country <- gapminder_flu_dengue %>% 
                  ggplot() + 
                  geom_col(aes(x = country, 
                               y = dengue_cases,
                               fill = country)) +
                  theme_bw() +
                  scale_fill_rcolorUtrecht(palette = "hu") +
  
                  ## turn the x axis label 45 degrees
                  theme(axis.text.x = element_text(angle = 45, 
                                     hjust = 1),
                        legend.position = "none") + 
  
                  ## explain the graph
                  labs(title = "Cases of Dengue Fever per country", 
                       x = "",
                       y = "Cases of Dengue Fever")
```

```{r together, message=FALSE, warning=FALSE, full.width=TRUE}
grid.arrange(flu_country, dengue_country, nrow = 1)
```


**Conclusions after joining**
Joining the dataset provides us with a singular object that contains data on all three diseases. This helps put things in perspective, but as we see in the graphs above, the conclusions previously drawn remains the same.