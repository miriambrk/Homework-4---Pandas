
# Academy of Py Data Analysis - Miriam Berkowitz<br>
### Observed Trends:<br>
1)  Schools that spent more per student had lower overall passing rates than the schools that spent the least. The schools that spent the least per student had the highest overall passing rates.
<br>
2)  Larger schools had worse passing rates than smaller schools.
<br>
3)  Charter schools had better passing rates than district schools.



```python
#Dependencies
import pandas as pd
```


```python
#store filepath in variable
file1 = "schools_complete.csv"
file2 = "students_complete.csv"

#Read CSV files
schools_df = pd.read_csv(file1, encoding = "utf-8")
students_df = pd.read_csv(file2, encoding = "utf-8")
```

### District Summary


```python
#compute district summary
total_schools = schools_df["School ID"].count()
total_students = students_df["Student ID"].count()
total_budget = schools_df["budget"].sum()
avg_math_score = round(students_df["math_score"].mean(),2)
avg_reading_score = round(students_df["reading_score"].mean(),2)

#MATH: assume the passing is a score of greater than 70
math_passed_table = students_df.loc[students_df["math_score"]> 70,:]

percent_passed_math = round((math_passed_table["Student ID"].count() / total_students),2) * 100
percent_passed_math
```




    72.0




```python
#READING: assume the passing is a score of greater than 70
reading_passed_table = students_df.loc[students_df["reading_score"]> 70,:]
percent_passed_reading = round((reading_passed_table["Student ID"].count() / total_students),2) * 100
percent_passed_reading

#compute overall passing rate -- average of math and reading passage averages
overall_passing_rate = (percent_passed_math + percent_passed_reading)/2
overall_passing_rate
```




    77.5




```python
#create a table with all the summary data computed above
summary_table = pd.DataFrame({"Total Schools": [total_schools], 
                              "Total Students": [total_students], 
                              "Total Budget": [total_budget], 
                              "Average Math Score": [avg_math_score], 
                              "Average Reading Score": [avg_reading_score], 
                              "Percentage Passing Math": [percent_passed_math], 
                              "Percentage Passing Reading": [percent_passed_reading], 
                              "Overall Passing Rate": [overall_passing_rate]})

#summary_table
#reorder the columns into the preferred order
summary_table = summary_table[["Total Schools",
                              "Total Students",
                              "Total Budget",
                              "Average Math Score",
                              "Average Reading Score",
                              "Percentage Passing Math",
                              "Percentage Passing Reading",
                              "Overall Passing Rate"]]


#format the budget as dollars and the passing rate fields as percents 
summary_table['Total Budget'] = summary_table['Total Budget'].map('${:,.0f}'.format)
summary_table['Percentage Passing Math'] = summary_table['Percentage Passing Math'].map('{:.0f}%'.format)
summary_table['Percentage Passing Reading'] = summary_table['Percentage Passing Reading'].map('{:.0f}%'.format)
summary_table['Overall Passing Rate'] = summary_table['Overall Passing Rate'].map('{:.0f}%'.format)

summary_table
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Percentage Passing Math</th>
      <th>Percentage Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>$24,649,428</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>72%</td>
      <td>83%</td>
      <td>78%</td>
    </tr>
  </tbody>
</table>
</div>



### School Summary


```python
#rename the school name in both tables so that they are the same column name
schools_df = schools_df.rename(columns={"name":"school_name"})
students_df = students_df.rename(columns={"school":"school_name"})

```


```python
#merge the school and students table, joining on school_name; inner join
merge_table = pd.merge(schools_df, students_df, on="school_name")
merge_table.head()

#compute average reading and math scores by school name
sch_avg_scores = pd.DataFrame(students_df.groupby("school_name").mean())

sch_avg_scores = sch_avg_scores.reset_index()
sch_avg_scores = sch_avg_scores.drop('Student ID', axis=1)
```


```python
#make copy of schools data frame; then add the rest of the data for each school
school_summary_df = schools_df

#merge avg reading and math into school summary
school_summary_df = school_summary_df.merge(sch_avg_scores,on="school_name")
```


```python
#compute "per student" budget; school budget/size
school_summary_df["Per Student Budget"] = round(school_summary_df["budget"] / school_summary_df["size"])
```


```python
#compute percent passing math and reading and overall passing rate

#MATH: assume the passing is a score of over 70

#create table of just the students who passed math; then group by school
math_passed_table = students_df.loc[students_df["math_score"]> 70,:]
math_passed_table.head()
math_passed_totals = math_passed_table.groupby("school_name")['Student ID'].count()
   
