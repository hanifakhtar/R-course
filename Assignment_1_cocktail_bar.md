---
title: "Assignment_1_cocktail_bar"
author: "Hanif Akhtar"
date: "5/27/2021"
output: 
  html_document: 
    keep_md: yes
editor_options: 
  chunk_output_type: console
---

# Skills needed to solve this assignment
-   Using R and RStudio, reading data
-   Reporting using RMarkdown
-   Using Git and Github (for submitting the task)
-   Data manipulation (e.g. dplyr, tidyr)
-   String manipulation (e.g. stringr)



# Background

Your Cuban uncle sets up a cocktail bar in downtown Budapest. He has a secret list of cocktails that he would like to serve in the bar. He asks you to do create a few lists and tables to set up the shop. As your uncle is a secret hipster, he has a dataset on Github that you can use for the task!

![](https://media1.tenor.com/images/57a519cffd0ad4693be3b9c94b211540/tenor.gif?itemid=5195211){width="320"}

Note: There are several different ways to solve these tasks, feel free to come up with your own.

## 1. Read the data

Read the cocktail dataset from: <https://github.com/nthun/cocktail-balance> You can also find the codebook there.


```r
cocktail <- readr::read_tsv("https://raw.githubusercontent.com/nthun/cocktail-balance/master/cocktail_data.tsv")
```

```
## 
## -- Column specification --------------------------------------------------------
## cols(
##   name = col_character(),
##   abv = col_double(),
##   acid = col_double(),
##   sugar = col_double(),
##   type = col_character(),
##   index = col_double(),
##   instructions = col_character(),
##   ingredients = col_character(),
##   ncotw = col_character()
## )
```

```r
head(cocktail)
```

```
## # A tibble: 6 x 9
##   name     abv  acid sugar type   index instructions     ingredients    ncotw   
##   <chr>  <dbl> <dbl> <dbl> <chr>  <dbl> <chr>            <chr>          <chr>   
## 1 Pisco~  12.1  0.68   7.2 eggwh~     5 Dry shake, shak~ 2 oz pisco (4~ "not ye~
## 2 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shak~ 1 1/2 oz Plym~ "not ye~
## 3 Clove~  13.6  0.49   6.7 eggwh~     7 Dry shake, shak~ 2 oz Plymouth~ "<a hre~
## 4 Whisk~  15.2  0.53   7.1 eggwh~     8 Dry shake, shak~ 2 oz rye (50%~ "not ye~
## 5 Daiqu~  14.7  0.94   8.7 shaken     9 Shake, coupe.    2 oz white ru~ "<a hre~
## 6 Honey~  15    0.85   8.9 shaken    10 Shake, coupe. G~ 2 oz white ru~ "<a hre~
```

## 2. Transform the data table and clean the ingredient variable!

The ingredients are currently in a single cell for each cocktail. It would be better to put them in separate rows. Also, the variable should be cleaned of all quantities (e.g. 1/2 oz, 2 dashes, etc.), and the indicator of alcohol content (e.g. 47.3% abv). You will need to use this cleaned table in later tasks.


```r
sep_cocktail <- cocktail %>% 
  separate_rows(ingredients, sep = "<br/>", convert = TRUE) %>% 
  separate_rows(ingredients, sep = "<br>", convert = TRUE) %>% 
  separate_rows(ingredients, sep = "<b4/>", convert = TRUE)

ingre <- gsub("[^A-Za-z ]","", sep_cocktail$ingredients)
ingre_clean <- ingre %>%
  gsub("  oz ","", .) %>%
  gsub(" bsp ","", .) %>%
  gsub(" dash ","", .) %>%
  gsub(" dashes ","", .) %>%
  gsub(" drop ","", .) %>%
  gsub(" drops ","", .) %>%
  gsub(" oz ","", .) %>%
  gsub("  abv","", .)
sep_cocktail$ingredients <- ingre_clean

cocktail_fix <- sep_cocktail %>% 
  mutate(ingredients = str_to_lower(ingredients))
cocktail_fix
```

```
## # A tibble: 223 x 9
##    name     abv  acid sugar type   index instructions        ingredients   ncotw
##    <chr>  <dbl> <dbl> <dbl> <chr>  <dbl> <chr>               <chr>         <chr>
##  1 Pisco~  12.1  0.68   7.2 eggwh~     5 Dry shake, shake w~ pisco         not ~
##  2 Pisco~  12.1  0.68   7.2 eggwh~     5 Dry shake, shake w~ egg white     not ~
##  3 Pisco~  12.1  0.68   7.2 eggwh~     5 Dry shake, shake w~ lime juice    not ~
##  4 Pisco~  12.1  0.68   7.2 eggwh~     5 Dry shake, shake w~ simple syrup  not ~
##  5 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shake w~ plymouth gin  not ~
##  6 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shake w~ egg white     not ~
##  7 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shake w~ lemon juice   not ~
##  8 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shake w~ grenadine     not ~
##  9 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shake w~ simple syrup  not ~
## 10 Pink ~  12.4  0.64   9   eggwh~     6 Dry shake, shake w~ lairds apple~ not ~
## # ... with 213 more rows
```

## 3. All ingredients in alphabetical order

Before opening the bar, you need to find a reliable supplier that has all the ingredients. You need to send a list of all possible ingredients you will need. They don't need the quantities (i.e. how many of these are needed), just the names of the ingredients.


```r
ingredients_name <- cocktail_fix %>%
  arrange(ingredients) %>%
  distinct(ingredients)
ingredients_name
```

```
## # A tibble: 63 x 1
##    ingredients                   
##    <chr>                         
##  1 absinthe                      
##  2 absolut citron vodka          
##  3 agarclarified grapefruit juice
##  4 amer picon                    
##  5 angostura bitters             
##  6 apple brandy                  
##  7 benedictine                   
##  8 blanco tequila                
##  9 bourbon                       
## 10 campari                       
## # ... with 53 more rows
```

## 4. Number of unique ingredients

How many different ingredients you will need?


```r
count(ingredients_name)
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1    63
```
There are 63 unique ingredients

## 5. What are the top 10 ingredients?

What are the 10 most frequently used ingredients? If there are ties, you can list more than 10.


```r
top10_ingredients <- cocktail_fix %>%
  count(ingredients, sort = T) %>% 
  top_n(10)
```

```
## Selecting by n
```

```r
top10_ingredients
```

```
## # A tibble: 11 x 2
##    ingredients            n
##    <chr>              <int>
##  1 lemon juice           15
##  2 simple syrup          14
##  3 sweet vermouth        13
##  4 angostura bitters     12
##  5 gin                   12
##  6 lime juice            12
##  7 saline solution       10
##  8 water                 10
##  9 rye                    8
## 10 cognac                 6
## 11 luxardo maraschino     6
```

## 6. Which cocktail(s) has/have the most ingredients?

Count the number of ingredients and filter all the cocktails that has that many.


```r
most_ingredients <- cocktail_fix %>%
  count(name, sort = T) %>% 
  top_n(1)
```

```
## Selecting by n
```

```r
most_ingredients
```

```
## # A tibble: 6 x 2
##   name                   n
##   <chr>              <int>
## 1 Blender Margarita      6
## 2 Carbonated Negroni     6
## 3 Clover Club            6
## 4 De La Louisiane        6
## 5 Pink Lady              6
## 6 Vieux Carre            6
```

## 7. How many ingredients appear in only one cocktail (rare ingredient)?


```r
rare_ingredients <- cocktail_fix %>%
  count(ingredients) %>% 
  filter(n == 1)
count(rare_ingredients)
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1    29
```

## 8. Which cocktail has an ingredient that is only used in one cocktail?


```r
only_one <- cocktail_fix %>%
  group_by(ingredients) %>% 
  filter(n() == 1)

cocktail_one <- unique(only_one$name)
cocktail_one
```

```
##  [1] "Pisco Sour"                        "Pink Lady"                        
##  [3] "Clover Club"                       "Blood and Sand"                   
##  [5] "Alexander"                         "Cosmopolitan (Modern/Bastardized)"
##  [7] "20th Century Cocktail"             "Aviation"                         
##  [9] "Last Word"                         "Blackthorn"                       
## [11] "Hanky Panky"                       "Martinez"                         
## [13] "Brooklyn"                          "Rusty Nail"                       
## [15] "De La Louisiane"                   "Blender Daiquiri"                 
## [17] "Blender Margarita"                 "Blender Whiskey Sour"             
## [19] "Gin and Tonic (Dry)"               "Carbonated Negroni"               
## [21] "Gin and Juice Centrifuge"          "Gin and Juice Agar"
```

## 9. What are the cocktails without rare ingredients?


```r
many_ingredients <- cocktail_fix %>%
  group_by(ingredients) %>% 
  filter(n() != 1)

cocktail_many <- unique(many_ingredients$name)
cocktail_many
```

```
##  [1] "Pisco Sour"                        "Pink Lady"                        
##  [3] "Clover Club"                       "Whiskey Sour"                     
##  [5] "Daiquiri with More Lime"           "Honeysuckle"                      
##  [7] "Classic Daiquiri"                  "Blood and Sand"                   
##  [9] "Alexander"                         "Hemingway Daiquiri"               
## [11] "Brown Derby"                       "Cosmopolitan (Modern/Bastardized)"
## [13] "Gold Rush"                         "Southside"                        
## [15] "Bee's Knees"                       "20th Century Cocktail"            
## [17] "Fresh Lime Gimlet"                 "Corpse Reviver #2"                
## [19] "Jack Rose"                         "Margarita"                        
## [21] "Aviation"                          "Sidecar"                          
## [23] "Champs-Elysses"                    "Last Word"                        
## [25] "Between the Sheets"                "Brady Crusta"                     
## [27] "Pegu Club"                         "Blinker"                          
## [29] "Negroni"                           "Blackthorn"                       
## [31] "Hanky Panky"                       "Martinez"                         
## [33] "Manhatan (Boubon, 45% abv)"        "Bobby Burns"                      
## [35] "Rob Roy"                           "Old Pal"                          
## [37] "Vieux Carre"                       "Brooklyn"                         
## [39] "Bijou"                             "Manhatan (Rye, 50% abv)"          
## [41] "Rusty Nail"                        "Improved Whiskey Cocktail"        
## [43] "De La Louisiane"                   "Widow's Kiss"                     
## [45] "Old Fashioned"                     "Blender Daiquiri"                 
## [47] "Blender Margarita"                 "Blender Whiskey Sour"             
## [49] "Carbonated Margarita"              "Carbonated Whiskey Sour"          
## [51] "Gin and Tonic (Dry)"               "Carbonated Negroni"               
## [53] "Gin and Juice Centrifuge"          "Gin and Juice Agar"               
## [55] "Chartruth"
```

## 10. Create a cheat sheet for the bartender!

Create a matrix that shows all cocktail names as rows and all ingredients as columns. When a cocktail requires an ingredient, there should be an "X" in the cell, otherwise, the cell should remain empty. Example:


```r
cheatsheet <- dcast(cocktail_fix, name ~ ingredients, length)
```

```
## Using ncotw as value column: use value.var to override.
```

```r
cheatsheet[cheatsheet == 0] <- ""
cheatsheet[cheatsheet == 1] <- "X"
cheatsheet
```

```
##                                 name absinthe absolut citron vodka
## 1              20th Century Cocktail                              
## 2                          Alexander                              
## 3                           Aviation                              
## 4                        Bee's Knees                              
## 5                 Between the Sheets                              
## 6                              Bijou                              
## 7                         Blackthorn                              
## 8                   Blender Daiquiri                              
## 9                  Blender Margarita                              
## 10              Blender Whiskey Sour                              
## 11                           Blinker                              
## 12                    Blood and Sand                              
## 13                       Bobby Burns                              
## 14                      Brady Crusta                              
## 15                          Brooklyn                              
## 16                       Brown Derby                              
## 17              Carbonated Margarita                              
## 18                Carbonated Negroni                              
## 19           Carbonated Whiskey Sour                              
## 20                    Champs-Elysses                              
## 21                         Chartruth                              
## 22                  Classic Daiquiri                              
## 23                       Clover Club                              
## 24                 Corpse Reviver #2                              
## 25 Cosmopolitan (Modern/Bastardized)                             X
## 26           Daiquiri with More Lime                              
## 27                   De La Louisiane        X                     
## 28                 Fresh Lime Gimlet                              
## 29                Gin and Juice Agar                              
## 30          Gin and Juice Centrifuge                              
## 31               Gin and Tonic (Dry)                              
## 32                         Gold Rush                              
## 33                       Hanky Panky                              
## 34                Hemingway Daiquiri                              
## 35                       Honeysuckle                              
## 36         Improved Whiskey Cocktail                              
## 37                         Jack Rose                              
## 38                         Last Word                              
## 39        Manhatan (Boubon, 45% abv)                              
## 40           Manhatan (Rye, 50% abv)                              
## 41                         Margarita                              
## 42                          Martinez                              
## 43                           Negroni                              
## 44                     Old Fashioned                              
## 45                           Old Pal                              
## 46                         Pegu Club                              
## 47                         Pink Lady                              
## 48                        Pisco Sour                              
## 49                           Rob Roy                              
## 50                        Rusty Nail                              
## 51                           Sidecar                              
## 52                         Southside                              
## 53                       Vieux Carre                              
## 54                      Whiskey Sour                              
## 55                      Widow's Kiss                              
##    agarclarified grapefruit juice amer picon angostura bitters apple brandy
## 1                                                                          
## 2                                                                          
## 3                                                                          
## 4                                                                          
## 5                                                                          
## 6                                                                          
## 7                                                                          
## 8                                                                          
## 9                                                                          
## 10                                                                         
## 11                                                                         
## 12                                                                         
## 13                                                                         
## 14                                                                         
## 15                                         X                 X             
## 16                                                                         
## 17                                                                         
## 18                                                                         
## 19                                                                         
## 20                                                           X             
## 21                                                                         
## 22                                                                         
## 23                                                                         
## 24                                                                         
## 25                                                                         
## 26                                                                         
## 27                                                           X             
## 28                                                                         
## 29                              X                                          
## 30                                                                         
## 31                                                                         
## 32                                                                         
## 33                                                                         
## 34                                                                         
## 35                                                                         
## 36                                                           X             
## 37                                                           X            X
## 38                                                                         
## 39                                                           X             
## 40                                                           X             
## 41                                                                         
## 42                                                           X             
## 43                                                                         
## 44                                                           X             
## 45                                                                         
## 46                                                           X             
## 47                                                                         
## 48                                                                         
## 49                                                           X             
## 50                                                                         
## 51                                                                         
## 52                                                                         
## 53                                                           X             
## 54                                                                         
## 55                                                                        X
##    benedictine blanco tequila bourbon campari
## 1                                            
## 2                                            
## 3                                            
## 4                                            
## 5                                            
## 6                                            
## 7                                            
## 8                                            
## 9                                            
## 10                                           
## 11                                           
## 12                                           
## 13           X                               
## 14                                           
## 15                                           
## 16                                  X        
## 17                          X                
## 18                                          X
## 19                                  X        
## 20                                           
## 21                                           
## 22                                           
## 23                                           
## 24                                           
## 25                                           
## 26                                           
## 27           X                               
## 28                                           
## 29                                           
## 30                                           
## 31                                           
## 32                                  X        
## 33                                           
## 34                                           
## 35                                           
## 36                                           
## 37                                           
## 38                                           
## 39                                  X        
## 40                                           
## 41                          X                
## 42                                           
## 43                                          X
## 44                                  X        
## 45                                          X
## 46                                           
## 47                                           
## 48                                           
## 49                                           
## 50                                           
## 51                                           
## 52                                           
## 53           X                               
## 54                                           
## 55           X                               
##    centrifugeclarified grapefruit juice champagne acid cherry herring
## 1                                                                    
## 2                                                                    
## 3                                                                    
## 4                                                                    
## 5                                                                    
## 6                                                                    
## 7                                                                    
## 8                                                                    
## 9                                                                    
## 10                                                                   
## 11                                                                   
## 12                                                                  X
## 13                                                                   
## 14                                                                   
## 15                                                                   
## 16                                                                   
## 17                                                                   
## 18                                                                   
## 19                                                                   
## 20                                                                   
## 21                                                                   
## 22                                                                   
## 23                                                                   
## 24                                                                   
## 25                                                                   
## 26                                                                   
## 27                                                                   
## 28                                                                   
## 29                                                                   
## 30                                    X              X               
## 31                                                                   
## 32                                                                   
## 33                                                                   
## 34                                                                   
## 35                                                                   
## 36                                                                   
## 37                                                                   
## 38                                                                   
## 39                                                                   
## 40                                                                   
## 41                                                                   
## 42                                                                   
## 43                                                                   
## 44                                                                   
## 45                                                                   
## 46                                                                   
## 47                                                                   
## 48                                                                   
## 49                                                                   
## 50                                                                   
## 51                                                                   
## 52                                                                   
## 53                                                                   
## 54                                                                   
## 55                                                                   
##    clarified lime juice clarified lime juice or champagne acid cognac cointreau
## 1                                                                              
## 2                                                                   X          
## 3                                                                              
## 4                                                                              
## 5                                                                   X          
## 6                                                                              
## 7                                                                              
## 8                                                                              
## 9                                                                             X
## 10                                                                             
## 11                                                                             
## 12                                                                             
## 13                                                                             
## 14                                                                  X          
## 15                                                                             
## 16                                                                             
## 17                    X                                                        
## 18                                                           X                 
## 19                    X                                                        
## 20                                                                  X          
## 21                    X                                                        
## 22                                                                             
## 23                                                                             
## 24                                                                            X
## 25                                                                            X
## 26                                                                             
## 27                                                                             
## 28                                                                             
## 29                                                                             
## 30                                                                             
## 31                    X                                                        
## 32                                                                             
## 33                                                                             
## 34                                                                             
## 35                                                                             
## 36                                                                             
## 37                                                                             
## 38                                                                             
## 39                                                                             
## 40                                                                             
## 41                                                                            X
## 42                                                                             
## 43                                                                             
## 44                                                                             
## 45                                                                             
## 46                                                                             
## 47                                                                             
## 48                                                                             
## 49                                                                             
## 50                                                                             
## 51                                                                  X         X
## 52                                                                             
## 53                                                                  X          
## 54                                                                             
## 55                                                                             
##    cranberry juice crem de violette curacao demerara syrup dolin dry vermouth
## 1                                                                            
## 2                                                        X                   
## 3                                 X                                          
## 4                                                                            
## 5                                         X                                  
## 6                                                                            
## 7                                                                            
## 8                                                                            
## 9                                                                            
## 10                                                                           
## 11                                                                           
## 12                                                                           
## 13                                                                           
## 14                                        X                                  
## 15                                                                           
## 16                                                                           
## 17                                                                           
## 18                                                                           
## 19                                                                           
## 20                                                                           
## 21                                                                           
## 22                                                                           
## 23                                                                          X
## 24                                                                           
## 25               X                                                           
## 26                                                                           
## 27                                                                           
## 28                                                                           
## 29                                                                           
## 30                                                                           
## 31                                                                           
## 32                                                                           
## 33                                                                           
## 34                                                                           
## 35                                                                           
## 36                                                                           
## 37                                                                           
## 38                                                                           
## 39                                                                           
## 40                                                                           
## 41                                                                           
## 42                                                                           
## 43                                                                           
## 44                                                                           
## 45                                                                           
## 46                                        X                                  
## 47                                                                           
## 48                                                                           
## 49                                                                           
## 50                                                                           
## 51                                                                           
## 52                                                                           
## 53                                                                           
## 54                                                                           
## 55                                                                           
##    drambuie dry vermouth egg white fernet branca gin gin  grapefruit juice
## 1                                                  X                      
## 2                                                                         
## 3                                                                         
## 4                                                  X                      
## 5                                                                         
## 6                                                  X                      
## 7                                                                         
## 8                                                                         
## 9                                                                         
## 10                                                                        
## 11                                                                       X
## 12                                                                        
## 13                                                                        
## 14                                                                        
## 15                     X                                                  
## 16                                                                       X
## 17                                                                        
## 18                                                      X                 
## 19                                                                        
## 20                                                                        
## 21                                                                        
## 22                                                                        
## 23                               2                                        
## 24                                                 X                      
## 25                                                                        
## 26                                                                        
## 27                                                                        
## 28                                                 X                      
## 29                                                 X                      
## 30                                                 X                      
## 31                                                 X                      
## 32                                                                        
## 33                                             X   X                      
## 34                                                                       X
## 35                                                                        
## 36                                                                        
## 37                                                                        
## 38                                                                        
## 39                                                                        
## 40                                                                        
## 41                                                                        
## 42                                                                        
## 43                                                 X                      
## 44                                                                        
## 45                     X                                                  
## 46                                                 X                      
## 47                               X                                        
## 48                               X                                        
## 49                                                                        
## 50        X                                                               
## 51                                                                        
## 52                                                 X                      
## 53                                                                        
## 54                               X                                        
## 55                                                                        
##    green chartreuse grenadine heavy cream hellfire bitters honey syrup
## 1                                                                     
## 2                                       X                             
## 3                                                                     
## 4                                                                    X
## 5                                                                     
## 6                 X                                                   
## 7                                                                     
## 8                                                                     
## 9                                                        X            
## 10                                                                    
## 11                                                                    
## 12                                                                    
## 13                                                                    
## 14                                                                    
## 15                                                                    
## 16                                                                   X
## 17                                                                    
## 18                                                                    
## 19                                                                    
## 20                X                                                   
## 21                X                                                   
## 22                                                                    
## 23                                                                    
## 24                                                                    
## 25                                                                    
## 26                                                                    
## 27                                                                    
## 28                                                                    
## 29                                                                    
## 30                                                                    
## 31                                                                    
## 32                                                                   X
## 33                                                                    
## 34                                                                    
## 35                                                                   X
## 36                                                                    
## 37                          X                                         
## 38                X                                                   
## 39                                                                    
## 40                                                                    
## 41                                                                    
## 42                                                                    
## 43                                                                    
## 44                                                                    
## 45                                                                    
## 46                                                                    
## 47                          X                                         
## 48                                                                    
## 49                                                                    
## 50                                                                    
## 51                                                                    
## 52                                                                    
## 53                                                                    
## 54                                                                    
## 55                                                                    
##    lairds applejack bottled in bond lemon juice lillet blanc lime juice
## 1                                             X            X           
## 2                                                                      
## 3                                             X                        
## 4                                             X                        
## 5                                             X                        
## 6                                                                      
## 7                                                                      
## 8                                                                     X
## 9                                                                     X
## 10                                            X                        
## 11                                                                     
## 12                                                                     
## 13                                                                     
## 14                                            X                        
## 15                                                                     
## 16                                                                     
## 17                                                                     
## 18                                                                     
## 19                                                                     
## 20                                            X                        
## 21                                                                     
## 22                                                                    X
## 23                                            X                        
## 24                                            X            X           
## 25                                                                    X
## 26                                                                    X
## 27                                                                     
## 28                                                                    X
## 29                                                                     
## 30                                                                     
## 31                                                                     
## 32                                            X                        
## 33                                                                     
## 34                                                                    X
## 35                                                                    X
## 36                                                                     
## 37                                            X                        
## 38                                                                    X
## 39                                                                     
## 40                                                                     
## 41                                                                    X
## 42                                                                     
## 43                                                                     
## 44                                                                     
## 45                                                                     
## 46                                                                    X
## 47                                X           X                        
## 48                                                                    X
## 49                                                                     
## 50                                                                     
## 51                                            X                        
## 52                                            X                        
## 53                                                                     
## 54                                            X                        
## 55                                                                     
##    luxardo maraschino luxardo marschino old tom gin orange bitters orange juice
## 1                                                                              
## 2                                                                              
## 3                   X                                                          
## 4                                                                              
## 5                                                                              
## 6                                                                X             
## 7                                                                X             
## 8                                                                              
## 9                                                                              
## 10                                                                            X
## 11                                                                             
## 12                                                                            X
## 13                                                                             
## 14                  X                                                          
## 15                  X                                                          
## 16                                                                             
## 17                                                                             
## 18                                                                             
## 19                                                                             
## 20                                                                             
## 21                                                                             
## 22                                                                             
## 23                                                                             
## 24                                                                             
## 25                                                                             
## 26                                                                             
## 27                                                                             
## 28                                                                             
## 29                                                                             
## 30                                                                             
## 31                                                                             
## 32                                                                             
## 33                                                                             
## 34                  X                                                          
## 35                                                                             
## 36                  X                                                          
## 37                                                                             
## 38                                    X                                        
## 39                                                                             
## 40                                                                             
## 41                                                                             
## 42                  X                             X              X             
## 43                                                                             
## 44                                                                             
## 45                                                                             
## 46                                                               X             
## 47                                                                             
## 48                                                                             
## 49                                                                             
## 50                                                                             
## 51                                                                             
## 52                                                                             
## 53                                                                             
## 54                                                                             
## 55                                                                             
##    peychauds bitters pisco plymouth gin quinine simple syrup raspberry syrup
## 1                                                                           
## 2                                                                           
## 3                                     X                                     
## 4                                                                           
## 5                                                                           
## 6                                                                           
## 7                                     X                                     
## 8                                                                           
## 9                                                                           
## 10                                                                          
## 11                                                                         X
## 12                                                                          
## 13                                                                          
## 14                                                                          
## 15                                                                          
## 16                                                                          
## 17                                                                          
## 18                                                                          
## 19                                                                          
## 20                                                                          
## 21                                                                          
## 22                                                                          
## 23                                    X                                    X
## 24                                                                          
## 25                                                                          
## 26                                                                          
## 27                 X                                                        
## 28                                                                          
## 29                                                                          
## 30                                                                          
## 31                                                         X                
## 32                                                                          
## 33                                                                          
## 34                                                                          
## 35                                                                          
## 36                                                                          
## 37                                                                          
## 38                                    X                                     
## 39                                                                          
## 40                                                                          
## 41                                                                          
## 42                                                                          
## 43                                                                          
## 44                                                                          
## 45                                                                          
## 46                                                                          
## 47                                    X                                     
## 48                       X                                                  
## 49                                                                          
## 50                                                                          
## 51                                                                          
## 52                                                                          
## 53                 X                                                        
## 54                                                                          
## 55                                                                          
##    rye saline saline solution scotch simple simple syrup sloe gin
## 1                                                                
## 2                                                                
## 3                                                                
## 4                                                                
## 5                                                                
## 6                                                                
## 7                                                               X
## 8                           X                                    
## 9                                                                
## 10                          X                                    
## 11   X                                                           
## 12                                 X                             
## 13                                 X                             
## 14                                                               
## 15   X                                                           
## 16                                                               
## 17                          X                          X         
## 18                          X                                    
## 19                          X                          X         
## 20                                                     X         
## 21                                                               
## 22                                                     X         
## 23                                                               
## 24                                                               
## 25                                                               
## 26                                                     X         
## 27   X                                                           
## 28                                                     X         
## 29                          X                                    
## 30                                        X                      
## 31                          X                                    
## 32                                                               
## 33                                                               
## 34                          X                                    
## 35                                                               
## 36   X                                                 X         
## 37                                                               
## 38          X                                                    
## 39                                                               
## 40   X                                                           
## 41                          X                          X         
## 42                                                               
## 43                                                               
## 44                                                     X         
## 45   X                                                           
## 46                                                               
## 47                                                     X         
## 48                                                     X         
## 49                                 X                             
## 50                                 X                             
## 51                                                     X         
## 52                                                     X         
## 53   X                                                           
## 54   X                      X                          X         
## 55                                                               
##    sugard proof rum sugared proof rye sweet vermouth water white crme de cacao
## 1                                                                            X
## 2                                                                             
## 3                                                                             
## 4                                                                             
## 5                                                                             
## 6                                                  X                          
## 7                                                  X                          
## 8                 X                                      X                    
## 9                                                        X                    
## 10                                  X                    X                    
## 11                                                                            
## 12                                                 X                          
## 13                                                 X                          
## 14                                                                            
## 15                                                                            
## 16                                                                            
## 17                                                       X                    
## 18                                                 X     X                    
## 19                                                       X                    
## 20                                                                            
## 21                                                       X                    
## 22                                                                            
## 23                                                                            
## 24                                                                            
## 25                                                                            
## 26                                                                            
## 27                                                 X                          
## 28                                                                            
## 29                                                       X                    
## 30                                                       X                    
## 31                                                       X                    
## 32                                                                            
## 33                                                 X                          
## 34                                                                            
## 35                                                                            
## 36                                                                            
## 37                                                                            
## 38                                                                            
## 39                                                 X                          
## 40                                                 X                          
## 41                                                                            
## 42                                                 X                          
## 43                                                 X                          
## 44                                                                            
## 45                                                                            
## 46                                                                            
## 47                                                                            
## 48                                                                            
## 49                                                 X                          
## 50                                                                            
## 51                                                                            
## 52                                                                            
## 53                                                 X                          
## 54                                                                            
## 55                                                                            
##    white mezcal white rum yellow chartreuse
## 1                                          
## 2                                          
## 3                                          
## 4                                          
## 5                       X                  
## 6                                          
## 7                                          
## 8                                          
## 9             X                           X
## 10                                         
## 11                                         
## 12                                         
## 13                                         
## 14                                         
## 15                                         
## 16                                         
## 17                                         
## 18                                         
## 19                                         
## 20                                         
## 21                                         
## 22                      X                  
## 23                                         
## 24                                         
## 25                                         
## 26                      X                  
## 27                                         
## 28                                         
## 29                                         
## 30                                         
## 31                                         
## 32                                         
## 33                                         
## 34                      X                  
## 35                      X                  
## 36                                         
## 37                                         
## 38                                         
## 39                                         
## 40                                         
## 41                                         
## 42                                         
## 43                                         
## 44                                         
## 45                                         
## 46                                         
## 47                                         
## 48                                         
## 49                                         
## 50                                         
## 51                                         
## 52                                         
## 53                                         
## 54                                         
## 55                                        X
```


Congrats, the bar is now officially open!

![](https://i.pinimg.com/originals/4e/c1/0c/4ec10c9d32b2c7c28b4b638c7f809ec5.gif){width="320"}
