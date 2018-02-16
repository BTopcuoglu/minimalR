---
title: "Session 4"
output: markdown_document
permalink: aggregating_data
---


## Learning goals
* Sourcing files
* Functions
* Logicals (ifelse)
* Factors
* Matrices
* Aggregating data by a categorical variable
* Representing categorical data with a bar plot
* Error bars


We're making some pretty good progress so far, well done! Over the last three sessions, we've gotten pretty good at merging our metadata and diversity data. It's kind of a pain every time we close R and come back that we have to remember what series of commands to execute to get things back to how they were before. Surely we're going to make some mistakes. Let's streamline things by putting our code into another file. First, let's create a folder in our project called `code` and we'll create a file called `baxter.R`. Within this file copy and paste the following commands...


```r
library("dplyr")
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
metadata <- read.table(file="data/baxter.metadata.tsv", header=T, sep='\t')
metadata$sample <- as.character(metadata$sample)
metadata$Hx_Prev <- as.logical(metadata$Hx_Prev)
metadata$Smoke <- as.logical(metadata$Smoke)
metadata$Diabetic <- as.logical(metadata$Diabetic)
metadata$Hx_Fam_CRC <- as.logical(metadata$Hx_Fam_CRC)
metadata$Hx_of_Polyps <- as.logical(metadata$Hx_of_Polyps)

metadata$NSAID <- as.logical(metadata$NSAID)
metadata$Diabetes_Med <- as.logical(metadata$Diabetes_Med)
metadata$stage <- as.factor(metadata$stage)

metadata[metadata$Height == 0 & !is.na(metadata$Height), "Height"] <- NA
metadata[metadata$Weight == 0 & !is.na(metadata$Weight), "Weight"] <- NA

alpha <- read.table(file="data/baxter.groups.ave-std.summary", header=T)
alpha$group <- as.character(alpha$group)
alpha_mean <- alpha[alpha$method == 'ave', ]

meta_alpha <- inner_join(metadata, alpha_mean, by=c("sample"="group"))
```

Now we can then load this code into R by running `source("code/baxter.R")` whenever we want to load the `meta_alpha` data frame. You might recall that in Sessions 1 and 2 we also made data frames where we had PCoA data. What would we do to make a `meta_pcoa` data frame? Well, we could tack code on to the end of our `baxter.R` file or we could make a new file for the PCoA data. Neither of these are super attractive. The first option is less desirable because I might not want to load both the PCoA and alpha diversity data. I would like to have more control. The second option is not desirable because I'm going to end up repeating a lot of code making things difficult to maintain. In cases where you need to reduce code repetition or want to make your code more modular, we would like to create a function.

The syntax you use to create a function looks like this.

```r
my_killer_function <- function(argument1, argument2){

	... special sauce ...

}
```

We'll get back to creating a function to generate `meta_alpha` and `meta_pcoa` in a moment, but let's play a bit with some functions to drive home some concepts. I'd like to create a function that tells me whether a number is even or odd.


```r
get_even_odd <- function(number){
	result <- ifelse(number %% 2 == 0, 'even', 'odd')
	return(result)
}
```

The syntax is essentially what we had in the template, the only difference is the inclusion of the special sauce that gives the function its utility. Here we see two new functions `%%` and `ifelse`. The `%%` is the modulus operator that tells us the remainder of dividing our number by another number - in this case 2. If we have `4 %% 2`, then the result will be `0` and if we have `23 %% 2`, then the result will be `1`. The `ifelse` function takes three arguments: a operation that returns `TRUE` or `FALSE` values and the value that the variable will take if the operation returns a `TRUE` or a `FALSE`. If we give `even_odd` `4`, then we should get back `"even"`. If we give `get_even_odd` `23`, then we should get back `odd`. Give it a shot. This will also work if we give `even_odd` a vector of values. Try this


```r
get_even_odd(1:10)
```

```
##  [1] "odd"  "even" "odd"  "even" "odd"  "even" "odd"  "even" "odd"  "even"
```

The current version of `get_even_odd` is pretty explicit about what's going on. It takes the argument `number`, generates a variable `result` and `return`s the value from the function. Strictly speaking the `return` function is not necessary and the `result <-` isn't either. This is because a function automatically returns the last value that is generated.


