Coursera: Getting and Cleaning Data - Project Week 3 
=====================================================

## Loading files into data.frames
This mainly is run_analysis.R, just lenty of comments and annotations.
Let's start from readng all files into data.frames.

### Activity and features
these are 2 labels (de-codification) tables, *Activity* containing the labels of activities to be used as value of the files Activity, and *features* that contains the names of the "columns" of the measures.
```{r }
# Activity
setwd("C:/Laurins/Safari_Downloads/00. AI-BI-DataManing-MachineLearning/__3.Getting.and.Cleaning.Data/3.Project/wk3/UCI HAR Dataset")
    Activity_Labels <- read.table(file="activity_labels.txt", , nrows=5)
    tmp_act <- paste(Activity_Labels[, 1], Activity_Labels[, 2], sep=".")
    Activity_Labels <- cbind(Activity_Labels, tmp_act)
    names(Activity_Labels) <- c("Activity_code", "Activity_Name", "Activity")
                remove(tmp_act) 
                # Both because my PC is small 
                #   and I like to avoid reading temporary tables by mistake, 
                #   I try to get rid of object not useful any longer
# features
    features <- read.table(file="features.txt", as.is=TRUE , nrows=5)
```
note on ***remove()***   
You will notice I use heavy the function  
*remove(my.data.frame)*
in order to try to get rid of object not useful any longer. This both because 
* I have a small PC, I am trying to keep memory available
* second (and more important) I like to avoid (reading) temporary tables by mistake  
  
### Train and Test
The actual data has been devided in 2 parts, ahead we get it. These 2 parts are Train and Test. This is a common procedure to prepare an analysis using the Traning part and then checking the performance of the analysis using only the Test part.  
At the moment we are not interested into this split, though we carry on the information about what are the raws related to train and test.

For each block we read 3 tables, containing info about 
* Subject_... (number identifying a person, though names are not given)
* y_... Activity (kind of activity done by the person, there are 6 types)
* x_... a lot of measures taken  

```{r }
# Train
    subject_train <- read.table(file="train/subject_train.txt", nrows=5)

    raw_y_train       <- read.table(file="train/y_train.txt", nrows=5)
    temp_train <- merge(x=Activity_Labels, y=raw_y_train, 
                        by.x="Activity_code", by.y="V1")
    y_train <- subset(x=temp_train, select=-c(Activity_code, Activity_Name))
    remove(raw_y_train, temp_train)
    names(y_train) <- c("Activity")

    x_train       <- read.table(file="train/x_train.txt", nrows=5)     
# Test
    subject_test <- read.table(file="test/subject_test.txt", nrows=5)
    
    raw_y_test   <- read.table(file="test/y_test.txt", nrows=5)
    temp_test <- merge(x=Activity_Labels, y=raw_y_test, 
                       by.x="Activity_code", by.y="V1")
    y_test <- subset(x=temp_test, select=-c(Activity_code, Activity_Name))
    names(y_test) <- c("Activity")
    remove(raw_y_test, temp_test)
    
    x_test       <- read.table(file="test/x_test.txt", nrows=5)      
```

## Getting columns names containing mean() and std(), for latter use
I prepare 2 "lists" (actually they are data.frames) containing all the column names for the "Base Table" (the one we will get the 2 final resulting datesets/files)
These lists are going to be used for putting the right names to the datasets headers and also to filter out the measures (columns) we are not interested.
* **features_all**: all features, in the proper order accordlying to the basetable
* **features_OK** : names of measure require (to keep in the final result)   

```{r}
    features_all <-  features    
    features_all [, 1] <- features_all [, 1] +3
    # features_all: all features, in the proper order accordlying to the basetable
    features_all <- rbind(c(1, "flagTrainTest"), c(2, "Subject")
                          , c(3, "Activity"), features_all)       
    list_indx_OK <- sort(unique(union((grep("mean", features$V2))
                                      ,( grep(c("std"), features$V2)))))
    indexes_all_features <- features[, 1]
    features_OK <- features[list_indx_OK, ]
    features_OK[, 1] <- features_OK[, 1] + 3
    # features_OK : names of measure require (to keep in the final result)
    features_OK <- rbind(c(1, "flagTrainTest"), c(2, "Subject"), c(3, "Activity"), features_OK)
```

