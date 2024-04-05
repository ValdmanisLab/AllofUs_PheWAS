# PheWAS Analysis in the NIH All of Us Dataset

## Overview

This repository contains scripts and documentation for performing a PheWAS (Phenome-wide association study) analysis in the NIH All of Us dataset using Python and R. The analysis involves various steps including data preprocessing, genotype processing, population stratification analysis, and PheWAS using R libraries.

## Analysis Steps

### 1st Step: Setup and Data Retrieval

- Install required Python libraries and packages.
- Initialize environment and download necessary files.
- Retrieve patient personal information and conditions from the database using SQL.

### 2nd Step: Variable Setup

- Set variables for SNP name, SNP position, ethnicity, reference allele, alternative allele, genome quality threshold, and minimum Phecode threshold.

### 3rd Step: Main Pipeline

- Perform data preprocessing including filtering, genotype processing, and population stratification analysis.
- Generate PCA graphs and prepare dataframes for analysis.
- Prepare data for R PheWAS.

### 4th Step: UMAP Usage

- Utilize UMAP for dimensionality reduction and clustering analysis.
- Visualize clusters and genotype distribution on the map.

### 5th Step: PheWAS Analysis in R

- Load R libraries and install PheWAS if not already installed.
- Read data produced from Python and perform PheWAS analysis with covariates.
- Generate Manhattan plots for visualization.

## Genotype Algorithm

The genotype algorithm is designed to process data from a dataframe containing information about allele rows, reference alleles, alternative alleles, and genotype information.

1. Make a variable message.
2. Remove IDs with missing genotype information.
3. Read the 'GT' column, for example '0/1'.
4. Split the allele info to create two new columns named Allele_1 and Allele_2, for example:
    
    <details>
    <summary>Example Step 4: Split Allele Info</summary>
        
        - Allele_1   Allele_2             GT
        - "0"            "1"            "0/1"
        - "1"            "1"            "1/1"
        - "0"            "0"            "0/0"
        - "1"            "2"            "1/2"
    </details>
5. From the allele row, read the alleles, for example: ['C','T','G','GTA'].
6. If the reference_allele input is provided (reference_allele != None), set the reference allele as the provided value.
7. Otherwise, choose the first allele from the allele row list (position 0), for example:
    <details>
    <summary>Example Step 7: Choose Reference Allele</summary>
        - 'C'            'T'            'G'            'GTA'</br>
        - 0               1                2                3        
        
    </details>

8. Create three new columns named Allele_1_map, Allele_2_map, and Allele_combination using the allele row to assign a letter to the two alleles, for example:
    <details>
    <summary>Example Step 8: Create Allele Maps</summary>
    
        -   GT    Allele_1       Allele_2       Allele_1_map           Allele_2_map           Allele_combination
        - "0/1"     "0"             "1"               "C"                     "T"                      "C/T"
        - "1/1"     "1"             "1"               "T"                      "T"                      "T/T"
        - "0/0"     "0"             "0"               "C"                     "C"                      "C/C"
        - "1/2"     "1"             "2"               "T"                      "G"                      "T/G"
    </details>
9. Create a column named Allele_count to count how many times each Allele_combination occurs, for example:
    <details>
    <summary>Example Step 9: Count Allele Combinations</summary>
        
        - Allele_combination   Allele_count
        - 'C/T'                            3000
        - 'T/T'                            100
        - 'C/C'                           10000
        - 'T/G'                               5
        - 'C/G'                              10
    </details>
10. Find the alternative allele:
    - If an alternative_allele is provided, use that.
    - Otherwise, take the reference_allele ('C') and find all combinations inside the Allele_combination.
11. Find the most frequent combination that is not the 'reference_allele/reference_allele', for example:
    <details>
    <summary>Example Step 11: Find Alternative Allele</summary>
    
    - Alternative Allele: 'T'
    </details>
12. Remove the reference allele from the most frequent combination and set it as the alternative_allele, for example:
    - 'C/T' (remove 'C/') = 'T'.
