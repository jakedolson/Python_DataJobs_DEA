# Overview

This project examines the U.S. data job market with a primary focus on data analyst positions. The goal of the analysis is to understand which skills employers request most often, how demand for those skills changes over time, and how skill demand compares with salary. These comparisons can help identify skills that may be valuable for someone preparing for a career in data analytics.

The analysis uses the dataset provided by [Luke Barousse](https://lukebarousse.com/python). The dataset contains information about job titles, salaries, locations, and skills listed in job postings. I used Python to investigate skill demand, changes in demand over time, salary differences between data roles, and the relationship between skill demand and compensation.


# The Questions

The project is organized around four main questions:

1. Which skills are most frequently requested for the three most common data roles?
2. How does demand for common data analyst skills change throughout the year?
3. What differences can be seen in salaries across data roles and skills?
4. Which data analyst skills combine relatively high demand with higher salaries?

# Tools I Used

The analysis was completed using several Python-based tools:

- **Python:** Used to perform the overall analysis and generate the results.
  - **Pandas:** Used for data manipulation, filtering, and analysis.
  - **Matplotlib:** Used to create visualizations.
  - **Seaborn:** Used to build statistical and categorical visualizations.
- **Jupyter Notebooks:** Used to run the analysis and document the individual steps.
- **Visual Studio Code:** Used as the primary development environment for the Python work.
- **Git & GitHub:** Used for version control and to organize and share the project code.

# Data Preparation and Cleanup

Before beginning the analysis, I prepared the dataset so that the relevant fields could be used consistently throughout the project.


## Import & Clean Up Data

The first step was to import the required libraries and load the job-posting dataset. I also converted the posting dates into a date format and converted the stored skill values into Python lists.

```python
# Importing Libraries
import ast
import pandas as pd
import seaborn as sns
from datasets import load_dataset
import matplotlib.pyplot as plt  

# Loading Data
dataset = load_dataset('lukebarousse/data_jobs')
df = dataset['train'].to_pandas()

# Data Cleanup
df['job_posted_date'] = pd.to_datetime(df['job_posted_date'])
df['job_skills'] = df['job_skills'].apply(lambda x: ast.literal_eval(x) if pd.notna(x) else x)
```

### Limiting the Analysis to U.S. Jobs

Because the project is focused on the U.S. job market, I filtered the dataset so that the remaining records represented positions located in the United States.

```python
df_US = df[df['job_country'] == 'United States']

```

# The Analysis

Each Jupyter notebook for this project aimed at investigating specific aspects of the data job market. Here’s how I approached each question:

## 1. Most Requested Skills for the Three Most Common Data Roles

I first identified the three most common data job titles and then examined the five skills that appeared most frequently for each role. This provides a comparison of the technical skills employers are requesting across different types of data positions.


View my notebook with detailed steps here: [2_Skill_Demand](3_Project\2_Skills_Count.ipynb).

### Visualize Data

```python
fig, ax = plt.subplots(len(job_titles), 1)


for i, job_title in enumerate(job_titles):
    df_plot = df_skills_perc[df_skills_perc['job_title_short'] == job_title].head(5)[::-1]
    sns.barplot(data=df_plot, x='skill_percent', y='job_skills', ax=ax[i], hue='skill_count', palette='dark:b_r')

plt.show()
```

### Results

![Likelihood of Skills Requested in the US Job Postings](3_Project\images\skill_demand_top3_data_roles.png)

*Bar graph visualizing the salary for the top 3 data roles and their top 5 skills associated with each.*

### Findings

- SQL is the most frequently requested skill for both Data Analysts and Data Scientists, appearing in more than half of the postings for each role. For Data Engineers, Python is the leading skill, appearing in 68% of postings.
- Data Engineering positions place more emphasis on specialized technologies such as AWS, Azure, and Spark. Data Analyst and Data Scientist positions rely more heavily on broadly used data management and analysis tools such as Excel and Tableau.
- Python is broadly applicable across all three roles. It is especially common in Data Scientist postings at 72% and Data Engineer postings at 65%.

## 2. Changes in Data Analyst Skill Demand

Next, I examined how frequently the leading skills appeared in Data Analyst job postings during 2023. I grouped the postings by month and compared the percentage of postings that requested each of the top five skills. This makes it possible to see how demand changed over the course of the year.


View my notebook with detailed steps here: [3_Skills_Trend](3_Skills_Trend.ipynb).

### Visualize Data

```python

from matplotlib.ticker import PercentFormatter

df_plot = df_DA_US_percent.iloc[:, :5]
sns.lineplot(data=df_plot, dashes=False, legend='full', palette='tab10')

plt.gca().yaxis.set_major_formatter(PercentFormatter(decimals=0))

plt.show()

```

### Results

![Trending Top Skills for Data Analysts in the US](3_Project\images\skill_trend_DA.png)  
*Line graph showing how the leading Data Analyst skills changed in demand during 2023.*

### Findings

- SQL remained the most consistently requested skill throughout the year, although its share of postings gradually declined.
- Excel showed a noticeable increase in demand beginning around September and had surpassed both Python and Tableau by the end of the year.
- Python and Tableau remained relatively stable overall, with some month-to-month variation. Power BI had lower demand than the other leading skills but showed a modest increase toward the end of the year.

## 3. Salary Differences Across Data Roles and Skills

For the salary portion of the project, I limited the data to U.S. positions and compared median annual salaries. I first examined salary distributions for common data roles, including Data Scientist, Data Engineer, and Data Analyst, to understand how compensation differed between positions.

View my notebook with detailed steps here: [4_Salary_Analysis](4_Salary_Analysis.ipynb).

#### Visualize Data 

```python
sns.boxplot(data=df_US_top6, x='salary_year_avg', y='job_title_short', order=job_order)

ticks_x = plt.FuncFormatter(lambda y, pos: f'${int(y/1000)}K')
plt.gca().xaxis.set_major_formatter(ticks_x)
plt.show()

```

#### Results

![Salary Distributions of Data Jobs in the US](3_Project\images\salary_poxplot.png)  
*Box plot visualizing the salary distributions for the top 6 data job titles.*

![Salary Distributions based on Data Analyst skills](3_Project\images\data_analyst_salary_by_skills.png)  
*Bar Chart empazsizing skill to salary relationship with in Data Analyst roles*

### Findings

- Salary ranges vary considerably between data job titles. Senior Data Scientist positions have the highest salary potential, reaching as high as $600K, which reflects the value associated with advanced skills and experience.
- Senior Data Scientist and Senior Data Engineer positions contain a substantial number of higher-end salary outliers. Data Analyst positions are more consistent overall and have fewer extreme salary values.
- Median compensation generally rises with greater seniority and specialization. Senior Data Scientist and Senior Data Engineer positions have higher median salaries and greater variation in typical compensation than the less senior roles.

### Highest-Paid and Most In-Demand Data Analyst Skills

I then narrowed the salary analysis to Data Analyst positions and compared skill demand with median salary. The purpose of this comparison was to identify skills that appear frequently in job postings while also being associated with higher compensation.


#### Visualize Data

```python
from matplotlib.ticker import PercentFormatter

# Create a scatter plot
scatter = sns.scatterplot(
    data=df_DA_skills_tech_high_demand,
    x='skill_percent',
    y='median_salary',
    hue='technology',  # Color by technology
    palette='bright',  # Use a bright palette for distinct colors
    legend='full'  # Ensure the legend is shown
)
plt.show()

```

#### Results

![Most Optimal Skills for Data Analysts in the US with Coloring by Technology](images/Most_Optimal_Skills_for_Data_Analysts_in_the_US_with_Coloring_by_Technology.png)  
*Scatter plot comparing skill demand and median salary for Data Analyst skills, grouped by technology category.*

### Findings

- Programming-related skills tend to cluster toward higher salary levels compared with several other categories, suggesting that programming expertise can provide a stronger salary advantage within data analytics.
- Database technologies such as Oracle and SQL Server are associated with some of the highest salaries among the Data Analyst skills examined. This indicates that database and data-management expertise can be highly valued in the market.
- Analyst tools such as Tableau and Power BI appear frequently in job postings while also showing competitive salaries. This reinforces the importance of visualization and data-analysis tools across a range of data-related tasks.

# What I Learned

This project strengthened my understanding of the data analyst job market while also giving me more experience working with Python for data analysis and visualization.

- **Using Python for Analysis:** I gained more experience using Pandas for data manipulation and Matplotlib and Seaborn for creating visualizations. These tools made it easier to work through a large dataset and communicate the results.
- **Importance of Data Preparation:** The analysis reinforced how important it is to clean and organize data before drawing conclusions from it. Missing or inconsistent values can affect the quality of the final results.
- **Connecting Skills to the Job Market:** Comparing skill demand, salary, and job availability showed how these factors can be considered together when deciding which technical skills may be worth developing.

# Overall Insights

Several broader observations came out of the analysis:

- **Demand and Compensation:** The analysis shows that some specialized or advanced skills are associated with higher salaries. Python and Oracle are examples of skills that appeared in this higher-value portion of the analysis.
- **Changing Skill Demand:** Skill demand is not completely static. The changes observed throughout 2023 demonstrate why it is useful to monitor job-posting trends when evaluating the skills employers want.
- **Choosing Skills Strategically:** Looking at both demand and salary provides a more useful picture than considering either measure by itself. Skills that perform well on both measures may be particularly valuable for someone planning a career in data analytics.

# Challenges

The project also presented several challenges that helped improve my understanding of the analysis process:

- **Inconsistent Data:** Missing and inconsistent entries required attention during the data-cleaning process so that they would not undermine the analysis.
- **Data Visualization:** Turning large amounts of information into charts that are easy to interpret required careful consideration of how the data should be presented.
- **Scope of the Analysis:** The project covered several different aspects of the data job market. Maintaining enough detail in each section while keeping the overall project focused required balancing breadth with depth.

# Conclusion

The analysis provides a snapshot of the U.S. data analyst job market and highlights several skills and trends that stand out in the dataset. SQL, Python, Excel, Tableau, Power BI, and database technologies each play different roles in the market, while salary levels vary substantially by position, seniority, and specialization.

More broadly, the project demonstrates the value of combining job-posting frequency with salary information when evaluating technical skills. Because the data job market changes over time, continuing to analyze new job postings can help identify shifts in employer demand and provide a stronger basis for future career and skill-development decisions.
