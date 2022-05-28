# Source-tracking for Contamination Removal in *micro*Biomes (SCRuB)

![Screenshot](Images/SCRuB_logo.png)

SCRuB is a tool designed to help researchers address the common issue of contamination in microbial studies. This package provides an easy to use framework to apply SCRuB to your projects. All you need to get started are n samples x m taxa count matrices for both your samples and controls. 


Support
-----------------------

For support using SCRuB, please use our <a href="https://github.com/korem-lab/SCRuB/issues">issues page</a> or email: gia2105@columbia.edu.


Software Requirements and dependencies
-----------------------

*SCRuB* is implemented in R (>= 3.6.3) and requires the following dependencies: **glmnet**, **torch**, **tidyverse**. Please install and load them prior to trying to install *SCRuB*. 

```
install.packages( c('glmnet', 'torch', 'tidyverse') )
```


Installation
---------------------------

*SCRuB* will be available on QIIME 2 very soon. Until then you can you can simply install *SCRuB* using **devtools**: 
```
devtools::install_github("korem-lab/SCRuB")
```

Runtime 
-------------------
As an expectation-maximization model, SCRuB is highly efficient, with very low computational costs. On a standard machine, it only requires 5 seconds to install SCRuB using devtools. For the example dataset consisting of 384 samples, we expect the SCRuB function to require at most 45 seconds of runtime. As a result, the full tutorial notebook we provide, which loads the data and runs through multiple variations of SCRuB, will run successfully in under one minute!


Usage
___________________

### As input, *SCRuB* takes arguments:

 _data_ - An ( n_samples + n_controls ) x n_taxa count matrix.
 
_metadata_ - a metadata matrix of 2 columns (and a third optional column), where each row name is aligned to the `data` parameter row name. 
The first metadata column is boolean, denoting `TRUE` if the corresponding `data` row belongs to a sample representing a contamination source, and `FALSE` otherwise. 
The second column is a string, identifying the type of each sample, such that SCRuB can identifying which control samples should be grouped together. 
The third (and optional, but highly recommended) column is a string entry identifying the well location of the corresponding sample, which allows SCRuB to track well leakage.
This must be in a standard *LETTER**NUMBER* format, i.e. A3, B12, D4...
 
(optional) _well_dist_ - An ( n_samples + n_controls ) x ( n_samples + n_controls ) distance matrix, summarizing the pairwise distance between each sample. Both the row names and column names of _well_dist_ must correspond to the row names of the _data_ matrix. See our <a href="https://korem-lab.github.io/SCRuB/tutorial.html">tutorial</a> for a demonstration of how to create this input, using Euclidean distance. 

(optional) _dist_threshold_ float - Determines the maximum distance between samples and controls which SCRuB determines as potential sources of well leakage. Default of 1.5.


#### As output, *SCRuB* returns a list containing:

 decontaminated_samples - a n_samples x n_taxa count matrix, representing the decontaminated samples.
 
 p - The fitted p parameter, as described in SCRuB's methods. An n_sample vector representing the estimate proportion of each observe sample that was not contamination. A dataset that had no contamination would have a p of 1s, while a dataset of entirely contamination would have a p of 0.

3) inner_iterations -- results from SCRuB's intermediary steps, which includes one list entry per type of control. Each iteration contains:

	- 3.1)d econtaminated_samples - a n_samples x n_taxa count matrix, representing the decontaminated samples.
 
 	- 3.2) p - The fitted p parameter, as described in SCRuB's methods. An n_sample vector representing the estimate proportion of each observe sample that was not contamination. A dataset that had no contamination would have a p of 1s, while a dataset of entirely contamination would have a p of 0.
 
 	- 3.3) alpha - The fitted \alpha parameter, as described in SCRuB's methods. An n_control x ( n_sample + 1 ) matrix, representing the estimated contribution of the contaminant and each sample to each control, where the (n_sample + 1)th column represents the contribution from the contamination to the control. Each row of alpha sums to 1, with each entry of the (n_sample + 1)th  column being 1 means there is zero estimated well leakage, while entries close to zero would indicate there is a high level of well leakage.
 
	- 3.4) gamma - the $\gamma$ parameter described in SCRuB's methods. An n_taxa vector representing the estimated relative abundance of the contamination community
loglikelihood - float. The log-likelihood of the inputted dataset based on SCRuB's fitted parameters.


 


Demo
-----------------------
We provide a dataset for an example of *SCRuB* usage. Download the demo files <a href="https://github.com/korem-lab/SCRuB/SCRuB/tutorial/">here</a>. We provide an rmarkdown notebook <a href="https://github.com/korem-lab/SCRuB/tutorial/tutorial.Rmd">here</a> to follow along with the example below.

First load the **SCRuB** packages into R:
```
library(SCRuB)
```

Then, load the datasets:
```
data <- read.csv('plasma_data.csv', row.names=1) %>% as.matrix()
```

Next, load the well positions metadata
```

metadata <- read.csv('plasma_metadata.csv', row.names=1)
```

Run *SCRuB*,:

```
scr_out <- SCRuB(data = data, 
                 metadata=metadata
                 )
```


Input format
-----------------------
The input to *SCRuB* is composed of one count matrix, and one metadata matrix (or data frame):

(1) data - ( n_samples + n_controls ) x n_taxa count matrix, where m is the number samples and n is the number of taxa. Row names are the sample ids ('SampleID'). Column names are the taxa ids. Every consecutive column contains read counts for each sample. Note that this order must be respected.


count matrix (first 4 rows and columns):

| | taxa_1 | taxa_2 | taxa_3 | taxa_4 |
| ------------- | ------------- |------------- |------------- |------------- |
| ERR525698  |  0 | 5 | 0|20 |
| ERR525693  |  15 | 5 | 0|0 |
| ERR525688  |  0 | 13 | 0| 200 |
| ERR525699  |  4 | 5 | 0|0 |

(2) data - (n_samples + n_controls x 2, or (n_samples + n_controls x 3. The columns must be ordered as described below, with the third column being optional. The three columns are 1) a boolean indicator identifying which entries correspond to control samples; 2) a string column denoting the types of control samples; and (optionally) 3) the plate location of each sample.

| | is_control | sample_type | sample_well |
| ------------- | ------------- |------------- |------------- |
| ERR525698  |  FALSE | plasma | A1|
| ERR525693  |  FALSE | plasma | A2|
| ERR525688  |  TRUE | extraction control | B1| 
| ERR525699  |  FALSE | plasma | B2|




 

Output format
-----------------------
SCRuB outputs a list containing the following:

(1) decontaminated_samples

| | taxa_1 | taxa_2 | taxa_3 | taxa_4 |
| ------------- | ------------- |------------- |------------- |------------- |
| ERR525698  |  0 | 4 | 0 | 0 |
| ERR525693  |  15 | 5 | 0 | 0 |
| ERR525699  |  4 | 5 | 0 | 0 |

(2) p - The estimated fraction of each sample that is not contamination
```
c(0.16, 1, 1)
```
(3) outputs from inner SCRuB iterations, which includes fitted (3.1) decontaminated samples and (3.2) p, as described above, and:

(3.3) alpha - the estimated composition of each control:
| | ERR525698  | ERR525693 | ERR525699 | contaminant  |  
| ------------- | ------------- | ------------- | ------------- | ------------- |
| ERR525688 | 0.001 | 0.001 | 0.001 | 0.997 |



(3.4) gamma - The estimated relative abundance of the contamination community
```
c(0, 0.05, 0, 0.95)
```