## Merges the training and the test sets to create one data set.
Put together train and test to abtain the "base Table", with all data we need to get the 2 final steps (and extracted files).
On the way I fill up a column flagTrainTest that keep tract of whether the raw belogs to the train or to the test block.
```{r }
# Train
    # before merge prepare a column "Subject" telling whether it is train/test data
    flagTrainTest <- rep("train", nrow(y_train))
    names(subject_train) <- c("Subject")
    Train_full <- cbind(flagTrainTest,subject_train,y_train,x_train) ; remove(flagTrainTest)
# Test
    flagTrainTest <- rep("test", nrow(y_test))
    names(subject_test) <- c("Subject")
    Test_full <- cbind(flagTrainTest,  subject_test, y_test, x_test)
# Train + Test
    dataFull <- rbind (Train_full, Test_full)
    remove(Train_full,Test_full,flagTrainTest,x_test,x_train,y_test
           ,y_train,subject_train,subject_test)
```


## Result 1st. 
### Dataset only the measurements on the mean and standard deviation for each measurement.  
We work on a copy of __dataFull__, our "base table", this because we need it unchanged.
We fix names of columns, keep only columns required by assignment
Save the result data.frame into a file txt.
```{r }
    # let's keep unchanged dataFull for next steps
    tmp_Only_mean_std <- dataFull 
    
    # put right name into the data.frame, i.e. "V1" --> "tBodyAcc-mean()-X"
    names(tmp_Only_mean_std) <- features_all[, 2] # force the actual features names
    
    # keep only "good" vars
    df_Only_mean_std <- subset(x=tmp_Only_mean_std, select=features_OK[, 2] )  
                        remove(tmp_Only_mean_std) 
    # save 1st dataset as txt files
    write.table(x=df_Only_mean_std, file="df_Only_mean_std.txt")
```

## Result 2nd. 
### A second, independent tidy data set with the average of each variable for each activity and each subject. 
Once more from _dataFull_, our "base table", we ___summarize___ with key ***flagTrainTest***, ***Subject***, ***Activity***.  
Manage columns names, keep only columns required.
Save the result data.frame into a file txt.

