# Simple Alert

A Databricks alert system for monitoring workspace costs and identifying high-spending SKUs across your organization.

## Overview

This alert monitors workspace spending in Databricks by analyzing billing data over a rolling 3-day window. It identifies the highest-cost SKU (Stock Keeping Unit) for each workspace on each day and sends a formatted email notification when usage data is available.

### Key Features

- **Automated Cost Monitoring**: Runs every 12 hours to track workspace spending
- **Top SKU Identification**: Pinpoints the highest-cost service for each workspace
- **Spend Threshold Filtering**: Only includes SKUs with daily spend exceeding $5
- **Percentage Breakdown**: Shows what percentage of workspace spending each top SKU represents
- **HTML Email Notifications**: Professionally formatted alerts with cost breakdowns

## Prerequisites

- Databricks workspace with access to `system.billing.usage` and `system.billing.list_prices` tables
- Permissions to create and manage alerts in Databricks
- Email notification system configured in Databricks

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/jneil17/simple_alert.git
   ```

2. Import the alert configuration into Databricks:
   - Navigate to your Databricks workspace
   - Click **Create**
   - Click **Git folder**
   - Paste the **HTTPS Github Link** for this repo [Green **< › Code** button above]

3. Configure notification settings:
   - Update alert recipients
   - Verify timezone settings (default: America/New_York)
   - Adjust schedule if needed

## Alert Configuration

### Schedule

The alert runs on the following schedule:
- **Cron Expression**: `22 0 8/12 * * ?`
- **Frequency**: Every 12 hours (8 AM and 8 PM)
- **Timezone**: America/New_York

### Data Window

- **Analysis Period**: Last 3 days from current date
- **Minimum SKU Spend**: $5 per day
- **Currency**: USD

### Trigger Condition

The alert triggers when `usage_date IS NOT NULL`, meaning it fires whenever there is usage data available in the specified time window.

## Query Logic

The alert query performs the following steps:

1. **Workspace Identification**: Identifies all distinct workspaces from billing data

2. **Date Filtering**: Filters usage data for the last 3 days

3. **Price Calculation**: Joins usage data with USD list prices to calculate costs

4. **SKU Aggregation**: Sums usage costs by date, workspace, and SKU

5. **Top SKU Selection**: Ranks SKUs by spend within each workspace/date combination and selects the highest

6. **Output Format**: Returns usage date, workspace ID, top SKU with percentage, and total daily cost

### Output Columns

- `usage_date`: The date of the usage
- `workspace`: Workspace identifier (prefixed with "id: ")
- `top_sku_percentage_usage`: Top SKU name with its percentage of workspace spend
- `total_usd`: Total daily workspace cost (rounded to nearest dollar)

## Email Alert Format

The alert sends a professionally styled HTML email with:

- **Alert Header**: Red banner indicating cost threshold triggered
- **Alert Details Table**: Shows alert name, status, and condition
- **Cost Breakdown Table**: Displays workspace costs with top SKU for each day
- **Call to Action**: Direct link to view alert in Databricks
- **Warning Banner**: Yellow notification about spending threshold

## Customization

### Adjusting the Time Window

To change the analysis period, modify the date filter in the query:

```sql
where u.usage_date between current_date() - interval 3 day and current_date()
```

Change `3 day` to your desired number of days.

### Modifying Spend Threshold

To adjust the minimum SKU spend filter:

```sql
having sum(usage_usd) > 5
```

Change `5` to your desired threshold amount.

### Updating Schedule

To change when the alert runs, modify the cron expression:

```json
"quartz_cron_schedule": "22 0 8/12 * * ?"
```

[Cron expression reference](https://docs.oracle.com/cd/E12058_01/doc/doc.1014/e12030/cron_expressions.htm)

### Customizing Email Style

The HTML template in `custom_description_lines` can be modified to adjust:
- Colors and branding
- Table formatting
- Additional sections or metrics
- Font styles and sizes

## Use Cases

This alert is useful for:

- **Cost Governance**: Track daily spending across all workspaces
- **Budget Management**: Identify workspaces approaching spending limits
- **Resource Optimization**: Pinpoint high-cost services for optimization
- **Anomaly Detection**: Spot unusual spending patterns early
- **Reporting**: Regular cost visibility for finance and operations teams

## Troubleshooting

### Alert Not Triggering

- Verify you have recent data in `system.billing.usage`
- Check that the evaluation condition is properly configured
- Ensure notification settings are correctly set up

### Missing Data in Email

- Confirm access to billing tables
- Verify date range captures relevant data
- Check that workspaces have spending above the $5 threshold

### Incorrect Cost Calculations

- Ensure `system.billing.list_prices` has current pricing data
- Verify currency code is set to USD (or update query for your currency)
- Check that price effective dates align with usage dates

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the MIT License.

## Support

For issues or questions:
- Open an issue on GitHub
- Review Databricks documentation for [SQL Alerts](https://docs.databricks.com/sql/user/alerts/index.html)

## Acknowledgments

Built for monitoring and optimizing Databricks workspace costs across organizations.
