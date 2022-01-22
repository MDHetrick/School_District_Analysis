# School District Analysis

## Project Overview
The purpose of this analysis is to evaluate the math and reading scores for a school district as they relate to several metrics. There is some question about the validity of the data for the 9th graders at Thomas High School (THS). This analysis is performed on the complete data, and with suspect data excluded.

## Resources
- Data Source: schools_complete.csv, students_complete.csv
- Software: Python 3.9.7, Jupyter Notebook 6.4.6

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

### Part 1:  Replace Ninth-Grade Reading and Math Scores 
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
Once the scores were replaced, I pulled up the dataframe to ensure only the scores were replaced, and performed a count on the dataframe to count the nulls.

![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/THS9gdf.png)

```
print(student_data_df.isnull().sum())
```
This gave me reading_score and math_score counts of 461 each, indicating that I successfully replaced 461 reading and math scores with "NaN".


### Part 2: Repeat the School District Analysis
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

THS_count = school_data_complete_df.loc[(school_data_complete_df["school_name"] == "Thomas High School") & 
                               (school_data_complete_df["grade"] == "9th"), :].count()

THS_count = THS_count["Student ID"]


THS_count

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
#### District DataFrame
![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/DF_by_school01.png)

I then ran code to create and format a school summary dataframe.
#### Updated District Summary DataFrame
![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/District_summary_DF01.png)


Next, I retrieved the number of 10th-12th grade students from THS.

```
THS_student_count = student_data_df[(student_data_df["school_name"] == "Thomas High School")].notnull().sum()

THS_student_count = THS_student_count['reading_score'].sum() 

THS_student_count
```

Once I had the student count, I created dataframes that contained students passing math, reading, and math and reading. With these dataframes, I also counted the number of students falling into each category, and I calculated the percentage of students falling in these categories as well.
```
# Step 6. Get all the students passing math from THS
THS_passing_math_count_df = school_data_complete_df.loc[(student_data_df["school_name"] == "Thomas High School") &
                                             (school_data_complete_df["math_score"] >= 70), :]["student_name"]

THS_passing_math_count = THS_passing_math_count_df.count()

THS_passing_math_count

# Step 7. Get all the students passing reading from THS
THS_passing_reading_count_df = school_data_complete_df.loc[(student_data_df["school_name"] == "Thomas High School") &
                                             (school_data_complete_df["reading_score"] >= 70), :]["student_name"]

THS_passing_reading_count = THS_passing_reading_count_df.count()

THS_passing_reading_count

# Step 8. Get all the students passing math and reading from THS
THS_passing_math_reading_count_df = school_data_complete_df.loc[(student_data_df["school_name"] == "Thomas High School") & 
                                                             (school_data_complete_df["reading_score"] >= 70) & 
                                                             (school_data_complete_df["math_score"] >= 70), :]["student_name"]

THS_passing_math_reading_count = THS_passing_math_reading_count_df.count()

THS_passing_math_reading_count 

# Step 9. Calculate the percentage of 10th-12th grade students passing math from Thomas High School. 
THS_percent_passing_math = (THS_passing_math_count / float(THS_student_count)) * 100

THS_percent_passing_math

# Step 10. Calculate the percentage of 10th-12th grade students passing reading from Thomas High School.
THS_percent_passing_reading = (THS_passing_reading_count / float(THS_student_count)) * 100

THS_percent_passing_reading

# Step 11. Calculate the overall passing percentage of 10th-12th grade from Thomas High School. 
THS_percent_passing_math_reading = (THS_passing_math_reading_count / float(THS_student_count)) * 100

THS_percent_passing_math_reading
```
With this information, I used the `.loc` function to replace the Thomas High School data from the original dataframe.

