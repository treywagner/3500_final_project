# 3500_final_project
Repository for Data Mining final project

---

1. Import Data
Load both datasets using pandas:

NFL_League_Master.csv — contains team-level season stats

passing_cleaned.csv — contains quarterback passing stats

python
Copy
Edit
import pandas as pd
nfl_data = pd.read_csv('NFL_League_Master.csv', index_col=0)
qb_data = pd.read_csv('passing_cleaned.csv', index_col=0)

---

2. Drop Unaligned Seasons
Remove the 2000 season from nfl_data (team data)

Remove 2022 and 2023 from qb_data (QB data) to ensure consistent year overlap

python
Copy
Edit
nfl_data = nfl_data[nfl_data['Year'] != 2000]
qb_data = qb_data[~qb_data['Year'].isin([2022, 2023])]

---

3. Normalize Team Names
QB data uses 3-letter team abbreviations; team data uses full team names

Use a mapping dictionary to convert all team abbreviations in qb_data to match full team names in nfl_data

python
Copy
Edit
team_abbr_to_full = {
    'ARI': 'Arizona Cardinals', 'ATL': 'Atlanta Falcons', ..., 'WAS': 'Washington Commanders'
}
qb_data['Team'] = qb_data['Team'].map(team_abbr_to_full)

---

4. Remove Multi-Team Rows
Drop QB entries labeled '2TM', '3TM', etc., as they represent players who played for multiple teams in one season and cannot be matched cleanly to a single team

python
Copy
Edit
qb_data = qb_data[~qb_data['Team'].isin(['2TM', '3TM', '4TM'])]

---

5. Keep Only Primary QBs
Sort quarterbacks by Year, Team, and descending Att (pass attempts)

Keep only the top QB per team per year (assumed to be the starter)

python
Copy
Edit
qb_data = qb_data.sort_values(by=['Year', 'Team', 'Att'], ascending=[True, True, False])
qb_data = qb_data.drop_duplicates(subset=['Year', 'Team'], keep='first')

---

6. Merge Datasets
Merge qb_data and nfl_data on Team and Year

python
Copy
Edit
merged_df = pd.merge(qb_data, nfl_data, on=['Team', 'Year'], how='inner')

---

7. Remove Duplicates and Erroneous Data
Check for and remove any rows with duplicate QB stat lines across years

Specifically remove 2007 data after discovering it exactly matched 2023 data

python
Copy
Edit
qb_data = qb_data[qb_data['Year'] != 2007]

---

8. Create Binary Classification Column
Add a new column Winning_Season:

1 if Win/Loss% is >= 0.500

0 otherwise

python
Copy
Edit
qb_data['Winning_Season'] = (qb_data['Win/Loss%'] >= 0.500).astype(int)

---

9. Export Final Dataset
Save the cleaned and merged data for modeling in Orange or other tools

python
Copy
Edit
qb_data.to_csv('merged_nfl_qb_data.csv', index=False)