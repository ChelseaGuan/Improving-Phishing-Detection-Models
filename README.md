# Improving Phishing Detection Models README

## About the Project
We wanted to expand on Off-the-Hook’s content-based phishing prevention application by using a stacked approach. The first level filters domains according to the lexical composition of the domain. The domains labeled phishing (TP and FP) as well as phishing domains falsely labeled benign (FN) from the first step would then be fed into Off-the-Hook’s content-based model in the second level.   

Note that this repos provides scripts to train both our domain-based model and Off-the-Hook's, as well as test our stacked model.

### Development
This project was developed in the PyCharm IDE from JetBrains. The language used is Python.
 
## Requirements
Test with Linux machine.

From the main Phishing-Detection-Automated-Testing directory containing the requirements file, run   
`pip install -r requirements.txt`
 
### How Run the Code

### Manually Collecting Training Data
Our domain-based model uses text files to train, whereas Off-the-Hook's content-based model uses JSON files. In order to use the same domains to train both, we follow the steps below.

#### Initial Text File Data Collection
To extract domains from Alexa, run   
`python3 get_alexa_domains.py [number of benign domains]`   
  [number of benign domains]: int representing the number of domains to extract.  

To extract domains from Phishtank, run   
`python3 get_phishing_domains.py [number of phishing domains]`   
   [number of phishing domains]: int representing the number of domains to extract.   
The script automatically verifies the Phishtank domains with VirusTotal and only writes to the text file the domains that have been flagged as phishing by at least than 5 engines. 

#### JSON File Data Collection and Final Text File Data Collection
To obtain a folder containing JSON files captured from the domain text files, run   
`python3 generate_JSON.py [path to training text file]`   
  [path to training text file]: str representing the path to the text file containing domains with which to train the model.   

In our case, we run it twice, once for each class of domain, benign and phishing.   
Run   
`python3 generate_JSON.py /var/tmp/phishing/benign_domains.txt`   
Manually rename the websites directory created in the Phishing-Detection-Automated-Testing project to "benign_train".   
Manually rename the after_filtering_domains.txt text file created in the Phishing-Detection-Automated-Testing project to "benign_train.txt". Move the file from the current directory to the /var/tmp/phishing/ directory which contains the other domain text files.    

Run   
`python3 generate_JSON.py /var/tmp/phishing/phishing_domains.txt`   
Manually rename the websites directory created in the Phishing-Detection-Automated-Testing project to "phishing_train".   
Manually rename the after_filtering_domains.txt text file created in the Phishing-Detection-Automated-Testing project to "phishing_train.txt". Move the file from the current directory to the /var/tmp/phishing/ directory which contains the other domain text files.    


### Training Our Domain-Based Model
To train our model, run   
`python3 train.py`   


### Training Off-the-Hook's Content-Based Model
We must first generate pkl files by running   
`python3 generate_JSON.py [websites_dir] [prefix] [target]`   
  [websites_dir] parameter: str representing the path to the directory containing the JSON files.   
  [prefix]: str representing the name to be assigned to the pickle file generated.   
  [target]: int representing the label of the domains from which features were extracted (benign: 0, phishing: 1).   

In our case, we run it twice, once for each class of domain, benign and phishing.   
Run  
`python3 generate_pkl.py benign_train benign_train 0`  
Run  
`python3 generate_pkl.py phishing_train phishing_train 1`  

To train Off-the-Hook's model, run  
`python3 build_model.py 0 benign_train_fvm.pkl phishing_train_fvm.pkl model`   
For more information on the build_model script, consult Off-the-Hook-README.txt.

### Testing the Stacked Model
To test the stacked model, run   
`python3 automate_testing.py [test case size] [upper rank]`   
  [test case size]: int representing both the number of benign domains and phishing domains to test the model with. The total number of domains tested is 2*[test case size].   
  [upper rank]: int representing the popoularity rank of the domain from which point on we will be extracting benign domains.   

## Desired Result
The final output should be a simple table with the number of FP, FN, TP and TN domains, and the accuracy of the stacked model.
