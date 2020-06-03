Course Project for Getting and Cleaning Data Course under Data Science specialization.
This document describes data transformation flow to obtain course objectives:

```
You should create one R script called run_analysis.R that does the following.
 Merges the training and the test sets to create one data set.
 Extracts only the measurements on the mean and standard deviation for each measurement.
 Uses descriptive activity names to name the activities in the data set
 Appropriately labels the data set with descriptive variable names.
 From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```

# Data Transformation flow

## Download and unzip data
```
## Download and unzip data for the project
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip", destfile ="UCI_HAR_Dataset.zip", method = "curl")
unzip("UCI_HAR_Dataset.zip")
## Remove .zip archive
file.remove("UCI_HAR_Dataset.zip")
```

## Test and Train sets are loaded into separate variables and joined using rbind() to form a variable storing full set
```
## Load train and test data sets
X_test <- read.table("UCI HAR Dataset/test/X_test.txt", quote="\"", comment.char="")
X_train <- read.table("UCI HAR Dataset/train/X_train.txt", quote="\"", comment.char="")
## Merge two data sets
X_full <- rbind(X_test, X_train)
```

## Import feature names and use regex to extract row numbers that will correspond to column numbers containing measurements for mean and standard deviation. Extract measurements using select() from dplyr library and save to new variable.
```
## Import features names from 'features.txt'
features <- read.table("UCI HAR Dataset/features.txt", quote="\"", comment.char="")
## Rename column name for convinience
colnames(features)[2]  <- "feature"
## Extract row numbers with measurement names containing only "mean()" or "std()" using regex
columnNumbers <- grep("(mean[(][)]|std[(][)])", features$feature)
## Select columns by numbers returned from previous function using "dplyr" library
library(dplyr)
X_mean_std <- select(X_full, all_of(columnNumbers))
```

## Assign descriptive variable names by setting column names of new variable to the values obtained from features file in the previous step
```
## Apply column names from features.txt based on selected numbers
colnames(X_mean_std) <- features$feature[columnNumbers]
```
## Add activity id's to the set by importing and merging test and training activity labels and appending as a new column
```
## Import training labels
y_test <- read.table("UCI HAR Dataset/test/y_test.txt", quote="\"", comment.char="")
y_train <- read.table("UCI HAR Dataset/train/y_train.txt", quote="\"", comment.char="")
## Add activities to single variable
y_full <- rbind(y_test, y_train)
## Rename column name for convinience
colnames(y_full) <- "activityId"
## Append to dataset
X_mean_std <- cbind(y_full, X_mean_std)
```

## Add test subject's id's to the set by importing and merging test and training subjects and appending as a new column
```
subject_test <- read.table("UCI HAR Dataset/test/subject_test.txt", quote="\"", comment.char="")
subject_train <- read.table("UCI HAR Dataset/train/subject_train.txt", quote="\"", comment.char="")
## Add subjects to single variable
subject_full <- rbind(subject_test, subject_train)
## Rename column name for convinience
colnames(subject_full) <- "subjectId"
## Append to dataset
X_mean_std <- cbind(subject_full, X_mean_std)
```

## Replace activity id's with activity names by importing activity labels from the corresponding file and then suing merge() by activity id. Delete the activity id field to tidy set.
```
## Import activity labels
activity_labels <- read.table("UCI HAR Dataset/activity_labels.txt", quote="\"", comment.char="")
## Rename column names for convinience
colnames(activity_labels)[1]  <- "activityId"
colnames(activity_labels)[2]  <- "activity"
## Merge with dataset by activityId to match ID with descriptive name
X_mean_std <- merge(X_mean_std, activity_labels, by='activityId')
## Delete activityId column since we do not need it anymore
X_mean_std <- select(X_mean_std, -(activityId))
```

## Form a new set by grouping by two variables - id of subject and activity name using group_by from dplyr library. Apply summarize_all from dplyr library using mean() function as an argument to apply to all variables in the set to obtain average values. Save to .csv file using write.table()

```
## Use summarize over grouped tbl by Subject Id and activity
Dataset <- X_mean_std %>%
  group_by(subjectId, activity) %>%
  summarise_all(mean) 

## Save dataset into .csv
write.table(Dataset, file = "UCI HAR Dataset/Dataset.csv", sep = ",", row.name=FALSE)
```