math_passed_totals = math_passed_totals.reset_index()
#correct the column name to math_passed instead of Student ID
math_passed_totals = math_passed_totals.rename(columns={'Student ID':'math_passed'})
```


```python
#create table of just the students who passed reading; then group by school
read_passed_table = students_df.loc[students_df["reading_score"]> 70,:]
read_passed_table.head()
read_passed_totals = read_passed_table.groupby("school_name")['Student ID'].count()
read_passed_totals = read_passed_totals.reset_index()
#correct the column name to read_passed instead of Student ID
read_passed_totals = read_passed_totals.rename(columns={'Student ID':'read_passed'})

```


```python
#merge in math_passed_totals and read_passed_totals and then compute percent instead of total; then compute overall %
school_summary_df = school_summary_df.merge(math_passed_totals,on="school_name")
school_summary_df = school_summary_df.merge(read_passed_totals,on="school_name")
school_summary_df['math_passed'] = (school_summary_df['math_passed'] / school_summary_df["size"]) * 100 
school_summary_df['read_passed'] = (school_summary_df['read_passed'] / school_summary_df["size"]) * 100
school_summary_df['% Overall Passing'] = (school_summary_df['math_passed'] + school_summary_df['read_passed'])/2
```


```python
#format and change column names
school_summary_df = school_summary_df.rename(columns= {"math_passed": "% Passing Math",
                                            "read_passed": "% Passing Reading",
                                                       "school_name": "School Name",
                                                       "type": "School Type",
                                                       "size": "Total Students",
                                                       "budget": "Total School Budget",
                                                       "reading_score": "Average Reading Score",
                                                       "math_score": "Average Math Score"})
                                     
school_summary_df['Per Student Budget'] = school_summary_df['Per Student Budget'].map('${:,.2f}'.format)
school_summary_df['Total School Budget'] = school_summary_df['Total School Budget'].map('${:,.2f}'.format)

# Drop the column with school ID 
school_summary_df.drop('School ID', axis=1,inplace=True)
```


```python
#reorder the columns
school_summary_df = school_summary_df[["School Name",
                              "School Type",
                              "Total Students",
                              "Total School Budget",
                                "Per Student Budget",
                              "Average Math Score",
                              "Average Reading Score",
                              "% Passing Math",
                              "% Passing Reading",
                              "% Overall Passing"]]

#order alphabetically by school
school_summary_df = school_summary_df.sort_values("School Name")

#display the data
school_summary_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>Bailey High School</td>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>$628.00</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>64.630225</td>
      <td>79.300643</td>
      <td>71.965434</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>91.711518</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>71.091896</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ford High School</td>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>65.753925</td>
      <td>77.510040</td>
      <td>71.631982</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>89.713896</td>
      <td>93.392371</td>
      <td>91.553134</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>71.467098</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>91.686183</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>71.066164</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>71.067003</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>91.943867</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>70.905226</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>89.892107</td>
      <td>92.617831</td>
      <td>91.254969</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>90.214067</td>
      <td>92.905199</td>
      <td>91.559633</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Wright High School</td>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>91.861111</td>
    </tr>
  </tbody>
</table>
</div>



### Top Performing Schools (By Passing Rate)


```python
#sort the data to find the 5 highest performing schools
#sort on overall passing rate (descending) 
sorted_df = school_summary_df.sort_values(["% Overall Passing"], ascending=[False])
```


```python
#top 5 (head) 
top_five = sorted_df.head()
top_five
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>91.943867</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Wright High School</td>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>91.861111</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>91.711518</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>91.686183</td>
    </tr>
  </tbody>
</table>
</div>



### Bottom Performing Schools (by Passing Rate)



```python
#note: the tail has the lowest performers, but they would be displayed reverse order, so resort ascending and display the first 5
sorted_df = school_summary_df.sort_values(["% Overall Passing"], ascending=[True])
sorted_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Name</th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>70.905226</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>71.066164</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Johnson High School</td>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>71.067003</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>71.091896</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>71.467098</td>
    </tr>
  </tbody>
</table>
</div>



### Math Scores by Grade


```python
#group the student data by school name and grade
grouped = students_df.groupby(['school_name','grade'])
```


```python
#get the average math score from the grouped data
math_df = pd.DataFrame(grouped['math_score'].mean())
math_df = math_df.reset_index()
```


```python
#reorganize the data for the output format
pivot_math = math_df.pivot(index='school_name',columns='grade',values='math_score')
```


```python
pivot_math = pivot_math.reset_index()

pivot_math.columns.name = None

#reorder the grade columns so they are in order!
pivot_math = pivot_math[['school_name','9th','10th','11th','12th']]
pivot_math.set_index('school_name', inplace=True)
pivot_math
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>



### Reading Scores by Grade


```python
#get the average reading score from the grouped data
read_df = pd.DataFrame(grouped['reading_score'].mean())
```


```python
read_df = read_df.reset_index()
#reorganize the data for the output format
pivot_read = read_df.pivot(index='school_name',columns='grade',values='reading_score')
pivot_read = pivot_read.reset_index()

