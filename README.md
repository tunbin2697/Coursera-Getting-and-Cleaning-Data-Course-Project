# Explanation of the `run_analysis.R` Script

This Markdown file explains the R script designed to clean, process, and summarize the Human Activity Recognition Using Smartphones Dataset (UCI HAR Dataset). The script performs several steps, from downloading the raw data to creating a tidy dataset with the average of each variable for each activity and each subject.

## Script Overview

The primary goal of this script is to:
1.  Download and extract the UCI HAR Dataset.
2.  Merge the training and test datasets.
3.  Extract only the measurements on the mean and standard deviation for each measurement.
4.  Apply descriptive activity names to name the activities in the dataset.
5.  Appropriately label the dataset with descriptive variable names.
6.  Create a second, independent tidy data set with the average of each variable for each activity and each subject.

## Step-by-Step Explanation

### 1. Load `dplyr` Library

```R
library(dplyr)
```

This line loads the `dplyr` package, which is a powerful tool for data manipulation in R. It provides functions for filtering, selecting, arranging, summarizing, and joining data frames.

### 2. Create Directory (if it doesn't exist)

```R
if(!file.exists("./getcleandata")) {
  dir.create("./getcleandata")
}
```

This code block checks if a directory named `getcleandata` exists in the current working directory. If it doesn't, the `dir.create()` function creates it. This directory will be used to store the downloaded and extracted dataset.

### 3. Download the Dataset

```R
fileurl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileurl, destfile = "./getcleandata/projectdataset.zip")
```

Here, the URL of the dataset zip file is stored in the `fileurl` variable. Then, the `download.file()` function is used to download the zip file from the specified URL and save it as `projectdataset.zip` within the `getcleandata` directory.

### 4. Unzip the Dataset

```R
unzip(zipfile = "./getcleandata/projectdataset.zip", exdir = "./getcleandata")
```

After downloading, this line unzips the `projectdataset.zip` file. The `exdir` argument specifies that the contents of the zip file should be extracted into the `getcleandata` directory, creating a new folder named `UCI HAR Dataset`.

### 5. Read Data into R

```R
x_train <- read.table("./getcleandata/UCI HAR Dataset/train/X_train.txt")
y_train <- read.table("./getcleandata/UCI HAR Dataset/train/y_train.txt")
subject_train <- read.table("./getcleandata/UCI HAR Dataset/train/subject_train.txt")

x_test <- read.table("./getcleandata/UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./getcleandata/UCI HAR Dataset/test/y_test.txt")
subject_test <- read.table("./getcleandata/UCI HAR Dataset/test/subject_test.txt")

features <- read.table("./getcleandata/UCI HAR Dataset/features.txt")

activityLabels <- read.table("./getcleandata/UCI HAR Dataset/activity_labels.txt")
```

This section reads various data files into R data frames:
*   `x_train` and `x_test`: Contain the main feature vectors (measurement data) for the training and test sets, respectively.
*   `y_train` and `y_test`: Contain the activity labels (numeric IDs) for the training and test sets.
*   `subject_train` and `subject_test`: Contain the subject IDs for each observation in the training and test sets.
*   `features`: Contains the names of the 561 features/measurements.
*   `activityLabels`: Contains the mapping between activity IDs (1-6) and their descriptive names (e.g., WALKING, SITTING).

### 6. Assign Column Names to Activity Labels

```R
colnames(activityLabels) <- c("activityID", "activityType")
```

This line assigns meaningful column names (`activityID` and `activityType`) to the `activityLabels` data frame, making it easier to understand and merge later.

### 7. Assign Column Names to Training and Test Data

```RR
colnames(x_train) <- features[, 2]
colnames(y_train) <- "activityID"
colnames(subject_train) <- "subjectID"

colnames(x_test) <- features[, 2]
colnames(y_test) <- "activityID"
colnames(subject_test) <- "subjectID"
```

Here, descriptive column names are assigned to the training and test datasets:
*   `x_train` and `x_test`: The feature names are taken from the second column of the `features` data frame.
*   `y_train` and `y_test`: The column is named `activityID` to represent the activity being performed.
*   `subject_train` and `subject_test`: The column is named `subjectID` to represent the participant who performed the activity.

### 8. Merge Training and Test Datasets

```R
alltrain <- cbind(y_train, subject_train, x_train)
alltest <- cbind(y_test, subject_test, x_test)
finaldataset <- rbind(alltrain, alltest)```