```{r}
    tmp_mean <- aggregate (cbind(V1,V2,V3,V4,V5,V6,V7,V8,V9,V10,V11,V12,V13,V14,V15,V16,V17,V18,V19,V20,V21,V22,V23,V24,V25,V26,V27,V28,V29,V30,V31,V32,V33,V34,V35,V36,V37,V38,V39,V40,V41,V42,V43,V44,V45,V46,V47,V48,V49,V50,V51,V52,V53,V54,V55,V56,V57,V58,V59,V60,V61,V62,V63,V64,V65,V66,V67,V68,V69,V70,V71,V72,V73,V74,V75,V76,V77,V78,V79,V80,V81,V82,V83,V84,V85,V86,V87,V88,V89,V90,V91,V92,V93,V94,V95,V96,V97,V98,V99,V100,V101,V102,V103,V104,V105,V106,V107,V108,V109,V110,V111,V112,V113,V114,V115,V116,V117,V118,V119,V120,V121,V122,V123,V124,V125,V126,V127,V128,V129,V130,V131,V132,V133,V134,V135,V136,V137,V138,V139,V140,V141,V142,V143,V144,V145,V146,V147,V148,V149,V150,V151,V152,V153,V154,V155,V156,V157,V158,V159,V160,V161,V162,V163,V164,V165,V166,V167,V168,V169,V170,V171,V172,V173,V174,V175,V176,V177,V178,V179,V180,V181,V182,V183,V184,V185,V186,V187,V188,V189,V190,V191,V192,V193,V194,V195,V196,V197,V198,V199,V200,V201,V202,V203,V204,V205,V206,V207,V208,V209,V210,V211,V212,V213,V214,V215,V216,V217,V218,V219,V220,V221,V222,V223,V224,V225,V226,V227,V228,V229,V230,V231,V232,V233,V234,V235,V236,V237,V238,V239,V240,V241,V242,V243,V244,V245,V246,V247,V248,V249,V250,V251,V252,V253,V254,V255,V256,V257,V258,V259,V260,V261,V262,V263,V264,V265,V266,V267,V268,V269,V270,V271,V272,V273,V274,V275,V276,V277,V278,V279,V280,V281,V282,V283,V284,V285,V286,V287,V288,V289,V290,V291,V292,V293,V294,V295,V296,V297,V298,V299,V300,V301,V302,V303,V304,V305,V306,V307,V308,V309,V310,V311,V312,V313,V314,V315,V316,V317,V318,V319,V320,V321,V322,V323,V324,V325,V326,V327,V328,V329,V330,V331,V332,V333,V334,V335,V336,V337,V338,V339,V340,V341,V342,V343,V344,V345,V346,V347,V348,V349,V350,V351,V352,V353,V354,V355,V356,V357,V358,V359,V360,V361,V362,V363,V364,V365,V366,V367,V368,V369,V370,V371,V372,V373,V374,V375,V376,V377,V378,V379,V380,V381,V382,V383,V384,V385,V386,V387,V388,V389,V390,V391,V392,V393,V394,V395,V396,V397,V398,V399,V400,V401,V402,V403,V404,V405,V406,V407,V408,V409,V410,V411,V412,V413,V414,V415,V416,V417,V418,V419,V420,V421,V422,V423,V424,V425,V426,V427,V428,V429,V430,V431,V432,V433,V434,V435,V436,V437,V438,V439,V440,V441,V442,V443,V444,V445,V446,V447,V448,V449,V450,V451,V452,V453,V454,V455,V456,V457,V458,V459,V460,V461,V462,V463,V464,V465,V466,V467,V468,V469,V470,V471,V472,V473,V474,V475,V476,V477,V478,V479,V480,V481,V482,V483,V484,V485,V486,V487,V488,V489,V490,V491,V492,V493,V494,V495,V496,V497,V498,V499,V500,V501,V502,V503,V504,V505,V506,V507,V508,V509,V510,V511,V512,V513,V514,V515,V516,V517,V518,V519,V520,V521,V522,V523,V524,V525,V526,V527,V528,V529,V530,V531,V532,V533,V534,V535,V536,V537,V538,V539,V540,V541,V542,V543,V544,V545,V546,V547,V548,V549,V550,V551,V552,V553,V554,V555,V556,V557,V558,V559,V560,V561) 
                            ~ flagTrainTest + Subject + Activity, dataFull, mean)

    names(tmp_mean) <- features_all[, 2]  # force actual features names

    df_Mean_flagTrainTest_Subject_Activity <- subset(x=tmp_mean, select=features_OK[, 2] )  
                        remove(tmp_mean)

    # save 2nd dataset as txt files
    write.table(x=df_Mean_flagTrainTest_Subject_Activity
                  , file="df_Mean_flagTrainTest_Subject_Activity.txt")

```
The following code confirm that a Subject (person) either belong to the training block or to the test block. This was not given in the description, it my also sound good for the experiment.   
Important is that the last data.frame ( __df_Mean_flagTrainTest_Subject_Activity__ ) rappresent also 
the average of each variable for each activity and each subject, ***not matter the falg Tran / Test*** 
maybe we chould just save with a different name "df_Mean_Subject_Activity" and removing the column flagTrainTest.
```{r }
# This code confirm that a subject either belong to the training block or to the test block.
    intersect(subject_test$V1, subject_train$V1)
    sort(union(subject_test$V1, subject_train$V1))
```