```r
get_even_odd <- function(number){
	ifelse(number %% 2 == 0, 'even', 'odd')
}
```

Go ahead and give this leaner version of the function a spin to show yourself that it works as expected. Why would you prefer one version of the function over another? As your functions get more complex, code like the first version is a bit more readable. But it is also more verbose and requires more typing relative to the second version.

#### Activity 1
To calculate a person's body mass index you need to know their weight in kilograms and height in meters

```r
bmi <- weight_kg / height_m ^ 2
```

Write a function that takes both parameters and returns the BMI. Use an online BMI calculator like the one at the [NIH](http://www.nhlbi.nih.gov/health/educational/lose_wt/BMI/bmi-m.htm) to double check your values. Try it out on the data in `meta_alpha` to generate a `bmi` column in the data frame.

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
get_bmi <- function(weight_kg, height_m){
	weight_kg / height_m ^ 2
}
```

</div>

We would also like to have a BMI category column in our data frame so that we can partition our data. Let's make a function! We can use the thresholds defined at the NIH site to create our categories. And we can probably get by using the `ifelse` command by nesting additional `ifelse` commands within the `FALSE` argument of the `ifelse` statement.


```r
get_bmi_category <- function(bmi){
	ifelse(bmi >= 30, "obese", ifelse(bmi >= 25, "overweight", ifelse(bmi >= 18.5, "normal", "underweight")))
}

get_bmi_category(c(10, 15, 20, 25, 30, 35, 40))
```

```
## [1] "underweight" "underweight" "normal"      "overweight"  "obese"      
## [6] "obese"       "obese"
```

Let's step through this assuming we have a BMI of 17. The first question is whether 17 is greater than or equal to 30. Nope. Next, is 17 greater than 25? Nope. Is 17 greater than 18.5? Nope. Then we return `"underweight"`. Alternatively, if the comparison were true at a previously step, we would have stopped and returned the `TRUE` value. You can probably appreciate that `ifelse` statement is a beast. It works, but is pretty hard to read and figure out what is going on. As a small point of style, we like to give our variables names that are names and our functions that are verbs. That way it is easier to understand what is going on when we read our code.


#### Activity 2
Generate a function `is_obese` that takes in a person's `bmi` and returns a `TRUE` or `FALSE` value indicating whether the person is obese.

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
is_obese <- function(bmi){
	return(get_bmi_category(bmi) == "obese")
}
```
</div>


#### Activity 3
* Convert the code in `baxter.R` into a function that we'll call `get_meta_alpha`. Don't include the `library(dplyr)` statement in the new function (leave it at line 1 of the file). The `get_meta_alpha` function won't take any arguments.
* Create a second function in `baxter.R` that we'll call `get_meta_pcoa`. Again, this won't take any arguments, but will return a single data frame analogous to that of `meta_alpha`
* What do you notice about these two functions? Can you solve this problem by creating a third function - `get_meta` - that you call in `get_meta_alpha` and `get_meta_pcoa`?
* Add `get_bmi`, `get_bmi_category`, and `is_obese` to `baxter.R`. Call these functions in `get_meta` to add three new columns: BMI, BMIcat, and is_obese to the data frame.
* Restart R and run

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
# The following solution is also saved as code/baxter_soln.R for future reference

library("dplyr")


# This function calculates a BMI value. Note that it assumes
# the person's weight is in kg and their height is in cm.
get_bmi <- function(weight_kg, height_cm){
  height_m <- height_cm/100
  bmi <- weight_kg / height_m ^2
  return(bmi)
}


# This function takes in a bmi value and based on NIH recommendations
# returns the BMI category that the value falls within
get_bmi_category <- function(bmi){
  category <- ifelse(bmi >= 30, "obese",
                     ifelse(bmi >=25, "overweight",
                            ifelse(bmi >= 18.5, "normal", "underweight")))
  return(category)
}


# this function takes in a BMI value and reports whether the individual is obese
is_obese <- function(bmi){
  get_bmi_category(bmi) == "obese"
}


get_meta <- function(){
	metadata <- read.table(file="data/baxter.metadata.tsv", header=T, sep='\t', stringsAsFactors=F)
	metadata$sample <- as.character(metadata$sample)
	metadata$Hx_Prev <- as.logical(metadata$Hx_Prev)
	metadata$Smoke <- as.logical(metadata$Smoke)
	metadata$Diabetic <- as.logical(metadata$Diabetic)
	metadata$Hx_Fam_CRC <- as.logical(metadata$Hx_Fam_CRC)
	metadata$Hx_of_Polyps <- as.logical(metadata$Hx_of_Polyps)

	metadata$NSAID <- as.logical(metadata$NSAID)
	metadata$Diabetes_Med <- as.logical(metadata$Diabetes_Med)
	metadata$stage <- as.factor(metadata$stage)

	metadata[metadata$Height == 0 & !is.na(metadata$Height), "Height"] <- NA
	metadata[metadata$Weight == 0 & !is.na(metadata$Weight), "Weight"] <- NA

	metadata$BMI <- get_bmi(weight_kg = metadata$Weight, height_cm = metadata$Height)
	metadata$BMIcat <- get_bmi_category(metadata$BMI)
	metadata$is_obese <- is_obese(metadata$BMI)

	return(metadata)
}


get_meta_alpha <- function(){
	metadata <- get_meta()

	alpha <- read.table(file="data/baxter.groups.ave-std.summary", header=T)
	alpha$group <- as.character(alpha$group)
	alpha_mean <- alpha[alpha$method == 'ave', ]

	meta_alpha <- inner_join(metadata, alpha_mean, by=c("sample"="group"))
}


get_meta_pcoa <- function(){
	metadata <- get_meta()

	pcoa <- read.table(file="data/baxter.thetayc.pcoa.axes", header=T)
	pcoa$group <- as.character(pcoa$group)

	meta_pcoa <- inner_join(metadata, pcoa, by=c("sample"="group"))
}
```

</div>


```r
source("code/baxter.R")
meta_alpha <- get_meta_alpha()
```

Let's pause and think about what we've been able to do with functions in this session. First, we've reduced the amount of duplicate code in our analysis. If I want the "Stage" column to be numerical instead of a factor, I only need to change one line and run one function. Similarly, if the guidelines for BMI thresholds change, I have those conversions in a single location that I can update without having to hunt for all of the places where I go from BMI to a category. Second, we've created a resource that others might find useful. Your future projects may need to generate BMI values and categories. Instead of trying to figure out how to code it up and test it, you can use your code from this project. Finally, we've put all of these functions into a file that will allow us to `source` it so that those features can persist long after our current session. Do you recall back in Session 1 we talked about the DRY - Don't Repeat Yourself - principle? Well with functions, we can make our code even more dry.

Let's do one more thing with `code/baxter.R`. If we are going to generate a handful of plots, we'd like them all to have the same coloring scheme. We could define `dx_color` and `dx_pch` in every script where we generate a figure, but let's do that in `code/baxter.R` instead. At the bottom of `code/baxter.R` put the following:

````
dx_color <- c(normal="black", adenoma="blue", cancer="red")
dx_pch <- c(normal=17, adenoma=18, cancer=19)
sex_color <- c(f="red", m="blue")
sex_symbol <- c(17, 19)
````

This way, if we want to use different colors or symbols, we only need to change them in one place. For instance, we may decide we don't want the colors in `sex_color` to overlap with those in `dx_color`. We could change that in `code/baxter.R` and we'll be good to go. Again, if we run `source('code/baxter.R')` at the top of a file, we'll automatically have all of the functions in that file available to us as well as the various plotting symbols and color schemes. Be sure to run `source('code/baxter.R')` before going further. If you get confused or lost along the way, you can run `source('code/baxter_soln.R')` instead - that's the code that I generated as the solution to Activity 3.


I'd like to start exploring how our continuous data like Shannon diversity vary by some of our categorical data such as cancer status, gender, and BMI category. One method we can use to explore such relationships is to use a bar plot. Here we'll plot the height of a bar as the average Shannon diversity for the individuals within a specific category. To calculate the average across a vector of numbers you can use the `mean` function.


#### Activity 4
Calculate the average Shannon diversity for those individuals with normal colons (`normal_mean`). Repeat for those with adenomas (`adenoma_mean`) and cancer (`cancer_mean`).

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
meta_alpha <- get_meta_alpha()

normal <- meta_alpha[meta_alpha$dx == "normal", "shannon"]
normal_mean <- mean(normal)

adenoma <- meta_alpha[meta_alpha$dx == "adenoma", "shannon"]
adenoma_mean <- mean(adenoma)

cancer <- meta_alpha[meta_alpha$dx == "cancer", "shannon"]
cancer_mean <- mean(cancer)
```

</div>


For a few conditions, this is fine, but if we wanted to get the average diversity for each center or we wanted to get the average diversity for men or women with adenomas, the expressions would become unwieldy. Without using any special packages we could use the `aggregate` function


```r
aggregate(shannon ~ BMIcat, data=meta_alpha, FUN=mean)
```

```
##        BMIcat  shannon
## 1      normal 3.598280
## 2       obese 3.449775
## 3  overweight 3.616646
## 4 underweight 3.371101
```

```r
aggregate(shannon ~ BMIcat + Gender, data=meta_alpha, FUN=mean)
```

```
##        BMIcat Gender  shannon
## 1      normal      f 3.590398
## 2       obese      f 3.427478
## 3  overweight      f 3.617696
## 4 underweight      f 3.371101
## 5      normal      m 3.610474
## 6       obese      m 3.472073
## 7  overweight      m 3.615995
```

```r
aggregate(cbind(shannon, sobs) ~ BMIcat + Gender, data=meta_alpha, FUN=mean)
```

```
##        BMIcat Gender  shannon     sobs
## 1      normal      f 3.590398 216.8960
## 2       obese      f 3.427478 188.6327
## 3  overweight      f 3.617696 210.7003
## 4 underweight      f 3.371101 219.1711
## 5      normal      m 3.610474 205.2582
## 6       obese      m 3.472073 192.5826
## 7  overweight      m 3.615995 208.3659
```

Do you see what's happening in these commands? Do you remember what the `~` represents? These commands are asking R to aggregate the variables on the left side of the `~` using the variables on the right side and to calculate the average value within each category. The other new function you see here is `cbind`, which takes two vectors that are the same length and binds them together to make a new object with two columns (think "column bind"). There's also a `rbind`, which binds together things by rows (think "row bind").

Something you may notice is that the rows of the output aren't in alphabetical order. Perhaps we'd like to treat the `BMIcat` column as ordinal data. We can create an ordinal version of `BMIcat` by using the `factor` function:


```r
BMIordinal <- factor(meta_alpha$BMIcat, levels=c("underweight", "normal","overweight", "obese"))
aggregate(shannon ~ BMIordinal, data=meta_alpha, FUN=mean)
```

```
##    BMIordinal  shannon
## 1 underweight 3.371101
## 2      normal 3.598280
## 3  overweight 3.616646
## 4       obese 3.449775
```

#### Activity 5
Using the `aggregate` function, create a variable, `shannon_colon_mean`, that contains the average Shannon diversity for each diagnosis. Treat the diagnosis groups as ordinal data. This object will replace the values in `normal_mean`, `adenoma_mean`, and `cancer_mean`. What type of variable is this?

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
dx_ordinal <- factor(meta_alpha$dx, levels=c("normal", "adenoma", "cancer"))
shannon_colon_mean <- aggregate(shannon ~ dx_ordinal, data=meta_alpha, FUN=mean)
str(shannon_colon_mean)
```

```
## 'data.frame':	3 obs. of  2 variables:
##  $ dx_ordinal: Factor w/ 3 levels "normal","adenoma",..: 1 2 3
##  $ shannon   : num  3.58 3.58 3.52
```

</div>

Alright, now we'd like to plot these data as a bar plot to see similar the Shannon diversity values are in these groups of individuals.


```r
barplot(height=shannon_colon_mean$shannon)
```

![plot of chunk unnamed-chunk-14](assets/images/04_aggregating_data//unnamed-chunk-14-1.png)

Well, that's a start. We will find that most of the plotting arguments are consistent across funtions. We can do this...


```r
barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5), ylab="Shannon Diversity Index", col=dx_color[shannon_colon_mean$dx_ordinal])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

