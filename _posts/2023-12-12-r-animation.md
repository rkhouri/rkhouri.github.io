---
layout: post
title: Animating plots in R
subtitle: Enhancing ggplot2 capabilities!
tags: [R, tidyverse]
comments: true
---

Though not appropriate to every medium, animated plots can breathe life
into a website or powerpoint. I have always wanted to play around with
plot animation, but it has never been compatible with the medium I
primarily use **R** for (reports and journal articles), so this post
gave me a great excuse to learn! As is the beauty of R, there are many
paths to achieve the same goal. In this post, I will explore two
packages (`gganimate` and `plotly`), which can be used to animate plots
created with [`ggplot2`](https://ggplot2.tidyverse.org/), one of the
packages in the amazing *tidyverse*. If you are unfamiliar with
`ggplot2`, I recommend you begin by checking out some of the [resources](#resources)
linked at the bottom of the page!

<br>

## Data Selection

With this exercise, I wanted to emphasize the ability of animation to
highlight data that changes over a series of “subsets.” The first use
that jumps to mind is annual data, which is appropriate because it is
ordinal—meaning it has an order that matters (e.g., 2001 is preceded by
2000 and succeeded by 2002)—and also discrete—meaning there is a logical
way to subset the data. Animation can be a good way to emphasize how
certain variable changes over time.

*Data:* World Population and Life Expectancy by Country, 1952–2007.

*Source:* Free data from UN POP via GAPMINDER.ORG (License: CC-BY 4.0).
Retrieved through the `gapminder` package in **R**.

*Dataset description:* `gapminder` is a helpful tool for teaching data
wrangling and visualization in R. It includes an excerpt of the data
available at [Gapminder.org](https://www.gapminder.org/data/),
specifically: life expectancy, GDP per capita, and population for 142
countries every five years (1952 to 2007).

[!WARNING]
From the package author: “this package
exists for the purpose of teaching and making code examples. It is an
excerpt of data found in specific spreadsheets on Gapminder.org circa 2010.
It is not a definitive source of socioeconomic data and I don’t
update it. Use other data sources if it’s important to have the current
best estimate of these statistics.”

<br>

## Animation using gganimate

The [`gganimate`](https://gganimate.com/) package (developed by Thomas
Lin Pedersen and David Robinson) is an extension to ggplot2 which adds
new “grammar” (aka “arguments”) to allow for animation by defining what
data will be shown in each frame of the animation and how the frames
should transition.

{: .box-note}
**Note:** The original iteration of gganimate (versions up
to and including v0.1.1) is now
[deprecated](https://github.com/thomasp85/gganimate/releases/tag/v0.1.1)
and code written for the old API will not work with the new iteration of
gganimate (v1.0.0 and beyond). This post will use grammar intended to
work with the newer iteration of gganimate.

Let’s start by creating a static plot showing the life expectancy of
each country as a function of the population, with the countries grouped
by continent:

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

Now we can add an animation with gganimate. Here, I define that the
frames I want to cycle through are a time series by using the
“transition_time” argument and my variable, which is “year”. This
creates a new variable called “frame_time”, which indicates which set of
data should be shown in each frame. We can use this variable to assign a
label for the plot which will change to show which year’s data is shown
in the current frame of the animation.

``` r
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

Alternatively, because the data set only includes data from every 5
years, I can define each year as a distinct “state” (using the
“transition_states” argument), rather than a time series. This places
more emphasis on each 5-year datapoint rather than creating what looks
like a “smooth” timeline. The “transition_states” argument has more
aesthetic options, the length of the transition, the length of time each
state is shown, etc. There are also three options for the frame label:
“previous_state,” “next_state,” or “closest_state.”

``` r
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

For more customization options, a full list of gganimate arguments can
be found [here](https://gganimate.com/reference/index.html).

<br>

## Animation using plotly

[`plotly`](https://plotly.com/r/getting-started/) is an R package for
creating *interactive* web-based graphs via a JavaScript graphing
library of the same name [plotly.js](https://plotly.com/javascript/).
Because `plotly` makes ggplots interactive, it opens up some exciting
functionality if you are creating visualizations for a website. This
suite of interactive features includes modifying axes to zoom in or out
on certain data points, saving a screenshot of the plot as a png, or
comparing data points. Compared to `gganimate` for animating plots,
`plotly` really shines by addition of an interactive slider alongside
your plot, which allows the viewer to select and focus on a specific
frame in the animation. Effectively, the reader can toggle between a
static and animated view.

Unlike gganimate, which works by adding new grammar to ggplot, `plotly`
is designed to work off of a ggplot which has already been created. We
will use the same base ggplot code to create a static graph, with one
addition: an aesthetic called “frame”, which defines the list of frames
that are cycled through in the animation. “frame” will *not* be
recognized by ggplot2, but it will be recognized by the subsequent
`plotly` function “ggplotly”.

<br>

``` r
library(ggplot2)
library(gapminder)
library(plotly)

p <- ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(aes(frame = year), size = 1.5) +
  # The "frame" aesthetic will not be recognized by ggplot, which will
  # trigger a warning message that the aesthetic is being ignored. This is OK.
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw()

ggplotly(p) %>%
  layout(showlegend = FALSE)
  # Some layout features from ggplot are overriden by ggplotly (i.e. legend modification).
  # As a result, changes should be made here and not in the ggplot syntax.
```
<br>

<iframe src="https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/plotly-simple.html" style="width:672px; height:480px"></iframe>

<br>

Let’s fine-tune the visualization by adjusting the button and slider
options.

``` r
p <- ggplot(gapminder, aes(pop, lifeExp, colour = country)) +
  geom_point(aes(frame = year), size = 1.5) +
  scale_colour_manual(values = country_colors) +
  scale_x_log10() +
  facet_wrap(~continent) +
  theme_bw() +
  labs(x = "Population", y = "Life Expectancy")

# Pipes (%>%) are a dplyr function. They require the `dplyr` or `tidyverse` package.
ggplotly(p) %>%
  layout(showlegend = FALSE) %>%
  animation_slider(currentvalue = list(prefix = "Source: UN POP, Year: ")) %>%
  animation_button(x = 0.9, y = 0.4)
```

<br>

<iframe src="https://rkhouri.github.io/assets/2023-12-12-r-animation_files/figure-gfm/plotly-enhanced.html" style="width:672px; height:480px"></iframe>

<br>

In-depth guides for `plotly` can be found
[here](https://plotly.com/ggplot2/#animations). Some useful starting
guides which apply to either static or animated plots are: [modifying
legends](https://plotly.com/r/legend/) and [configuring the interactive
features](https://plotly.com/ggplot2/configuration-options/).

<br>

## Summary

Animation is a simple yet effective way to improve viewer engagement and
understanding when plots are presented in a digital medium. For
animations made in R, there are two package options to animate plots
made with `ggplot2`. The `gganimate` package expands on ggplot2 grammar
to introduce animation, which can be rendered as a gif or video. The
`plotly` package can also be used to animate plots created with ggplot2,
but the output is an HTML widget (benefit: the plots are automatically
made interactive by the `plotly` package; downside: plots are only
viewable on the web).

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
**Note:** When using gganimate, a renderer is required to
combine the frames into an animation (without a renderer, the frames
will generate as individual png files instead of an animation). I used
the `gifski` package, which renders as a *gif*. If you prefer a *video*
output, you can install an alternate renderer (such as the `av` package)
and specify it in your code to override the default use of gifski. The
arguments used to change the renderer for `gganimate` can be found
[here](https://gganimate.com/reference/index.html).

<br>

---

### Learning Roundup

| New Skills         | Refresher Skills |
| :----------------- | :--------------- |
| Animation in R     | Plotting in R    |
| GitHub Pages       | Markdown         |
| Visual Studio Code | Creative writing |


I love how much bang for your buck you get from animating plots. With only a few lines of code added to R, you can add a huge amount of value to visualizations. I also like that animation can streamline data, particularly when you have a third or fourth variable that you want to present without needing numerous panels or plots. Learning how to animate plots in R was relatively easy because I already knew how to navigate R and have a good handle on ggplot grammar. I would say it had a great payout for the amount of time invested into it, and I really appreciate that both `gganimate` and `plotly` build off of `ggplot2`, which is a powerful and flexible tool on its own for data viz, and a package that most R users are already familiar with. For someone with no foundational knowledge of R or ggplot, there would certainly be a steeper learning curve.

I did run into a pretty frustrating knowledge gap when I tried to integrate processes that I am very comfortable with (creating an R Markdown file in RStudio) into a new process (version control and hosting content on GitHub Pages). Implementing version control is possible directly from RStudio, and there are some great guides about this (e.g., [How to Combine R-Markdown and GitHub](https://medium.com/geekculture/how-to-combine-r-markdown-and-github-7bc4fedd929a)). However, I am using Visual Studio Code (VS Code) for anything related to this site that doesn't require R. I quickly realized that making local changes to the same GitHub repository from two different programs (RStudio and VS Code) was a recipe for disaster. I also didn't want to only make this blog post using RStudio, because I wanted to use some helpful VS Code extensions such as "Git Graph" for source control, "Prettier" for code formatting, and the built-in option to preview markdown files. So, I installed the ["R"](https://code.visualstudio.com/docs/languages/r) and the "vscode-pandoc" extensions to be able to create, edit, and knit (render) R Markdown (rmd) files in VS Code. I also discovered and opted to use the ["github_document"](https://rmarkdown.rstudio.com/github_document_format.html) output option for rmd fils. Of course, nothing good can be easy, so I was ramming my head against the wall because 

From what I could glean online, there is some conflict between my site (some posts online suggest it's a problem with Github or Jekyll itself, specifically Ruby). It seems that, for security reasons, GitHub Pages does not like markdown files that have JavaScript elements. When I knit my rmd file as an HTML document and tried uploading that as a blog post (my site template, beautiful-jekyll, treats files in the "_blog" folder in a special way), my site would not even deploy, which makes me think that there is a conflict with that code and the script or Ruby that is foundational to beautiful-jekyll.

I got an idea [here](https://community.plotly.com/t/how-to-display-plotly-graph-on-github-pages/44398) to save the plotly graphs as standalone HTML files and to upload those files to my repo instead of including the full HTML/js code in the markdown file. Using this guide to [Saving and embedding HTML](https://plotly-r.com/saving.html) and this article on [Export plotly Graph as PNG, JPEG & HTML File in R (Example)](https://statisticsglobe.com/export-interactive-plotly-graph-png-jpeg-html-r), I was able to save the plots (which plotly generates as HTML widgets) as stand-alone HTML files, which I uploaded to my repository in a separate folder and linked to iframes in my markdown. The site deploys fine when the plotly HTML code is not in the "_blogs" folder, so I hope I won't find any issues with the site as I go along! All in all, it's not a great solution, because I ended up going back to RStudio to export the plots. However, the solution seems functional and isn't worth sinking more time into right now because I don't know if I will ever need to add more plotly graphs to this site in the future.

---

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
    function names. They may be overwhelming if you are a first-time
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
