## GitView Sample Queries

Below are sample queries for GitView's custom reports and dashboard feature at [GitView.com](https://gitview.com)

### Impact by contributor last 12 Months

```sql
SELECT date_trunc('month', authored_at) as Month,
        contributor_parents.name as Name,
        SUM(commits.impact)
    FROM commits
    INNER JOIN contributors ON commits.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where authored_at > date_trunc('month', current_date - interval '12' month)
    GROUP BY Month, contributor_parents.id
```

### Deploys in last 6 months

```sql
SELECT date_trunc('month', external_created_at) as Month, COUNT(*) FROM pull_requests
    where external_created_at > date_trunc('month', current_date - interval '6' month)
    AND title iLIKE ANY(ARRAY['%release%', '%deploy%'])
    GROUP BY Month
```

### PRs Merged in last 6 months By Contributor

```sql
SELECT date_trunc('month', external_merged_at) as Month,
        contributor_parents.name as ContributorName,
        COUNT(pull_requests)
    FROM pull_requests
    INNER JOIN contributors ON pull_requests.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where external_merged_at > date_trunc('month', current_date - interval '6' month)
    GROUP BY Month, ContributorName, contributor_parents.id
```

### Comments Added Last 6 Months By Contributor

```sql
SELECT date_trunc('month', comments.external_created_at) as Month,
        contributor_parents.name as Name,
        COUNT(*)
    FROM comments
    INNER JOIN contributors ON comments.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where comments.external_created_at > date_trunc('month', current_date - interval '6' month)
    GROUP BY Month, contributor_parents.id
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




### Issues Closed Last 12 Months Grouped By Assignee

```sql
SELECT date_trunc('month', external_closed_at) as Month,
        contributor_parents.name AS Name,
        COUNT(issues)
    FROM issues
    INNER JOIN issue_assignees ON issue_assignees.issue_id=issues.id
    INNER JOIN contributors ON issue_assignees.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where external_closed_at > date_trunc('month', current_date - interval '12' month)
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

### Comments last 6 months

```sql
SELECT date_trunc('month', comments.external_created_at) as Month,
        COUNT(*)
    FROM comments
    where comments.external_created_at > date_trunc('month', current_date - interval '12' month)
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
SELECT date_trunc('month', authored_at) as Month, SUM(commits.number_of_added_lines_not_filtered_out) - SUM(commits.number_of_added_test_lines_not_filtered_out) as NonTestLines, SUM(commits.number_of_added_test_lines_not_filtered_out) as TestLines FROM commits
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

### Pull Request Iterations last 12 months(commits after comments on p.r.s)

```sql
SELECT date_trunc('month', commits.authored_at) as Month,
        COUNT(*)
    FROM pull_request_commits
    INNER JOIN  commits ON commits.id = pull_request_commits.commit_id
    where commits.authored_at > date_trunc('month', current_date - interval '12' month)
    AND pull_request_commits.is_pr_iteration=true
    GROUP BY Month
```

### Comments that let to iteration last 12 months (comments that came before a commit on a p.r.)

```sql
SELECT date_trunc('month', comments.external_created_at) as Month,
        COUNT(*)
    FROM comments
    where comments.external_created_at > date_trunc('month', current_date - interval '12' month)
    AND comments.did_lead_to_pr_iteration=true
    GROUP BY Month
```

### Lines Added By Contributor w/ Line and File Filters Applied

*Note the line filters can be applied in repository settings*


```sql
SELECT date_trunc('month', authored_at) as Month,
        contributor_parents.name as ContributorName,
        SUM(commits.number_of_added_lines_not_filtered_out)
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


### Clean Impact 

Impact produced with the least churn involved

```sql
SELECT  contributor_parents.name,
 SUM(commits.impact) * (1 - (SUM(commits.churn) / NULLIF(SUM(commits.impact),0))) as clean_impact,
 (SUM(commits.churn) / NULLIF(SUM(commits.impact),0)) as churn_ratio,
 SUM(commits.impact) as impact
    FROM commits
    INNER JOIN contributors ON commits.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where commits.authored_at > {start_time} AND commits.authored_at < {end_time}
    AND commits.is_merged=true
    GROUP BY contributor_parents.name
    ORDER BY clean_impact desc nulls last
```

### PR Effeciency by Developer (PRs Merged / PR Iterations)

```sql
SELECT contributor_parents.name as contributor_name,
    (COUNT(distinct pull_requests.id) * 1.0 / NULLIF(SUM(CASE WHEN pull_request_commits.is_pr_iteration THEN 1 ELSE 0 END), 0)) as efficiency_ratio,
    COUNT(distinct pull_requests.id) as prs_merged,
    SUM(CASE WHEN pull_request_commits.is_pr_iteration THEN 1 ELSE 0 END) AS iterations
    FROM pull_requests
    LEFT OUTER JOIN pull_request_commits ON pull_request_commits.pull_request_id=pull_requests.id
    INNER JOIN contributors ON pull_requests.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where pull_requests.external_created_at > '2022-01-01'
    AND pull_requests.external_merged_at IS NOT NULL
    GROUP BY contributor_name
    ORDER BY efficiency_ratio desc nulls last
```


### Custom Commits Stat Sheet

```sql
SELECT contributor_parents.name as contributor_name,
    ROUND(SUM(commits.impact)) as impact,
    SUM(commits.new_work) as new_work,
    SUM(commits.simple_work) as simple_work,
    SUM(commits.churn) as churn,
    SUM(commits.legacy_refactor) as legacy_refactor,
    SUM(commits.help_others) as help_others,
    SUM(commits.ignored_lines) as ignored_lines,
    SUM(commits.isolated_removals) as isolated_removals,
    SUM(commits.number_of_added_lines_total) as total_lines_added,
    SUM(commits.number_of_added_test_lines_total) as test_lines_added
    FROM commits
    INNER JOIN contributors ON commits.contributor_id=contributors.id
    INNER JOIN contributor_parents ON contributor_parents.id=contributors.contributor_parent_id
    where commits.authored_at > '2022-04-23'
    GROUP BY contributor_name
    ORDER BY impact desc nulls last
```
