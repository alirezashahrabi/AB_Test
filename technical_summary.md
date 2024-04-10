# A/B Test Report: Food and Drink Banner

## Purpose

GloBox is primarily known amongst its customer base for boutique fashion items and high-end decor products. However, their food and drink offerings have grown tremendously in the last few months, and the company wants to bring awareness to this product category to increase revenue.
The Growth team decides to run an A/B test that highlights key products in the food and drink category as a banner at the top of the website.  
I will analyze the results of the A/B test to provide a recommendation to the stakeholders about whether GloBox should launch the experience to all users.
## Hypotheses

**Conversion Ratio**:  
*Null Hypothesis*: There is no difference between the Conversion rate of groups. ($H_0 : p_A = p_B$).  
*Alternative Hypothesis* : There is significant difference between the Conversion rate of groups. ($H_1 : p_A \neq p_B$)  

**Average Amount Spent**:  
*Null Hypothesis*: There is no difference between the Average Amount Spent of groups. ($H_0 : \mu_A = \mu_B$).  
*Alternative Hypothesis* : There is significant difference between the Average Amount Spent of groups. ($H_1 : \mu_A \neq \mu_B$)  

## Methodology

### Test Design

- **Population:** The experiment is only being run on the mobile website. A user visits the GloBox main page and is randomly assigned to either the control or test group. The page loads the banner if the user is assigned to the test group, and does not load the banner if the user is assigned to the control group. There were 24,600 users in the treatment, 24343 in the control, and 48,943 total.

- **Duration:** The experiment ran for 2 weeks in Q1 2023. It started at 25 Jan, 2023 and ended at 4 Feb, 2023.
- **Success Metrics:** Two metrics are used to measure success:

1. Conversion Rate: The conversion rate is the number of successful conversions (users who purchased) divided by the total number of users

2. Average Amount Spent: The average amount spent for all users (include users who did not convert).

## Results

### Data Analysis

- **Pre-Processing Steps:** In the following, there are SQL scripts to clean and extract data.

 - To provide a table containing the user ID, the user’s country, the user’s gender, the user’s device type, the user’s test group, whether or not they converted (spent > $0), and how much they spent in total ($0+), the following SQL script is used:

``` sql
-- a common table expression (CTE) is used to find total money spent by each user
WITH total_activity AS(
  SELECT 
  u.id
 ,SUM(COALESCE(ac.spent,0)) as total_spent -- find total money that a user spent

 FROM users AS u
 LEFT JOIN activity AS ac
 ON u.id = ac.uid
 GROUP BY u.id
 ORDER BY total_spent DESC
                     )
 
SELECT 
   total_activity.id
  ,total_activity.total_spent
  ,COALESCE(users.country,'N/A') as country
  ,COALESCE(users.gender,'N/A') as gender
  ,COALESCE(groups.device,'N/A') as device
  ,groups.group
  -- to find customers who made purchase (total_spent > 0 means they bought an item(s) )
  ,CASE WHEN total_spent > 0 THEN 1 
   ELSE 0
   END convert_status
FROM total_activity
LEFT JOIN users
ON total_activity.id = users.id
LEFT JOIN groups
ON total_activity.id = groups.uid
```

- To check the Novelty Effect the following SQL script is written to return the date and the metrics for each group in separate columns.

``` sql
WITH total_activity AS(
  SELECT 
  u.id,
    SUM(COALESCE(ac.spent,0)) as total_spent

 FROM users AS u
 LEFT JOIN activity AS ac
 ON u.id = ac.uid
 GROUP BY u.id
  ORDER BY total_spent DESC
           )
SELECT 
join_dt as Date,
AVG(CASE WHEN
     groups.group='A' THEN total_spent
    END) AS total_spent_A,
AVG(CASE WHEN
     groups.group='B' THEN total_spent
    END) AS total_spent_B,
 
SUM (
  CASE WHEN 
  groups.group = 'A' AND total_spent>0 THEN 1
   ELSE 0 
   END
  ):: FLOAT/
SUM (
  CASE WHEN 
  groups.group = 'A' THEN 1 
  ELSE 0 
  END
    )  AS Conversion_A,
SUM (
  CASE WHEN 
  groups.group = 'B' AND total_spent>0 THEN 1 
  ELSE 0 
  END
  ):: FLOAT/
SUM (
  CASE WHEN groups.group = 'B' THEN 1 ELSE 0 END
)  AS Conversion_B  

FROM total_activity as tc
LEFT JOIN groups
ON tc.id = groups.uid

GROUP BY join_dt
ORDER BY join_dt

```

