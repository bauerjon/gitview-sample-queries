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
    GROUP BY Month, Name, contributor_parents.id
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