To label the actual bars, we need to use the `names.arg` argument. This argument takes a vector of names, which we can take from `shannon_colon_mean$dx`


```r
barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5), ylab="Shannon Diversity Index", col=dx_color[as.character(shannon_colon_mean$dx_ordinal)], names.arg=shannon_colon_mean$dx)
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

Nice, that's looking better. You'll notice a subtle function call that we added, `col=dx_color[as.character(shannon_colon_mean$dx_ordinal)]`. That `as.character` function call, converts the output of `shannon_colon_mean$dx_ordinal` to a character vector. What is it otherwise? If you run `str(shannon_colon_mean)`, you'll notice that the `dx` column is a "`Factor w/ 3 levels "normal","adenoma",..: 1 2 3`". A factor represents categorical data. R stores a factor as a numerical vector (i.e. 1, 2, 3), but it gives those numbers character values (i.e. "normal", "adenoma", "cancer"). This can be a pain and is a frequent source of frustration in R. If the values in `dx_color` were not in the "correct" order, the colors would be screwed up because the values of `shannon_colon_mean$dx_ordinal` that go into `dx_color` are actually numbers. But we want to pull values out of `dx_color` using the diagnosis names. We don't really need `as.character` here, but it is a good practice to use it.

Something we'd like to do to make the figure more "publication quality" is to format the categories so the first letter is capitalized. We could do some fancy string manipulations to achieve this, but for now, let's create a named vector to do the conversion. Can you create a vector called `dx_convert` so that if we call `dx_convert['normal']` we'll get back `"Normal"`? Go ahead and add this to the barplot command so the categories are properly named.

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
dx_convert <- c(normal="Normal", adenoma="Adenoma", cancer="Cancer")
barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5), ylab="Shannon Diversity Index", col=dx_color[shannon_colon_mean$dx], names.arg=dx_convert[shannon_colon_mean$dx_ordinal])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

</div>

I like to have a box around my plots, we can get a box by using the `box` function


```r
dx_convert <- c(normal="Normal", adenoma="Adenoma", cancer="Cancer")
barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5), ylab="Shannon Diversity Index", col=dx_color[shannon_colon_mean$dx], names.arg=dx_convert[shannon_colon_mean$dx_ordinal])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

