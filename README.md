## GitView Sample Queries

Below are sample queries for GitView's custom reports and dashboard feature at [GitView.com](https://gitview.com)

### Impact by contributor last 6 Months

```sql
SELECT date_trunc('month', authored_at) as Month,
        (SELECT name
            FROM contributors
            WHERE contributor_parents.id=contributors.contributor_parent_id LIMIT 1)
            AS Name,
        SUM(commits.impact)
    FROM commits
    INNER JOIN contributors ON commits.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where authored_at > date_trunc('month', current_date - interval '6' month)
    GROUP BY Month, contributor_parents.id
```

### Deploys in last 6 months

```sql
SELECT date_trunc('month', external_created_at) as Month, COUNT(*) FROM pull_requests
    where external_created_at > date_trunc('month', current_date - interval '6' month)
    AND title iLIKE ANY(ARRAY['%release%', '%deploy%'])
    GROUP BY Month
```

### PRs Merged in last 6 months

```sql
SELECT date_trunc('month', external_merged_at) as Month,
        (SELECT name
            FROM contributors
            WHERE contributor_parents.id=contributors.contributor_parent_id LIMIT 1)
            AS Name,
        COUNT(pull_requests)
    FROM pull_requests
    INNER JOIN contributors ON pull_requests.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where external_merged_at > date_trunc('month', current_date - interval '6' month)
    GROUP BY Month, Name, contributor_parents.id
```

### Bug fixes last 6 months

```sql
SELECT date_trunc('month', external_created_at) as MONTH, COUNT(*) FROM pull_requests
    where external_created_at > date_trunc('month', current_date - interval '6' month)
    AND (title iLIKE ANY(ARRAY['%bug%', '%fix%']) OR body iLIKE ANY(ARRAY['%bug%', '%fix%']))
    GROUP BY MONTH
```


### Merged Pull Requests Over Last 6 Months by week

```sql
SELECT date_trunc('week', external_created_at) as Week, COUNT(*) FROM pull_requests
    where external_created_at > date_trunc('month', current_date - interval '6' month)
    AND external_merged_at IS NOT NULL
    GROUP BY Week
```

### Issues Created Last 6 Months By Contributor

```sql
SELECT date_trunc('month', external_created_at) as Month,
        (SELECT name
            FROM contributors
            WHERE contributor_parents.id=contributors.contributor_parent_id LIMIT 1)
            AS Name,
        COUNT(issues)
    FROM issues
    INNER JOIN contributors ON issues.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where external_created_at > date_trunc('month', current_date - interval '6' month)
    GROUP BY Month, contributor_parents.id
```


### Issues Closed Last 6 Months Grouped By Assignee

```sql
SELECT date_trunc('month', external_closed_at) as Month,
        (SELECT name
            FROM contributors
            WHERE contributor_parents.id=contributors.contributor_parent_id LIMIT 1)
            AS Name,
        COUNT(issues)
    FROM issues
    INNER JOIN issue_assignees ON issue_assignees.issue_id=issues.id
    INNER JOIN contributors ON issue_assignees.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where external_closed_at > date_trunc('month', current_date - interval '6' month)
    GROUP BY Month, contributor_parents.id
```

### Issues created with title, body, or label with word "bug" last 6 months

```sql
SELECT date_trunc('month', external_created_at) as Month,
        COUNT(issues)
    FROM issues
    INNER JOIN issue_labels ON issue_labels.issue_id = issues.id
    INNER JOIN labels ON issue_labels.label_id = labels.id
    where external_created_at > date_trunc('month', current_date - interval '6' month)
    AND (labels.name ILIKE 'bug' OR issues.body ILIKE '%bug%' OR issues.title ILIKE '%bug%')
    GROUP BY Month
```

### Issues over time

```sql
SELECT date_trunc('month', external_created_at) as Month,
        COUNT(issues)
    FROM issues
    where external_created_at > date_trunc('month', current_date - interval '24' month)
    GROUP BY Month
```

### Pull Requests By Day of Week

```sql
SELECT  extract(isodow from external_created_at) || '-' || To_Char(external_created_at, 'DAY') as DayOfWeek,
        COUNT(*)
    FROM pull_requests
    INNER JOIN contributors ON pull_requests.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where external_created_at > {start_time} AND external_created_at < {end_time}
    GROUP BY DayOfWeek
```

### Automated Test Lines of Code vs Non Test Lines of Code (Merged Lines Only)

```sql
SELECT date_trunc('month', authored_at) as Month, SUM(commits.number_of_added_lines_filtered) - SUM(commits.number_of_added_test_lines_filtered) as NonTestLines, SUM(commits.number_of_added_test_lines_filtered) as TestLines FROM commits
    where authored_at > date_trunc('month', current_date - interval '12' month)
    AND commits.is_merged=true
    GROUP BY Month
```

### Impact last 12 months

```sql
SELECT date_trunc('month', authored_at) as Month,
        SUM(commits.impact)
    FROM commits
    where authored_at > date_trunc('month', current_date - interval '12' month)
    AND commits.is_merged=true
    GROUP BY Month
```

### Lines Added By Contributor w/ Line and File Filters Applied

*Note the line filters can be applied in repository settings*


```sql
SELECT date_trunc('month', authored_at) as Month,
        contributor_parents.name as ContributorName,
        SUM(commits.number_of_added_lines_filtered)
    FROM commits
    INNER JOIN contributors ON commits.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where authored_at > date_trunc('month', current_date - interval '12' month)
    AND commits.is_merged=true
    GROUP BY Month, ContributorName
```

### In depth commit analysis by contributor

```sql
SELECT commits.impact, commits.number_of_added_lines_total, commits.new_work, commits.simple_work, commits.churn, commits.help_others, commits.legacy_refactor, commits.ignored_lines, commits.number_of_removed_lines_total, number_of_added_lines_total, commits.authored_at, commits.sha
    FROM commits
    INNER JOIN contributors ON commits.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where authored_at > date_trunc('month', current_date - interval '12' month)
    AND contributor_parents.id='Contributor Parent Id'
    AND commits.is_merged=true
    ORDER BY commits.authored_at
```
