# microFIM

![alt text](microFIM_framework.jpg)

## Overview
**microFIM (microbial Frequent Itemset Mining) is a Python tool for the integration of Frequent Itemset Mining approach (also known as Association Rule Mining - ARM) into microbiome pattern analysis**.

The tool is developed to create a bridge between microbial ecology researchers and ARM technique, integrating the common microbiome outputs (in particular, OTU, ESV and taxa table), metadata files typically used in microbiome analysis, and it provides similar microbiome outputs that help scientists to integrate ARM in microbiome pipelines. 

In detail, microFIM generates the **pattern table** - an OTU table built with the patterns extracted via ARM (see Figure above as an example)- that can be used for further statistical analysis and microbiome visualization strategies.

microFIM is also versatile. It is suited for metabarcoding projects and datasets that respect the input format requests (see the **microFIM input** section for details and examples).

### Frequent Itemset Mining (or ARM) in a nutshell
Association rule mining is a supervised machine learning method for discovering interesting relations between variables in large databases. It is intended to identify patterns (group of items) and strong rules discovered in databases using some measures of interestingness.

microFIM is implemeted to integrate ARM analysis in 16S rRNA microbiome pipelines. An example of applications is provided in the figure below.

![alt text](arm_microbiome_applications.png)

microFIM, in particular, supports **pattern mining phase** (itemset extraction), in order to extract interesting patterns from 16S rRNA microbiome data.
Details about ARM approach are available in the work of Naulearts et al., 2016 [[1]](#1) and Agrawal et al., 1993 [[2]](#2).

microFIM framework is developed in **6 main steps** (see the next sections for details and the main figure for an **overview**) and it integrates the following metrics:

* to extract patterns, the mandatory parameters (also available in [input_templates](input_templates) directory) are the following:
   * minimum support to be considered to extract interesting patterns, intended as Support = frq(X,Y)/N, e.g. the frequency of the pattern in the dataset;
   * minimum length, intended as the minimum number of elements that composed the patterns;
   * maximum length, intended as the maximum number of elements that composed the patterns.

* measures included to evaluate patterns extracted and available in microFIM outputs:
   * Support of the patterns;
   * length of the patterns;
   * number of samples in which the patterns were found.
   
* interest measures integrated in microFIM framework:
   * all-confidence - metrics to evaluate 'hypercliques patterns' [[3]](#3). 
   
     Considering a pattern ‘X’ composed of different items, all-confidence is calculated as the ratio between the support of ‘X’ and the highest support   retrieved from the elements of the pattern ‘X’.
   
      all-confidence(X) = min{P(X|Y),P(Y|X)}
   
       All-confidence means that all rules which can be generated from itemset X have at least a confidence of all-confidence(X). 
       For details, see [[3]](#3)[[4]](#5).
       
    Example: Considering a pattern ‘X’ composed of different items, all-confidence is calculated as the ratio between the support of ‘X’ and the highest support retrieved from the elements of the pattern ‘X’. For example, a pattern X is composed of 3 elements that, considering the entire dataset, have the following support threshold: 0.3, 0.6 and 0.8. Overall, the pattern X has a support of 0.3. All-confidence will be calculated as the ratio between the support of X - 0.3 - and the higher support within X - 0.8, resulting in 0.37. All-confidence, in this way, is defined as the smallest confidence of all rules which can be produced from a pattern, i.e., all rules produced from a pattern will have a confidence greater or equal to its all-confidence value [[3]](#3)[[5]](#5). In detail, confidence is an indication of how often a rule has been found to be true, so it is considered as a measure of rule reliability [[1]](#1)[[6]](#6).

Below, installation, instructions of use and tutorials are provided.

## Installation
1. Download Python 3 if you haven’t already at https://www.python.org/
2. Clone github repository (link: https://github.com/qLSLab/microFIM.git) and:
    * Download ZIP from microFIM github home page, then decompress it\
    or 
    * Use git from command line: `git clone https://github.com/qLSLab/microFim.git`

3. We suggest to install Conda or Mininconda to run microFIM - here you can find how to install Miniconda3: https://conda.io/projects/conda/en/latest/user-guide/install/index.html

4. Once Miniconda is installed, create the conda environment with the command below: \
`conda create --name microFIM --file requirements.txt --channel default --channel conda-forge --channel plotly`

5. (Optional) Test microFIM before starting with your analysis - see Tutorial section for details

## Usage
microFIM can be used via guided scripts or python functions. \
Please see [microfim_tutorial_notebook](microfim_tutorial_notebook.ipynb) for complete tutorials for both usage.
Below, recomendations about input files format and explanation about the structure and functions module are available.

### Input/output microFIM files
microFIM accepts as input taxa table and metadata in CSV or TSV format. An example of taxa table is provided in [tutorials](tutorials) (e.g. as test1.csv or test2.csv). In particular, the file is composed by the rows and columns representing the taxa with their abundances for each sample (visible also in the figure below). This kind of file derives from the conversion of the BIOM file into a CSV file (https://biom-format.org/). There are different ways to convert a BIOM file into a TSV or CSV file, at the end of the page we provide some options. In general, you can find taxa tables in different sources, as for example in QIITA platform (https://qiita.ucsd.edu/; [[7]](#7)) or in MLrepo (https://knights-lab.github.io/MLRepo/; [[8]](#8)).
Considering microbiome analysis, QIIME2 provide complete frameworks and scripts to analyse and obtain taxa tables (https://qiime2.org/; [[9]](#9)).

![alt text](taxa_table_example.png)

#### Input description
Before starting with microFIM, be sure to set the following requirements for **inputs**:
* #ID \
In the taxa table, the column describing taxa must be filled as #ID. You can rename it with `sed -i 's/SEARCH_REGEX/REPLACEMENT/g' INPUTFILE` \
or a text editor;
* #SampleID \
If you want to filter your taxa table with a list of samples or metadata file, the column name must be filled as #SampleID; 
* CSV or TSV format \
Taxa table and metadata file must be provided as CSV or TSV files. Template file is already defined as CSV in [input_templates](input_templates) directory.

#### Output description
* transactional file: taxa table is converted into a transactional file, where each row represents a sample with the list of elements it contains;
* pattern datadrame: for each pattern, interesting measures are calculated and represented in CSV dataframe;
* pattern table as CSV: after pattern extraction, a presence-absence matrix is calculated, considering for each pattern if it is present or not in each sample - the results is similar to a taxa table, where the first column contains the patterns while the others describe in which sample are found. Columns considering the length of the patterns and their support are reported. Additional metrics can be also added (see Next section for details) - see the Figure below for an example;
* plots as SVG and HTML: visualizations as barplot, scatter plot and heatmap can be generated.


### Structure
Guided scripts are defined in **6 main phases**, also represented in [microFIM_framework](microFIM_framework.jpg) figure.
For complete tutorials, see [microfim_tutorial_notebook](microfim_tutorial_notebook.ipynb). 

#### Script usage
* Step 1: **importing and filtering taxa table** - in the first phase you can filter your taxa table using the metadata file (via #SampleID column). 
If you want to run microFIM on your taxa table without filtering, go to the next step.
The script to filter is [script_1_filtertable.py](script_1_filtertable.py). 

* Step 2: **conversion into a transactional dataset** - taxa table must be converted in a transactional file (see the previous description for details). The script to convert it is [script_2_tableconversion.py](script_2_tableconversion.py).

* Step 3: **patterns extraction** - microFIM extract patterns from the transactional file (see previous step). To run [script_3_microfimcalculation.py](script_3_microfimcalculation.py), files with parameters must be given as input, you can find examples to be filled in [template_inputs](template_inputs).
The parameters to be filled are: 
   * minimum support (minsupp);
   * minimum length (zmin);
   * maximum length (zmax).
We recomment to keep the option **report = [asS** in order to correctly used our scripts.

* Step 4: **integration of additional interest measures** - microFIM provides additional interest metrics to evaluate patterns, in particular all-Confidence is added. However, other metrics can be added. For each pattern, the value of the interest measure is added in a separate column, in order to guarantee further analysis and filtering steps. The script to calculate additional interest measures is [script_4_additionalmeasures.py](script_4_additionalmeasures.py). 

* Step 5: **creation of the pattern table** - similar to a taxa table, the pattern table describes the presence-absence of a pattern in the samples analysed. A CSV file is obtained. The script to obtain the pattern table is [script_5_generatepatterntable.py](script_5_generatepatterntable.py). 

* Step 6: **visualization of results** - from the pattern table, plot can be obtained. In particular, bar plot, scatter plot and heatmap are available. The standard output are SVG and HTML format. The script to generate visualizations is [script_6_generateplots.py](script_6_generateplots.py).

#### Function usage
* microdir module: functions for setting project directory.
* microimport module: functions for importing input files.
* microfim module: main functions of microFIM - converting data, extract patterns, generated main outputs.
* microinteresmeasures module: functions implementing interesting measures.
* microplots module: functions for visualization of results.

## Contacts
For any doubt and help, please contact g.agostinetto@campus.unimib.it

## Cite us
Link

## References
<a id="1">[1]</a> 
Naulaerts, S., Meysman, P., Bittremieux, W., Vu, T. N., Vanden Berghe, W., Goethals, B., & Laukens, K. (2015). A primer to frequent itemset mining for bioinformatics. Briefings in bioinformatics, 16(2), 216-231. \
<a id="2">[2]</a> 
Agrawal, R., Imieliński, T., & Swami, A. (1993, June). Mining association rules between sets of items in large databases. In Proceedings of the 1993 ACM SIGMOD international conference on Management of data (pp. 207-216). \
<a id="1">[3]</a> 
Omiecinski, E. R. (2003). Alternative interest measures for mining associations in databases. IEEE Transactions on Knowledge and Data Engineering, 15(1), 57-69. \
<a id="1">[4]</a> 
Xiong, H., Tan, P. N., & Kumar, V. (2006). Hyperclique pattern discovery. Data mining and knowledge discovery, 13(2), 219-242. \
<a id="1">[5]</a> 
Tan, P. N., Kumar, V., & Srivastava, J. (2002, July). Selecting the right interestingness measure for association patterns. Proc. ACM SIGKDD Int. (pp. 32-41). \
<a id="1">[6]</a> 
Hahsler, M., Chelluboina, S., Hornik, K., & Buchta, C. (2011). The arules R-package ecosystem: analyzing interesting patterns from large transaction data sets. The Journal of Machine Learning Research, 12, 2021-2025. \
<a id="1">[7]</a> 
Gonzalez, A., Navas-Molina, J. A., Kosciolek, T., McDonald, D., Vázquez-Baeza, Y., Ackermann, G., ... & Knight, R. (2018). Qiita: rapid, web-enabled microbiome meta-analysis. Nature methods, 15(10), 796-798. \
<a id="2">[8]</a> 
Bolyen, E., Rideout, J. R., Dillon, M. R., Bokulich, N. A., Abnet, C. C., Al-Ghalith, G. A., ... & Caporaso, J. G. (2019). Reproducible, interactive, scalable and extensible microbiome data science using QIIME 2. Nature biotechnology, 37(8), 852-857. \
<a id="3">[9]</a> 
Vangay, P., Hillmann, B. M., & Knights, D. (2019). Microbiome Learning Repo (ML Repo): A public repository of microbiome regression and classification tasks. Gigascience, 8(5), giz042.
