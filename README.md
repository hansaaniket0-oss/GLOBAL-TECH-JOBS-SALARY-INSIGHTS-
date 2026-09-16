Global Data & Tech Job Market Analysis (2023)

An end-to-end data analytics project on 32,671 global data/tech job postings from 2023 — cleaned and transformed with pandas, modeled and visualized in an interactive Power BI dashboard.
![Global Data & Tech Job Market Analysis Dashboard](https://github.com/user-attachments/assets/a336e303-5c5a-48e9-bb60-46febd15395e)

📌 Project Overview

This project analyzes a real-world dataset of data and tech job postings (Data Analyst, Data Scientist, Data Engineer, Business Analyst, Cloud Engineer, and more) to answer:

What skills are most in demand, and which pay the most?
How does salary vary by job schedule type (Full-time, Contractor, Internship, etc.)?
Are there seasonal trends in hiring and pay throughout the year?
Which skills combine strong demand with strong pay — and which are common but lower-paying?
🗂️ Dataset
Source: raw job postings dataset, ~32,672 rows before cleaning
Fields: job title, company, location, country, salary (annual/hourly), schedule type, remote-work flag, degree requirement, posted date, and required skills (stored as a list per posting)
Coverage: global postings, all dated within 2023
🛠️ Tools & Tech Stack
Stage	Tool
Data cleaning & transformation	Python (pandas, ast)
Dashboarding & visualization	Power BI (Power Query, DAX, data modeling)
🧹 Data Cleaning (pandas)

Key steps performed in clean_jobs_simple.py:

Removed exact duplicate rows
Consolidated salary_year_avg and salary_hour_avg (which are complementary, split by pay type) into a single salary_year_est column, converting hourly rates to an annual estimate (× 2,080 standard work hours/year)
Parsed the job_skills column — stored as a stringified Python list (e.g. "['sql', 'python']") — into a real list using ast.literal_eval()
Deduplicated skills within each job's own list (some postings listed the same skill twice, e.g. ['sql', 'sas', 'sas']), which was silently inflating skill-demand counts
Simplified messy combined schedule-type values (e.g. "Full-time and Part-time") into a single primary category
Split posting dates into year-month and day-of-week fields for time-based analysis
Exploded the cleaned data into two output tables:
cleaned_jobs.csv — one row per job posting
job_skills_exploded.csv — one row per (job, skill) pair, joined back via job_id
📊 Data Model (Power BI)

The two tables above are connected in a simple star-schema-style model:

cleaned_jobs (1) ────< job_skills_exploded (many)
      job_id                    job_id
Relationship: one-to-many on job_id
Cross-filter direction: Both — required so filtering by skill (on the "many" side) correctly filters salary and job-count measures defined on cleaned_jobs (the "1" side)
Key DAX Measures
dax
Job Count = DISTINCTCOUNT(cleaned_jobs[job_id])
Median Salary = MEDIAN(cleaned_jobs[salary_year_est])
Total Jobs per Skill = COUNTROWS(job_skills_exploded)
Total Unique Skills Tracked = DISTINCTCOUNT(job_skills_exploded[skill])

Median (not mean) is used throughout to reduce the influence of high-end salary outliers (dataset max ≈ $960,000).

📈 Dashboard Features
KPI cards — total postings, median salary, unique skills tracked
Top Skills table — job count & median salary per skill, sorted by demand
Month-wise trend chart — median salary & job count by month (combo chart, dual axis)
Job schedule comparison chart — median salary & job count by Full-time/Contractor/Internship/etc.
Top Skills priority chart — Top 15 skills by demand, comparing salary vs. job count
Slicers — job title, country, schedule type, posted date range
🔍 Key Findings
SQL and Python dominate demand, appearing in ~57% and ~54% of postings respectively — far ahead of the next tier (Tableau, R, AWS)
Demand and pay don't always align — specialized skills like Spark ($135,000 median) and Scala ($135,200 median) out-earn higher-demand skills like SQL ($115,000) and Excel ($88,400)
Full-time postings dominate the market (~83%) and pay close to Contractor roles, while Internships and Part-time roles pay substantially less
Seasonal pattern: postings peaked in August, dipped in November; median salary trended higher in the first half of 2023 than the second half