```r
box()
```

```
## Error in box(): plot.new has not been called yet
```

At this point, there doesn't appear to be a difference in diversity beetween the three diagnosis groups. We can test this using the `summary` and `aov` functions.


```r
summary(aov(shannon ~ dx, data=meta_alpha))
```

```
##              Df Sum Sq Mean Sq F value Pr(>F)
## dx            2   0.26  0.1316   0.631  0.533
## Residuals   487 101.61  0.2087
```

With a P-value of 0.5327414 it's clear that there isn't a significant effect of diagnosis on Shannon diversity. This test tells us that the variation between individuals within a category is greater than the differences between the categories. Let's try to visualize the variation between individuals within a category. We'd like to plot the standard error for each category. We'll start by defining a `se` function. The standard error is the square root (`sqrt`) of the standard deviation (`sd`) divided by the number of observations minus 1. We can get the number of observations by getting the length (`length`) of the vector containing our diversity values.


```r
se <- function(x){
	sd(x) / sqrt(length(x))
}
```

#### Activity 6
Using our new `se` function, create a variable called `shannon_colon_se`.

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
shannon_colon_se <- aggregate(shannon ~ dx_ordinal, data=meta_alpha, se)
```

</div>

To create our error bars we will make use of the `arrows` command. If you think about it, an error bar is really an arrow where the sides of the arrow point are a 90 degree angle to the shaft of the arrow. How would we go about finding information on how to run this command?

```
arrows                package:graphics                 R Documentation

