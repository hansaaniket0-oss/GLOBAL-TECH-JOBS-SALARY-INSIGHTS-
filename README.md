Global Data & Tech Job Market Analysis (2023)

An end-to-end data analytics project analyzing 32,671 data and tech job postings from 2023 — cleaned and transformed in Python (pandas), modeled and visualized in an interactive Power BI dashboard.

The goal: understand which skills are most in demand, how they relate to salary, and how job schedule type and posting timing shape the data/tech job market.

Dashboard Preview


Tech Stack
Tool	Purpose
Python (pandas)	Data cleaning, parsing, transformation
Power Query	CSV import and type handling
Power BI	Data modeling (star schema), DAX measures, interactive dashboard
Excel / CSV	Raw data source and cleaned output format
Dataset
Size: 32,671 job postings (after removing duplicates)
Period: Full calendar year 2023
Scope: Data/tech roles — Data Analyst, Data Scientist, Data Engineer, Business Analyst, Cloud Engineer, and more
Key fields: job title, company, location, country, salary (annual or hourly), schedule type, remote-work flag, posted date, required skills
Data Cleaning (pandas)

The raw dataset had several real-world quality issues. Key steps in clean_jobs_simple.py:

Removed duplicate rows — one exact duplicate found via drop_duplicates()
Added a stable job_id — primary key needed once skills are split into their own table
Cleaned job_via → job_platform — stripped the "via " prefix ("via LinkedIn" → "LinkedIn")
Filled missing text values — job_location and job_schedule_type set to "Not specified"
Simplified schedule types — job_schedule_type held combined values like "Full-time and Part-time"; created job_schedule_primary with just the first listed type for clean grouping
Consolidated salary into one column — salary_year_avg and salary_hour_avg are complementary (each posting has exactly one, based on salary_rate). Combined into salary_year_est, converting hourly pay to an annual estimate (hourly × 2,080 standard work hours/year)
Split posted date — added job_posted_month and job_posted_day for time-based filtering in Power BI
Parsed the skills column — job_skills was stored as a stringified Python list (e.g. "['sql', 'python', 'aws']"). Parsed with ast.literal_eval(), which is safer than eval() since it only allows literal data structures and won't execute arbitrary code
Deduplicated skills within each posting — some jobs listed the same skill twice (e.g. ['python', 'sql', 'sas', 'sas', 'oracle']), inflating demand counts. Fixed with list(dict.fromkeys(skills_list)), preserving order while removing repeats
Data Modeling (Star Schema)

Rather than one wide flat table, the cleaned data is split into two related tables:

Table	Grain	Rows
cleaned_jobs.csv	One row per job posting	32,671
job_skills_exploded.csv	One row per job–skill pair	~164,000

Why split? A posting can list 18+ skills. Exploding skills into the main table would duplicate every job-level field (salary, company, location) once per skill — breaking any job-level aggregate, since each job would be counted once per skill instead of once overall.

Relationship: One-to-many, cleaned_jobs[job_id] → job_skills_exploded[job_id], with cross-filter direction set to Both so filtering by skill (the "many" side) correctly filters salary measures living on cleaned_jobs (the "one" side).

DAX Measures
dax
Job Count = DISTINCTCOUNT(cleaned_jobs[job_id])

Median Salary = MEDIAN(cleaned_jobs[salary_year_est])

Total Jobs per Skill = COUNTROWS(job_skills_exploded)

Total Unique Skills Tracked = DISTINCTCOUNT(job_skills_exploded[skill])

Why median over average? Salaries range from $15,000 to $960,000. A mean would be pulled sharply upward by a handful of executive-level outliers; median better represents what a typical posting actually pays.

Dashboard Features
KPI cards — total postings, median salary, postings with skills listed, unique skills tracked (221)
Top Skills table — job count and median salary per skill, sorted by demand
Month-wise trend — combo chart plotting median salary (columns) against job count (line) across 2023
Job schedule comparison — salary and volume by Full-time / Contractor / Part-time / Internship / Temp work
Top Skills priority chart — demand vs. pay for the top 15 skills, via a Top N filter
Interactive slicers — job title, country, schedule type, and posted-date range, cross-filtering every visual
Key Insights

1. SQL and Python dominate demand They appear in ~57% and ~54% of all postings (18,499 and 17,689) — far ahead of the next tier (Tableau ~7,045, R ~6,929, AWS ~6,844).

2. Demand and pay don't always align

Skill	Postings	Median Salary
Scala	2,505	$135,200
Spark	5,294	$135,000
Snowflake	3,492	$130,000
SQL	18,499	$115,000
Excel	6,264	$88,400

Excel and Power BI are high-demand but lower-paying — common baseline tools for junior-to-mid roles rather than specialized senior skills.

3. Schedule type strongly predicts pay

Schedule	Postings	Median Salary
Contractor	4,801	$114,400
Full-time	27,273	$111,041
Part-time	280	$72,800
Internship	96	$54,020

Full-time postings make up ~83% of the market.

4. Seasonal hiring pattern Postings peaked in August (3,554) and bottomed out in November (1,963). Median salary held steady around $111K–$115K from January–July, then dipped to $104K–$107K from August onward.

Repository Structure
├── clean_jobs_simple.py          # pandas cleaning script
├── data/
│   ├── raw_job_analysis.xlsx     # raw source data
│   ├── cleaned_jobs.csv          # cleaned job-level table
│   └── job_skills_exploded.csv   # job–skill bridge table
├── images/
│   └── dashboard.png             # dashboard screenshot
├── job_market_dashboard.pbix     # Power BI report file
└── README.md
How to Run
bash
