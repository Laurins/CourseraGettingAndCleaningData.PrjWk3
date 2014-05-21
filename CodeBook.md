# Code book 
## Getting and Cleaning Data:  _Project Assignment: Week 3_

Raw Data gathered from :
Machine Learning Repository
[Human Activity Recognition Using Smartphones Data Set](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones)


## Input files:
There are 2 sets of files: one for ___training___ (70% of data) and one for ___test___. This experiment organization is replicate in the structure of the directories/files. In the project (root) directory there are some explaning the experiment for tecnically, plus there is a file with the labels (de-codefication) of the activity codes.
* 'features_info.txt': information about features
* 'features.txt': labels(decodification) of features variable
* 'activity_labels.txt': labels(decodification) of activities

* 'train/X_train.txt': Measure - Training set
* 'train/y_train.txt': Training labels
* 'train/subject_train.txt': subject  

* 'test/X_test.txt': Measure - Test set
* 'test/y_test.txt': Test labels
* 'test/subject_train.txt': subject  
  
  
## Variables
We can split the fileds we have into 2 groups:
### ID / key / Dimension
These ID column are the key to understand what about the measure is talkin about.
These ID are:
- *flagTrainTest*
    * this tell whether the row is suppose to be used for the training stage or latter on for test. 
- *Subject*
    * the person the data is about, at present only a number of reference is available
- *Activity*
    * this tell what the person was doing for that mewasure: 
        * LAYING 
        * SITTING 
        * STANDING 
        * WALKING 
        * WALKING_DOWNSTAIRS 
        * WALKING_UPSTAIRS

### Measures
The measure is what we have measured with the device. The number of measure is very long just report some, the variable label is self-explaining, for instance:
*tBodyAcc-mean()-X* mean accelaration of the body
  
  
## Data transformations
* Loading files into data.frames
    * source file are loaded into data.frames
    * attach for training data.frames suject + activity + measures get a data.frame with IDs+Measures
    * same for test data.frames
    * append training and test data.frames to create one data set ***dataFull***
* Extracts only the measurements on the mean and standard deviation for each measurement. 
    * remove all measure we are not interested for this assignment
    * get final result ***Only_mean_std*** as require
* Second independent tidy data set with the average of each variable for each activity and each subject. 
    * summarize / aggregate with the usual key *flagTrainTest*, *Subject*, *Activity*
    * also here keep only measures we are interested to
    * get final result ***Full_mean*** as require
  
## Data (project result)

***df_Only_mean_std***  
* this is the whole data melt together,  
* with the activity filed with its labels, 
* only the mesure requires by assignment and these have the proper labels, not the initial ones (V1,V2,...).

***df_Mean_flagTrainTest_Subject_Activity***  
* this ia a  ___summarized___ version of the table above, where we get the mean for each measure keeping the usual key ***flagTrainTest***, ***Subject***, ***Activity***.  

* Actually the data segmentation Train/Test is done per Subject, i.e. a given Subject can belong only to train or (exclusive) test. For this reason this last dataset is also the  
mean for each mean on the "reduced" key ***Subject***, ***Activity*** not considering Train/Tets.  