- **Statistical Tests Used:**

  - Since Conversion Rate is a proportion and its difference between two groups are desirable , **two sample z-test** is selected. In addition, based on the hypothesis, the test is "two-sided".

  - The **two sample t-test**, used to compare the average amounts spent by the two groups and the test is "two-sided".

- **Results Overview:**

The z-test for the conversion rate yields a p-value of approximately 0.0001.

The t-test, applied to compare the average spending between the two groups, produces a p-value of approximately 0.944. 

### Findings

## Interpretation

- **Outcome of the Test(s):**

   The z-test for the conversion rate yields a p-value of approximately 0.0001. As this p-value falls below our significance threshold of 0.05, it indicates a statistically significant difference in conversion rates between Group A (Test) and Group B (Treatment). Consequently, we **reject the Null Hypothesis**.

   The t-test, applied to compare the average spending between the two groups, produces a p-value of approximately 0.944. As this value greatly exceeds the usual significance threshold of 0.05, it indicates no statistically significant difference in average spending between Group A and Group B. Thus, we **fail to reject the Null Hypothesis**.

- **Confidence Level:**
The 95% confidence interval for the difference in conversion rate between the two groups is (0.0035,0.017).  
The 95% confidence interval for the difference in average amount spent between the two groups is (-0.4387,0.4713), which includes 0.

- **Power Analysis**:  To determine the necessary sample size for detecting a minimum relative change of 10%, we performed a power analysis using the following parameters:  
**Parameters Used**:  
Desired Power: (80%)  
Level of Significance: 0.05  
Minimum Detectable Effect: 10%  
**Findings**:
For the conversion rate, both groups have already exceeded the required sample size of 20,133 users.
For average spending, it’s estimated that approximately 76 to 79 more days are needed for both groups to reach the desired sample size of 90,875 users, assuming consistent join rates.

## Conclusions
- **Key Takeaways:** 
  - Reliable results for conversion rate due to adequate sample size.
  - Caution advised in interpreting average spending results due to current insufficient sample size.
  - **Conversion Rate Impact:** The A/B test revealed a statistically significant difference in conversion rates between Group A (Control) and Group B (Treatment), with Group B showing a higher conversion rate. This suggests that the changes implemented in the treatment group positively impacted user behavior.
  - **Spending Patterns:** The analysis of average spending showed no significant difference between the two groups. This indicates that while the treatment may influence the likelihood of users making a purchase, it doesn't significantly affect the amount they spend.
- **Limitations/Considerations:**
  - **Sample Size:** Insufficient as per power analysis parameters.
  - **Duration of the Test:** The test duration might not have been long enough to capture long-term behavioral changes, such as adjustments to novelty effects.
  - **External Factors:** External factors not accounted for in the test could have influenced user behavior, such as seasonal trends or concurrent marketing campaigns.


## Recommendations
- **Next Steps:** 
- **Regarding the Conversion Rate**: Since the sample size is adequate, we recommend considering the current findings for the conversion rate when making decisions about the banner implementation.
- **For the Average Spending Analysis**: We recommend re-running and extending the A/B test duration by approximately 76 to 79 days to accumulate the necessary sample size for a more reliable analysis of average spending.
- **Further Analysis:**
  - **Segmented Analysis:** Further analyze the data by segmenting users based on demographics, user behavior, or other relevant factors to uncover deeper insights.
  - **Explore Additional Variables:** Investigate other variables that may influence spending behavior, such as user engagement levels or specific features of the new design.