Add Arrows to a Plot

Description:

     Draw arrows between pairs of points.

Usage:

     arrows(x0, y0, x1 = x0, y1 = y0, length = 0.25, angle = 30,
            code = 2, col = par("fg"), lty = par("lty"),
            lwd = par("lwd"), ...)

Arguments:

  x0, y0: coordinates of points *from* which to draw.

  x1, y1: coordinates of points *to* which to draw.  At least one must
          the supplied

  length: length of the edges of the arrow head (in inches).

   angle: angle from the shaft of the arrow to the edge of the arrow
          head.

    code: integer code, determining _kind_ of arrows to be drawn.

col, lty, lwd: graphical parameters, possible vectors.  ‘NA’ values in
          ‘col’ cause the arrow to be omitted.

     ...: graphical parameters such as ‘xpd’ and the line
          characteristics ‘lend’, ‘ljoin’ and ‘lmitre’: see ‘par’.
```

Cool, this function draws arrows between points. Which points would we like to draw an arrow between? Looking at the arguments description we need to set the coordinates to draw the arrow from (`x0` and `y0`) and the coordinates to draw the arrow to (`x1` and `y1`). They `y0` and `y1` values come from `shannon_colon_mean` and `shannon_colon_se`. We can set the values for all of the columns by giving these arguments these vectors. By what about `x0` and `x1`? Do you think these values will be different from each other? We could guess at the values - perhaps their 1, 2, and 3? What function did we learn before for placing the legend? Give that a shot. For the legend we gave the `n` argument the value of 1, use 3 here and see what we get. Ugh, that's pretty ugly. When we run the `barplot` function we see a plot as an output. But there's other output values when the function is assigned to a variable - these values are the midpoints for the bars.


```r
colon_plot <- barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5),
											ylab="Shannon Diversity Index",
											col=dx_color[shannon_colon_mean$dx_ordinal],
 											names.arg=dx_convert[shannon_colon_mean$dx_ordinal])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

