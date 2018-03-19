

```python
import pandas as pd
import os

```


```python
#students pathway
students=os.path.join("raw_data","students_complete.csv")
pd_students =pd.read_csv(students, encoding= "UTF-8")

```


```python
#schools pathway,rename schools
schools=os.path.join("raw_data","schools_complete.csv")
pd_schools =pd.read_csv(schools, encoding= "UTF-8")
pd_schools.rename(columns={"name":"school"},inplace=True)

```


```python
#sum of budget
Total_Budget=pd_schools["budget"].sum()
Total_Students=pd_schools["size"].sum()
Total_Schools=pd_schools["School ID"].count()

```


```python
# merge data sets 
merged= pd.merge(pd_students,pd_schools[["School ID","school","type","size","budget"]], on="school")

```


```python
#collect data for all districts; math score, reading score, % pass for both 
read_avg=merged["reading_score"].mean()
math_avg=merged["math_score"].mean()
read_pass_percent_dist=round(len(merged.loc[merged["reading_score"] > 70])/Total_Students*100,2)
math_pass_percent_dist= round(len(merged.loc[merged["math_score"]>70])/Total_Students*100,2)
overall_passing_dist=(read_pass_percent_dist+math_pass_percent_dist)/2

# district summary 
#total schools, total budget, average math score ,average reading score , % passing math, % passing reading, % overall passing
dist_sum= {"Total Schools":[Total_Schools],"Total Budget":[Total_Budget],"Average Math Score":[math_avg],
           "Average Reading Score":[read_avg],"% Passing Math":read_pass_percent_dist,
           "% Passing Reading":math_pass_percent_dist,"% Overall Passing":overall_passing_dist}
dist_sum_df=pd.DataFrame(data= dist_sum)
dist_sum_df=dist_sum_df[["Total Schools","Total Budget","Average Math Score",
                         "Average Reading Score","% Passing Math","% Passing Reading","% Overall Passing"]]
dist_sum_df.head()

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
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>24649428</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>82.97</td>
      <td>72.39</td>
      <td>77.68</td>
    </tr>
  </tbody>
</table>
</div>




```python
# school summary; use merge DataFrame,drop unneeded colums 
school_summary_df= merged
#calculate averages per school 
school_summary_df=school_summary_df.groupby(['school','type', 'size', 'School ID','budget'])['math_score','reading_score'].mean()
school_summary_df=school_summary_df=school_summary_df.reset_index()
school_summary_df.rename(columns={"math_score":"Average Math Score","reading_score":"Average Reading Score"}, inplace=True)
school_summary_df["per student budget"]=school_summary_df["budget"]/school_summary_df["size"]
#calculate passing average per school
school_summary_df=school_summary_df.set_index(["school"])


# find count of passing reading grades
passing_count_dist_reading=merged.loc[merged["reading_score"]>70].groupby(["school"]).count()

passing_count_dist_reading["count"]=passing_count_dist_reading["budget"]
passing_count_dist_reading["count"]
# find count of passing math grades
passing_count_dist_math=merged.loc[merged["math_score"]>70].groupby(["school"]).count()
passing_count_dist_math["count"]=passing_count_dist_math["budget"]
# put passing % into school summary
school_summary_df["% Passing Reading"]=round((passing_count_dist_reading["count"]/school_summary_df["size"])*100,2)
school_summary_df["% Passing Math"]=round((passing_count_dist_math["count"]/school_summary_df["size"])*100,2)
school_summary_df["Overall Passing"]= round((school_summary_df["% Passing Reading"]
                                             +school_summary_df["% Passing Math"])/2,2)
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
      <th>type</th>
      <th>size</th>
      <th>School ID</th>
      <th>budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>per student budget</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>7</td>
      <td>3124928</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>628.0</td>
      <td>79.30</td>
      <td>64.63</td>
      <td>71.96</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>6</td>
      <td>1081356</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>582.0</td>
      <td>93.86</td>
      <td>89.56</td>
      <td>91.71</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>1</td>
      <td>1884411</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>639.0</td>
      <td>78.43</td>
      <td>63.75</td>
      <td>71.09</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>13</td>
      <td>1763916</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>644.0</td>
      <td>77.51</td>
      <td>65.75</td>
      <td>71.63</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>4</td>
      <td>917500</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>625.0</td>
      <td>93.39</td>
      <td>89.71</td>
      <td>91.55</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>3</td>
      <td>3022020</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>652.0</td>
      <td>78.19</td>
      <td>64.75</td>
      <td>71.47</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>8</td>
      <td>248087</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>581.0</td>
      <td>92.74</td>
      <td>90.63</td>
      <td>91.68</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>0</td>
      <td>1910635</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>655.0</td>
      <td>78.81</td>
      <td>63.32</td>
      <td>71.06</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>12</td>
      <td>3094650</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>650.0</td>
      <td>78.28</td>
      <td>63.85</td>
      <td>71.06</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>9</td>
      <td>585858</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>609.0</td>
      <td>92.20</td>
      <td>91.68</td>
      <td>91.94</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>11</td>
      <td>2547363</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>637.0</td>
      <td>77.74</td>
      <td>64.07</td>
      <td>70.90</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>2</td>
      <td>1056600</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>600.0</td>
      <td>92.62</td>
      <td>89.89</td>
      <td>91.26</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>14</td>
      <td>1043130</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>638.0</td>
      <td>92.91</td>
      <td>90.21</td>
      <td>91.56</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>5</td>
      <td>1319574</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>578.0</td>
      <td>93.25</td>
      <td>90.93</td>
      <td>92.09</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>10</td>
      <td>1049400</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>583.0</td>
      <td>93.44</td>
      <td>90.28</td>
      <td>91.86</td>
    </tr>
  </tbody>
