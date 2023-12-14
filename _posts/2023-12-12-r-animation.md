---
layout: post
title: Animating plots in R
subtitle: Enchancing ggplot2 capabilities!
tags: [R, tidyverse]
comments: true
---

## Uses of Animation in R

Many R users are already familiar with
[`ggplot2`](https://ggplot2.tidyverse.org/), one of the packages in the
amazing _tidyverse_. `ggplot2` elevates plots with capabilities well
beyond base R plotting. I have always wanted to play around with plot
animation, but it has never been compatible with the medium I use most
often to present plots: reports or journal articles. Though not
appropriate to every medium, animated plots can breathe life into a
website or powerpoint.

As is the beauty of R, there are many paths to achieve the same goal. In
this post, I will explore two packages (`gganimate` and `plotly`), which
can be used to animate plots created with `ggplot2`. If you are
unfamiliar with `ggplot2`, I recommend you begin by checking out some of
the resources linked at the bottom of the page!

<br>

## Data Selection

With this exercise, I wanted to emphasize the ability of animation to
highlight one subset of data at a time. The first use that jumps to mind
is annual data, which is appropriate because it is ordinal—meaning it
has an order that matters (e.g., 2001 is preceded by 2000 and succeeded
by 2002)—and also discrete—meaning there is a logical way to subset the
data.

_Data:_ World Population and Life Expectancy by Country, 1952–2007.

_Source:_ Free data from UN POP via GAPMINDER.ORG (License: CC-BY 4.0).
Retrieved through the `gapminder` package in **R**.

_Dataset description:_ `gapminder` is a helpful tool for teaching data
wrangling and visualization in R. It includes an excerpt of the data
available at [Gapminder.org](https://www.gapminder.org/data/),
specifically: life expectancy, GDP per capita, and population for 142
countries every five years (1952 to 2007).

{: .box-warning}
**Warning** from the package author: “this package
exists for the purpose of teaching and making code examples. It is an
excerpt of data found in specific spreadsheets on Gapminder.org circa 2010. It is not a definitive source of socioeconomic data and I don’t
update it. Use other data sources if it’s important to have the current
best estimate of these statistics.”

<br>

## Animation using gganimate

[`gganimate`](https://gganimate.com/) (developed by Thomas Lin Pedersen
and David Robinson) is an extension to ggplot2 which adds new “grammar”
to define animation in the final plot.

{: .box-note}
**Note:** The original iteration of gganimate (versions up
to and including v0.1.1) is now
[deprecated](https://github.com/thomasp85/gganimate/releases/tag/v0.1.1)
and code written for the old API will not work with the new iteration of
gganimate (v1.0.0 and beyond). This post will use grammar intended to
work with the newer iteration of gganimate.

Let’s create a static plot showing the life expectancy of each country
as a function of the population, with the countries grouped by
continent:

```r
library(ggplot2)
library(gapminder)

ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(show.legend = FALSE, size = 1.5) +
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw()
```

![](https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/gapminder%20static-1.png)<!-- -->

<br>

Now add an animation with gganimate. Here, I define that the frames I
want to cycle through are a time series. This creates a new variable
called “frame_time” which can be used to create a label for the plot.

```r
library(gganimate)

ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(show.legend = FALSE, size = 1.5) +
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw() +
  labs(title = "Year: {frame_time}", x = "Population", y = "Life Expectancy") +
  transition_time(year) +
  ease_aes("cubic-in-out")
```

![](https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/gganimate%20timeframe-1.gif)<!-- -->

<br>

Alternatively, becuase there is only data avilable for every 5 years, I
can define the frames as sepearate states, which makes the animation
less focused on a smooth transition which “fills in” the gaps in the
5-year periods, and places more emphasis on each 5-year datapoint. Now,
we have three options for the label: “previous_state,” “next_state,” or
“closest_state.”

```r
ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(show.legend = FALSE, size = 1.5) +
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw() +
  labs(title = "Year: {closest_state}", x = "Population", y = "Life Expectancy") +
  transition_states(year, transition_length = 2, state_length = 4) +
  ease_aes("cubic-in-out") +
  exit_fade(alpha = 0)
```

![](https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/gganimate%20stateframe-1.gif)<!-- -->

A full list of gganimate functions to customize aesthetics can be found
[here](https://gganimate.com/reference/index.html).

<br>

## Animation using plotly: “ggplotly” function

[`plotly`](https://plotly.com/r/getting-started/) is an R package for
“creating interactive web-based graphs via the open source JavaScript
graphing library [plotly.js](https://plotly.com/javascript/).” `plotly`
makes ggplots interactive, which allows for some exciting functionality
if you are creating visualizations for a website. Compared to
`gganimate` for animation purposes, `plotly` really shines by addition
of an interactive slider alongside your plot, which allows the viewer to
select and focus on a specific frame in the animation. It also includes
a suite of interactive features that can be used for static or animated
plots.

Unlike gganimate, which adds new grammar to ggplot, `plotly` is designed
to work off of a ggplot which has already been created. We will use the
same base ggplot code to create a static graph, with one addition: a key
called “frame”, which defines the list of frames that are cycled through
in the animation. We can add this variable when creating the plot.
“frame” will not be recognized by ggplot2, but it will be recognized by
the `plotly` function “ggplotly”.

<br>

```r
library(ggplot2)
library(gapminder)
library(plotly)

p <- ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(aes(frame = year), size = 1.5) +
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw()

ggplotly(p) %>%
  layout(showlegend = FALSE)
# We no longer need the hide legend syntax in ggplot, because it is overriden by plotly.
# Instead, we can opt to hide the legend in the ggplotly syntax.
```
<iframe src="https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/plotly-simple.html" style="width:672px; height:480px"></iframe>

<br>

Let’s fine-tune the visualization by adjusting the button and slider
options.

```r
p <- ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(aes(frame = year), size = 1.5) +
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw() +
  labs(x = "Population", y = "Life Expectancy")

# Using pipes (%>%) here, which is a dplyr function
ggplotly(p) %>%
  layout(showlegend = FALSE) %>%
  animation_slider(currentvalue = list(prefix = "Source: UN POP, Year: ")) %>%
  animation_button(x = 0.9, y = 0.4)
```
<iframe src="https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/plotly-enhanced.html" style="width:672px; height:480px"></iframe>

In-depth guides to customize visualizations made with `plotly` can be
found [here](https://plotly.com/ggplot2/#animations). Some useful
starting guides include [modifying
legends](https://plotly.com/r/legend/) and [interaction configuration
options](https://plotly.com/ggplot2/configuration-options/).

<br>

## Summary

Animation is a simple yet effective way to improve viewer engagement and
interpretation of plots presented in a digital medium. For animations
made in R, there are two important packages. The `gganimate` package
expands on ggplot2 grammar to introduce animation, which can be rendered
as a gif or video. The `plotly` package can also be used to animate
plots created with ggplot2, but it can also add interactive elements to
web-based (HTML) visualizations.

---

### Learning Roundup

| New Skills         | Refresher Skills |
| :----------------- | :--------------- |
| Animation in R     | Plotting in R    |
| GitHub             | Markdown         |
| Visual Studio Code | Creative writing |

---

### Software Used

**[R](https://www.r-project.org) version**: 4.3.2

**[RStudio](https://posit.co/download/rstudio-desktop/) version**:
2023.09.1+494

**R Packages**:
[`gapminder`](https://cran.r-project.org/web/packages/gapminder/index.html)
(v1.0.0); [`ggplot2`](https://cran.r-project.org/web/packages/ggplot2/)
(v3.4.4);
[`gganimate`](https://cran.r-project.org/web/packages/gganimate/)
(v1.0.8); [`gifski`](https://cran.r-project.org/web/packages/gifski/)
(v1.12.0-2); [`plotly`](https://cran.r-project.org/web/packages/plotly/)
(v4.10.3)

{: .box-note}
**Note:** gifski is a renderer, which is required if you
want the output of gganimate to be a gif rather each frame as a seperate
image. If you prefer a video output over a gif, you can use an alternate
renderer (such as the `av` package) and specify it in your code to
override the deafult use of gifski. However, a renderer is required to
combine the frames into an animation.

<br>

## Resources

- Learning to use ggplot2:
  - [R For Data Science - Chapter 3: Data
    visualisation](https://r4ds.had.co.nz/data-visualisation.html)
  - [ggplot2 Introductory
    Webinar](https://www.youtube.com/watch?v=h29g21z0a68&feature=youtu.be)
    by Thomas Lin Penderson: a thorough breakdown of theory of ggplot2
    to help any beginner or intermediate user understand how each part
    of the syntax translates to the output. Also includes working
    examples and code to follow along with.
  - [ggplot2 Cheat
    Sheet](https://rstudio.github.io/cheatsheets/html/data-visualization.html):
    Posit (the company behind RStudio and the tidyverse) has a great
    collection of cheatsheets that are particularly helpful when you
    already have some experience with ggplot2 and are trying to remember
    function names. They may be overwhleming if you are a first-time
    user.
- R Graphics and Animation:
  - [R Graphics Cookbook](https://r-graphics.org/) by Winston Chang
  - [gganimate Resources](https://gganimate.com/)
  - [plotly Resources](https://plotly.com/r/getting-started/)
- Finding and using public databases:
  - Built-in data sets in R’s [`dataset`
    package](https://stat.ethz.ch/R-manual/R-devel/library/datasets/html/00Index.html)
  - [Open Data Network](https://www.opendatanetwork.com/)
  - [Summary of CC
    licenses](https://creativecommons.org/share-your-work/cclicenses/)
