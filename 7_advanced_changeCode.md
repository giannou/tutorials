Advanced: Change MAgPIE GAMS Code
================
Florian Humpenöder (<humpenoeder@pik-potsdam.de>)
25 April, 2019

-   [Learning objectives](#learning-objectives)
-   [Getting started](#getting-started)
    -   [Step 1](#step-1)
    -   [Step 2](#step-2)
    -   [Step 3](#step-3)
    -   [Step 4](#step-4)

Learning objectives
===================

MAgPIE has a modular concept. Each module (e.g. cropland) can have several realization (e.g. dynamic and static). This tutorial shows how to add a new realization to a module in 4 steps:

1.  create new realization via copying existing one
2.  include it properly via lucode::update\_modules\_embedding()
3.  do simple modification in new realization
4.  choose realization in cfg and run model

Getting started
===============

We want to add a new realization to the cropland module. In the MAgPIE 4.0 release the cropland module (30\_crop) has only one realization called "endo\_jun13". In this realization irrigation of bioenergy crops is prohibited. We will add a new realization, based on "endo\_jun13", which allows for irrigation of bioenergy crops. We will call the new new realization "bioen\_irrig".

Step 1
------

We first copy-paste the folder "endo\_jun13" and the file "endo\_jun13.gms" (both located in "modules/30\_crop"), and rename them to "bioen\_irrig" and "bioen\_irrig.gms" respectivly.

Step 2
------

To include the new realization "bioen\_irrig" properly into the GAMS code we run a specific R command in the main folder. First navigate in your command line to the MAgPIE main folder, then open a new R session (type "R" followed by ENTER), and then copy-paste the following R command:

``` r
lucode::update_modules_embedding()
```

Step 3
------

The prohibition of irrigated bioenergy is specified in the file "presolve.gms" within the "bioen\_irrig" realization. We just have to remove (or comment out) lines 7-9 from this file to allow for irrigation of bioenergy crops.

``` r
*vm_area.fx(j,"begr","irrigated")=0;
*vm_area.fx(j,"begr","irrigated")=0;
*vm_area.fx(j,"betr","irrigated")=0;
```

Step 4
------

To test the new realization we have to make a change in the file "config/default.cfg" in line 218. We change

``` r
cfg$gms$crop    <- "endo_jun13"
```

to

``` r
cfg$gms$crop    <- "bioen_irrig"
```

We can then start the model run with

``` r
Rscript start.R -> 1: default -> 1: Direct execution
```