</table>
</div>




```python
# top five schools sorted 

school_summary_top_five=school_summary_df.sort_values("Overall Passing",ascending=False)

school_summary_top_five.head()
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
      <th>type</th>
      <th>size</th>
      <th>School ID</th>
      <th>budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>per student budget</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>5</td>
      <td>1319574</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>578.0</td>
      <td>93.25</td>
      <td>90.93</td>
      <td>92.09</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>9</td>
      <td>585858</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>609.0</td>
      <td>92.20</td>
      <td>91.68</td>
      <td>91.94</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>10</td>
      <td>1049400</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>583.0</td>
      <td>93.44</td>
      <td>90.28</td>
      <td>91.86</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>6</td>
      <td>1081356</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>582.0</td>
      <td>93.86</td>
      <td>89.56</td>
      <td>91.71</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>8</td>
      <td>248087</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>581.0</td>
      <td>92.74</td>
      <td>90.63</td>
      <td>91.68</td>
    </tr>
  </tbody>
</table>
</div>




```python
#bottom 5 schools 
school_summary_bottom_five=school_summary_df.sort_values("Overall Passing",ascending=True)

school_summary_bottom_five.head()

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
      <th>type</th>
      <th>size</th>
      <th>School ID</th>
      <th>budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>per student budget</th>
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>11</td>
      <td>2547363</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>637.0</td>
      <td>77.74</td>
      <td>64.07</td>
      <td>70.90</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>0</td>
      <td>1910635</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>655.0</td>
      <td>78.81</td>
      <td>63.32</td>
      <td>71.06</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>12</td>
      <td>3094650</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>650.0</td>
      <td>78.28</td>
      <td>63.85</td>
      <td>71.06</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>1</td>
      <td>1884411</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>639.0</td>
      <td>78.43</td>
      <td>63.75</td>
      <td>71.09</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>3</td>
      <td>3022020</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>652.0</td>
      <td>78.19</td>
      <td>64.75</td>
      <td>71.47</td>
    </tr>
  </tbody>
</table>
</div>




```python
# math score by grade scores_df=merged
scores_df=merged
scores_df=scores_df.groupby(['school','grade'])['math_score','reading_score'].mean()
math_score_df=scores_df.copy()
math_score_df.reset_index().pivot(index='school', columns='grade', values='math_score')


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
      <th>grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
      <td>77.083676</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
      <td>83.094697</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
      <td>76.403037</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
      <td>77.361345</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
      <td>82.044010</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
      <td>77.438495</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
      <td>83.787402</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
      <td>77.027251</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
      <td>77.187857</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
      <td>83.625455</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
      <td>76.859966</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
      <td>83.420755</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
      <td>83.590022</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
      <td>83.085578</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
      <td>83.264706</td>
    </tr>
  </tbody>
</table>
</div>




```python

