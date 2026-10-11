# Kickstarter Crowdfunding Dashboard (Tableau)

An interactive Tableau dashboard analysing Kickstarter projects: how many were created, where and in which categories, how much successful projects raised, and what share of projects succeed.

## Overview

Kickstarter is an all-or-nothing crowdfunding platform. Creators set a funding goal and a deadline, and backers are charged only if the goal is reached. This dashboard answers the questions in `Crowdfunding_Questionnaires.txt` using the project data and three lookup tables. The background deck is `Kickstarter_Overview.pdf`.

## Files

| File | Description |
|---|---|
| `Kickstart_Dashboard.twbx` | Tableau packaged workbook (dashboard, worksheets, calculated fields, data) |
| `Crowdfunding_projects_1.xlsx` | Main fact table: one row per project |
| `crowdfunding_Category.xlsx` | Category lookup (`id`, `name`, `parent_id`, `position`) |
| `Crowdfunding_Creator.xlsx` | Creator lookup (`id`, `name`, `chosen_currency`) |
| `Crowdfunding_Location.xlsx` | Location lookup (`id`, `displayable_name`, `type`, `name`, `state`, `short_name`, `is_root`, `country`, `localized_name`) |
| `Crowdfunding_Questionnaires.txt` | The eight analysis tasks the dashboard answers |
| `Kickstarter_Overview.pdf` | Background deck on crowdfunding and Kickstarter |
| `new_working_file_AutoRecovered__1.xlsx` | Excel working file used while preparing the data |

## Data Model

The workbook uses Tableau's relationship model, with the projects table in the centre:

```
Crowdfunding_projects_1 (Sheet1)
   |-- category_id  = Category.id
   |-- location_id  = Dim_Location.id
   |-- creator_id   = Dim_creator.id
```

Key columns in the projects table: `id`, `state`, `name`, `country`, `creator_id`, `location_id`, `category_id`, `created_at`, `launched_at`, `deadline`, `state_changed_at`, `successful_at`, `goal`, `pledged`, `currency`, `usd_pledged`, `static_usd_rate`, `backers_count`, `spotlight`, `staff_pick`.

## Calculated Fields

Dates in the source are Unix epoch seconds, so they are converted in Tableau (Questionnaire Q1). The Q2 calendar is also built from calculated fields instead of a separate table.

| Field | Logic |
|---|---|
| `created_date`, `Launched_date`, `Deadline_date`, `State_Changed_date` | `DATEADD('second', <epoch field>, #1970-01-01#)` |
| `Year`, `Month_No.`, `Month_Name`, `Quarter`, `Year_Month`, `Weekday_No.`, `Weekday_Name` | Derived from `created_date`. Quarter is `Q1`-`Q4` and Year_Month is `YYYY-MMM` |
| `Financial_Month` | April = FM1 ... March = FM12 |
| `Financial_Quarter` | April-June = FQ-1, July-September = FQ-2, and so on |
| `Goal_Amount` | `goal * static_usd_rate` (returns NULL if either is missing) |
| `Total_Project` | `COUNT([id])` |
| `Successful Project` | `COUNTD(IF [state]="successful" THEN [name] END)` |
| `Success %` | Successful projects / total projects |
| `Successful Amount Raised` | `SUM(IF [state]="successful" THEN [usd_pledged] END)` |
| `No. of Backers` | `SUM(IF [state]="successful" THEN [backers_count] END)` |
| `Days to success` | Days between two date fields for successful projects only |
| `Avg No. of Days` | Average of `Days to success` |

## Dashboard Contents

`Dashboard 1` is a fixed-size 1640 x 910 layout, plus a scrolling phone layout. It is built from these worksheets:

## Dashboard 
https://public.tableau.com/app/profile/pratham.vishwakarma/viz/KickStarterCrowdFundingDashBoard/Dashboard1


| Section | Worksheets | Questionnaire task |
|---|---|---|
| Headline KPIs | `Total Project`, `Success %`, `S. Amount raised`, `S. Backer Count`, `No. of backers`, `Static USD Rate` | Q4, Q5, Q6, Q8 |
| Projects by outcome | `TP Outcome` | Q5 |
| Projects by location | `TP Location` (map) | Q5 |
| Projects by category | `TP category`, `Top 10 category` | Q5, Q7 |
| Projects over time | `TP Created by year`, `TP created by Quarter`, `TP created by Month` | Q5 |
| Success rate analysis | `Success %`, `Q2.` | Q8 |
| Filters | `Filters` (interactive filters, including project state) | All |

## How to Use

1. Open `Kickstart_Dashboard.twbx` in Tableau Desktop, or in Tableau Public (the workbook was created on Tableau Public build 2025.3).
2. Go to `Dashboard 1`. Use the filters to narrow by outcome and date.
3. Click any bar, map location or category to cross-filter the other charts.

If Tableau reports missing data sources, point them at the four Excel files above. The packaged workbook already contains the data.

## Questionnaire Coverage

| # | Task | Where it is answered |
|---|---|---|
| 1 | Convert epoch dates to natural dates | Date calculated fields |
| 2 | Build a calendar table | Calendar calculated fields, including financial month and quarter |
| 3 | Build the data model | Relationships between the four Excel files |
| 4 | Goal amount in USD | `Goal_Amount` |
| 5 | Projects by outcome, location, category, year, quarter, month | `TP ...` worksheets |
| 6 | Successful projects: amount raised, backers, average days | `S. Amount raised`, `S. Backer Count`, `No. of backers`, `Avg No. of Days` |
| 7 | Top successful projects by backers and amount | `Top 10 category` and related views |
| 8 | Success percentage overall, by category, time and goal range | `Success %`, `Q2.` |

## Things to Check

I read the workbook's structure and formulas, not the rendered dashboard, so please verify these before sharing:

- **`Successful Project` counts distinct project names.** Two successful projects with the same name would be counted once. Using `COUNTD([id])` is safer.
- **The Q7 top projects view.** The worksheet I found is "Top 10 category". If you also have a top-projects-by-backers and by-amount view, mention it in the table above.
- **Goal range analysis (Q8).** I could not confirm which worksheet holds the goal-range buckets, so make sure it is included.
- **Currency.** Amounts raised use `usd_pledged`, which is already in USD. Goal amounts use `goal * static_usd_rate`. Do not mix the two when comparing raised vs goal.
- **Reference benchmark.** Kickstarter's own published overall success rate in the overview deck is 36.02%. Your dataset is a sample, so the figures will differ, but a large gap is worth investigating.
