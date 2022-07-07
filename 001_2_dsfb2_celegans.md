# Reproducibility in Data Science

At the institute for Life Sciences a dose response analysis was performed on the C. elegans. The effect of compounds Decane, Naphthalene and 2,6-diisopropylnaphthalene were the number of offspring. From the course instructors I have acquired a dataset which contains the number of offspring and the concentrations of the compounds. In R I visualize the effects using ggplot2.

The main objective of the experiment was to establish whether the compounds Decane, Nephtalene and 2,6-diisopropylnaphthalene causes a decrease in the number of offpsring seen in the C. elegans and if so, in what concentrations. Ethanol was used as a positive control, meaning the number of offspring will definitely drop when being exposed to it. S-medium was used as a negative control, which means nothing will happen when the C. elegans is exposed to it. 68 hours after exposure the number of offspring were counted under a microscope.

In R, I loaded the dataset and inspected it. The column RawData contains the number of offspring after 68 hours of exposure to the compounds. The column compName shows the compound to which C. elegans was exposed to and lastly the column compConcentration shows the concentration of the compound.

A problem I came across was that the column compConcentration was classified as a character rather than numeric. The reason for this is because the value separator was a comma instead of a fullstop. A way of changing is is by using the as.numeric function however that will result in values being removed. Alternatively, using the str_replace function works better and will not remove any values.

1. Load packages
2. Data import
3. Inspect data
4. Clean data
5. Tidy data
6. Visualization
7. Conclusions


```{r, include=FALSE, message=FALSE, warning=FALSE}
## install and/or load packages
if (!require(rcolorUtrecht)) install.packages("rcolorUtrecht")  ## color palette
if (!require(tidyverse)) install.packages("tidyverse")
if (!require(ggplot2)) install.packages("ggplot2")            ## data visualization
if (!require(janitor)) install.packages("janitor")            ## import excel files
if (!require(stringr)) install.packages("stringr")            ## replace , for .
if (!require(timetk)) install.packages("timetk")
if (!require(ggtext)) install.packages("ggtext")              ## text modeling
if (!require(readxl)) install.packages("readxl")              ## import excel files
if (!require(dplyr)) install.packages("dplyr")                ## tidy data
if (!require(glue)) install.packages("glue")
if (!require(drc)) install.packages("drc")                    ## dose response curve
```


```{r data import, message=FALSE, warning=FALSE}
## import the data as a dataframe using read_xlsx
## by default sheet1 will be imported
df <- as.data.frame(readxl::read_xlsx(here::here(
      "raw_data/CE.LIQ.FLOW.062_Tidydata.xlsx")))
```

```{r inspect, message=FALSE, warning=FALSE}
## inspect the data
dplyr::glimpse(df)
```

```{r correct data, message=FALSE, warning=FALSE, include=FALSE}
## replace comma separator with fullstop
df$compConcentration <- df$compConcentration %>% str_replace(",", ".")

## change compConcentration class
df$compConcentration <- df$compConcentration %>% as.numeric() 

## remove scientific notation
format(df, scientific=F)
```

```{r scatter plot, message=FALSE, warning=FALSE}
df %>% ggplot(aes(x = compConcentration,
                  y = RawData,
                  color = compName,
                  shape = expType)) +
       geom_point() +
       theme_minimal() +
       labs(title = "Dose Response Reaction of\nDecane, Nepthalane and 2,6-diisopropylnaphthalene",
            subtitle = "Data collected after 68 hours of dose response time",
            x = "Compound concentration in nM",
            y = "Number of offspring at 68 hours",
            caption = "Data source: J. Louter of the Institute of Life Sciences") +
  scale_color_rcolorUtrecht(palette = "miffy")
  
```

This dataset was a good way to practice with data normalization considering the positively skewed plot. This makes the data difficult to read, which means I needed to do a log10 transformation.

```{r plot2, warning=FALSE, message=FALSE}
## the data is difficult to interpret
## perform a log10 analysis on the compConcentration
df %>% ggplot(aes(x = (log10(compConcentration + 0.001)),
                  y = RawData, 
                  color = compName, 
                  shape = expType)) +
       geom_point() +
       geom_jitter(position = position_jitter(0.2)) +
       theme_minimal() +
       scale_color_rcolorUtrecht(palette = "miffy") +
       labs(title = "Dose Response Reaction of\nDecane, Nepthalane and 2,6-diisopropylnaphthalene",
            subtitle = "Data collected after 68 hours of dose response time",
            x = "Compound concentration in nM",
            y = "Number of offspring at 68 hours",
            caption = "Data source: J. Louter of the Institute of Life Sciences")
```

A log10 transformation helps with interpreting results. The higher the concentration, the less offspring is counted. After this, I normalized the data for the negative control. Why? To get a more relative view of the experiment. If I calculate the mean value for the negative control and then normalize those values, I can compare each data point with that normalized control. This way we may get a more relative conclusion of the plot. 

```{r normalize negative control, message=FALSE, warning=FALSE}
## to make the results more relative

## summarize the data of the negative control
ethanol_summary <- df %>% 
  group_by(compName, compConcentration) %>% 
  summarize(by = "compName", 
            mean = mean(RawData, na.rm = TRUE))

## check the mean value of the ethanol values
ethanol <- df$RawData[df$expType=="controlNegative"] %>% mean()   ##85.9

## normalize the results for the negative control
ethanol_summary <- ethanol_summary %>% mutate(normalized_mean = mean / 85.9)

## and compare each value with this normalized data
ethanol_summary %>% 
                   ggplot(aes(x = (log10(compConcentration + 0.001)), 
                              y = normalized_mean, 
                              colour = compName, 
                              shape = compName)) +
                   geom_point() +
                   geom_line() +
                   labs(title = "Dose Response Reaction of\nDecane, Nepthalane and 2,6-diisopropylnaphthalene",
                        x = "Compound concentration in nM",
                        y = "Number of offspring") +
                   theme_minimal() +
                   scale_color_rcolorUtrecht(palette = "miffy")
```
What can be concluded from this graph is that with increasing concentrations the number of offspring also drop. However what we can see now, what we couldn't before is that Naphthalene shows the most severe drop in offspring, becoming more severe with higher concentrations.

This is a predictable conclusion, but also has shortcomings because it says little about the IC50 and EC50. A way to establish a conclusion on these factors would be to use the drc package.

See my source code in file named 001_dsfb2_celegans.Rmd.
The data file is named CE.LIQ.FLOW.062_Tidydata.xlsx and may be found in the folder raw_data.

To understand how I applied the Guerilla Method to manage my files, take a look at my article Data & File management

**References:**
<br>
[A Brief Introduction to C. elegans](https://www.youtube.com/watch?v=zjqLwPgLnV0)
C. elegans dataset provided by the course instructors, however the experiment was conducted by J. Louter.