# reading score by grade
scores_df=merged
scores_df=scores_df.groupby(['school','grade'])['math_score','reading_score'].mean()
reading_score_df=scores_df.copy()
reading_score_df.reset_index().pivot(index='school', columns='grade', values='reading_score')
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
      <th>grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
      <td>81.303155</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
      <td>83.676136</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
      <td>81.198598</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
      <td>80.632653</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
      <td>83.369193</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
      <td>80.866860</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
      <td>83.677165</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
      <td>81.290284</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
      <td>81.260714</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
      <td>83.807273</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
      <td>80.993127</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
      <td>84.122642</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
      <td>83.728850</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
      <td>83.939778</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
      <td>83.833333</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Scores by School Spending
# make bins and labels:
school_spending_df=school_summary_df
school_spend_bins = [0,585,615,645,675]
school_spend_labels = ["<$585", "$585-615", "$615-645","$645-675"]
school_spending_df=school_spending_df.reset_index()
del school_spending_df["type"]
del school_spending_df["size"]
del school_spending_df["School ID"]
del school_spending_df["school"]
del school_spending_df["budget"]


school_spending_df["School_Spending"] = pd.cut(school_spending_df["per student budget"], school_spend_bins, labels=school_spend_labels)

school_spending_df.reset_index()
del school_spending_df["per student budget"]
school_spending_df=school_spending_df.groupby(school_spending_df['School_Spending'], as_index=True, sort=False, group_keys=True).mean()
school_spending_df
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
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing</th>
    </tr>
    <tr>
      <th>School_Spending</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>93.322500</td>
      <td>90.350000</td>
      <td>91.835000</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>92.410000</td>
      <td>90.785000</td>
      <td>91.600000</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>79.079225</td>
      <td>81.891436</td>
      <td>83.213333</td>
      <td>73.020000</td>
      <td>78.115000</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>78.426667</td>
      <td>63.973333</td>
      <td>71.196667</td>
    </tr>
  </tbody>
</table>
</div>




```python
# scores by school size 

school_size_df=school_summary_df
school_size_bins = [0,1000,2000,5000]
school_size_labels = ["Small (<1000) ", "Medium (1000-2000)", "Large (2000-5000)"]
school_size_df=school_size_df.reset_index()
del school_size_df["type"]
del school_size_df["School ID"]
del school_size_df["school"]
del school_size_df["budget"]
del school_size_df["per student budget"]
school_size_df["School_Size"] = pd.cut(school_size_df["size"], school_size_bins, labels=school_size_labels)



del school_size_df["size"]

school_size_df.reset_index()
school_size_df=school_size_df.groupby(school_size_df['School_Size'], as_index=True, sort=False, group_keys=True).mean()
school_size_df
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
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing</th>
    </tr>
    <tr>
      <th>School_Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>83.821598</td>
      <td>83.929843</td>
      <td>92.47000</td>
      <td>91.15500</td>
      <td>91.8100</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.374684</td>
      <td>83.864438</td>
      <td>93.24400</td>
      <td>89.93000</td>
      <td>91.5880</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>77.746417</td>
      <td>81.344493</td>
      <td>80.18875</td>
      <td>67.63125</td>
      <td>73.9075</td>
    </tr>
  </tbody>
</table>
</div>




```python
school_type_df=school_summary_df
#school_type_bins = [0,1000,2000,5000]
school_type_bins =["Charter","District"]
school_type_labels = ["Charter","District"]
school_type_df=school_type_df.reset_index()
del school_type_df["School ID"]
del school_type_df["school"]
del school_type_df["budget"]
del school_type_df["size"]
del school_type_df["per student budget"]
school_type_df=school_type_df.groupby(school_type_df['type'], as_index=True, sort=False, group_keys=True).mean()
school_type_df
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
      <th>% Passing Reading</th>
      <th>% Passing Math</th>
      <th>Overall Passing</th>
    </tr>
    <tr>
      <th>type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>78.322857</td>
      <td>64.302857</td>
      <td>71.31000</td>
    </tr>
    <tr>
      <th>Charter</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>93.051250</td>
      <td>90.361250</td>
      <td>91.70625</td>
    </tr>
  </tbody>
</table>
</div>



-Charter Schools performed better over District schools
-Large Schools (large school size) performed the worst compared to small and medium schools.
-Schools that spent the most per student performed worse than schools that spent less money per student
-All top 5 performing schools were charter schools
