# ggpathway
A tutorial for pathway visualization using tidyverse, igraph, and ggraph. 

![Krebs cycle](https://github.com/cxli233/ggpathway/blob/main/Results/TCA_2.svg) 

[![DOI](https://zenodo.org/badge/574269912.svg)](https://zenodo.org/badge/latestdoi/574269912)


# Table of contents

1. [Introduction](https://github.com/cxli233/ggpathway#introduction)
     - [Dependencies](https://github.com/cxli233/ggpathway#dependencies)
     - [The theory behind this workflow](https://github.com/cxli233/ggpathway#the-theory-behind-this-workflow)
     - [Required input](https://github.com/cxli233/ggpathway#required-input)
2. [Example 1: simple linear pathway](https://github.com/cxli233/ggpathway#example-1-simple-linear-pathway) 
3. [Example 2: more complex pathway](https://github.com/cxli233/ggpathway#example-2-more-complex-pathway)
4. [Example 3: circular pathway](https://github.com/cxli233/ggpathway#example-3-circular-pathway) 
5. [Subsetting pathway](https://github.com/cxli233/ggpathway#subsetting-pathway)
6. [Combinding pathways](https://github.com/cxli233/ggpathway#combining-pathways)

# Introduction 

This markdown page describes how to make pathway diagrams using ggplot compatible functions. 
It requires: 

* [R](https://cran.r-project.org/)
* [RStudio](https://posit.co/downloads/)
* Rmarkdown, can be downloaded using `install.packages("rmarkdown")` in R.

## Dependencies 

The workflow is built upon [tidyverse](https://www.tidyverse.org/) and [igraph](https://igraph.org/).
Interactions between `ggplot` & `igraph` functions are achieved via [ggraph](https://ggraph.data-imaginist.com/). 

If you want to read in excel files, you will need the `readxl` package. 

```r
library(tidyverse)
library(igraph)
library(ggraph)

library(readxl)

library(viridis)
library(RColorBrewer)
library(rcartocolor)
```

The rest of the loaded packages are for data visualization only (some nice colors in graphs). 

## The theory behind this workflow

To plot a pathway, we can model the pathway as a network, or a "graph" in graph theory. 
In mathematics, a graph is a structure that models the relationship between objects. 
A network can be constructed by: 

* an edge table 
* a node table 

For example, we want to visualize a metabolic pathway. 
In this context, each metabolite is a node; each enzyme is an edge that connects the metabolites.
This concept can be applied to signaling pathways as well, with modifications. 

We will use `tidyverse` functions to handle tabular data operations regarding the edge and node tables.
We will then use `igraph` functions to produce a network object from edge and node tables. 
Finally, we will use `ggraph`, a `ggplot` extension of `igraph` to make pretty plots. 

## Required input

* Edge table - each row is an edge, with the following columns: 
     - from: where the edge starts, e.g., name of metabolite (**required**).
     - to: where the edge ends, e.g., name of metabolite (**required**).
     - label: if you want the edge to be labeled, e.g., name of the enzyme.
     - other information as different columns, e.g., condition, tissue, cell types... 

* Node table - each row is a node, with the following columns: 
     - name: name of the node, e.g., name of the metabolite (**required**).
     - x: x coordinate of the node on the graph.
     - y: y coordinate of the node on the graph.
     - other information as different columns, e.g., molecular weight, localization... 

Note that the edge and node tables are tidy data frames. 
Each row is an observation, and each column is a variable. 
Also note that the union of `from` and `to` columns in the edge table should be identical to the `name` column of the node table. 

Hopefully the above explanation will become more straightforward when do an example. 
Example input files can be found in the [Data](https://github.com/cxli233/ggpathway/tree/main/Data) folder. 

# Example 1: simple linear pathway 

We will start with a simple example, a linear pathway with 3 steps and 4 metabolites. 
We will use the oxidative segment of [pentose phosphate pathway](https://en.wikipedia.org/wiki/Pentose_phosphate_pathway) as an example.  

This is a very short pathway, so we can actually write the tables in R by hand. 
We can write the tables row-by-row using the `tribble()` function in `tidyverse`.

## Edge table
```r
example1_edge_table <- tribble(
  ~from, ~to,  ~label,
  "Glc6P", "6P-gluconolactone",  "Glc6PHD",
  "6P-gluconolactone", "6P-glucoconate",  "6P-gluconolactonase",
  "6P-glucoconate", "Ru5P", "6P-gluconateDH"
)

head(example1_edge_table)
```
## Node table
```r
example1_nodes_table <- tribble(
  ~name, ~x,  ~y,
  "Glc6P", 1, 0,
  "6P-gluconolactone", 2, 0,  
  "6P-glucoconate", 3, 0,
  "Ru5P", 4, 0
)

head(example1_nodes_table)
```
Notice here I provided a manual layout; each node is given an x and y coordinate. 
For example, Glc6P will show up at (1, 0) on the graph and so on. 

## Make network object and graph 
Once the node and edge tables are written, we can combined them into a network object. 
We use the `graph_from_data_frame()` function from `igraph`. 

```r
example1_network <- graph_from_data_frame(
  d = example1_edge_table,
  vertices = example1_nodes_table,
  directed = T
)
```

Note that the `directed` argument is set to `TRUE`. 

Once the network object is made, we can visualize it using `ggraph()`

```r
ggraph(example1_network, layout = "manual", 
      x = x, y = y) +
  geom_node_text(aes(label = name), hjust = 0.5) +
  geom_edge_link(aes(label = example1_edge_table$label), 
                   angle_calc = 'along',
                   label_dodge = unit(2, 'lines'),
                   arrow = arrow(length = unit(0.5, 'lines')), 
                   start_cap = circle(4, 'lines'),
                   end_cap = circle(4, 'lines')) +
  theme_void()  

ggsave("../Results/Pentose_1.svg", height = 2, width = 6.5, bg = "white")
ggsave("../Results/Pentose_1.png", height = 2, width = 6.5, bg = "white")
```
![OPPP_short](https://github.com/cxli233/ggpathway/blob/main/Results/Pentose_1.svg)

And there it is!
Not very sophisticated, but now we have the frame work to build more complex pathways.  

# Example 2: more complex pathway

For the 2nd example, let's do a more complex pathway.
By more complex I mean more edges and more nodes, as well as branches. 
We will use the rest of the pentose phosphate pathway. 

Once the pathway gets complex enough, it's better to prepare edge & node tables in Excel. 
Once they are written, you can load them into R. 

```r
example2_edges <- read_excel("../Data/OPPP_edges.xlsx")
example2_nodes <- read_excel("../Data/OPPP_nodes.xlsx")

head(example2_edges)
head(example2_nodes)
```

**Important!** If a compound appears multiple times in the pathway at different locations, each instance *must* have a different name. 

In this example, Xu5P, Glyceral-3P, and Frc-6P all appear twice. 
So I named them {name}{1} or {name}{2}. 
For aesthetic purposes, we can make a new column in the node table called "label",
such that different nodes can have the same label, but they must have unique names. 

```r
example2_nodes <- example2_nodes %>% 
  mutate(label = str_remove(name, "_\\d"))


head(example2_nodes)
```

I think we are all good to go. 
```r
example2_network <- graph_from_data_frame(
  d = example2_edges,
  vertices = example2_nodes,
  directed = T
)
```

For a complex pathway with multiple branch points, instead of manual layout, we can also use the layout methods provides by `igraph` and `ggraph`. 
Read more about layouts [here](https://www.data-imaginist.com/2017/ggraph-introduction-layouts/).

```r
ggraph(example2_network, layout = "kk") +
  geom_node_point(size = 3, aes(fill = as.factor(carbons)), 
                  alpha = 0.8, shape = 21, color = "grey20") +
  geom_node_text(aes(label = label), hjust = 0.5, repel = T) +
  geom_edge_link(#aes(label = example2_edges$label), 
                   #angle_calc = 'along',
                   label_dodge = unit(2, 'lines'),
                   arrow = arrow(length = unit(0.4, 'lines')), 
                   start_cap = circle(1, 'lines'),
                   end_cap = circle(2, 'lines')) +
  scale_fill_manual(values = carto_pal(7, "Vivid")) +
  labs(fill = "Carbons") +
  theme_void()  

ggsave("../Results/Pentose_2.svg", height = 5, width = 4, bg = "white")
ggsave("../Results/Pentose_2.png", height = 5, width = 4, bg = "white")
```
![OPPP_2](https://github.com/cxli233/ggpathway/blob/main/Results/Pentose_2.svg)

That looks fine to me. 
I turned off the edge labels, because it's too much text to look at. 
We can incorporate other info on the graph, such as number of carbons each metabolite has. 
A purpose of the pentose phosphate pathway is to toggle between 6 or 3 carbon molecules for glycolysis and 5 carbon molecules for nucleotide biosynthesis. 


# Example 3: circular pathway 
For the next example, let's do a circular pathway. 
An archtypal example is the TCA cycle, aka the Krebs cycle. 
Let's read in the nodes and edges. 

```r
example3_edges <- read_excel("../Data/TCA_cycle_edges.xlsx")
example3_nodes <- read_excel("../Data/TCA_cycle_nodes.xlsx")

head(example3_edges)
head(example3_nodes)
```
In this example, I also included co-factors (Co-enzymeA, NAD+/NADH, ATP...).
Again, when a molecule appears multiple times, each instance *must* have unique names. 
For aesthetics only, let's make a label column. 

```r
example3_nodes <- example3_nodes %>% 
  mutate(label = str_remove(name, "_\\d"))


head(example3_nodes)
```

I did some high school math to layout the pathway around a circle. 

```r
example3_network <- graph_from_data_frame(
  d = example3_edges,
  vertices = example3_nodes,
  directed = T
)
```

```r
ggraph(example3_network, layout = "manual",
       x = x, y = y) +
  geom_node_point(size = 3, aes(fill = as.factor(carbons)), 
                  alpha = 0.8, shape = 21, color = "grey20") +
  geom_edge_link(arrow = arrow(length = unit(0.4, 'lines')), 
                   start_cap = circle(0.5, 'lines'),
                   end_cap = circle(0.5, 'lines'), 
                 width = 1.1, alpha = 0.5) +
  geom_node_text(aes(label = label), hjust = 0.5, repel = T) +
  annotate(geom = "text", label = "TCA Cycle", 
           x = 0, y = 0, size = 5, fontface = "bold") +
  scale_fill_manual(values = carto_pal(7, "Vivid")) +
  labs(fill = "Carbons") +
  theme_void() +
  coord_fixed()

ggsave("../Results/TCA_1.svg", height = 4, width = 5, bg = "white")
ggsave("../Results/TCA_1.png", height = 4, width = 5, bg = "white")
```
![TCA1](https://github.com/cxli233/ggpathway/blob/main/Results/TCA_1.svg)

This looks fine to me. 
I had to play around with the line and arrow size. 
Maybe I was too ambitious to put all the cofactors on this. 

# Subsetting pathway

We can subset a pathway by removing nodes and edges. 
```r
example3_nodes_trim <- example3_nodes %>% 
  filter(carbons != "cofactor")

example3_edges_trim <- example3_edges %>% 
  filter(from %in% example3_nodes_trim$name &
           to %in% example3_nodes_trim$name)
```

Now re-make the network object
```r
example3_network_trim <- graph_from_data_frame(
  d = example3_edges_trim,
  vertices = example3_nodes_trim,
  directed = T
)
```

```r
ggraph(example3_network_trim, layout = "manual",
       x = x, y = y) +
  geom_node_point(size = 3, aes(fill = as.factor(carbons)), 
                  alpha = 0.8, shape = 21, color = "grey20") +
  geom_edge_link(arrow = arrow(length = unit(0.4, 'lines')), 
                   start_cap = circle(0.5, 'lines'),
                   end_cap = circle(1, 'lines'), 
                 width = 1.1, alpha = 0.5) +
  geom_node_text(aes(label = label), hjust = 0.5, repel = T) +
  annotate(geom = "text", label = "TCA Cycle", 
           x = 0, y = 0, size = 5, fontface = "bold") +
  scale_fill_manual(values = carto_pal(7, "Vivid")) +
  labs(fill = "Carbons") +
  theme_void() +
  coord_fixed()

ggsave("../Results/TCA_2.svg", height = 4, width = 5, bg = "white")
ggsave("../Results/TCA_2.png", height = 4, width = 5, bg = "white")
```
![TCA_2](https://github.com/cxli233/ggpathway/blob/main/Results/TCA_2.svg)
 
That's it! 

# Combining pathways
When we need to combine two pathways, the new edge and node tables are the unions of edges and nodes, respectively. 
This can be achieved by binding the tables as rows and then removing redundant rows using `distinct(..., .keep.all = T)` .

First read in data.
```r
calvin_edges <- read_excel("../Data/Calvin_cycle_edges.xlsx")
calvin_nodes <- read_excel("../Data/Calvin_cycle_nodes.xlsx")

PR_edges <- read_excel("../Data/Photorespiration_edges.xlsx")
PR_nodes <- read_excel("../Data/Photorespiration_nodes.xlsx")
```

Then combine edges and remove redundant ones.
```r
combined_edges <- rbind(
  calvin_edges, 
  PR_edges
) %>% 
  distinct(from, to, .keep_all = T)
```

Then combine nodes and remove redundant ones.
```r
combined_nodes <- rbind(
  calvin_nodes %>% 
    select(-carbon),
  PR_nodes
) %>% 
  distinct( .keep_all = T)
```

Then make graph object. 
```r
combined_network <- graph_from_data_frame(
  d = combined_edges,
  vertices = combined_nodes,
  directed = T
)
```

Finally, plot! 
```r
ggraph(combined_network, layout = "kk") +
  geom_node_point(size = 3, aes(fill = localization), 
                  alpha = 0.8, shape = 21, color = "grey70") +
  geom_edge_link(label_dodge = unit(2, 'lines'),
                   arrow = arrow(length = unit(0.4, 'lines')), 
                   start_cap = circle(0.75, 'lines'),
                   end_cap = circle(0.75, 'lines'),
                 alpha = 0.5, width = 1.1, color = "grey30") +
  geom_node_text(aes(label = name), hjust = 0.5, repel = T) +
  scale_fill_manual(values = carto_pal(7, "Vivid")[c(4, 2, 5)],
                    limits = c("chloroplast", "peroxisome", "mitochondria")) +
  labs(fill = "Localization",
       title = "Calvin cycle & photorespiration") +
  theme_void() +
  theme(
    legend.position = c(0.8, 0.2)
  ) +
  scale_y_reverse()

ggsave("../Results/Calvin_PS_comb.svg", height = 4.5, width = 5.5, bg = "white")
ggsave("../Results/Calvin_PS_comb.png", height = 4.5, width = 5.5, bg = "white")
```

![combined_pathway](https://github.com/cxli233/ggpathway/blob/main/Results/Calvin_PS_comb.svg) 

Done!
Example script on Calvin cycle, photorespiration, and combined can be found [here](https://github.com/cxli233/ggpathway/blob/main/Scripts/calvin_cycle.Rmd). 