What are the values of `colon_plot`? Do they look like the values you saw with the `locator` function? Great - now we know everything we need to add the error bars.


```r
colon_plot <- barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5),
											ylab="Shannon Diversity Index",
											col=dx_color[shannon_colon_mean$dx_ordinal],
 											names.arg=dx_convert[shannon_colon_mean$dx_ordinal])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

```r
arrows(x0=colon_plot, x1=colon_plot, y0=shannon_colon_mean$shannon,
			 y1=shannon_colon_se$shannon)
```

```
## Error in arrows(x0 = colon_plot, x1 = colon_plot, y0 = shannon_colon_mean$shannon, : object 'colon_plot' not found
```

Ruh roh. Do you see what we did wrong there?


```r
colon_plot <- barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5),
											ylab="Shannon Diversity Index",
											col=dx_color[shannon_colon_mean$dx_ordinal],
 											names.arg=dx_convert[shannon_colon_mean$dx])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

```r
arrows(x0=colon_plot,
				x1=colon_plot,
				y0=shannon_colon_mean$shannon,
 				y1=shannon_colon_mean$shannon+shannon_colon_se$shannon)
```

```
## Error in arrows(x0 = colon_plot, x1 = colon_plot, y0 = shannon_colon_mean$shannon, : object 'colon_plot' not found
```

That's better, but we need to adjust the angle of the arrow heads.


```r
colon_plot <- barplot(height=shannon_colon_mean$shannon, ylim=c(0,5.5),
											ylab="Shannon Diversity Index",
											col=dx_color[shannon_colon_mean$dx_ordinal],
 											names.arg=dx_convert[shannon_colon_mean$dx_ordinal])
```

```
## Error in barplot.default(height = shannon_colon_mean$shannon, ylim = c(0, : object 'dx_color' not found
```

```r
arrows(x0=colon_plot,
				x1=colon_plot,
				y0=shannon_colon_mean$shannon,
 				y1=shannon_colon_mean$shannon+shannon_colon_se$shannon,
				angle=90)
```

```
## Error in arrows(x0 = colon_plot, x1 = colon_plot, y0 = shannon_colon_mean$shannon, : object 'colon_plot' not found
```

Nice.


#### Activity 7
There's been much made about the relationship between obesity category and diversity. The thought is that obese individuals have a lower diversity than normal people. Modify our code to generate a bar plot that displays the variation in Shannon diversity for each obesity category. Is there a significant difference between the categories?

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
bmi_convert <- c(underweight="Underweight", normal="Normal", overweight="Overweight", obese="Obese")

shannon_obesity_mean <- aggregate(shannon ~ BMIordinal, data=meta_alpha, FUN=mean)
shannon_obesity_se <- aggregate(shannon ~ BMIordinal, data=meta_alpha, FUN=se)

obesity_plot <- barplot(height=shannon_obesity_mean$shannon,
												ylim=c(0,4),
												ylab="Shannon Diversity Index",
												names.arg=bmi_convert[shannon_obesity_mean$BMIordinal])
