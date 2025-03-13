# Contrast Everything: A Hierarchical Contrastive Framework for Medical Time-Series (Neurips 2023)  

#### Authors: 
#### [Yihe Wang](https://webpages.charlotte.edu/ywang145/)(ywang145@uncc.edu), [Yu Han]()(hanyu21@mails.ucas.ac.cn)
####  [Haishuai Wang]()(haishuai.wang@zju.edu.cn), [Xiang Zhang](http://xiangzhang.info/)(xiang.zhang@uncc.edu)

#### COMET Paper: [NeurIPS 2023](https://openreview.net/forum?id=sOQBHlCmzp), [Preprint](https://arxiv.org/abs/2310.14017)
  
## Overview  
This repository contains the description of three datasets and the code of the COMET model for the paper *Contrast Everything: A Hierarchical Contrastive Framework for Medical Time-Series (COMET)*. We present COMET, an innovative hierarchical framework that leverages data consistencies at all inherent levels in medical time series. We conduct experiments in the challenging patient-independent setting. We compare COMET against six baselines using three diverse datasets, which include ECG signals for myocardial infarction and EEG signals for Alzheimer's and Parkinson's diseases. The following figure illustrates the structure of medical time series. Medical time series commonly have four levels (coarse to fine): patient, trial, sample, and observation. An observation is a single value in univariate time series and a vector in multivariate time series.


![Medical Time Series](Fig/patient-data-structure-v2.png)
 
## Key idea of COMET
Capturing data consistency is crucial in the development of a contrastive learning framework. Data consistency refers to the shared commonalities preserved within the data, which provide a supervisory signal to guide model optimization. Utilizing this consistency as prior knowledge can reduce the reliance on labeled data for downstream tasks. Contrastive learning captures data consistency by contrasting positive and negative data pairs, where positive pairs share commonalities and negative pairs do not. We propose consistency across four data levels: observation, sample, trial, and patient, from fine-grained to coarse-grained in the medical time series. Although we present four levels here, our model can easily be adapted to accommodate specific datasets by adding or removing data levels.
 
