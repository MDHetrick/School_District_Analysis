# School District Analysis

## Project Overview

## Resources
- Data Source: 
- Software: Python xxxxxx, Jupyter Notebook

## Performing the Analysis
In performing the analysis, first, the data was loaded and cleaned.

```
# Dependencies and Setup
import pandas as pd

# File to Load (Remember to change the path if needed.)
school_data_to_load = "Resources/schools_complete.csv"
student_data_to_load = "Resources/students_complete.csv"

# Read the School Data and Student Data and store into a Pandas DataFrame
school_data_df = pd.read_csv(school_data_to_load)
student_data_df = pd.read_csv(student_data_to_load)

# Cleaning Student Names and Replacing Substrings in a Python String
# Add each prefix and suffix to remove to a list.
prefixes_suffixes = ["Dr. ", "Mr. ","Ms. ", "Mrs. ", "Miss ", " MD", " DDS", " DVM", " PhD"]

# Iterate through the words in the "prefixes_suffixes" list and replace them with an empty space, "".
for word in prefixes_suffixes:
    student_data_df["student_name"] = student_data_df["student_name"].str.replace(word,"", regex=False)

# Check names.
student_data_df.head(10)
```
Then I performed a comprehensive analysis on the original complete data. I created a district summary that showed a summary of district data. `district_summary_df_original`
I also calculated the statistics for students at each school. `per_school_summary_df_original`
Using this dataframe, I sorted % overall passing both ascending and descending to obtain the top 5 and bottom 5 Schools. `top_schools_original.head()` , `bottom_schools_original.head()`
Then I grouped math and reading scores by grade, and created dataframes for each. `math_scores_by_grade_original` , `reading_scores_by_grade_original`
Next, I created spending bins for the "Per Student Budget", and assigned each school to a bin, sorted by the bins, and was able to get a summary of scores based on spending bins. `spending_summary_df_original` 
I then performed similar steps to put the schools in bins based on school size. `size_summary_df_original`
The final piece of analysis was to look at school type as it relates to the relevant scores.  `type_summary_df_original`



### Deliverable 1:  Replace Ninth-Grade Reading and Math Scores 
Per school district instructions, the reading and math scores for 9th graders at Thomas High School are suspect, and should be excluded from analysis. To do this, I used the `loc` method to filter the `student_data_df` for 9th graders (from the grade column) and Thomas High School(from the school_name column). Within the `loc` command, I also pulled up the rows with reading_score, and replaced the scores with `NaN`. I did the same for math_scores, replacing the socres with `NaN`

```
# Install numpy using conda install numpy or pip install numpy. 
# Step 1. Import numpy as np.
import numpy as np
# Step 2. Use the loc method on the student_data_df to select all the reading scores from the 9th grade at Thomas High School and replace them with NaN.
student_data_df.loc[(student_data_df['grade'] == '9th') & 
                    (student_data_df['school_name'] == 'Thomas High School'),
                   'reading_score'] = np.nan
#  Step 3. Refactor the code in Step 2 to replace the math scores with NaN.
student_data_df.loc[(student_data_df['grade'] == '9th') & 
                    (student_data_df['school_name'] == 'Thomas High School'),
                   'math_score'] = np.nan
```
Once the scores were replaced, I performed a count on the dataframe to count the nulls.

```
print(student_data_df.isnull().sum())
```
This gave me reading_score and math_score counts of 461 each, indicating that I successfully replaced 461 reading and math scores with "NaN".


### Deliverable 2: Repeat the School District Analysis
First, I combined the `student_data_df` and the `school_data_df`dataframes to create a comprehensive dataframe.