arrows(x0=obesity_plot,
				x1=obesity_plot,
				y0=shannon_obesity_mean$shannon,
 				y1=shannon_obesity_mean$shannon+shannon_obesity_se$shannon,
				angle=90)
```

![plot of chunk unnamed-chunk-26](assets/images/04_aggregating_data//unnamed-chunk-26-1.png)

</div>


A three or four category bar plot is nice and all, but really isn't that interesting. We'd like to know whether the Shannon diversity varies between diagnosis category and gender. We think we'd like to see the bars clustered by gender, but we might also want to see them clustered by category. The first thing we need to do is to use our aggregate command. Can you create the `shannon_dx_gender_mean` and `shannon_dx_gender_se` variable using aggregate commands?


<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
shannon_dx_gender_mean <- aggregate(shannon ~ dx_ordinal + Gender, data=meta_alpha, FUN=mean)
shannon_dx_gender_se <- aggregate(shannon ~ dx_ordinal + Gender, data=meta_alpha, FUN=se)
```

</div>


Great. For the `barplot` function we need to give the function heights as a vector or matrix (see `?barplot`). Previously, we supplied a vector, now we'd like to supply a matrix. But we haven't seen matrices yet! Matrices are essentially the same as a data frame, except that all of the values in the matrix are the same type. In a data frame we might have had a column of text, numbers, or booleans. But all of the values in a matrix must be text, numbers, or booleans, it can't change between columns. Let's start by generating a generic matrix.


```r
my_vector <- 1:12
my_first_matrix <- matrix(data=my_vector, nrow=3, byrow=T)
my_second_matrix <- matrix(data=my_vector, nrow=3, byrow=F)
```

What's the difference between the two matrices? That's right, in the first, the numbers are laid out in the matrix by rows (`byrow=T`) and in the second they're laid out by columns (`byrow=F`). We'd like to layout the values of `shannon_dx_gender_mean` in a matrix where the rows represent the diagnosis (`dx`) and the columns represent gender. What should the code look like?


```r
mean_matrix <- matrix(shannon_dx_gender_mean$shannon, nrow=3, byrow=F)
rownames(mean_matrix) <- c("normal", "adenoma", "cancer")
colnames(mean_matrix) <- c("female", "male")
```

Now let's give that to `barplot`


```r
barplot(mean_matrix, beside=T, ylim=c(0.0,5.5))
```