#remove grade index column
pivot_read.columns.name = None
```


```python
#reorder the grade columns so they are in order!
pivot_read = pivot_read[['school_name','9th','10th','11th','12th']]

pivot_read.set_index('school_name', inplace=True)
pivot_read
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school_name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>



### Scores by School Spending


```python
#put school summary data into schools_df
schools_df = school_summary_df
```


```python
#set the per_student_budget to float in a new column
schools_df['Spending Range'] = schools_df['Per Student Budget'].replace('\$','',regex=True).astype('float')

#set 4 bins for budget per student
bins = [0,580,610,640,670]
group_names = ['Less than $580','\$580-$610','\$610-$640','More than $640']

#use group_indices to keep the data in the order I want; change to the group_names at the end
group_indices = ["1","2","3","4"]

schools_df["Spending Range Bins"] = pd.cut(schools_df["Spending Range"], bins, labels=group_indices)
```


```python
#group by the bins
school_groups = schools_df.groupby("Spending Range Bins")
school_means = school_groups.mean()
```


```python
#remove unneeded columns
school_means.drop('Total Students', axis=1,inplace=True)
school_means.drop('Spending Range', axis=1,inplace=True)
school_means.reset_index(inplace=True)
```


```python
#Replace the temporary index with the actual group names 
#this was done to keep the rows in the correct order
school_means['Spending Range Bins'] = school_means['Spending Range Bins'].replace(['1','2','3','4'], group_names)
school_means.set_index('Spending Range Bins', inplace=True)
school_means
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
    </tr>
    <tr>
      <th>Spending Range Bins</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Less than $580</th>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>92.093736</td>
    </tr>
    <tr>
      <th>\$580-$610</th>
      <td>83.549353</td>
      <td>83.903238</td>
      <td>90.408972</td>
      <td>92.974087</td>
      <td>91.691529</td>
    </tr>
    <tr>
      <th>\$610-$640</th>
      <td>79.474551</td>
      <td>82.120471</td>
      <td>74.474926</td>
      <td>84.355203</td>
      <td>79.415064</td>
    </tr>
    <tr>
      <th>More than $640</th>
      <td>77.023555</td>
      <td>80.957446</td>
      <td>64.417757</td>
      <td>78.198366</td>
      <td>71.308062</td>
    </tr>
  </tbody>
</table>
</div>



### Scores by School Size


```python
#set 3 bins for school size
bins = [0,1500,2500,5000]
group_names = ['Small (<1500)','Medium (1500-2500)','Large (>2500)']
group_indices = ["1","2","3"]
schools_df["Size Bins"] = pd.cut(schools_df["Total Students"], bins, labels=group_indices)
```


```python
#group by the bins
school_groups = schools_df.groupby("Size Bins")
school_means = school_groups.mean()

#remove unneeded columns
school_means.drop('Total Students', axis=1,inplace=True)

school_means.reset_index(inplace=True)
```


```python
#Replace the temporary index with the actual group names 
#this was done to keep the rows in the correct order
school_means['Size Bins'] = school_means['Size Bins'].replace(['1','2','3'], group_names)

school_means.set_index('Size Bins', inplace=True)
school_means
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
      <th>Spending Range</th>
    </tr>
    <tr>
      <th>Size Bins</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1500)</th>
      <td>83.664898</td>
      <td>83.892148</td>
      <td>90.676736</td>
      <td>92.778720</td>
      <td>91.727728</td>
      <td>605.000000</td>
    </tr>
    <tr>
      <th>Medium (1500-2500)</th>
      <td>83.359224</td>
      <td>83.898984</td>
      <td>90.175120</td>
      <td>93.217267</td>
      <td>91.696193</td>
      <td>596.200000</td>
    </tr>
    <tr>
      <th>Large (&gt;2500)</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>64.302528</td>
      <td>78.324559</td>
      <td>71.313543</td>
      <td>643.571429</td>
    </tr>
  </tbody>
</table>
</div>



### Scores by School Type


```python
#group by type
school_groups = schools_df.groupby("School Type")
school_means = school_groups.mean()

#remove unneeded columns
school_means.drop('Total Students', axis=1,inplace=True)
del school_means['Spending Range']
school_means
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>90.363226</td>
      <td>93.052812</td>
      <td>91.708019</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>64.302528</td>
      <td>78.324559</td>
      <td>71.313543</td>
    </tr>
  </tbody>
</table>
</div>






```python

```
