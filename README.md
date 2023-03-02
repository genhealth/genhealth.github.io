<!-- ABOUT THE PROJECT -->
## Overview

GenHealth's submission to the [NIH NCAT's Bias Detection Tools in HealthCare Challenge]( https://expeditionhacks.com/bias-detection-healthcare/)


<!-- GETTING STARTED -->
### Folder Structure
The files in the repo are organized as follows:
```/
├── js/                                      # javascript for html landing page 
├── css/                                     # css files for html landing page
├── assets/img                               # images for the team submission landing page
├── demo                                     # demo folder containing demo CSV file
├── measure_disparity/measure_disparity.py   # python script for measuring disparity
├── mitigate_disparity.py                    # python script for measuring disparity
├── README.md                                # this file
```

## Setup

Here's how to setup the project locally:

We recommend using virtualenv to create a local environment for this project. The following steps will create a virtual 
environment and install the required packages:
```
virtualenv -p python3.10 genhealth_bias
source genhealth_bias/bin/activate
pip install -r requirements.txt
```

## Running the Scripts

### Measure Disparity
The `measure_disparity.py` script by default expects a CSV file as input. For convenience, we have included a demo CSV file
which includes diabetes admission and treatment information in the `demo` directory:

The script can be run with the following command line arguments:

| Name                  | Description                                                                          | Expected Values                        | Example                                        |
|-----------------------|--------------------------------------------------------------------------------------|----------------------------------------|------------------------------------------------|
| probability-col       | A column containing the model's predicted probability of the outcome being true (1)  | float between 0 and 1                  | 0.7                                            |
| binary-outcome-col    | The binary outcome predicted for the row                                             | 0/1 or y/n or true/false               | true                                           |
| pos-outcome-indicator | The label indicating a "true" value from the binary outcome column. (Case sensitive) | string                                 | 1                                              |
| sample-weights-col    | The sample weights                                                                   | float                                  | 5.2                                            |
| protected-classes     | A comma separated list of columns containing protected classes                       | pro1,pro2,pro3                         | sex_f,race_african_american                    |
| reference-classes     | A comma separated list of columns containing reference classes                       | ref1,ref2,ref3                         | sex_m,race_white                               |
| input-file            | The location of the input file                                                       | An absolute or relative path to a file | demo/compass_data_with_predict_probability.csv |
| debug                 | Enable additional debug logging                                                      | True/False                             | True                                           |


Example:
```
python measure_disparity.py  \
    --probability-col=predict_probability \
    --binary-outcome-col=readmitted \
    --protected-classes=race_AfricanAmerican,race_Asian,race_Hispanic,gender_Female \
    --reference-classes=sex_Male,race_Caucasian \
    --input-file=demo/diabetes_with_predict_probability.csv \
    --pos-outcome-indicator=1 \
    --reference-classes=race_Caucasian,gender_Male
```

Expected Output:
```
| protected_or_reference   | class                |   num_samples |   equal_odds_difference |   equal_odds_ratio |   demographic_parity_difference |   demographic_parity_ratio |
|--------------------------|----------------------|---------------|-------------------------|--------------------|---------------------------------|----------------------------|
| protected                | race_AfricanAmerican |         19210 |                   0.044 |              0.923 |                           0.034 |                      0.934 |
| protected                | race_Asian           |           641 |                   0.107 |              0.705 |                           0.128 |                      0.738 |
| protected                | race_Hispanic        |          2037 |                   0.052 |              0.857 |                           0.039 |                      0.919 |
| protected                | gender_Female        |         54708 |                   0.005 |              0.993 |                           0.006 |                      0.987 |
| reference                | race_Caucasian       |         76099 |                   0.025 |              0.961 |                           0.001 |                      0.998 |
| reference                | gender_Male          |         47055 |                   0.001 |              0.997 |                           0.007 |                      0.986 |

```

### Mitigate Disparity
The `mitigate_disparity.py` script by default also expects a CSV file as input. For convenience, we have included a demo CSV file
`demo/diabetes_with_predict_probability.csv` which can be used to test the output. 

| Name                           | Description                                                                                                                              | Expected Values          | Example                            |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|------------------------------------|
| do-demo                        | Perform demo. Performs optimization on embedded diabetes file                                                                            | -                        | -                                  |
| input-filename                 | The absolute or relative path to the train file                                                                                          | directory + filename     | ./demo/diabetes.csv                |
| test-filename                  | The absolute or relative file to the test file. If no test file is defined,  the code will divide the input file into a train/test split | directory + filename     | ./demo/diabetes.csv                |
| sensitive-classes              | Comma separated list of sensitive classes                                                                                                | string                   | race_AfricanAmerican,race_Hispanic |
| reference-classes              | Comma separated list of reference classes                                                                                                | string                   | race_White,sex_M                   |
| binary-outcome-column          | Column containing binary outcome data on patient                                                                                         | 0/1 or y/n or true/false | 1                                  |
| use-pos-weights                | Use autogenerated positive weights. Note: This will ignore the sample-weights if those are specified.                                    | True or False            | True                               |
| sample-weights-column          | Sample weights column. Note: specifying use-pos-weights will cause this to be ignored.                                                   | string                   | sample_weights                     |
| do-hyperparameter-optimization | Optional flag to perform hyperparameter optimization                                                                                     | True or False            | True                               |
| train-test-split               | Train/test ratio split of data. Default's to 0.8                                                                                         | Float                    | 0.7                                |
| random-state                   | Seed for initializing the random state. Default to 0                                                                                     | Integer                  | 2                                  |
| debug                          | Enable additional debug logging                                                                                                          | True/False               | True                               |


To run the demo script with a custom file, use the following command:
```
python mitigate_disparity.py \
    --binary-outcome-col=readmitted \
    --protected-classes=race_AfricanAmerican,race_Asian,race_Hispanic,gender_Female \
    --reference-classes=sex_Male,race_Caucasian \
    --input-file=demo/diabetes.csv \
    --pos-outcome-indicator=1 \
    --reference-classes=race_Caucasian,gender_Male
```

Expected output:
```
| model   | protected_or_reference   | class                |   num_samples |   equal_odds_difference |   equal_odds_ratio |   demographic_parity_difference |   demographic_parity_ratio |
|---------|--------------------------|----------------------|---------------|-------------------------|--------------------|---------------------------------|----------------------------|
| fair    | protected                | race_AfricanAmerican |          3817 |                   0.017 |              0.962 |                           0.006 |                      0.99  |
| fair    | protected                | race_Asian           |           142 |                   0.095 |              0.854 |                           0.09  |                      0.829 |
| fair    | protected                | race_Hispanic        |           401 |                   0.002 |              0.996 |                           0.015 |                      0.971 |
| fair    | protected                | gender_Female        |         10962 |                   0.031 |              0.953 |                           0.01  |                      0.981 |
| fair    | reference                | race_Caucasian       |         15278 |                   0.018 |              0.97  |                           0.01  |                      0.98  |
| fair    | reference                | gender_Male          |          9392 |                   0.028 |              0.958 |                           0.006 |                      0.988 |
```

### Background and Script Design
The `measure_disparity.py` file uses the `fairlearn` library's in-built functions to derive the `equalized_odds_difference`,
`equalized_odds_ratio`, `demographic_parity_difference`,  and `demographic_parity_ratio` values for the input data sets.

The `mitigate_disparity.py` file creates a binary classifier that dynamically adjusts weights to minimize the equal odds 
difference between the protected and reference classes. It also feeds the results into the `measure_disparity.py` script to
measure the reduction in bias after training.

<!-- LICENSE -->
## License

Distributed under the BSD 3 License. See `license.txt` for more information.


<!-- CONTACT -->
## Contact

* Ricky Sahu 
* Eric Marriott
* Ethan Siegel

Project Link: [https://github.com/](https://github.com/)
