# Making R package rcolorUtrecht

*Art is the Queen of all Sciences Communicating Knowledge to All the Generations of the World.* ***(Leonardo da Vinci)***

The Dutch city Utrecht is one of the most beautiful places in the world, and a beacon of art and education, housing more than 21 universities and colleges and 400 art installations. Not long ago when I was lying sick in bed, scrolling the internet for some distraction and at photographs of art pieces, when I stumbled upon the R package by Edwin Thoen (GitHub: EdwinTh) named dutchmasters. I was immediately inspired.

As a Data Scientist you are constantly making graphs that convey to someone either a difference or correlation. That proces is important because looking at large files of data can be daunting and makes us prone to making mistakes. To make a statistical graphs more appealing, we use distinctive colors. To add to the color palettes currently available in R, I used the beauty of a city. 

After gathering some photographs of street art found in Utrecht, I extracted the color palettes with the site ginifab.com. The hex codes from the images were stored in a variable, specific to every art piece.

The saving of those color codes means that the package that I named rcolorUtrecht now contains a function with which you can make a graph using the color codes I stored in that function.

My next step was to use a function for the scale_fill option and the scale_color option. The colors will be the same however when using custom color palette packages in R, and let's say we want to make a bar chart that compares the honey production per month in a given year, we must add the code scale_fill_rcolorUtrecht(palette = "the_wonders").
Now let's say we want to make a scatterplot that seeks to verify whether there is a correlation between cat breeds and tail lengths (yes, I chose this example for a specific reason) when you would use the code scale_color_rcolorUtrecht(palette = "the_wonders"). 

Lastly, I added the every elusive vignettes
This is kind of like a instruction manual you receive when buying an appliance. This way when you install my package by running 

```{r install, eval=FALSE, include=TRUE}
 devtools::install_github("cherjuliette/rcolorUtrecht") 
```

and are wondering how to use the package, you may run the code...
```{r help, eval=FALSE, include=TRUE}
help(package = "rcolorUtrecht") 
```

Let me show you what a graph using my package would look like.
```{r plot, message=FALSE, warning=FALSE}
## package that stores the starwars dataset
if (!require(tidyverse)) install.packages("tidyverse")

## visualize
plot <- starwars %>% 
  filter(hair_color == "black" | hair_color == "brown") %>% 
  drop_na(sex) %>% 
  ggplot(aes(x = hair_color,
             fill = sex)) +
  geom_bar(position = "dodge",
           alpha = 0.8) +
  theme_minimal() +
  theme(panel.grid.major = element_blank())

## color palette
plot + scale_fill_rcolorUtrecht(palette = "hu")
```

Wondering about the package? Or do you just want to see the source code? Take a look at [my GitHub repository](https://github.com/cherjuliette/rcolorUtrecht)

**References**

[Hogeschool Utrecht/Institute for Life Sciences at the Uithof](https://www.internationalhu.com/campus-utrecht-science-park)

[City ambassador group Thirty030, member patatjes4life](https://thirty030.nl/vankastjenaarcanvasje-3/)
    IG: patatjes4life

[De Wonderen/The Wonders mural in Pijlsweerd by Boukje Lootsma and Germa Borst](http://www.muurschilderingenplus.nl/sprookjes-van-de-21ste-eeuw-muurschilderingen/#lightbox[auto_group1]/8/)

[Stories from the Neighborhood in Dichterswijk by Munir de Vries] (www.duic.nl/cultuur/gigantische-muurschildering-verhalen-bewoners-croeselaan/)
IG: https://www.instagram.com/munir_de_vries/?hl=en

[Ducdalf met Schepen (1978)/ Ducdalf with Ships (1978) by Anne P. Boer in the Adelaarstraat](https://www.kunstinopenbareruimte-utrecht.nl/kunstwerken/schepen-met-ducdalf)

[Nijntje/Miffy at the Mariaplaats by De Strakke Hand](https://www.destrakkehand.nl/nijntje-museum)

[Groceries mural above the Albert Heijn grocery shop in Amsterdamse Straatweg 367 by Jan is De Man](https://girlswanderlust.com/exploring-street-art-in-utrecht/)
IG: https://www.instagram.com/janisdeman/?hl=en

[Train Station from the Past at Meidoornstraat by De Verfdokter](https://www.verfdokter.com/)

[Derk by Derk Wessels and Jan is De Man at Lauwerecht 55](https://www.instagram.com/janisdeman/?hl=en)

[De Walvis/The Wale by Studio KCA at the Tivolivredenburg](https://www.uu.nl/organisatie/faculteit-recht-economie-bestuur-en-organisatie/samenwerking-en-samenleving/skyscraper)

[Glimpse into the Past above the Ibis Hotel at Bizetlaan by De Strakke Hand](https://www.duic.nl/algemeen/muurschildering-geeft-doorkijkje-naar-historisch-den-hommel/)

[Geese at the Gansstraat 64 by De Strakke Hand](https://www.destrakkehand.nl/)

[3D organs at the Berlijnplein bij Leon Keer](https://www.instagram.com/leonkeer/)

[Birds mural at the Vogelenbuurt by Jan is De Man](https://www.instagram.com/janisdeman/?hl=en)

[Phase Constract Microscope of Caroline Bleeker by De Strakke Hand](https://streetartcities.com/cities/utrecht/markers/40184)

[Utrecht University at the Domplein](https://www.youtube.com/watch?v=QmkkKRwrqZk)

