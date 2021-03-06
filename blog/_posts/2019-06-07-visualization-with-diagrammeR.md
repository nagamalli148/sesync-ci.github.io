---
# DO NOT EDIT THE .md ON GITHUB
# DO EDIT THE .Rmd AND then run BUILD ALL
title: Creating visualizations with DiagrammeR
tags:
  - Visualization
  - R
author: rblake
output:   
  md_document:
    preserve_yaml: true
always_allow_html: yes
---

Have you ever needed to create a visualization of a research process or statistical model that isn't directly plotted from data?  For example, a conceptual diagram, mind map, flowchart of your research process, or statistical model diagram.  The R package [DiagrammeR](http://rich-iannone.github.io/DiagrammeR/index.html) makes it much easier to create high quality figures and diagrams in situations like these.  

There are three main ways to use the DiagrammeR package.  

1) The package contains R functions that help you [create a diagram](http://rich-iannone.github.io/DiagrammeR/graph_creation.html).  These include `create_graph` for defining the structure of your diagram, and `render_graph` for printing your diagram in the RStudio viewer.  With `render_graph` you can also output your graph in DOT (graph description language).  

2) DOT is also used by [Graphviz](https://www.graphviz.org/), which is implemented in DiagrammeR.  Diagrams specified with Graphviz must pass a valid diagram specification in DOT to the function `grViz()`.  At the minimum, you will need graph, node, and edge statements (see example below).  

3) The [mermaid](https://mermaidjs.github.io/) library is also implemented in DiagrammeR.  Diagrams specified with mermaid must pass a valid diagram specification to the function `mermaid()`.  You must begin by specifying `graph` in this function (see example below).  

For exporting/printing/saving your diagrams, the package [DiagrammeRsvg](https://github.com/rich-iannone/DiagrammeRsvg) can be useful.  You can also use the Export functionality in RStudio to save graphs.  



```r
library(DiagrammeR) ; library(DiagrammeRsvg) ; library(rsvg) 
```

## Example using DiagrammeR functions

A simple diagram:


```r
# define nodes dataframe
nodes <- create_node_df(n = 4, 
                        type = "lower",
                        style = "filled",
                        color = "teal", 
                        shape = "circle", 
                        data = c(3.5, 2.6, 9.4, 2.7))

# define edges dataframe
edges <- create_edge_df(from = c(1, 2, 3, 3),
                        to = c(2, 4, 4, 2))

# create graph
my_graph <- create_graph(nodes_df = nodes, edges_df = edges)

# print graph
render_graph(my_graph)
```

<div class="figure">
<!--html_preserve--><div id="htmlwidget-25a9dc81912a0efa6fca" style="width:504px;height:504px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-25a9dc81912a0efa6fca">{"x":{"diagram":"digraph {\n\ngraph [layout = \"neato\",\n       outputorder = \"edgesfirst\",\n       bgcolor = \"white\"]\n\nnode [fontname = \"Helvetica\",\n      fontsize = \"10\",\n      shape = \"circle\",\n      fixedsize = \"true\",\n      width = \"0.5\",\n      style = \"filled\",\n      fillcolor = \"aliceblue\",\n      color = \"gray70\",\n      fontcolor = \"gray50\"]\n\nedge [fontname = \"Helvetica\",\n     fontsize = \"8\",\n     len = \"1.5\",\n     color = \"gray80\",\n     arrowsize = \"0.5\"]\n\n  \"1\" [style = \"filled\", color = \"teal\", shape = \"circle\", fillcolor = \"#F0F8FF\", fontcolor = \"#000000\"] \n  \"2\" [style = \"filled\", color = \"teal\", shape = \"circle\", fillcolor = \"#F0F8FF\", fontcolor = \"#000000\"] \n  \"3\" [style = \"filled\", color = \"teal\", shape = \"circle\", fillcolor = \"#F0F8FF\", fontcolor = \"#000000\"] \n  \"4\" [style = \"filled\", color = \"teal\", shape = \"circle\", fillcolor = \"#F0F8FF\", fontcolor = \"#000000\"] \n  \"1\"->\"2\" \n  \"2\"->\"4\" \n  \"3\"->\"4\" \n  \"3\"->\"2\" \n}","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
<p class="caption">plot of chunk unnamed-chunk-2</p>
</div>

Use the `export_graph` function to save the figure as an image file, in this case a PNG:


```r
# export graph
export_graph(my_graph, file_name = "simple.png",
             file_type = "PNG")
```


## Research provenance example using Graphviz
I've found it easiest and most customizable to create diagrams using Graphviz. 

A simple diagram:


```r
my_graphviz <- grViz("digraph{
         
                     graph[rankdir = LR]
                     
                     node[shape = rectangle, style = filled]  
                     A[label = 'Figure']
                     B[label = 'Analysis.R']
                     C[label = 'Data.csv']

                     edge[color = black]
                     B -> A
                     C -> B
                     
                     }")

my_graphviz
```

<div class="figure">
<!--html_preserve--><div id="htmlwidget-1d7ababb01058e9ed1ac" style="width:504px;height:504px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-1d7ababb01058e9ed1ac">{"x":{"diagram":"digraph{\n         \n                     graph[rankdir = LR]\n                     \n                     node[shape = rectangle, style = filled]  \n                     A[label = \"Figure\"]\n                     B[label = \"Analysis.R\"]\n                     C[label = \"Data.csv\"]\n\n                     edge[color = black]\n                     B -> A\n                     C -> B\n                     \n                     }","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
<p class="caption">plot of chunk unnamed-chunk-4</p>
</div>

A GraphViz object requires a few more steps to save as an image file, such as a PNG:


```r
# export graph
export_svg(my_graphviz) %>%
  charToRaw() %>%
  rsvg() %>%
  png::writePNG("simple_grv.png")
```

A more complex diagram, specifying different colors for some nodes, and using subgraph clustering.  


```r
grViz("digraph{

      graph[rankdir = LR]
  
      node[shape = rectangle, style = filled]
  
      node[fillcolor = Coral, margin = 0.2]
      A[label = 'Figure 1: Map']
      B[label = 'Figure 2: Metrics']
  
      node[fillcolor = Cyan, margin = 0.2]
      C[label = 'Figures.Rmd']
  
      node[fillcolor = Violet, margin = 0.2]
      D[label = 'Analysis_1.R']
      E[label = 'Analysis_2.R']
  
      subgraph cluster_0 {
        graph[shape = rectangle]
        style = rounded
        bgcolor = Gold
    
        label = 'Data Source 1'
        node[shape = rectangle, fillcolor = LemonChiffon, margin = 0.25]
        F[label = 'my_dataframe_1.csv']
        G[label = 'my_dataframe_2.csv']
      }
  
      subgraph cluster_1 {
         graph[shape = rectangle]
         style = rounded
         bgcolor = Gold
    
         label = 'Data Source 2'
         node[shape = rectangle, fillcolor = LemonChiffon, margin = 0.25]
         H[label = 'my_dataframe_3.csv']
         I[label = 'my_dataframe_4.csv']
      }
  
      edge[color = black, arrowhead = vee, arrowsize = 1.25]
      C -> {A B}
      D -> C
      E -> C
      F -> D
      G -> D
      H -> E
      I -> E
      
      }")
```

<div class="figure">
<!--html_preserve--><div id="htmlwidget-2efc9607dfdbe1cd1b8b" style="width:504px;height:504px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-2efc9607dfdbe1cd1b8b">{"x":{"diagram":"digraph{\n\n      graph[rankdir = LR]\n  \n      node[shape = rectangle, style = filled]\n  \n      node[fillcolor = Coral, margin = 0.2]\n      A[label = \"Figure 1: Map\"]\n      B[label = \"Figure 2: Metrics\"]\n  \n      node[fillcolor = Cyan, margin = 0.2]\n      C[label = \"Figures.Rmd\"]\n  \n      node[fillcolor = Violet, margin = 0.2]\n      D[label = \"Analysis_1.R\"]\n      E[label = \"Analysis_2.R\"]\n  \n      subgraph cluster_0 {\n        graph[shape = rectangle]\n        style = rounded\n        bgcolor = Gold\n    \n        label = \"Data Source 1\"\n        node[shape = rectangle, fillcolor = LemonChiffon, margin = 0.25]\n        F[label = \"my_dataframe_1.csv\"]\n        G[label = \"my_dataframe_2.csv\"]\n      }\n  \n      subgraph cluster_1 {\n         graph[shape = rectangle]\n         style = rounded\n         bgcolor = Gold\n    \n         label = \"Data Source 2\"\n         node[shape = rectangle, fillcolor = LemonChiffon, margin = 0.25]\n         H[label = \"my_dataframe_3.csv\"]\n         I[label = \"my_dataframe_4.csv\"]\n      }\n  \n      edge[color = black, arrowhead = vee, arrowsize = 1.25]\n      C -> {A B}\n      D -> C\n      E -> C\n      F -> D\n      G -> D\n      H -> E\n      I -> E\n      \n      }","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
<p class="caption">plot of chunk unnamed-chunk-6</p>
</div>

Adapted from a more detailed "real life" example located in this research [data package](https://knb.ecoinformatics.org/view/urn:uuid:64e28478-7964-4fcb-b002-49a7915fbe4e).  


## Structural equation model example using mermaid

It's also pretty easy to create diagrams in mermaid, but it seems slightly less customizable.  It also not currently possible to export a diagram created in mermaid to a static file (ex: svg, png).  If you need to create a figure for publication, for example, it may be best to use the Graphviz implementation above.   

A simple diagram:


```r
mermaid("
        graph LR
        A[Nutrients]
        A-->B[Phytoplankton]
        B-->B1[Mussels]
        ")
```

<div class="figure">
<!--html_preserve--><div id="htmlwidget-79967e688303dcf1d865" style="width:504px;height:504px;" class="DiagrammeR html-widget"></div>
<script type="application/json" data-for="htmlwidget-79967e688303dcf1d865">{"x":{"diagram":"\n        graph LR\n        A[Nutrients]\n        A-->B[Phytoplankton]\n        B-->B1[Mussels]\n        "},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
<p class="caption">plot of chunk unnamed-chunk-7</p>
</div>

A more complex diagram specifying colors and shapes for nodes, and labels for edges.


```r
mermaid("
        graph BT
        A((Salinity))
        A-->B(Barnacles)
        B-.->|-0.10|B1{Mussels}
        A-- 0.30 -->B1

        C[Air Temp]
        C-->B
        C-.->E(Macroalgae)
        E-->B1
        C== 0.89 ==>B1

        style A fill:#FFF, stroke:#333, stroke-width:4px
        style B fill:#9AA, stroke:#9AA, stroke-width:2px
        style B1 fill:#879, stroke:#333, stroke-width:1px
        style C fill:#ADF, stroke:#333, stroke-width:2px
        style E fill:#9C2, stroke:#9C2, stroke-width:2px

        ")
```

<div class="figure">
<!--html_preserve--><div id="htmlwidget-4b6ceb7421d993430672" style="width:504px;height:504px;" class="DiagrammeR html-widget"></div>
<script type="application/json" data-for="htmlwidget-4b6ceb7421d993430672">{"x":{"diagram":"\n        graph BT\n        A((Salinity))\n        A-->B(Barnacles)\n        B-.->|-0.10|B1{Mussels}\n        A-- 0.30 -->B1\n\n        C[Air Temp]\n        C-->B\n        C-.->E(Macroalgae)\n        E-->B1\n        C== 0.89 ==>B1\n\n        style A fill:#FFF, stroke:#333, stroke-width:4px\n        style B fill:#9AA, stroke:#9AA, stroke-width:2px\n        style B1 fill:#879, stroke:#333, stroke-width:1px\n        style C fill:#ADF, stroke:#333, stroke-width:2px\n        style E fill:#9C2, stroke:#9C2, stroke-width:2px\n\n        "},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
<p class="caption">plot of chunk unnamed-chunk-8</p>
</div>

Adapted from Klinger & Blake (in prep.)