```
# Step 12. Replace the passing math percent for Thomas High School in the per_school_summary_df.
per_school_summary_df.loc["Thomas High School", "% Passing Math"] = THS_percent_passing_math

# Step 13. Replace the passing reading percentage for Thomas High School in the per_school_summary_df.

per_school_summary_df.loc["Thomas High School", "% Passing Reading"] = THS_percent_passing_reading

# Step 14. Replace the overall passing percentage for Thomas High School in the per_school_summary_df.
per_school_summary_df.loc["Thomas High School", "% Overall Passing"] = THS_percent_passing_math_reading

```
Then, the schools were again sorted by overall passing rate to see the top 5 and bottom 5

```
# Sort and show top five schools.
top_schools = per_school_summary_df.sort_values(["% Overall Passing"], ascending=False)

top_schools.head()

# Sort and show top five schools.
bottom_schools = per_school_summary_df.sort_values(["% Overall Passing"], ascending=True)

bottom_schools.head()
```
Next, scores were grouped by grade level and combined into dataframes. 

```
# Create a Series of scores by grade levels using conditionals.
ninth_graders = school_data_complete_df[school_data_complete_df["grade"] == "9th"]
tenth_graders = school_data_complete_df[school_data_complete_df["grade"] == "10th"]
eleventh_graders = school_data_complete_df[school_data_complete_df["grade"] == "11th"]
twelfth_graders = school_data_complete_df[school_data_complete_df["grade"] == "12th"]

# Group each school Series by the school name for the average math score.
ninth_grade_math_scores = ninth_graders.groupby(["school_name"]).mean()["math_score"]
tenth_grade_math_scores = tenth_graders.groupby(["school_name"]).mean()["math_score"]
eleventh_grade_math_scores = eleventh_graders.groupby(["school_name"]).mean()["math_score"]
twelfth_grade_math_scores = twelfth_graders.groupby(["school_name"]).mean()["math_score"]

# Group each school Series by the school name for the average reading score.
ninth_grade_reading_scores = ninth_graders.groupby(["school_name"]).mean()["reading_score"]
tenth_grade_reading_scores = tenth_graders.groupby(["school_name"]).mean()["reading_score"]
eleventh_grade_reading_scores = eleventh_graders.groupby(["school_name"]).mean()["reading_score"]
twelfth_grade_reading_scores = twelfth_graders.groupby(["school_name"]).mean()["reading_score"]

print(ninth_graders)
print(ninth_grade_math_scores)
print(ninth_grade_reading_scores)

# Combine each Series for average math scores by school into single data frame.
math_scores_by_grade = pd.DataFrame({
    "9th": ninth_grade_math_scores,
    "10th": tenth_grade_math_scores,
    "11th": eleventh_grade_math_scores,
    "12th": twelfth_grade_math_scores
})



# Combine each Series for average reading scores by school into single data frame.
reading_scores_by_grade = pd.DataFrame({
    "9th": ninth_grade_reading_scores,
    "10th": tenth_grade_reading_scores,
    "11th": eleventh_grade_reading_scores,
    "12th": twelfth_grade_reading_scores
})



```

The next analysis piece was to look at the average scores by school spending per student. To do this, several spending bins were created, and a data frame was created.

