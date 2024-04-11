# U.S. Men's National Team vs. World in Soccer (1872 - Jan 2024)

## Table of Content
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Preparation](#data-preparation)
  - [SQL](#sql)
  - [R](#r)
- [Tableau](#tableau)

## Project Overview
The goal of this project is to create a world map showing the results of the U.S. Men's National Team (soccer) against every country it has played. The data is from 1872 through January 2024.

## Data Source
The source file is ['results.csv'](results.csv), which was created by Mart Jürisoo on obtained [Kaggle](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017). 


## Tools
- SQL (pgAdmin 4)
- R Studio
- Tableau

## Data Preparation
### SQL
SQL was used to narrow down the data set to only matches played by the United States. Then, the results (wins, draws, losses) were grouped by opponent and summed.

```sql
CREATE TEMPORARY TABLE usa_results AS
SELECT date,
    home_team,
    away_team,
    home_score,
    away_score,
    tournament,
        CASE
            WHEN home_score > away_score AND home_team::text = 'United States'::text THEN 'W'::text
            WHEN home_score = away_score AND home_team::text = 'United States'::text THEN 'D'::text
            WHEN home_score < away_score AND home_team::text = 'United States'::text THEN 'L'::text
            WHEN away_score > home_score AND away_team::text = 'United States'::text THEN 'W'::text
            WHEN away_score = home_score AND away_team::text = 'United States'::text THEN 'D'::text
            WHEN away_score < home_score AND away_team::text = 'United States'::text THEN 'L'::text
            ELSE NULL::text
        END AS result
   FROM results
  WHERE home_team::text = 'United States'::text OR away_team::text = 'United States'::text;
  
CREATE TEMPORARY TABLE usa_opponents AS
SELECT *,
CASE
	  WHEN home_team::text = 'United States'::text THEN away_team
    WHEN away_team::text = 'United States'::text THEN home_team
    ELSE NULL::character varying
END AS opponent
FROM usa_results;

SELECT opponent, result, COUNT(*) AS count
FROM usa_opponents
GROUP BY opponent, result
ORDER BY opponent, (
	CASE
        WHEN result = 'W'::text THEN 1
        WHEN result = 'D'::text THEN 2
        WHEN result = 'L'::text THEN 3
        ELSE NULL::integer
    END)
```

### R
This data was expored to R Studio, where it was easier to pivot the table and replace null values with zero. Percentages were calculated for wins, draws, and losses.

```R
library(tidyr)
library(dplyr)

us_vs_world <- read.csv("us_vs_world.csv")

wide_data <- pivot_wider(us_vs_world,
                         names_from = result,
                         values_from = count)

cleaned_data <- wide_data %>% replace(is.na(.), 0)

us_world <- cleaned_data %>%
  mutate(total = W+D+L) %>%
  mutate(win_percent = round((W/total)*100, 2), 
         draw_percent = round((D/total)*100, 2), 
         loss_percent = round((L/total)*100, 2))
```

## Tableau
This finalized data set was uploaded into Tableau and [visualized as an interactive world map](https://public.tableau.com/app/profile/alex.berezow/viz/USMNTvsWorld/Dashboard1).
