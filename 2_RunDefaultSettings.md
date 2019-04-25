Start running MAgPIE with default settings
================
Kristine Karstens (<karstens@pik-potsdam.de>)
25 April, 2019

-   [1. Introduction](#introduction)
    -   [Learning objectives](#learning-objectives)
-   [2. Start Scripts](#start-scripts)
    -   [More details:](#more-details)
-   [3. Phases of model execution](#phases-of-model-execution)
    -   [Preprocessing](#preprocessing)
    -   [GAMS model execution](#gams-model-execution)
    -   [Postprosessing](#postprosessing)
-   [4. First ideas of troubleshooting](#first-ideas-of-troubleshooting)
-   [5. Stop the model](#stop-the-model)
-   [7. Lessons learned](#lessons-learned)

1. Introduction
===============

Whereas MAgPIE inner core is written in GAMS, it comes with an outer layer for data handling in R. This also applies to the start of MAgPIE. Moreover this nested structure leads to some characteristics in code execution, that should been understand to do basic troubleshooting.

### Learning objectives

The goal of this exercise is to run MAgPIE with default settings. After completion of this exercise, you'll be able to:

1.  Start MAgPIE with the predefined start scripts.
2.  Understand the stages of model execution.
3.  Find basic indicators in the case of errors.
4.  Stop MAgPIE code.

2. Start Scripts
================

To run the model execute within in terminal (cmd for Windows, shell for Linux, MacOS) in the main folder of the model:

``` bash
Rscript start.R 
```

or from within R

``` r
source("start.R") 
```

This will give you a list of available run scripts you can choose from, looking as follows:

``` bash
Global .Rprofile loaded!


Choose start script:
1: default
2: check code
3: download data only
4: recalibrate
5: aff test
6: bmi shr
7: bra comp
8: cemics2
9: demandtest
10: disagg
11: emulator
12: fable prep
13: fable start
14: factor cost comparison
15: fix som
16: inms
17: inms2
18: MAg4 candidate
19: run time
20: sim4nexus temporary
21: sim4nexus
22: ssp cc
23: ssp deforestation
24: sustag
25: testruns
26: tradetest
Number:
```

To run a **single model run with settings as stated in default.cfg** you can choose start script **`default`**, which can be done by typing `1` and confirm via `Enter`. A new selecting list to choose the way of executing the code will show up:

``` bash
Choose submission type:
1: Direct execution
2: Background execution
3: Debug mode
Number:
```

To run a the code within your terminal you choose **`Direct execution`** (again via `1` and `Enter`).

> **Exercise**: Start a magpie run with the `default` start scrpit as `Direct execution`.

### More details:

-   **`check code`** will execute a test script within R, that check consistence of the code.
-   **`download data only`** will just execetute a download script.
-   **`recalibrate`** will recalculate the yield calibration factors. This is usually not nessessary (only if the input files change and than default settings will automatically run the recalibration with the **`default`** run script.)
-   All other start scripts refer to quite specific run settings from individual MAgPIE developers. When you become a more advanced users, you can also add your own run scripts by saving them in the folder `scripts/start`.
-   **`Background execution`** will start the model as a job in the background even running, if you close your terminal. The output will be written into \[run\_title\].log
-   **`Debug mode`** is similar to normal **`Direct execution`**.
-   If you run the code on a high performance cluster handling jobs with `SLURM`, you maybe also get a 4. and 5. option for job execution. **\``SLURM [priority/standby]`** will handle job submission to hpc. This is customize to PIK-cluster settings and may lead to problems on other hpc.

3. Phases of model execution
============================

As pointed out before the execution of the GAMS model execution is nested in pre- and postprosessing framework written R.

Preprocessing
-------------

Preprocessing starts with the execution of `Rscript start.R` and includes the following steps:

<table>
<colgroup>
<col width="35%" />
<col width="36%" />
<col width="27%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">step</th>
<th align="left">tasks:</th>
<th align="left">embedded in:</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">1. job submission</td>
<td align="left">load choosen start script, apply choosen submission type</td>
<td align="left">start.R</td>
</tr>
<tr class="even">
<td align="left">- lock model folder -</td>
<td align="left">create <code>.lock</code> folder to stop co-execution</td>
<td align="left">scripts/start_function.R</td>
</tr>
<tr class="odd">
<td align="left">2. configurate run and code check</td>
<td align="left">load libraries, configure settings, run <code>settingsCheck()</code> (lucode) to check code for consistency</td>
<td align="left">scripts/start_function.R</td>
</tr>
<tr class="even">
<td align="left">3. input data</td>
<td align="left">check, if data download is nessessary, download data</td>
<td align="left">scripts/downloader/download.R</td>
</tr>
<tr class="odd">
<td align="left">4. npi/ndc calculation</td>
<td align="left">calculate for specific cluster and regional settings the representation of land based npi/ndc policies within the model</td>
<td align="left">scripts/npi_ndc/start_npi_ndc.R</td>
</tr>
<tr class="even">
<td align="left">5. yield calibration</td>
<td align="left">calculates a regional yield calibration factor based on a pre run of magpie to be inline with FAO production data</td>
<td align="left">scripts/calibration/calc_calib.R</td>
</tr>
<tr class="odd">
<td align="left">6. gams code submission</td>
<td align="left">execute gams command to final run the gams model, start post-processing after run finished</td>
<td align="left">scripts/run_submit/submit.R</td>
</tr>
<tr class="even">
<td align="left">- unlock model folder -</td>
<td align="left">delete <code>.lock</code> folder, be ready for next call of start script</td>
<td align="left">scripts/start_function.R</td>
</tr>
</tbody>
</table>

Several of these steps will generate terminal output.

> **Exercise**: Match the terminal output to steps of preprocessing.

GAMS model execution
--------------------

The GAMS code execution is started with submit.R and by default there is no output on your terminal with regard to the optimizations prozess. You can find output in the output folder of the run:

-   `output/[run_title]/full.lst` - complitation, execution & iteration log and summaries
-   `output/[run_title]/full.log` - optimization log (detailed solver output)

<table>
<colgroup>
<col width="67%" />
<col width="32%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">step</th>
<th align="left">more information in:</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">1. code complilation</td>
<td align="left">full.lst</td>
</tr>
<tr class="even">
<td align="left">2. code execution for each time step:</td>
<td align="left">full.lst</td>
</tr>
<tr class="odd">
<td align="left">2.1. solve food demand model</td>
<td align="left">full.lst, full.log</td>
</tr>
<tr class="even">
<td align="left">2.2. solve magpie model</td>
<td align="left">full.lst, full.log</td>
</tr>
<tr class="odd">
<td align="left">2.3. iterate food demand and magpie model till convergence is reached</td>
<td align="left">full.lst</td>
</tr>
</tbody>
</table>

> **Exercise**: Open the `full.lst` and locate the different steps of gams model run.

Postprosessing
--------------

Postprocessing starts after gams runs finished. If a `fulldata.gdx` was created, the following postprocessing steps are executed:

<table>
<colgroup>
<col width="35%" />
<col width="36%" />
<col width="27%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">step</th>
<th align="left">tasks:</th>
<th align="left">embedded in:</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">1. Submit run statistics</td>
<td align="left">Submit run statistics repository</td>
<td align="left">submit.R</td>
</tr>
<tr class="even">
<td align="left">2. Execute configured output scripts</td>
<td align="left">Run output.R in postprocessing mode</td>
<td align="left">output.R</td>
</tr>
<tr class="odd">
<td align="left">2.1. rds report</td>
<td align="left">Create rds report with magpie4 library</td>
<td align="left">scripts/output/single/rds_report.R</td>
</tr>
<tr class="even">
<td align="left">2.2. validation</td>
<td align="left">Create based on report-functions (magpie4) a validation.pdf</td>
<td align="left">scripts/output/single/validation.R</td>
</tr>
<tr class="odd">
<td align="left">2.3. interpolation</td>
<td align="left">Disaggregate land use pattern to 0.5° grid, generate spam-files</td>
<td align="left">scripts/output/single/interpolation.R</td>
</tr>
<tr class="even">
<td align="left">2.4. (others)</td>
<td align="left">Several other scripts</td>
<td align="left">scripts/output/[single/comparison]/*.R</td>
</tr>
</tbody>
</table>

Several of these steps will generate terminal output. More information in tutorial `5_AnalysingModelOutputs.Rmd`.

4. First ideas of troubleshooting
=================================

Here we listed some troubles and where to find them:

<table>
<colgroup>
<col width="7%" />
<col width="37%" />
<col width="54%" />
</colgroup>
<thead>
<tr class="header">
<th align="left">step</th>
<th align="left"></th>
<th align="left">possible issues:</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">pre1.</td>
<td align="left">job submission</td>
<td align="left">General R issues (missing PATH variables)</td>
</tr>
<tr class="even">
<td align="left"></td>
<td align="left">- lock model folder -</td>
<td align="left"><code>.lock</code> folder not deleted after termination of a run</td>
</tr>
<tr class="odd">
<td align="left">pre2.</td>
<td align="left">configurate run and code check</td>
<td align="left">missing libraries, failed code check (after change in the code)</td>
</tr>
<tr class="even">
<td align="left">pre3.</td>
<td align="left">input data</td>
<td align="left">no internet connection, input data not available (check spelling)</td>
</tr>
<tr class="odd">
<td align="left">pre4.</td>
<td align="left">npi/ndc calculation</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left">pre5.</td>
<td align="left">yield calibration</td>
<td align="left">general gams issues (compilation or solver failures, missing PATH variables)</td>
</tr>
<tr class="odd">
<td align="left">pre6.</td>
<td align="left">gams code submission</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left"></td>
<td align="left">- unlock model folder -</td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left">gams1.</td>
<td align="left">code complilation</td>
<td align="left">general gams issues (compilation or solver failures, missing PATH variables)</td>
</tr>
<tr class="even">
<td align="left">gams2.</td>
<td align="left">code execution for each time step:</td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left">gams2.1.</td>
<td align="left">solve food demand model</td>
<td align="left">Infeasibilties</td>
</tr>
<tr class="even">
<td align="left">gams2.2.</td>
<td align="left">solve magpie model</td>
<td align="left">Infeasibilties</td>
</tr>
<tr class="odd">
<td align="left">gams2.3.</td>
<td align="left">iterate food demand and magpie model till convergence is reached</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left">post1.</td>
<td align="left">Submit run statistics</td>
<td align="left">No access to repository (not critical)</td>
</tr>
<tr class="odd">
<td align="left">post2.</td>
<td align="left">Execute configured output scripts</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left">post2.1.</td>
<td align="left">rds report</td>
<td align="left">missing libraries (specially gdx, gdxrrw, magpie4)</td>
</tr>
<tr class="odd">
<td align="left">post2.2.</td>
<td align="left">validation</td>
<td align="left">latexrelated r-extension are not working or missing</td>
</tr>
<tr class="even">
<td align="left">post2.3.</td>
<td align="left">interpolation</td>
<td align="left"></td>
</tr>
<tr class="odd">
<td align="left">post2.4.</td>
<td align="left">(others)</td>
<td align="left">r extension are missing (e.g. ncdf)</td>
</tr>
</tbody>
</table>

> **Exercise**: If your run fails, try to find out with the help of terminal output and `full.lst`, `full.log`, what went wrong.

5. Stop the model
=================

-   The model can be stopped with `Crtl` + `C`.
-   If you run it in `background mode` you have to kill the job over the Task Manager or process handler (linux: `top`).
-   Make sure that you delete the `.lock` folder, if it was not deleted automatically to unlock the model after a termination of a run.

> **Exercise**: Stop the magpie run with `Crtl` + `C`.

7. Lessons learned
==================

1.  You started a MAgPIE run with the predefined **`default` start scripts**.
2.  You had a look into on the **terminal output** and the **full.lst**.
3.  Maybe: you solved some first issues.
4.  You stopped a MAgPIE run.