This section combines the individual data components:
*   `alltrain`: Combines `y_train`, `subject_train`, and `x_train` column-wise (`cbind`) to create a complete training dataset.
*   `alltest`: Combines `y_test`, `subject_test`, and `x_test` column-wise (`cbind`) to create a complete test dataset.
*   `finaldataset`: Combines `alltrain` and `alltest` row-wise (`rbind`) to create a single, comprehensive dataset.

### 9. Extract Mean and Standard Deviation Measurements

```R
mean_and_std <- grepl("activityID|subjectID|mean\\(\\)|std\\(\\)", colnames(finaldataset))
setforMeanandStd <- finaldataset[, mean_and_std]
```

This step identifies and extracts only the relevant columns:
*   `grepl()`: This function searches for patterns in the column names of `finaldataset`. The pattern `"activityID|subjectID|mean\\(\\)|std\\(\\\\)"` selects columns that contain "activityID", "subjectID", "mean()", or "std()" (the double backslashes are needed to escape the parentheses as they are special characters in regular expressions). It returns a logical vector.
*   `setforMeanandStd`: This new data frame is created by subsetting `finaldataset` using the `mean_and_std` logical vector, keeping only the identified columns.

### 10. Merge with Activity Names

```R
setWithActivityNames <- merge(setforMeanandStd, activityLabels, by = "activityID", all.x = TRUE)
```

This line merges the `setforMeanandStd` data frame with the `activityLabels` data frame.
*   `by = "activityID"`: Specifies that the merge should be performed based on the common `activityID` column.
*   `all.x = TRUE`: Ensures that all rows from `setforMeanandStd` are kept, and matching `activityType` information from `activityLabels` is added. This results in a dataset where numerical activity IDs are replaced with descriptive names.

### 11. Appropriately Label the Dataset with Descriptive Variable Names

```R
colnames(setWithActivityNames) <- gsub("^t", "time", colnames(setWithActivityNames))
colnames(setWithActivityNames) <- gsub("^f", "frequency", colnames(setWithActivityNames))
colnames(setWithActivityNames) <- gsub("Acc", "Accelerometer", colnames(setWithActivityNames))
colnames(setWithActivityNames) <- gsub("Gyro", "Gyroscope", colnames(setWithActivityNames))
colnames(setWithActivityNames) <- gsub("Mag", "Magnitude", colnames(setWithActivityNames))
colnames(setWithActivityNames) <- gsub("BodyBody", "Body", colnames(setWithActivityNames))
```

This series of `gsub()` (global substitute) functions cleans up and makes the variable names more readable and descriptive:
*   `"^t"` is replaced with `"time"`: Prefixes like `tBodyAcc` become `timeBodyAccelerometer`.
*   `"^f"` is replaced with `"frequency"`: Prefixes like `fBodyAcc` become `frequencyBodyAccelerometer`.
*   `"Acc"` is replaced with `"Accelerometer"`.
*   `"Gyro"` is replaced with `"Gyroscope"`.
*   `"Mag"` is replaced with `"Magnitude"`.
*   `"BodyBody"` is corrected to `"Body"` (due to a typo in the original feature names).

### 12. Create a Tidy Dataset with Averages

```R
tidySet <- setWithActivityNames %>%
  group_by(subjectID, activityID, activityType) %>%
  summarise_all(mean)
```

This block creates the final tidy dataset:
*   `setWithActivityNames %>%`: The pipe operator `%>%` passes `setWithActivityNames` as the first argument to the next function.
*   `group_by(subjectID, activityID, activityType)`: Groups the data by `subjectID`, `activityID`, and `activityType`. This means that subsequent operations will be performed independently within each unique combination of subject and activity.
*   `summarise_all(mean)`: Calculates the mean of *every other column* (all columns except the grouping variables) for each of these groups. This results in a tidy dataset where each row represents the average of all measurement variables for a specific subject performing a specific activity.

### 13. Write the Tidy Dataset to a File

```R
write.table(tidySet, "tidySet.txt", row.names = FALSE)
```

Finally, this line writes the `tidySet` data frame into a text file named `tidySet.txt` in the current working directory.
*   `row.names = FALSE`: Prevents R from writing the row numbers as a column in the output file, as they are not part of the actual data.

This completes the explanation of the R script, detailing each step from raw data to the final tidy and summarized dataset.