```
# Combine the data into a single dataset
school_data_complete_df = pd.merge(student_data_df, school_data_df, how="left", on=["school_name", "school_name"])
```
Then, I counted the total number of schools, students, and the total budget as well as the average reading and math scores.
```
# Calculate the Totals (Schools and Students)
school_count = len(school_data_complete_df["school_name"].unique())
student_count = school_data_complete_df["Student ID"].count()

# Calculate the Total Budget
total_budget = school_data_df["budget"].sum()

# Calculate the Average Scores using the "clean_student_data".
average_reading_score = school_data_complete_df["reading_score"].mean()
average_math_score = school_data_complete_df["math_score"].mean()

```
Then, I performed a count on the number of 9th grade students at Thomas High School. To do this, I summed the number of null values in the student data dataframe. I was able to do this because I replaced all the reading and math scores for Thomas High School 9th graders above. To get an updated total student count, I subtracted the Thomas High School 9th grader count from the original total student count.
```
# Step 1. Get the number of students that are in ninth grade at Thomas High School.
 
THS_count = student_data_df['reading_score'].isnull().sum()

print(THS_count)

# Get the total student count 
student_count = school_data_complete_df["Student ID"].count()

# Step 2. Subtract the number of students that are in ninth grade at 
# Thomas High School from the total student count to get the new total student count.
new_total_student_count = student_count - THS_count

new_total_student_count
```
The count for THS 9th graders was 461, the total student count was 39,170, and the new total student count was 38,709.

I then performed analysis using the updated dataset. 

The first analysis I performed using the updated dataset was to count the number of students passing math and reading.
```
# Calculate the passing rates using the "clean_student_data".
passing_math_count = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)].count()["student_name"]
passing_reading_count = school_data_complete_df[(school_data_complete_df["reading_score"] >= 70)].count()["student_name"]
```

Using these numbers, I also calculated the percentages of students passing math and reading from the updated dataset.

```
# Step 3. Calculate the passing percentages with the new total student count.
passing_math_percentage = passing_math_count / float(new_total_student_count) * 100

passing_reading_percentage = passing_reading_count / float(new_total_student_count) * 100

print(passing_math_percentage)
print(passing_reading_percentage)
```
Next, I calculated how many students from the updated dataset passed both math and reading, as well as what percentage of students this represents.
```
# Calculate the students who passed both reading and math.
passing_math_reading = school_data_complete_df[(school_data_complete_df["math_score"] >= 70)
                                               & (school_data_complete_df["reading_score"] >= 70)]

# Calculate the number of students that passed both reading and math.
overall_passing_math_reading_count = passing_math_reading["student_name"].count()


# Step 4.Calculate the overall passing percentage with new total student count.

overall_passing_percentage = overall_passing_math_reading_count / float(new_total_student_count) * 100
print(overall_passing_percentage)
```
With this updated information, I created and formatted a district summary dataframe
```
# Create a DataFrame
district_summary_df = pd.DataFrame(
          [{"Total Schools": school_count, 
          "Total Students": student_count, 
          "Total Budget": total_budget,
          "Average Math Score": average_math_score, 
          "Average Reading Score": average_reading_score,
          "% Passing Math": passing_math_percentage,
         "% Passing Reading": passing_reading_percentage,
        "% Overall Passing": overall_passing_percentage}])
# Format the "Total Students" to have the comma for a thousands separator.
district_summary_df["Total Students"] = district_summary_df["Total Students"].map("{:,}".format)
# Format the "Total Budget" to have the comma for a thousands separator, a decimal separator and a "$".
district_summary_df["Total Budget"] = district_summary_df["Total Budget"].map("${:,.2f}".format)
# Format the columns.
district_summary_df["Average Math Score"] = district_summary_df["Average Math Score"].map("{:.1f}".format)
district_summary_df["Average Reading Score"] = district_summary_df["Average Reading Score"].map("{:.1f}".format)
district_summary_df["% Passing Math"] = district_summary_df["% Passing Math"].map("{:.1f}".format)
district_summary_df["% Passing Reading"] = district_summary_df["% Passing Reading"].map("{:.1f}".format)
district_summary_df["% Overall Passing"] = district_summary_df["% Overall Passing"].map("{:.1f}".format)

# Display the data frame
district_summary_df
```



## Results

There is a bulleted list that addresses how each of the seven school district metrics was affected by the changes in the data

 - How is the district summary affected?
 - How is the school summary affected?
 - How does replacing the ninth graders’ math and reading scores affect Thomas High School’s performance relative to the other schools?
 - How does replacing the ninth-grade scores affect the following:
   - Math and reading scores by grade
   - Scores by school spending
   - Scores by school size
   - Scores by school type

## Summary
Summarize four changes in the updated school district analysis after reading and math scores for the ninth grade at Thomas High School have been replaced with NaNs.