![COMET](https://github.com/DL4mHealth/COMET/blob/main/Fig/comet-framework-v7.png)  
**Overview of COMET approach.**  Our COMET model consists of four contrastive blocks, each illustrating the formulation of positive pairs and negative pairs at different data levels. In the observation-level contrastive, an observation and its augmented view serve as a positive pair. Similarly, in the sample-level contrastive, a sample and its augmented view form a positive pair. Moving to the trial-level contrastive, two samples from the same trial are considered to be a positive pair. The patient-level contrastive follows a similar pattern, where two samples from the same patient are regarded as a positive pair. Positive pairs and corresponding negative pairs will be utilized to build contrastive loss in embedding space after being processed by encoder $G$.




  
  
## Datasets  
### Data preprocessing
(1) **[AD dataset](https://osf.io/jbysn/)** comprises EEG recordings from 12 patients with Alzheimer’s disease and 11 healthy controls. Each patient has an average of 30.0 ± 12.5 trials. Each trial corresponds to a 5-second interval with 1280 timestamps (sampled at 256Hz) and includes 16 channels. Prior to further processing, each trial is scaled using a standard scaler. To facilitate analysis, we segment each trial into nine half-overlapping samples, where each sample has a duration of 256 timestamps (equivalent to 1 second). Additionally, we assign a unique trial ID and patient ID to each sample based on its origin in terms of the patient and trial. We split training, validation, and test sets in a patient-independent way. We use samples from patient IDs 17 and 18 as the validation set and samples from IDs 19 and 20 as the test set. The rest of the samples are all put into the training set.

(2) **[PTB dataset](https://physionet.org/content/ptbdb/1.0.0/)** consists of ECG recordings from 290 patients, with 15 channels sampled at 1000 Hz. There are a total of 8 types of heart diseases present in the dataset. For this paper, we focus on binary classification using a subset of the dataset that includes 198 patients with major disease
labels, specifically Myocardial infarction and healthy controls. To preprocess the ECG signals, they are first normalized using a standard scaler after being resampled to a frequency of 250 Hz. Due to special peak information in ECG signals, a regular sliding window segmentation approach may result in the loss of crucial information. To address this issue, a different segmentation strategy is employed. Instead of sliding windows, the raw trials are segmented into individual heartbeats, with each heartbeat treated as a sample. To perform this segmentation, (1) the first step involves determining the maximum duration. The median value of R-Peak intervals across all channels is calculated for each raw trial, and outliers are removed to obtain a reasonable maximum interval as the standard heartbeat duration. (2) The next step is to determine the position of the first R-Peak. The median value of the first R-Peak position is calculated and used for all channels. (3) Next, the raw trials are split into individual heartbeat segments based on the median value of their respective R-Peak intervals. Each heartbeat is sampled starting from the R-Peak position, with the segments extending to both ends with half the length of the median interval. (4) To ensure the same length of the heartbeat samples, zero padding is applied to align their lengths with the maximum duration. (5) Finally, the samples are merged into trials, where 10 nearby samples are grouped together to form a pseudo-trial, similar to the neighborhood idea presented in [TNC](https://github.com/sanatonek/TNC_representation_learning). We split training, validation, and test sets in a patient-independent way. We use samples from 28 patients(7 healthy and 21 positive) as the validation set and samples from another 28 patients(7 healthy and 21 positive) as the test set. The rest of the samples are all put into the training set.


(3) **[TDBrain dataset](https://brainclinics.com/resources/)** is a large dataset that monitors the brain signals of 1274 patients with 33 channels (500 Hz) during EC (Eye closed) and EO (Eye open) tasks. The dataset consists of 60 types of diseases, and it is possible for a patient to have multiple diseases simultaneously. This paper focuses on a subset of the dataset, specifically 25 patients with Parkinson’s disease and 25 healthy controls. Only the EC task trials are used for representation learning. To process the raw EC trials, we employ a sliding window approach that continuously moves from the middle of the trial to both ends without any overlap. Each raw EC trial is divided into processed pseudo-trials with a length of 2560 timestamps (10 seconds) after resampling to 256 Hz. These processed pseudo-trials are then scaled using a standard scaler. Furthermore, each pseudo-trial is split into 19 half-overlapping samples, with each sample having a length of 256 timestamps (1 second). In addition to the binary label indicating Parkinson’s disease or healthy, each sample is assigned a patient and trial ID based on the patient and processed trial it originates from. It is important to note that the trial ID refers to the ID of the processed pseudo-trial and not the raw EC trial. We split training, validation, and test sets in a patient-independent way. We use samples from 8 patients(4 healthy and 4 positive) as the validation set and samples from another 8 patients(4 healthy and 4 positive) as the test set. The rest of the samples are all put into the training set.


### Processed data 
Download the raw data from the links above and run notebooks in the folder `data_processing/` for each raw dataset to get the processed dataset. The folder for processed datasets has two directories: `Feature/` and `Label/`. The folder `Feature/` contains files named in the format `feature_ID.npy` files for all the patients, where ID is the patient ID. Each`feature_ID.npy` file contains trials belonging to the same patient and stacked into a 3-D array with shape [N_r, T, C], where N_r denotes the number of trials in the patient with XX ID, T denotes the timestamps for a trial, and C denotes the feature dimensions. Notice that different patients have different numbers of trials.

The folder `Label/` has a file named `label.npy`. This label file is a 2-D array with shape [N_p, 2], where N_p denotes the number of patients. The first column is the patient's label(e.g., healthy or AD), and the second column is the patient ID, ranging from 1 to N.  

The processed data should be put into `datasets/DATA_NAME/` so that each patient file can be located by `datasets/DATA_NAME/Feature/feature_ID.npy`, and the label file can be located by `datasets/DATA_NAME/Label/label.npy`.  

The processed datasets can be manually downloaded at the following links.
* AD dataset: https://figshare.com/ndownloader/files/43196127 
* PTB dataset: https://figshare.com/ndownloader/files/43196133 

Since TDBrain is not a public dataset, we do not provide a download link here. The users need to request permission to download on the TDBrain official website and process the raw data.

## Experimental setups

We evaluate our model with patient-independent train-val-test split in two settings: partial fine-tuning and full fine-tuning, and in comparison with six state-of-the-art methods: [TS2vec](https://github.com/yuezhihan/ts2vec), [Mixing-up](https://github.com/Wickstrom/MixupContrastiveLearning), [TS-TCC](https://github.com/emadeldeen24/TS-TCC), [SimCLR](https://github.com/iantangc/ContrastiveLearningHAR), [CLOCS](https://github.com/danikiyasseh/CLOCS) and [TF-C](https://github.com/mims-harvard/TFC-pretraining).

**Setting 1: Partial fine-tuning.** We add a logistic regression classifier $L$ on top of the pre-trained encoder $G$. The training process utilizes 100% labeled training data for sample classification. Notably, the parameters of the encoder $G$ are frozen during training, ensuring that only the classifier $L$ is fine-tuned.

**Setting 2: Full fine-tuning.** We add a two-layer fully connected network classifier $P$ to the pre-trained encoder $G$. The training process utilizes 100%, 10%, and 1% of the labeled training data for sample classification. Unlike the Partial fine-tuning setting, both the encoder $G$ and classifier $P$ are trainable, allowing for fine-tuning of the entire network structure. 


## Requirements  
  
The recommended requirements are specified as follows:  
* Python 3.10  
* Jupyter Notebook  
* scikit-learn==1.2.1    
* torch==1.13.1+cu116    
* matplotlib==3.7.1    
* numpy==1.23.5    
* scipy==1.10.1    
* pandas==1.5.3    
* wfdb==4.1.0    
* neurokit2==0.2.4  
  
The dependencies can be installed by:  
```bash  
pip install -r requirements.txt
```
  
## Running the code  
  
We provide jupyter notebook examples for each dataset. To train and evaluate COMET on a dataset, simply run `DatasetName_example.ipynb`, such as `AD_example.ipynb`.  All the setups, including pre-training, partial fine-tuning, and full fine-tuning, are running step by step with commands in the notebook.  

After training and evaluation, the pre-training model and fine-tuning model can be found in`test_run/models/DatasetName/`; and the logging file for validation and test results can be found in  `test_run/logs/DatasetName/`. You could modify all the parameters, the working and logging directory in `config_files/DatasetName_Configs.py`.  
