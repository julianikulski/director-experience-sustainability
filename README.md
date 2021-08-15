# Classifying biographies of directors on boards of companies

This project is part of my Master thesis investigating whether prior CSR-related professional experience of directors sitting on boards of companies affects the corporate social performance (CSP) of those companies. CSR stands for corporate social responsibility and describes company conduct that is guided not only by economic concerns, but also by concerns about the environment, society, stakeholders and other discretionary factors. I investigated two hypothesis and both were supported.


## Table of Contents
* [Installation](#Installation)
* [Project Motivation and Description](#motivation)
* [File Description](#description)
* [Results](#Results)
* [Licensing, Authors, Acknowledgements](#licensing)


## Installation
The code requires Python versions of 3.*, general libraries available through the Anaconda package, as well as the following:
* transformers==4.2.2
* pytorch-lightning==1.2.7
* torch==1.8.0
* linearmodels==4.24


## Project Motivation and Description <a name="motivation"></a>
Investors are more and more concerned with the sustainability and CSR of companies. Research suggests that better CSP leads to better financial performance of companies. Much research has been conducted on the antecedents of CSP. The board of directors is starting to attract more attention, especially the characteristics of individual directors and their link to CSP. The non-profit organization Ceres suggested in 2015, that directors with prior professional experience in CSR-related areas would lead to higher CSP. Thus, this master thesis dealt with the following research question: _Does the prior professional experience of directors in CSR-related areas positively affect the corporate social performance of companies?_


## File Description <a name="description"></a>
This paragraph lists the names of all jupyter notebook files, their inputs and their outputs, as well as what the file does. Some of the files are not included in this public GitHub repo but they were still necessary to generate the analysis and results.


**`data-collection/eikon_python_api.ipynb`:**
* _Inputs_: No files but it accesses the Refinitiv Eikon API
* _Outputs_:
    * `founding_year.csv`: Lists the founding year for all companies in the S&P 500 index between 2010 and 2016 as well as the years that a company was a part of the S&P 500 index. Also includes the ISIN for each company.
    * `csr_committee.csv`: CSR sustainability committee variable for all companies in the S&P 500 index between 2010 and 2016.
    * `csr_scores.csv`: All ESG scores for all companies for 2011 until 2016.
* _What it does_: It accesses the Refinitiv Eikon API to get data on control and dependent variables, wrangles them and writes them to csv files.


**`data-collection/director_company_data.ipynb`:**
* _Inputs_:
    * `founding_year.csv`
    * All Excel files in the folder `/Board and Committee Memberships` which include the board and committee memberships for the sample companies
    * `control_vars.xlsx`: Excel file with data on control variables for years 2010 until 2016 for all companies that were a part of the S&P 500 index during those years.
* _Outputs_:
    * `all_control_vars.csv`: All control variables for each firm with the ISIN.
    * `all_directors.csv`: Includes 9465 unique directors and their board memberships and tenures for the companies of the S&P 500; this does include directors who only served outside the sample window of 2011 until 2015.
    * `all_committees.csv`: Includes all directors that sat on any committees on the S&P500 companies during any years; also includes the committee names.
* _What it does_: It uses the data on the board and committee memberships procured from Refinitiv's Eikon to create a file that only includes directors (and no executives) and their tenure on each of their companies. It also maps the control variables to the ISIN of each company. Finally, a csv file with all committee names and members of the S&P 500 companies is created.


**`preprocessing/biography_matching.ipynb`:**
* _Inputs_:
    * `all_directors.csv`
    * All files in the `/S&P Capital IQ Biographien` folder that contain the director biographies sourced from S&P Capital IQ
* _Outputs_:
    * `all_directors_rel.csv`: CSV file with all directors that sat on the S&P 500 companies during 2011 and 2015
    * `dir_bio_manual_review.xlsx`: File that contains a subset of directors that were matched but had the same name and therefore, this file had duplicate director entries that were spot checked. This files was not used for anything else.
    * `director_bios_all.xlsx`: File that contained all directors from the sample and the matched biographies that were available.
* _What it does_: The available biographies from the S&P Capital IQ dataset are matched to the directors in the sample and the file is saved to Excel for further manual research of the missing biographies.


**`csr-committee/csr_committees.ipynb`:**
* _Inputs_:
    * `all_committees.csv`
    * `CSR_scores_committee.xlsx`: ESG score data downloaded from Refinitiv Eikon.
    * `director_bios_all.xlsx`
* _Outputs_:
    * `people_rel_comms.xlsx`: List with people that sat on a environmental, social, or csr board committee.
    * `comps_rel_comms.xlsx`: List with companies that have a CSR-related board committee.
    * `dir_bio_comm_all.xlsx`: List with all directors in the sample, their tenure, biographies (if available) and their committee memberships.
* _What it does_: This file adds committee memberships and information to each applicable director in the overall director sample file (`director_bios_all.csv`) which will then be used for the manual research of biographies and committee information in the DEF 14A statements.


**`preprocessing/prepare_director_comp_data.ipynb`:**
* _Inputs_:
    * `dir_bio_comm_all_added.xlsx`: Excel file containing all manually researched biographies and committee information from the DEF 14A statements as well as notes on things, such as which directors are duplicates or not really directors and should be removed from the sample.
    * `dupes_directors_checked.xlsx`: List to do spot check of directors with same names but different biographies.
    * `all_directors_rel.csv`
* _Outputs_:
    * `dupes_directors.xlsx`: List with duplicate director names but different biographies to determine whether these directors are the same person or different people.
    * `complete_sample.xlsx`: File that includes the sample of companies and directors with their biographies and committee memberships which is used for the biography classification.
* _What it does_: This file takes the file that has all the manually researched biographies and committee information and merges it with all directors determined to be in the final sample used for the biography classification.


**`preprocessing/csr_experience_ETL.ipynb`:**
* _Inputs_:
    * All files in the `/S&P Capital IQ Biographien` folder that contain the director biographies sourced from S&P Capital IQ
    * `SP500.xlsx`: Constituents list of the S&P 500 index for the years 2011 until 2015 procured from Refinitiv Eikon.
    * `complete_sample.xlsx`
* _Outputs_:
    * `sp500_biographies_2015.xlsx`: File that includes the matched directors from the S&P 500 constituents list of 2015 and the S&P Capital IQ biographies.
    * `train_150.xlsx`: 150 randomly chosen biographies from the S&P Capital IQ biographies.
    * `train_second_50.xlsx`: 50 randomly chosen biographies from the manually researched DEF 14A statements.
* _What it does_: This file generates 200 training samples for the Longformer model. 150 biographies are chosen from the S&P 500 Capital IQ dataset and 50 biographies are chosen from the DEF 14A statements.


**`model/baseline_models.ipynb`:**
* _Inputs_:
    * `train_rev.xlsx`: File containing the 150 manually reviewed and classified biographies from the file `train_150.xlsx`.
    * `train_second_rev.xlsx`: File containing the 50 manually reviewed and classified biographies from the file `train_second_50.xlsx`.
    * `complete_sample_no_missing.csv`: File generated by `exploratory_data_analysis.ipynb` containing the final director sample after all companies and directors have been removed due to missing data in the control, independent, or dependent variables. Contains 5,276 unique directors.
* _Outputs_:
    * `all_200_rev.csv`: File containing the overall training sample with the 200 classified biographies.
* _What it does_: This file takes the manually reviewed biography samples from the two separate files, merges them and writes them to a CSV file. It also runs four different baseline models on the overall biography dataset.
* _Note_: This file generates an output that is required by `csr_experience_fine_tuning.ipynb` which generates output that is in turn required by `exploratory_data_analysis.ipynb`, but then this file generates an output that is fed into `baseline_models.ipynb`. Therefore, these files cannot be run through completely at once, but only iteratively.


**`model/csr_experience_fine_tuning.ipynb`:**
* _Inputs_:
    * `all_200_rev.csv`
    * `complete_sample_avail_score.csv`: This file contains the semi-final director sample. It slightly differs from the `complete_sample_no_missing.csv` because it still included some directors from companies that had to be removed for the final sample because they had some missing data and needed to be excluded from the final sample. Contains 5,287 unique directors.
* _Outputs_:
    * `predicted_bios.csv`: File contains the biographies with the predicted social and environmental labels
    * `csr-director-checkpoint_318.ckpt`: This is the model checkpoint that was saved for the best model.
* _What it does_: This file creates the training pipeline and fine-tunes the Longformer model with PyTorch Lightning. Once the best performing model is chosen, it is used to predict the social and environmental labels of each biography in the `complete_sample_avail_score.csv` dataset.
* _Note_: The `complete_sample_avail_score.csv` that is read in by this file slightly differs from the `complete_sample_avail_score.csv` that is produced by `exploratory_data_analysis.ipynb` because small changes were made to the EDA file after the biographies were already predicted. Therefore, the `csr_experience_fine_tuning.ipynb` file shows 5,279 unique directors in the `complete_sample_avail_score.csv` file, while the EDA file shows 5,287 unique directors.


**`analysis/exploratory_data_analysis.ipynb`:**
* _Inputs_:
    * `founding_year.csv`
    * `all_directors_rel.csv`
    * `complete_sample.xlsx`
    * `csr_scores.csv`
    * `predicted_bios.csv`
    * `all_200_rev.csv`
    * `all_control_vars.csv`
    * `industry_sector.xlsx`: File containing the supersector and ISIN for each company in the sample. Data retrieved from Refinitiv Eikon.
* _Outputs_:
    * `complete_sample_avail_score.csv`
    * `complete_sample_no_missing.csv`
    * `no_outliers_env.csv`: Contains the final firm-year observations with all control and independent variables for the environmental score. The dataset was truncated at the 1st- and 99th-percentile. Total observations: 1,397.
    * `no_outliers_soc.csv`: Contains the final firm-year observations with all control and independent variables for the social score. The dataset was truncated at the 1st- and 99th-percentile. Total observations: 1,700.
    * `winsorized_env.csv`: Contains the final firm-year observations with all control and independent variables for the environmental score. The dataset was winsorized at the 1st- and 99th-percentile. Total observations: 2,201.
    * `winsorized_soc.csv`: Contains the final firm-year observations with all control and independent variables for the social score. The dataset was winsorized at the 1st- and 99th-percentile. Total observations: 2,201.
    * `env_score_dataset`: Contains the final firm-year observations with all control and independent variables for the environmental score.
    * `soc_score_dataset`: Contains the final firm-year observations with all control and independent variables for the social score.
* _What it does_: This file determines the final sample size. It merges the dependent, independent, and control variables into one dataset. It generates a final firm-year observation dataset for both the environmental and social scores, a truncated dataset, and a winsorized dataset. The data is explored visually and statistically and almost all of the data, figures, and tables from chapter 4 and 5.1 are generated by this file.


**`analysis/regression_analysis.ipynb`:**
* _Inputs_:
    * `no_outliers_env.csv`
    * `no_outliers_soc.csv`
    * `winsorized_env.csv`
    * `winsorized_soc.csv`
    * `env_score_dataset`
    * `soc_score_dataset`
* _Outputs_: None
* _What it does_: This file conducts the two-way fixed effects regression analysis on the original dataset, the truncated, and the winsorized dataset.


## Results
H1: CSR-related professional experience of the directors on the board will positively affect the CSP of the company.

H2: The positive relationship between the CSR-related professional experience of the directors on the board and the CSP of the company will increase if these directors are members of the CSR committee.

Both hypotheses were supported. The link between environmental experience and social experience of directors sitting on committees and the CSP of a company was higher than the link between the social and environmental experience on the overall board and CSP. Please refer to the `exploratory_data_analysis.ipynb` notebook and the `regression_analysis.ipynb` notebook for more details.


## Licensing, Authors, Acknowledgements <a name="licensing"></a>
The results of this thesis as well as the code can be shared and adjusted, as long as the original author (Julia Nikulski) is cited.