![plot of chunk unnamed-chunk-30](assets/images/04_aggregating_data//unnamed-chunk-30-1.png)

We can transpose the columns and rows of a matrix using the `t` function. With barplot, this will switch the grouping


```r
barplot(t(mean_matrix), beside=T, ylim=c(0.0,5.5))
```

![plot of chunk unnamed-chunk-31](assets/images/04_aggregating_data//unnamed-chunk-31-1.png)

Let's stick with the first and think about adding our error bars. The output of the bar plot is actually a matrix. We can convert this to a vector by using the `as.vector` command


```r
matrix_plot <- barplot(mean_matrix, beside=T, ylim=c(0.0,5.5),
											 col=dx_color[rownames(mean_matrix)], ylab="Shannon Diversity Index")
```

```
## Error in barplot.default(mean_matrix, beside = T, ylim = c(0, 5.5), col = dx_color[rownames(mean_matrix)], : object 'dx_color' not found
```

```r
x_positions <- as.vector(matrix_plot)
```

```
## Error in as.vector(matrix_plot): object 'matrix_plot' not found
```

```r
arrows(x0=x_positions, x1=x_positions, y0=shannon_dx_gender_mean$shannon,
			 y1=shannon_dx_gender_mean$shannon+shannon_dx_gender_se$shannon, angle=90)
```

```
## Error in arrows(x0 = x_positions, x1 = x_positions, y0 = shannon_dx_gender_mean$shannon, : object 'x_positions' not found
```

```r
box()
```

```
## Error in box(): plot.new has not been called yet
```

We still have one small problem - the viewer doesn't know what our colors represent. We need a legend. See if you can't create a legend that goes in the upper right corner of the plot. In this case we don't want to use the `pch` or `col` arguments, rather we want to use the `fill` argument to specify the colors.


```r
matrix_plot <- barplot(mean_matrix, beside=T, ylim=c(0.0,5.5),
											 col=dx_color[rownames(mean_matrix)], ylab="Shannon Diversity Index")
```

```
## Error in barplot.default(mean_matrix, beside = T, ylim = c(0, 5.5), col = dx_color[rownames(mean_matrix)], : object 'dx_color' not found
```

```r
x_positions <- as.vector(matrix_plot)
```

```
## Error in as.vector(matrix_plot): object 'matrix_plot' not found
```

```r
arrows(x0=x_positions, x1=x_positions, y0=shannon_dx_gender_mean$shannon,
			 y1=shannon_dx_gender_mean$shannon+shannon_dx_gender_se$shannon, angle=90)
```

```
## Error in arrows(x0 = x_positions, x1 = x_positions, y0 = shannon_dx_gender_mean$shannon, : object 'x_positions' not found
```

```r
box()
```

```
## Error in box(): plot.new has not been called yet
```

```r
legend("topright", legend=rownames(mean_matrix), fill=dx_color[rownames(mean_matrix)])
```

```
## Error in strwidth(legend, units = "user", cex = cex, font = text.font): plot.new has not been called yet
```

#### Activity 8
Modify the last code chunk to nicely format the diagnosis and sex labels

<input type=button class=hideshow style="margin-bottom: 20px"></input>
<div style="display: none">


```r
dx_convert <- c(normal="Normal", adenoma="Adenoma", cancer="Cancer")
sex_convert <- c(female="Female", male="Male")

matrix_plot <- barplot(mean_matrix, beside=T, ylim=c(0.0,5.5),
											 	col=dx_color[rownames(mean_matrix)],
												ylab="Shannon Diversity Index",
												names.arg=sex_convert[colnames(mean_matrix)])
```

```
## Error in barplot.default(mean_matrix, beside = T, ylim = c(0, 5.5), col = dx_color[rownames(mean_matrix)], : object 'dx_color' not found
```

```r
x_positions <- as.vector(matrix_plot)
```

```
## Error in as.vector(matrix_plot): object 'matrix_plot' not found
```

```r
arrows(x0=x_positions, x1=x_positions, y0=shannon_dx_gender_mean$shannon,
			 y1=shannon_dx_gender_mean$shannon+shannon_dx_gender_se$shannon, angle=90)
```

```
## Error in arrows(x0 = x_positions, x1 = x_positions, y0 = shannon_dx_gender_mean$shannon, : object 'x_positions' not found
```

```r
box()
```

```
## Error in box(): plot.new has not been called yet
```

```r
legend("topright", legend=dx_convert[rownames(mean_matrix)], fill=dx_color[rownames(mean_matrix)])
```

```
## Error in strwidth(legend, units = "user", cex = cex, font = text.font): plot.new has not been called yet
```

</div>


Again we can test the significance using an analysis of variance with a model formula similar to what we used in the aggregate function.


```r
summary(aov(shannon ~ dx + Gender, data=meta_alpha))
```

```
##              Df Sum Sq Mean Sq F value Pr(>F)
## dx            2   0.26  0.1316   0.630  0.533
## Gender        1   0.10  0.1042   0.499  0.480
## Residuals   486 101.51  0.2089
```

As we probably expected from the plot, there's no significant effect of the patient's diagnosis or their gender on the Shannon diversity. This test does not include an interaction term. Instead of using `+` in the forumla, we could have used a `*` to generate the interaction


```r
summary(aov(shannon ~ dx * Gender, data=meta_alpha))
```

```
##              Df Sum Sq Mean Sq F value Pr(>F)
## dx            2   0.26  0.1316   0.631  0.532
## Gender        1   0.10  0.1042   0.500  0.480
## dx:Gender     2   0.63  0.3171   1.521  0.219
## Residuals   484 100.88  0.2084
```

Still nothing.




<script>
$( "input.hideshow" ).each( function ( index, button ) {
  button.value = 'Show an answer';
  $( button ).click( function () {
    var target = this.nextSibling ? this : this.parentNode;
    target = target.nextSibling.nextSibling;
    if ( target.style.display == 'block' || target.style.display == '' ) {
      target.style.display = 'none';
      this.value = 'Show an answer';
    } else {
      target.style.display = 'block';
      this.value = 'Hide answer';
    }
  } );
} );
</script>