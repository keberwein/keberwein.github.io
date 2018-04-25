Install from CRAN
-----------------

`install.packages("mlbgameday")`

Gathering Data
--------------

The package is primarily a data package, and has no native plotting
tools. However, there are several plotting options available by
leveraging one of the R language's excellent plotting libraries.

For all of the following examples, we will use The pitch data for Jake
Arrieta's no-hitter, which occurred on April 21, 2016.

    library(mlbgameday)
    library(dplyr)

    # Grap some Gameday data. We're specifically looking for Jake Arrieta's no-hitter.
    gamedat <- get_payload(start = "2016-04-21", end = "2016-04-21")

    # Subset that atbat table to only Arrieta's pitches and join it with the pitch table.
    pitches <- inner_join(gamedat$pitch, gamedat$atbat, by = c("num", "url")) %>%
        subset(pitcher_name == "Jake Arrieta")

Visualizing With ggplot
-----------------------

    library(ggplot2)

    # basic example
    ggplot() +
        geom_point(data=pitches, aes(x=px, y=pz, shape=type, col=pitch_type)) +
        coord_equal() + geom_path(aes(x, y), data = mlbgameday::kzone)

![](2018-04-11_gameday_plots1_files/figure-markdown_strict/unnamed-chunk-3-1.png)

Including Batter Stance
-----------------------

    library(ggplot2)

    # basic example with stand.
    ggplot() +
        geom_point(data=pitches, aes(x=px, y=pz, shape=type, col=pitch_type)) +
        facet_grid(. ~ stand) + coord_equal() +
        geom_path(aes(x, y), data = mlbgameday::kzone)

![](2018-04-11_gameday_plots1_files/figure-markdown_strict/unnamed-chunk-4-1.png)

Visualizing With Other Tools
----------------------------

The R language has no shortage of visualization tools. Other examples,
including Plotly, can be found in the [package
vignettes](https://github.com/keberwein/mlbgameday/blob/master/vignettes/pitch_plotting.Rmd).

![](https://raw.githubusercontent.com/keberwein/mlbgameday/master/man/figures/mlbgameday_hex.png)