13. Now, we have the reference and alternative alleles ('C' and 'T').
14. Update the GT column with the final information:
    - Set 'reference_allele/reference_allele' = 0, indicating those without the SNP.
    - Set 'reference_allele/alternative_allele' = 1 and 'alternative_allele/reference_allele' = 1 for heterozygous genotypes.
    - Set 'alternative_allele/alternative_allele' = 2 for homozygous genotypes.
    <details>
    <summary>Example Step 14: Update GT Column</summary>
        
        - GT   Allele_1   Allele_2   Allele_combination
        - 'reference_allele/reference_allele' = 0: 'C/C'
        - 'reference_allele/alternative_allele' = 1: 'C/T'
        - 'alternative_allele/reference_allele' = 1: 'T/C'
        - 'alternative_allele/alternative_allele' = 2: 'T/T'
    </details>






15.  Remove any rows in the dataset that do not have 0,1 or 2 genotypes.


### File Structure

The pipeline generates graphs and information about each step of the analysis as PDF files. At the end of the analysis, navigate to the `download` directory and then to `new_run` to download the files. 

- **Graphs**: Contains visualizations generated during the analysis.
- **PheWAS Output**: In the `new_run` directory, you will find the output of the PheWAS, including CSV files and Manhattan plots of the analysis.


### Example of how to set your Variables

#### Python
- `all_messages` = "" # Set up the message variable
- `rs_name` = 'rs1065853' # SNP's Name
- `rs_position` = 'chr19:44909976-44909977' # position 
- `sex` = None  # Choose the sex variable ('Male', 'Female', or None). Default is None.<br />
- `pop_variable` = 'all'  # Choose the population variable ['afr', 'amr', 'eas','eur', 'mid', 'sas', 'all']. Default is 'all'.<br />
- `ethnicity` = None  # Choose the ethnicity ('Hispanic or Latino' or 'Not Hispanic or Latino'), or None for classic run. Default is None.<br />
- `reference_allele` = None  # Choose the reference allele. Default is None.<br />
- `alternative_allele` = None  # Choose the alternative allele. Default is None.<br />
- `GQ_threshold` = 20  # Genome Quality threshold. Default is 20.<br />
- `phecode_min` = 0  # Phecode minimum threshold. Default is 0.<br />
- `icd9cm` = True  # Use ICD-9 codes. Default is True.<br />
- `icd10cm` = True  # Use ICD-10 codes. Default is True.<br />

#### R
- `rs_name` <- 'rs199768005' # Name of the SNP.
- `pop_variable` <- 'all' # Name of the population.
- `i` <- sprintf('as Cov Sex+ 1-3 principals_comp+ age+ %s with all Diseases',pop_variable) # name of the end of the files.



## Installation

### Python

1. Open a Jupyter Notebook.
2. In the first cell, set up the environment and read necessary functions. This step is required each time you open the Jupyter Notebook.
    - Install required Python packages:
        <details>
        <summary>Packages we need to pip install:</summary>

        - geopandas<br />
        - pgeocode<br />
        - geopy<br />
        - flexitext<br />
        - umap<br />
        - umap-learn<br />
        - datashader<br />
        - bokeh<br />
        - holoviews<br />
        - scikit-image<br />
        - colorcet<br />
        - dask[complete]<br />

        </details>
    - Install required Python libraries:
        <details>
        <summary>Python libraries list</summary>

        - pandas<br />
        - os<br />
        - subprocess<br />
        - scipy.stats<br />
        - statsmodels.stats.multicomp<br />
        - numpy<br />
        - matplotlib.pyplot<br />
        - seaborn<br />
        - geopandas<br />
        - pgeocode<br />
        - geopy.geocoders.Nominatim<br />
        - geopy.exc.GeocoderTimedOut<br />
        - hail<br />
        - hail.plot.show<br />
        - sklearn.decomposition.PCA<br />
        - ast<br />
        - matplotlib.lines.Line2D<br />
        - tabulate<br />
        - umap.plot<br />
        - sklearn.preprocessing.PowerTransformer<br />
        - sklearn.cluster.KMeans<br />
        - sklearn.preprocessing.OneHotEncoder<br />
        - importlib.util<br />
        - multiprocessing<br />

        </details>





### R

1. Open an R Jupyter Notebook.
2. Install the PheWAS library from GitHub. This step is only required once. 
    <details>
    <summary>Install PheWAS</summary>
    
    - install.packages("devtools")<br />
    - devtools::install_github("PheWAS/PheWAS")<br />
        
    </details>