```
# Establish the spending bins
spending_bins = [0, 585, 630, 645, 675]
per_school_capita.groupby(pd.cut(per_school_capita, spending_bins)).count()

# Establish group names.
spending_bins = [0, 585, 630, 645, 675]
group_names = ["< $584", "$585-629", "$630-644", "$645-675"]

# Categorize spending based on the bins.
pd.cut(per_school_capita, spending_bins)

per_school_summary_df["Spending Ranges (Per Student)"] = pd.cut(per_school_capita, spending_bins, labels=group_names)

#determine data types
per_school_summary_df.dtypes

#convert object to float

per_school_summary_df["Average Math Score"] = pd.to_numeric(per_school_summary_df["Average Math Score"])
per_school_summary_df["Average Reading Score"] = pd.to_numeric(per_school_summary_df["Average Reading Score"])
per_school_summary_df["% Passing Math"] = pd.to_numeric(per_school_summary_df["% Passing Math"])
per_school_summary_df["% Passing Reading"] = pd.to_numeric(per_school_summary_df["% Passing Reading"])
per_school_summary_df["% Overall Passing"] = pd.to_numeric(per_school_summary_df["% Overall Passing"])

per_school_summary_df.dtypes

# Calculate averages for the desired columns. 
spending_math_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Math Score"]

spending_reading_scores = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["Average Reading Score"]

spending_passing_math = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Math"]

spending_passing_reading = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Passing Reading"]

overall_passing_spending = per_school_summary_df.groupby(["Spending Ranges (Per Student)"]).mean()["% Overall Passing"]

# Create the DataFrame
spending_summary_df = pd.DataFrame({
          "Average Math Score" : spending_math_scores,
          "Average Reading Score": spending_reading_scores,
          "% Passing Math": spending_passing_math,
          "% Passing Reading": spending_passing_reading,
          "% Overall Passing": overall_passing_spending})

# Format the DataFrame 
spending_summary_df["Average Math Score"] = spending_summary_df["Average Math Score"].map("{:.1f}".format)
spending_summary_df["Average Reading Score"] = spending_summary_df["Average Reading Score"].map("{:.1f}".format)
spending_summary_df["% Passing Math"] = spending_summary_df["% Passing Math"].map("{:.1f}".format)
spending_summary_df["% Passing Reading"] = spending_summary_df["% Passing Reading"].map("{:.1f}".format)
spending_summary_df["% Overall Passing"] = spending_summary_df["% Overall Passing"].map("{:.1f}".format)

spending_summary_df.index.name = None

spending_summary_df
```
Next the schools were placed into bins based on their sizes, and a similar analysis was performed.

```
# Establish the bins.
size_bins = [0, 1000, 2000, 5000]

group_names = ["Small (<1000)", 
               "Medium (1000-2000)", 
               "Large (2000-5000)"]

#look at bin distribution
per_school_counts.groupby(pd.cut(per_school_counts, size_bins)).count()

# Categorize spending based on the bins.
per_school_summary_df["School Size"] = pd.cut(per_school_counts, size_bins, labels=group_names)

# Calculate averages for the desired columns. 
size_math_scores = per_school_summary_df.groupby(["School Size"]).mean()["Average Math Score"]

size_reading_scores = per_school_summary_df.groupby(["School Size"]).mean()["Average Reading Score"]

Size_passing_math = per_school_summary_df.groupby(["School Size"]).mean()["% Passing Math"]
Size_passing_reading = per_school_summary_df.groupby(["School Size"]).mean()["% Passing Reading"]
Size_overall_passing = per_school_summary_df.groupby(["School Size"]).mean()["% Overall Passing"]

Size_overall_passing

# Assemble into DataFrame. 
size_summary_df = pd.DataFrame({
          "Average Math Score" : size_math_scores,
          "Average Reading Score": size_reading_scores,
          "% Passing Math": Size_passing_math,
          "% Passing Reading": Size_passing_reading,
          "% Overall Passing": Size_overall_passing
})

# Format Math and Reading scores
size_summary_df["Average Math Score"] = size_summary_df["Average Math Score"].map("{:.1f}".format)

size_summary_df["Average Reading Score"] = size_summary_df["Average Reading Score"].map("{:.1f}".format)

# format percentages
size_summary_df["% Passing Math"] = size_summary_df["% Passing Math"].map("{:.1f}".format)
size_summary_df["% Passing Reading"] = size_summary_df["% Passing Reading"].map("{:.1f}".format)
size_summary_df["% Overall Passing"] = size_summary_df["% Overall Passing"].map("{:.1f}".format)

size_summary_df

```
#### Updated Size Summary

![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/updated_size_summary.png)

#### Original Size Summary

![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/original_size_summary.png)

The final step was to perform an analysis based on the type of school.

