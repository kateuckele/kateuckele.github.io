---
title: "Plotting QTL intervals with karyoploteR"
categories:
  - blog
tags:
  - karyoploteR
  - R
  - ideogram
---

I am absolutely loving the [karyoploteR][karyoploteR_paper] package and wanted to share what I've learned using `karyoploteR` to visualize quantitative trait loci (QTL) across the genome as part of my *Costus* QTL mapping project. 

When I started this project, I had four specific functionalities in mind for an ideogram plotting tool:

1. Plot the positions of the genetic markers used to detect the QTL.
2. Add colored rectangles adjacent to chromosomes to represent QTL confidence intervals.
3. Include directional arrows on these rectangles to indicate QTL additive effect direction.
4. Adjust the width of these rectangles to visually indicate QTL effect size.

While I didn't find a specific tutorial for plotting QTL intervals on the [github tutorial][karyoploteR_github], the versatility of the `karyoploteR` package allowed me to design my plot using the established functions--I just had to get a little creative. 

# Horizontal Ideograms and Plots

In `karyoploteR`, the chromosomes and associated data panels are plotted horizontally, like this:

![ideogram plotted horizontally](/assets/images/example_karyoploteR_plot.png)

This horizontal arrangement simplifies adding multiple tracks of data, each displaying different genomic features or quantitative data.

# My Approach: Markers and QTL Intervals

To achieve my visualization goals, I set up two main data tracks:

- **Markers Track**: Vertical lines representing specific genetic markers used in QTL mapping.
- **QTL Track**: Colored rectangles representing QTL confidence intervals, customized by width to reflect effect sizes.

# Setting Up the Plot

First, I created a custom `GRanges` object for the *Costus lasius* genome containing the chromosome names and their respective lengths in base pairs. To create a new empty plot with *Costus lasius* karyotype, I used the `plotKaryotype` function. Once the empty plot is initialized, the data tracks can be plotted above the chromosomes. 

```R
kp <- plotKaryotype(genome=custom.genome, ideogram.plotter = NULL)
```

# Plotting markers

`karyoploteR` uses a really intuitive system (inspired by the Circos visualization tool) for arranging data vertically using parameters `r0` and `r1`, defining the bottom and top of each track. To automate the parameters for my tracks, I used `autotrack()`:

```R
marker.track <- autotrack(current.track = 1, total.tracks = 2)
kpPlotRegions(kp, data=markers, r0=marker.track$r0, r1=marker.track$r1)
```
Because my marker intervals were single positions (start = end), they rendered neatly as vertical lines (my partner joked they look like barcodes!).

![ideogram plotted horizontally](/assets/images/karyoploteR_markers.png)
 
# Plotting QTL intervals

Next, for QTL intervals, I needed more flexibility than `kpPlotRegions()`. `kpRect()` allows precise rectangle control using x0, x1, y0, and y1 coordinates. I looped through all 20 traits, placing each in its own mini-track within the larger QTL track: 

```R
qtl.track <- autotrack(current.track = 2, total.tracks = 2)
for (i in 1:length(traits)) {
  trait.track <- autotrack(current.track = i, total.tracks = length(traits), r0=qtl.track$r0, r1=qtl.track$r1)
  GR <- makeGRangesFromDataFrame(
    bounds[bounds$trait == traits[i],],
    seqnames.field       = "chr",
    start.field          = "start",
    end.field            = "end",
    keep.extra.columns   = TRUE
  )
  kpRect(kp, data=GR, x0=GR$start, x1=GR$end, y0=0.2, y1=0.8, r0=trait.track$r0, r1=trait.track$r1, col=cols[i])
}
```

This loop neatly plots each trait's QTL intervals, clearly differentiating them by color and allowing easy comparison of interval sizes.



[karyoploteR_paper]: https://pmc.ncbi.nlm.nih.gov/articles/PMC5870550/
[karyoploteR_github]: https://bernatgel.github.io/karyoploter_tutorial/
[datapanels]: https://bernatgel.github.io/karyoploter_tutorial//Tutorial/DataPanels/DataPanels.html
[plasmodium_tutorial]: https://bernatgel.github.io/karyoploter_tutorial//Examples/PVivaxGenes/PVivaxGenes.html