3. Load libraries: 
    <details>
    <summary>Load Libraries</summary>

    - library(tidyverse)  # Data wrangling packages.<br />
    - library(dplyr)<br />
    - library(parallel)<br />
    - library(PheWAS)<br />
    
    </details>
## Usage



1. **Install necessary libraries and packages:**
   - Make sure to install all required Python libraries and packages. You can do this by executing the provided list of required packages.

2. **Set up the environment:**
   - In the first cell of your Jupyter Notebook, set up the environment by importing necessary libraries, initializing functions, and initializing the Hail environment. 
   - This step is essential and should be executed each time you open the notebook. 
   - Additionally, download useful files such as relatedness and ancestry information, and read patient personal information and conditions from the database. 
   - Note that the first step is only run the first time the notebook is opened.

3. **Set variables for analysis:**
   - Define variables such as SNP name, SNP position, ethnicity, reference allele, alternative allele, genome quality, and minimum Phecode threshold. 
   - Decide whether to use ICD-9 and/or ICD-10 Phecodes.

4. **Start the analysis:**
   - Begin the main pipeline for analysis.

5. **Usage of UMAP:**
   - Utilize UMAP for dimensionality reduction and visualization of data.


6. **PheWAS Analysis in R**
    - Install the PheWAS library from GitHub.
    - Load R libraries and install PheWAS if not already installed.
    - Read data produced from Python and perform PheWAS analysis with covariates.
    - Generate Manhattan plots for visualization.
## All Of Us Helping Information

### Creating Cohort

#### 1st Step

Choose what our Data Set should have. For example:
- Contains EHR Data Code
- Short Read WGS

If you want to exclude a characteristic or specific type of people in the dataset, use the second column.

#### 2nd Step

We need to Create a concept:
1. Press the + to start the process of creating a Dataset.
2. Select Concept sets.
3. Then you can choose your filters and the information that your data should have.
4. Choose the Disease that we want these datasets to have or specific characteristics about the population. Then we must Save the Concept Set to use it also in the future. We have 2 choices to create a new Description or upload an old one. First, we need to add, for example, conditions:
   - We can create a new set with a Description of the conditions that we removed and what is the target of this experiment.
   - Or we can update an old one.

#### 3rd Step

We need to create a Dataset.
- Choose the specific Cohort that we created or All Participants (Default cohort).
- Then choose the Concepts Sets. Here is the place that we pull information about the patients. As you can see in the picture, we can create a concept by choosing all the diseases or the Zip-Code data.
- Also, we have to pick what columns we want to have in the Dataframe that we will download inside the Jupiter Notebook. It is easier to press "View Preview Table" so that we understand what the data are. Also, the columns that we don't want to use can be unchecked.
- Next, we must Create the Dataset:
    - Choose a Name and give some Description.

#### 4th Step

If this is the first Jupiter notebook, then choose the language and the type of notebook, the name and then export the Notebook.

Tip: If you already have a Jupiter Notebook, then it is best to take the SQL command to pull the data that you want:
- Then you can Copy the SQL command to your Jupiter Notebook.

And with this, you will Create a Dataframe to do your analysis.

### Creating a Cloud Environment


To read the data frames at the Python Pipeline without crashing the virtual environment, you need at least 16 CPUs and 104 RAM. The RAM that we have in this option we cannot run the R PheWAS, but we can program and do Data Analysis.

If we want to have an environment to run the Python Pipeline and the R pipeline, then we need at least 64 CPUs and 416 GB RAM:



## References

- [PheWAS GitHub Repository](https://github.com/PheWAS/PheWAS): This repository contains resources and information related to PheWAS analysis. It serves as a valuable reference for understanding and implementing PheWAS methodologies.


## Contributors

- **Writer:** Evangelos Nizamis
- **Helper:** Eli Kaufman
- **Principal Investigator (PI):** Valdmanis Paul


## License

Copyright (c) [2024] [Evangelos Nizamis, Eli Kaufman, Paul Valdmanis]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

If you use this software in your work or research, we kindly request that you acknowledge its use by citing the following reference: [[https://github.com/ValdmanisLab/AllofUs_PheWAS](https://github.com/ValdmanisLab/AllofUs_PheWAS)].

