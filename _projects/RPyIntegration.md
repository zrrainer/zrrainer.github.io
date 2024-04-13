---
layout: page
title: R-python integration
description: a quick physics simulation
---
Pain and suffering. 

I have written some simple code that runs a physics simulation, calculated via python and visualized via the R package ggplot2. This repo documents me playing with (losing sanity over) rpy2, the most popular python package for integrating R code into python, and reticulate, themost popular R package for integrating python code into R.  



some reflections on rpy2:

rpy2 has very obvious weak points: its syntax are arduous and, frankly, very ugly. Every function in the R script needs to be imported into a python object individually, on top of sourcing the R script itself, leading to ridiculous code such as

RcreateDf = robjects.globalenv['createDf']  
RcreateDf()

Oh, rpy2 also does not have windows binaries. So.  





reflections on reticulate:

direct access to the methods in the python script. cool. awesome. 10/10. the trouble is displaying the ggplot objects.

while using rpy2 I took the easy way out and wrote all the code in a ipynb file, and displayed the outputs using, uh, display(). R markdown do not have the same behavior. the main loop for the animation never ends, and the ggplot objects are never actually printed out.  




2024.1.31

display() was more of an easy way out than I thought. without using the IPython.display module, it is nigh impossible to convert the ggplot object into a useful python object... i am temporarily resorting to doing it the heathen way - saving the ggplot object as a local file and reading it with PIL. which works. for now. (i am deeply exasperated)

additionally, ktinker....... every time i call update on a tk window it flashes for a second. which would be fine if i werent updating every 10 ms. 

![](https://github.com/zrrainer/RPyIntegration/blob/main/images/devil-may-cry-devil-may-cry3.gif)

ah. maybe the lesson here is that dont bother integrating other languages if not necessary. i have a feeling i can get matplotlib working within a couple hours.... 















After some wrangling, a quick EDA plotting measurements over time, such as these ones:
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/MOVERTcbd.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/movertcbn.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/movertTHCCOOH.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>



Now we needed something to quantify how well a given cutoff works for any specific biomarker. With a helper function that takes in a cutoff as a parameter and return that cutoff's sensitivity (true positives / (total # of observations)) and specificity (true negatives / (total # of observations)), we can then visualize the effectiveness of every possible cutoff. 

tdlr: we want a cutoff with **high sensitivity** and **high specificity**. 


{% raw %}
```r
#calculate sensiticity and specificity for each cutoff of each biomarker
output_WB <- map_dfr(compounds_WB,
                     ~ sens_spec_cpd(
                       dataset = WB,
                       cpd = all_of(.x),
                       timepoints =  timepoints_WB
                     )) |> clean_gluc()

output_BR <- map_dfr(compounds_BR, 
                     ~ sens_spec_cpd(
                       dataset = BR,
                       cpd = all_of(.x),
                       timepoints = timepoints_BR
                     ))  |> clean_gluc()

output_OF <- map_dfr(compounds_OF,
                     ~ sens_spec_cpd(
                       dataset = OF,
                       cpd = all_of(.x),
                       timepoints = timepoints_OF
                     ))  |> clean_gluc()

#plotting
ss_bottom_row <-
  plot_grid(
    ss_OF,
    ss_BR,
    labels = c('B', 'C'),
    label_size = 12,
    ncol = 2,
    rel_widths = c(0.66, .33)
  )
plot_grid(
  ss_WB,
  ss_bottom_row,
  labels = c('A', ''),
  label_size = 12,
  ncol = 1
)
```
{% endraw %}

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/DLvsSS.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Here we see the sensitivity of every possible cutoff at different time windows. We as a team decided that the cutoffs should be judged upon their average sensiticity and specificity. (Although, in hindsight, it might've been a good idea to base our analysis solely on the ealier time windows - we are trying to detect *recent* canibis use after all.) Plotting the average sensitivity and specificity of each cutoff, we get:

Whole blood:
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/avgSS.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Oral fluid:
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/avgSSOF.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
Breath:
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/avgSSBR.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

It should be apparent that OF-THC and WB-THC are the superior choices. The average sensitivity and specificity intersects at some cutoffs, meaning at that point, both the sensitivity and specificity is maximized. Additionally, if we eye ball their point of intersection, OF-THC has a significantly higher sensitivity and specificity at that point, making it the most optimal biomarker. 

Now we investigate OF-THC further to find the specific cutoff. In short, we basically zoom in on the graph. Giving us:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cs01assets/OFTHCzoomed.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

And the specific detection limit:


{% raw %}
```r
output_OF_avg |>
  mutate(diff = abs(average_sensitivity-average_specificity)) |>
  arrange(diff)


  ## # A tibble: 101 × 5
  ##    compound detection_limit average_sensitivity average_specificity    diff   
  ##    <chr>              <dbl>               <dbl>               <dbl>   <dbl>  
  ##  1 THC                 0.82               0.893               0.890 0.00338   
  ##  2 THC                 0.84               0.893               0.890 0.00338  
  ##  3 THC                 0.86               0.893               0.890 0.00338  
  ##  4 THC                 0.88               0.893               0.890 0.00338  
  ##  5 THC                 0.9                0.893               0.890 0.00338  
  ##  6 THC                 0.7                0.903               0.879 0.0240   
  ##  7 THC                 0.72               0.903               0.879 0.0240   
  ##  8 THC                 0.74               0.903               0.879 0.0240   
  ##  9 THC                 0.76               0.903               0.879 0.0240   
  ## 10 THC                 0.78               0.903               0.879 0.0240   
  ## # ℹ 91 more rows  
```
{% endraw %}

  At the cutoff range of 0.82 to 0.90, the difference between the average sensitivity and average specificity is minimized. We will settle with **>0.85ng/mL of THC in oral fluid** sample as the final detection limit. 