```
# Calculate averages for the desired columns. 
type_math_scores = per_school_summary_df.groupby(["School Type"]).mean()["Average Math Score"]
type_reading_scores = per_school_summary_df.groupby(["School Type"]).mean()["Average Reading Score"]
type_passing_math = per_school_summary_df.groupby(["School Type"]).mean()["% Passing Math"]
type_passing_reading = per_school_summary_df.groupby(["School Type"]).mean()["% Passing Reading"]
type_overall_passing = per_school_summary_df.groupby(["School Type"]).mean()["% Overall Passing"]

# Assemble into DataFrame. 
type_summary_df = pd.DataFrame({
          "Average Math Score" : type_math_scores,
          "Average Reading Score": type_reading_scores,
          "% Passing Math": type_passing_math,
          "% Passing Reading": type_passing_reading,
          "% Overall Passing": type_overall_passing    
})

# # Format the DataFrame 
type_summary_df["Average Math Score"] = type_summary_df["Average Math Score"].map("{:.1f}".format)
type_summary_df["Average Reading Score"] = type_summary_df["Average Reading Score"].map("{:.1f}".format)
type_summary_df["% Passing Math"] = type_summary_df["% Passing Math"].map("{:.1f}".format)
type_summary_df["% Passing Reading"] = type_summary_df["% Passing Reading"].map("{:.1f}".format)
type_summary_df["% Overall Passing"] = type_summary_df["% Overall Passing"].map("{:.1f}".format)

type_summary_df.index.name=None
```
## Results
 - How is the district summary affected?
    - The average math score decreased from 79.0 to 78.9
    - There was no change in the average reading score
    - The % passing math decreased from 75.0% to 74.8%
    - The % passing reading decreased from 85.8% to 85.7%
    - The overall % passing decreased from 65.2% to 64.9% 
![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/district_summary_comparison.png)

 - How is the school summary affected? For Thomas High School:  
    - There was no change in the average reading score
    - The average math score increased from 83.8 to 83.9
    - The % passing math decreased from 93.3% to 93.2%
    - The % passing reading decreased from 97.3% to 97.0%
    - The overall % passing decreased from 90.9% to 90.6%  
![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/THS_comparison.png)

 - How does replacing the ninth graders’ math and reading scores affect Thomas High School’s performance relative to the other schools?
    - Initially, Thomas High School was 2nd of all schools when looking at % overall passing, with 90.9% overall passing. 
    - After replacing the scores for the ninth graders, Thomas High School dropped to 3rd with 90.6% overall passing.
    - Bottom 5 schools were not impacted.

![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/top5_comparison.png) ![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/bottom5_comparison.png)



 - How does replacing the ninth-grade scores affect the following:
   - Math and reading scores by grade, average 9th grade reading scores decreased from 82.5 to 82.4 after THS 9th graders were excluded. Average 9th grade math scores decreased from 80.4 to 80.1 after THS 9th graders were excluded.
   - Scores by school spending: THS falls within the $630-644 per capita spending bin. Within this bin, the % passing reading decreased from 84.4% to 84.3%, and % overall passing decreased from 62.9% to 62.8%
   
![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/spending_summary_comparison.png)


   - Scores by school size: THS is a medium sized school. Within this bin, the % passing decreased from 96.8% to 96.7%, and the % overall passing decreased from 90.6% to 90.5%

![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/size_summary_comparison.png)


   - Scores by school type: THS is a charter school. Within this bin, the % passing reading decreased from 96.6% to 96.5%.

![Image](https://github.com/MDHetrick/School_District_Analysis/blob/main/resources/type_summary_comparison.png)



## Summary
After replacing the THS 9th grade reading and math scores with NaN, the overall % passing for THS decreased, as did the overall % passing for the per capita bin and the school size bin that THS is in. This change also bumped THS from 2nd to 3rd in the list of top 5 schools, with Griffin High school moving from 3rd to 2nd. 

