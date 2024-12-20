# Introduction
Dive into the data job market! Focusing on data analyst roles, this project explores top-paying job, in-demand skills, and where high demand meets high salary in data analytics.

SQL queries? Check them out here: [project_sql folder](/project_sql/)

# Background
Driven by a quest to navigate the data analyst job market more effectively, this project was born from a desire to pinpoint top-paid and in-demand skills, streamlining others work to find optimal jobs.

Data hails from [Luke's SQL Course](https://lukebarousse.com/sql). It's packed with insights on job titles, salaries, locations, and essential skills.

### The questions I wanted to answer through my SQL queries were:
1. What are the top-paying data analysts roles?
2. What are the skills required for these top-paying roles?
3. What are the most in-demand skills those roles?
4. What are the top skills based on salary for those roles?
5. What are the most optimal skills to learn?
	- Optimal: High Demand AND High Paying 

# Tools I Used
For my deep dive into the data analyst job market, I harnessed the power of several key tools:

- **SQL:** The backbone of my analysis, allowing me to query the database and unearth critical insights.
- **PostgreSQL:** The chosen database management system, ideal fo rhandling the job posting data.
- **Visual Studio Code:** My go-to for database management and executing SQL queries.
- **Git & GitHub:** Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.

# The Analysis
Each query for this project aimed at investigating specific aspects for the data analyst job market. Here's how I approached each question:

### 1. Top Paying Data Analyst Jobs
- Identify the top 10 highest-paying Data Analyst roles that are available remotely.
- Focuses on job postings with specified salaries (remove nulls).
- Why? Highlight the top-paying opportunities for Data Analysts, offering insights into employee options and location flexibility.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    name AS company_name,
    job_schedule_type,
    salary_year_avg,
    job_posted_date
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
```

### 2. What skills are required for the top-paying data analyst jobs? 
- Use the top 10 highest-paying Data Analyst jobs from first query
- Add the specific skills required for these roles
- Why? It provides a detailed look at which high-paying jobs demand certain skills,
    helping job seekers understand which skills to develop that align with top salaries.

```sql
WITH top_paying_jobs AS (
	SELECT 
		job_id, 
		job_title,
		salary_year_avg,
		job_posted_date,
		name AS company_name
	FROM
		job_postings_fact
	LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
	WHERE
		job_title_short = 'Data Analyst' AND
		job_location = 'Anywhere' AND
		salary_year_avg IS NOT NULL
	ORDER BY salary_year_avg DESC	
	LIMIT 10
)
SELECT
	top_paying_jobs.*,
	skills_dim.skills
FROM
	top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
	salary_year_avg DESC
```

### 3. What are the most in-demand skills for data analysts?
- Join job postings to inner join table similar to query 2
- Identify the top 5 in-demand skills for a data analyst.
- Focus on all job postings.
- Why? Retrieves the top 5 skills with the highest demand in the job market, 
    providing insights into the most valuable skills for job seekers.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS skill_demand
FROM job_postings_fact
INNER JOIN skills_job_dim ON skills_job_dim.job_id = job_postings_fact.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE 
    job_title_short = 'Data Analyst'
GROUP BY 
    skills
ORDER BY 
    skill_demand DESC
LIMIT 5;
```
### 4. What are the top skills based on salary?
- Look at the average salary associated with each skill for Data Analyst positions
- Focuses on roles with specified salaries, regardless of location
- Why? It reveals how different skills impact salary levels for Data Analysts and 
    helps identify the most financially rewarding skills to acquire or improve.

```sql
SELECT 
	skills,
	ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM
	job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
	job_title_short = 'Data Analyst'
	AND salary_year_avg IS NOT NULL
	AND job_work_from_home = True
GROUP BY
	skills
ORDER BY
	avg_salary DESC
LIMIT 25;
```

### 5. What are the most optimal skills to learn (high demand + high paying)?
- Identify skills in high demand and associated with high average salaries for Data Analyst roles
- Concentrates on remote positions with specified salaries
- Why? Targets skills that offer job security (high demand) and financial benefits (high salaries), 
    offering strategic insights for career development in data analysis

```sql
WITH skills_demand AS (
	SELECT 
		skills_dim.skill_id,
		skills_dim.skills,
		COUNT(skills_job_dim.job_id) AS demand_count
	FROM
		job_postings_fact
	INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
	WHERE
		job_title_short = 'Data Analyst' 
		AND salary_year_avg IS NOT NULL
		AND job_work_from_home = True
	GROUP BY
		skills_dim.skill_id
), average_salary AS (
	SELECT 
		skills_dim.skill_id,
		skills_dim.skills,
		ROUND(AVG(salary_year_avg), 0) AS avg_salary
	FROM
		job_postings_fact
	INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
	WHERE
		job_title_short = 'Data Analyst'
		AND salary_year_avg IS NOT NULL
		AND job_work_from_home = True
	GROUP BY
		skills_dim.skill_id
)
SELECT
	skills_dim.skill_id,
	skills_dim.skills,
	COUNT(skills_job_dim.job_id) AS demand_count,
	ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
	job_title_short = 'Data Analyst'
	AND salary_year_avg IS NOT NULL
	AND job_work_from_home = True
GROUP BY
	skills_dim.skill_id
HAVING
	COUNT(skills_job_dim.job_id) > 10
ORDER BY
	avg_salary DESC,
	demand_count DESC
LIMIT 25;
```

# What I Learned
Throughout this adventure, I have turbocharged my SQL toolkit with some serious firepower:

- **Complex Query Crafting:** Mastered the art of advanced SQL, merging tables like a pro and wielding WITH clauses for ninka-level temp table maneuvers.
- **Data Aggregation:** Got cozy with GROUP BY and turned aggregate functions like COUNT() and AVG() into my data-summarizing sidekicks.
- **Analytical Wizardry:** Leveled up my real-world puzzle-solving skills, turning questions into actionable, insightful SQL queries.

# Conclusions

### Insights
From the analysis, several general insights emerged:

1. **Top-Paying Data Analyst Jobs:** The highest-paying jobs for data analysts that allow remote work offer a wide range of salaries, the highest at $650,000!
2. **Skills for Top-Paying Jobs:** High-paying data analyst jobs require advanced proficiency in SQL, suggesting it's a crucial skill for earning a top salary.
3. **Most In-Demand Skills:** SQL is also the most demanded skill in the data analyst job market, thus making it essential for job seekers.
4. **Skills with Higher Salaries:** Specialized skills, such as SVN and Soldity, are associated with highest average salaries, indicating a premium on niche expertise.
5. **Optimal Skills for Job Market Value:** SQL leads in demand and offers for a higher average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value.

### Closing Thoughts

This project enhanced my SQL skills and provided valuable insights into the data analyst job market. The findings from the analysis serve as a guide to prioritizing skill development and job search efforts. Aspiring data analysts can better position themselves in a competitive job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of data analytics.
