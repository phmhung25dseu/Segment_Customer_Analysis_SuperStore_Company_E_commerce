# Customer Segment - RFM Analysis 
## I. Introduction
  SuperStore Company is a global retail company. Therefore, it has a large number of customers.
  On the occasion of Christmas and New Year, the Marketing Department wants to run marketing campaigns to show appreciation to customers who have supported the company over time, as well as to target potential customers who may become loyal in the future.
  However, the Marketing Department has not yet segmented the customers for this year because the dataset is too large to process manually like in previous years. Therefore, they have requested support from the Data Analytics Department to implement a segmentation model to categorize each customer, enabling the deployment of appropriate marketing programs for each group.
  The Marketing Director has also proposed using the RFM model. In the past, when the company was smaller, the team could calculate and classify segments using Excel. But now, with the large volume of data, they hope the Data Department can develop a segmentation pipeline using Python programming.
## II. Objective
  The objective of this project is to:
  Develop an automated customer segmentation system using the RFM (Recency, Frequency, Monetary) model with Python to support targeted marketing campaigns.
_**🔍 Key Goals:**_
1.	Segment Customers: Classify customers into distinct groups based on their purchasing behavior.
2.	Enable Targeted Marketing: Help the Marketing Department deploy customized campaigns for:

o	Loyal customers (retention and appreciation)

o	Potential loyal customers (conversion)

o	Inactive or at-risk customers (re-engagement).

3.	Automate the Process: Replace manual segmentation (previously done in Excel) with a scalable, efficient Python-based solution.
4.	Support Data-Driven Decisions: Provide insights and visualizations to help the marketing team understand customer value and engagement.
## III. Methods
_**📊 What is RFM Analysis?**_
RFM Analysis is a customer segmentation technique used in marketing and analytics to evaluate customer value based on three key factors:

Metric	Description

Recency (R): 	How recently a customer made a purchase. More recent = more engaged.

Frequency (F): 	How often a customer makes a purchase. More frequent = more loyal.

Monetary (M): 	How much money a customer has spent. Higher spend = more valuable.
________________________________________
_**💡 Why Use RFM Analysis?**_
1.	Simple and Effective Segmentation

RFM uses real behavioral data (transactions), not surveys or guesswork, to group customers.

2.	Identify High-Value Customers

Quickly find:
  o	VIPs (recent, frequent, high spenders)
  
  o	At-risk customers (haven’t bought recently)
  
  o	New customers
  
  o	Bargain hunters (frequent but low spend)
3.	Targeted Marketing
Tailor campaigns based on customer type:

o	Rewards for loyal customers

o	Win-back strategies for inactive ones

o	Intro offers for new customers

4.	Better ROI

Focus marketing efforts where they matter most, instead of using generic mass campaigns.

5.	Scalable & Data-Driven

Easy to implement with large datasets using Python or SQL, and adaptable across industries (retail, e-commerce, SaaS, etc.)

## IV. Process
**1. Import libraries and data**

_**+ Import libraries**_

```python
# import required libraries for dataframe and visualization
import numpy as np
import pandas as pd
import datetime as dt
import time, warnings
import matplotlib.pyplot as plt
import seaborn as sns
import squarify
```

_**+ Import data**_
```python
# Load data into Colab
from google.colab import drive
drive.mount('/content/drive')

# Define the path to the project folder
path = '/content/drive/MyDrive/RFM_Segment/'

# Load data from Excel file
cus_retail = pd.read_excel(path + 'ecommerce retail.xlsx', sheet_name='ecommerce retail')
seg = pd.read_excel(path + 'ecommerce retail.xlsx', sheet_name='Segmentation')

# Convert the files to CSV format
cus_retail.to_csv(path + 'ecommerce.csv', index=False)
seg.to_csv(path + 'segmentation.csv', index=False)
```

**2. EDA – Exploration Data Analysis**
_**2.1. Generate Data Profiling Report**_
```python
!pip install ydata-profiling

from ydata_profiling import ProfileReport

# Create report
profile = ProfileReport(cus_retail, title="Profiling Report", explorative=True)

# Show report in notebook
profile.to_notebook_iframe()
```
![image](https://github.com/user-attachments/assets/6a9c07c3-800e-4142-b8b1-225e3288178c)
![image](https://github.com/user-attachments/assets/b1440196-f9f2-40b7-b413-186d8b442858)

_**2.2. Check data type**_
```python
#Checking the information of df
cus_retail.info()
```
![image](https://github.com/user-attachments/assets/a4ae0a2a-e7a4-4563-b10f-5a80a7135133)

_**2.3. Check missing value**_
```python
#Counting number of missing value in each column
Cus_retail_na = cus_retail.isna().sum()
print(Cus_retail_na)
```

![image](https://github.com/user-attachments/assets/f68b55d9-1bf7-4237-8ba7-1a3861e331ad)

![image](https://github.com/user-attachments/assets/618508d8-cafd-4162-b831-8ccc1db6cfc2)

_**Notes:**_
•	The data contains negative values in the Quantity and Unit Price fields.

•	Description have missing value, but it is not affect analysis process, so can not edit this column.

•	CustomerID have missing values --> Delete row is null.

•	Datatype of CustomerID is not true, needing convert datatype to int64.

**3. Clean data**
_**3.1. Delete missing value**_
```python
#Drop missing value
cus_retail.dropna(subset=['CustomerID'], inplace=True)
```
_**+ Checking after delete missing value**_
```python
cus_retail.isna().sum()
```
![image](https://github.com/user-attachments/assets/aaf67f15-959e-4f54-9a35-38183c9a5b16)

_**3.2. Convert data type of CustomerID**_
```python
#Convert datatype CustomerID to int
cus_retail['CustomerID'] = cus_retail['CustomerID'].astype(int)
```
_**+ Checking after converting**_
```python
#Checking after converting
cus_retail.info()
```
![image](https://github.com/user-attachments/assets/0ca00a3b-a1eb-4f7d-8b18-36ed6536f686)

3.3. Remove quantities less than 0
```python
#Remove cancel order
cus_retail = cus_retail[cus_retail['Quantity']>0]
cus_retail.shape
```
4. Segment Analyst
```python
#Datetime now
present = dt.date(2011,12,31)
print(present)
#Create RFM data frame
cus_retail['Totalrevenue'] = cus_retail['Quantity'] * cus_retail['UnitPrice']
cus_retail['InvoiceDate'] = pd.to_datetime(cus_retail['InvoiceDate'])
cus_retail['Date'] = cus_retail['InvoiceDate'].dt.date
RFM_df = cus_retail.groupby('CustomerID').agg({'Date':'max','InvoiceNo':'count','Totalrevenue':'sum'}).reset_index()
RFM_df.columns = ['CustomerID','LastPurchase','F_value','M_value']
RFM_df['R_value'] = RFM_df['LastPurchase'].apply(lambda x: (present - x).days)
RFM_df = RFM_df.drop('LastPurchase', axis=1)
RFM_df.columns = ['CustomerID','F_value','M_value','R_value']
RFM_df
#R score
RFM_df['R_score'] = pd.qcut(x = RFM_df['R_value'], q = 5, labels = [5, 4, 3, 2, 1])
# F score
RFM_df['F_score'], bins = pd.qcut(RFM_df['F_value'].rank(method='first'),
                                  q = 5,
                                  labels= [1, 2, 3, 4, 5],
                                  retbins = True,
                                  precision=5,
                                  duplicates='drop')
#M score
RFM_df['M_score'] = pd.qcut(x = RFM_df['M_value'], q = 5, labels = [1, 2, 3, 4, 5])
#RFM score
RFM_df['RFM_score'] = (RFM_df['R_score'].astype(str) + RFM_df['F_score'].astype(str) + RFM_df['M_score'].astype(str))
RFM_df
#Transform Segmentation table segment
seg['RFM_score'] = seg['RFM Score'].str.split(',')
seg = seg.explode('RFM_score').reset_index(drop=True)
seg.column = ['Segment', 'RFM_score']
print(seg)
#Merging RFM with segment
RFM = RFM_df.merge(seg, how = 'left', on = 'RFM_score')
RFM
```
_**+ Result**_

![image](https://github.com/user-attachments/assets/c99770fa-91d2-4384-bf96-8651b3ff47aa)

## V. Results
**1. Retention Customer**
_**+ Code**_
```python
#function for month
def get_month(x):
  return dt.datetime(x.year, x.month, 1)
#apply the function
data = cus_retail
data['InvoiceMonth'] = data['InvoiceDate'].apply(get_month)
data.tail()
#create a column index with the minimum invoice date
data['CohortMonth'] = data.groupby('CustomerID')['InvoiceMonth'].transform('min')
#Create a date element function to get a series
def get_date_elements(df, column):
  day = df[column].dt.day
  month = df[column].dt.month
  year = df[column].dt.year
  return day, month, year
#get date element for our cohort and invoice columns
_, Invoice_Month, Invoice_Year = get_date_elements(data, 'InvoiceMonth')
_, Cohort_Month, Cohort_Year = get_date_elements(data, 'CohortMonth')
#Create an index cohort
year_diff = Invoice_Year - Cohort_Year
month_diff = Invoice_Month - Cohort_Month
data['CohortIndex'] = year_diff*12 + month_diff + 1
data.head()
#Create pivot table
Cohort_data = data.groupby(['CohortMonth', 'CohortIndex'])['CustomerID'].nunique().reset_index()
Cohort_table = Cohort_data.pivot(index = 'CohortMonth', columns = ['CohortIndex'], values = 'CustomerID')
#Change index
Cohort_table.index = Cohort_table.index.strftime('%B %Y')
#Create a percentage
Cohort_table_ = Cohort_table.divide(Cohort_table.iloc[:,0], axis = 0)
#Visualization
plt.figure(figsize=(12, 6))  # Set the figure size before the heatmap
sns.heatmap(Cohort_table_, annot=True, cmap='Blues', fmt='.0%')
plt.title('Retention by Monthly Cohorts')
plt.show()
```
_**+ Result**_

![image](https://github.com/user-attachments/assets/fe9b6ce7-7165-4ccc-a2a4-73a9e44071fa)

_**🔍 Key Observations**_
1.	Initial Drop-off:

o	All cohorts start with 100% (by definition).

o	The steepest drop-off typically occurs between Month 1 and Month 2, which is common in user behavior analytics.

2.	December 2010 Cohort is Exceptional:

o	This cohort shows relatively strong and consistent retention over time.

o	It has the highest long-term retention, with noticeable bumps at Month 6 (40%), Month 10 (40%), and Month 11 (50%).

3.	Recent Cohorts (Late 2011):

o	These cohorts have fewer data points (as they joined more recently).

o	However, they also show a weaker retention trend, with retention dropping to 11% or lower by the second or third month.

4.	Mid-Year 2011 Cohorts:

o	May, June, and July 2011 cohorts show slight improvements in Month 5-6 retention (e.g., June 2011 has 33% in Month 6), but this is not sustained.

_**💬 Discussion and Implications**_
•	Product Changes or Marketing Shifts?

  o	The December 2010 cohort's strong retention might suggest a period of particularly effective onboarding, product quality, or targeted user acquisition.
  
  o	What changed in 2011 that might have caused lower retention? This is worth investigating—perhaps a change in user acquisition strategy or product updates.
  
•	User Engagement Strategies Needed:

  o	The steep drop-off in most cohorts suggests that users lose interest quickly.
  
  o	Improving early engagement (onboarding, education, feature exposure) could improve these metrics.
  
•	Cohort Analysis Utility:

  o	This kind of visualization is excellent for identifying long-term trends, seasonality, or the impact of interventions.
  
  o	It’s a critical tool for SaaS, mobile apps, and any recurring usage product.

_**✅ Recommendations**_

•	Analyze December 2010 cohort more closely—what worked, and can it be replicated?

•	Implement onboarding improvements and track cohort changes over time.

•	Consider segmenting by user type, acquisition source, or geography to find more insights.

_**2. Recency value, Frequency value, Monetary value**_
_**+ Code**_
```python
#Histogram
joined = pd.DataFrame(RFM)
colnames = ['R_value', 'F_value', 'M_value']

for col in colnames:
    plt.figure(figsize=(12, 3))
    sns.histplot(joined[col], kde=False)
    plt.title('Distribution of %s' % col)
    plt.xlabel(col)
    plt.ylabel('Frequency')
    plt.show()
```
_**+ Result**_

![image](https://github.com/user-attachments/assets/20d4c555-be20-4143-9f80-1851178bc35a)
![image](https://github.com/user-attachments/assets/6ba4a306-67fb-4aa5-92d3-d152ca8c08e3)
![image](https://github.com/user-attachments/assets/4358c84c-8181-4aff-99f6-3e1a1dcdf022)

_**🔍 Key Observations**_

_**📊 1. Distribution of R_value (Recency)**_

•	Shape: Right-skewed distribution.

•	Interpretation:

  o	A large number of customers have low R values, meaning they purchased recently.

  o	Fewer customers have higher R values, indicating less recent activity.

•	Insight:

  o	The business likely retains a good portion of active or recently engaged customers.

  o	However, there's a long tail of inactive users, suggesting a need for re-engagement strategies.

_**📊 2. Distribution of F_value (Frequency)**_

•	Shape: Heavily right-skewed with a sharp drop-off.

•	Interpretation:

  o	The vast majority of users make very few purchases.

  o	A small number of users are very frequent purchasers, visible as the long tail.

•	Insight:

  o	This is a typical “power law” distribution in e-commerce or subscription services.

  o	Consider identifying and nurturing high-frequency users—they could be VIPs or loyal customers.

_**📊 3. Distribution of M_value (Monetary)**_

•	Shape: Extremely right-skewed, even more than Frequency.

•	Interpretation:

  o	Most customers spend small amounts.
  
  o	A few customers spend significantly more, leading to a long tail stretching past 250,000 in value.
  
•	Insight:

  o	Indicates the presence of high-value customers who may account for a disproportionate share of revenue (Pareto principle).
  
  o	Segmenting and giving special attention to these high-M customers could boost lifetime value.

_**💡 Overall Commentary:**_

•	Skewed distributions in all three RFM components are typical and valuable:

  o	Allow tiered segmentation (e.g., Low/Medium/High).

  o	Help in targeting marketing campaigns (e.g., re-engage high-R, reward high-F and high-M).

•	Combining RFM values can guide you in building customer personas like:

  o	“Champions” – low R, high F and M.

  o	“At-risk” – high R, previously high F or M.

  o	“Potential loyalists” – low R, moderate F.

_**3. Contribution by Segment**_
_**+ Code**_
```python
# Define colors for each segment
colors = ["gold", "silver", "orange", "green", "blue", "red", "pink", "brown", "gray", "cornsilk", "black"
]
#Creating dataframe including segment and number of customer each segment
Cus_seg = RFM.groupby('Segment')['CustomerID'].count().reset_index()
Cus_seg = Cus_seg.sort_values(by='CustomerID', ascending = False)
Cus_seg.columns = ['Segment','No_customer']
print(Cus_seg)
#Treemap
fig, ax = plt.subplots(1, figsize = (15, 8))
squarify.plot(sizes=Cus_seg["No_customer"],
              label=Cus_seg["Segment"],
              value=[f'{x / Cus_seg["No_customer"].sum() * 100:.2f}%' for x in Cus_seg["No_customer"]],
              alpha=.8,
              color=colors)
plt.title('RFM Segments of Customer Count', fontsize = 16)
plt.axis('off')
plt.show()
```
_**+ Result**_
![image](https://github.com/user-attachments/assets/8325b570-ded2-4490-b39d-c6ad21a8d6d8)

_**🔍 Key Observations**_

_**🧠 Strategic Takeaways**_

•	The largest segments are "Hibernating" and "Champions", indicating a split between highly engaged and disengaged users. A clear sign of engagement polarity.

•	With nearly 10% "At Risk" and 10% "Lost Customers", retention strategies should be a top priority.

•	"Potential Loyalists" and "New Customers" together make up nearly 19%—these are key targets for growth and conversion to "Champions".

•	Small but important segments like "Cannot Lose Them" (2.01%) may require bespoke outreach given their past value.

_**✅ Actionable Summary**_
•	Double down on "Champions" and "Loyal" to retain high-value users.

•	Rescue "At Risk" and "Hibernating" with personalized engagement.

•	Convert "Potential Loyalists" and "Promising" into long-term customers.

•	Use automation to re-engage "Lost" or "About to Sleep" cohorts efficiently.

_**4. Segment by Spending**_
_**+ Code**_
```python
# Calculating spending (M_value) each segment
segment_by_spending = (
    RFM[RFM['M_value'] > 0][['Segment', 'M_value']]
    .groupby('Segment')['M_value']
    .sum()
    .reset_index()
    .rename(columns={'M_value': 'Spending'})
)

# Calculate percentage value to segment labels
segment_by_spending['percent_segment_by_spending'] = round(
    segment_by_spending['Spending'] / segment_by_spending['Spending'].sum() * 100
)

# Convert the percentage values to integer type for better readability
segment_by_spending['percent_segment_by_spending'] = segment_by_spending['percent_segment_by_spending'].astype(int)

# Append percentage values to segment labels
segment_by_spending['Segment'] = (
    segment_by_spending['Segment'] + ' ' +
    segment_by_spending['percent_segment_by_spending'].astype(str) + '%'
)

# Remove segments with 0% contribution.
segment_by_spending = segment_by_spending[segment_by_spending['percent_segment_by_spending'] > 0]

# Plot a treemap to visualize spending distribution by segment
import matplotlib.pyplot as plt
import squarify
import seaborn as sns

plt.figure(figsize=(12, 8))
squarify.plot(
    sizes=segment_by_spending['percent_segment_by_spending'],
    label=segment_by_spending['Segment'],
    color=sns.color_palette("Set2"),
    alpha=0.8,
    text_kwargs={'fontsize': 8}
)
plt.title("Segmentation by Spending", fontsize=10)
plt.axis('off')
plt.show()
```
_**+ Result**_
![image](https://github.com/user-attachments/assets/41f9923f-f043-41ea-bd11-cf873a8a91a0)

_**🔍 Key Observations**_
_**📉 Strategic Contrast to Customer Count**_

•	While Champions are only ~18% of customers, they bring in 61% of the revenue.

•	Segments like Hibernating (18.6% of customers) only account for 4% of spending, confirming that not all customer segments are equally valuable.

•	High-potential segments (e.g., Potential Loyalists, Promising) are underperforming in revenue and may need nudging to unlock growth.

_**✅ Strategic Recommendations**_

•	Double down on Champions with VIP experiences, loyalty programs, and retention efforts.

•	Rescue high-spending "At Risk" and "Cannot Lose Them" customers with win-back campaigns—these are proven high-value clients.

•	Develop "Potential Loyalists" and "Promising" segments through targeted promotions or early loyalty rewards.

•	Deprioritize or automate engagement for Lost/New/Low-contributing segments, unless cost-effective.

_**5. Segmentation by Frequency**_
_**+ Code**_
```python
# Calculate the average frequency of purchases for each segment
segment_by_frequency = RFM[RFM.F_value > 0][['Segment', 'F_value']].groupby('Segment')['F_value'].mean().reset_index()

# Plot a bar chart to visualize the average frequency per segment
plt.figure(figsize=(12, 8))
ax = sns.barplot(data=segment_by_frequency, x='F_value', y='Segment')

# Add value labels to the bars
for i in ax.containers:
    ax.bar_label(i, fmt='%.2f', label_type='edge', padding=3)
    
# Customize the plot
plt.xlabel('Average Purchase Frequency')
plt.ylabel('Segment')
plt.title('Segmentation by Frequency')
plt.show()
```
+ Result
![image](https://github.com/user-attachments/assets/1bb7518e-0a31-450b-8d3a-5c4542fcb906)
_**🔍 Strategic**_

1.	Champions dominate in purchase activity—your top-tier loyalty builders.

o	Their high engagement makes them ideal for upselling, beta testing, referrals, and feedback loops.

2.	Loyal and At Risk customers show substantial frequency too:

o	“At Risk” users may not be recent but are still quite active—a good opportunity for targeted re-engagement.

3.	Potential Loyalists have strong frequency and are excellent candidates to nurture into Champions.
  
4.	Segments below 25 frequency (e.g., Hibernating, Promising, Lost):

o	Require automation-based nurturing or reactivation campaigns.

o	Consider whether it's worth investing marketing dollars here based on historical return rates.

_**📌 Strategy Table**_

| Segment                            | Focus Action                                      |
|------------------------------------|---------------------------------------------------|
| Champions                          | Retain & reward                                   |
| Loyal                              | Reinforce loyalty                                 |
| At Risk                            | Re-engage with urgency                            |
| Potential Loyalist                 | Convert to Champions                              |
| Need Attention                     | Survey for feedback, spark interest               |
| Cannot Lose Them                  | High-priority win-back                             |
| Promising / New                    | Educate & incentivize                             |
| Hibernating / Lost / About to Sleep| Low-cost automated win-back or re-targeting       |

_**6. Segmentation by Recency**_
_**+ Code**_
```python
# Calculate the average recency (days since last purchase) for each segment
segment_by_recency = RFM[RFM.R_value > 0][['Segment', 'R_value']].groupby('Segment')['R_value'].mean().reset_index()

# Plot a bar chart to visualize the average frequency per segment
segment_by_recency = segment_by_recency.rename(columns={'R_value': 'Recency'})

# Add value labels to the bars
plt.figure(figsize=(12, 8))
ax = sns.barplot(data=segment_by_recency, x='Recency', y='Segment')

# Plot a bar chart to visualize the average recency per segment
for container in ax.containers:
    ax.bar_label(container, fmt='%.2f', label_type='edge', padding=3)

plt.xlabel('Average Recency (Days)')
plt.ylabel('Segment')
plt.title('Segmentation by Recency')
plt.show()
```
_**+ Result**_
![image](https://github.com/user-attachments/assets/b6470262-b03d-4ad6-8509-6c1b896c589e)

_**🔍 Key Observations**_
•	Champions, Promising, and Potential Loyalists show low recency, suggesting recent, healthy engagement.

•	At Risk, Hibernating, and Cannot Lose Them have not been active for over 170 days and represent priority segments for re-engagement.

•	Lost Customers likely require cost-sensitive or last-chance offers due to their long inactivity.

_**📌 Strategy Table**_
| Segment                                  | Recency Status        | Recommended Action                               |
|------------------------------------------|-----------------------|--------------------------------------------------|
| Champions                                | Very recent           | Reward and retain                                |
| Loyal / Potential Loyalists              | Recent                | Encourage advocacy, upgrade loyalty tier         |
| Need Attention / Promising               | Moderate              | Use nudges and personalization                   |
| At Risk / Hibernating / Cannot Lose Them | High                  | Prioritize win-back campaigns                    |
| Lost Customers                           | Very high (churned)   | Final reactivation attempt or sunset             |

_**7. Top 5 segment by revenue**_
_**+ Code**_
```python
# Group by 'Segment' and sum the 'M_value'
seg_revenue = RFM.groupby('Segment')['M_value'].sum().reset_index()

# Rename the columns for clarity
seg_revenue.columns = ['Segment', 'Total_revenue']

# Sort the dataframe by 'Total_revenue' in descending order
seg_revenue = seg_revenue.sort_values(by='Total_revenue', ascending=False)

# Select the top 5 segments with the highest revenue
top5_revenue = seg_revenue.head(5)

# Define colors for the top 5 segments
Color_5 = ['#55AD9B', '#809BAD', '#95D2B3', '#D8EFD3', '#F1F8E8']

# Visualize with a barplot
plt.figure(figsize=(12, 3))
sns.barplot(x='Total_revenue', y='Segment', data=top5_revenue, palette=Color_5)

# Add value labels to the bars
for index, value in enumerate(top5_revenue['Total_revenue']):
    plt.text(value, index, f'${value}', va='center', fontsize=10)

# Customize the plot
plt.xlabel("Total Revenue")
plt.ylabel("Segment")
plt.title("Top 5 Segments with Highest Revenue")

# Show the plot
plt.show()
```

_**+ Result**_

![image](https://github.com/user-attachments/assets/bb60feb6-a34c-4ca7-9be5-606af997d531)

🔍 Strategic 
Champions alone contribute over five times more revenue than Loyal customers, reaffirming the need for:

  o	Exclusive perks
  
  o	Upsell/cross-sell offers
  
  o	Priority support and recognition
  
•	At Risk segment holds enormous recovery potential:

  o	They are disengaging but still profitable—target them urgently with win-back campaigns.

•	Need Attention and Hibernating segments together bring in over $650k, suggesting:

  o	There's meaningful value in nurturing mid-tier customers with behavioral nudges or dynamic discounts.

_**✅ Action**_
| Segment        | Action                                                                 |
|----------------|------------------------------------------------------------------------|
| Champions      | VIP treatment, retention-focused                                       |
| Loyal          | Loyalty programs, upsell                                               |
| At Risk        | Personalized reactivation                                              |
| Need Attention | Trigger-based engagement (e.g., abandoned cart, inactivity emails)     |
| Hibernating    | Reactivation with incentives or exit surveys                           |

_**8. Top 5 segment by transactions**_
_**+ Code**_
```python
# Group by 'Segment' and sum the 'F_value'
seg_frequency = RFM.groupby('Segment')['F_value'].sum().reset_index()

# Rename the columns for clarity
seg_frequency.columns = ['Segment', 'Frequency']

# Sort the dataframe by 'Frequency' in descending order
seg_frequency = seg_frequency.sort_values(by='Frequency', ascending=False)

# Select the top 5 segments with the highest frequency
top5_frequency = seg_frequency.head(5)

# Define colors for the top 5 segments
Color_5 = ['#55AD9B', '#809BAD', '#95D2B3', '#D8EFD3', '#F1F8E8']

# Visualize with a barplot
plt.figure(figsize=(12, 3))
sns.barplot(x='Frequency', y='Segment', data=top5_frequency, palette=Color_5)

# Add value labels to the bars
for index, value in enumerate(top5_frequency['Frequency']):
    plt.text(value, index, f'{value:,.0f}', color='black', va="center", fontsize=10)

# Customize the plot
plt.xlabel("Frequency")
plt.ylabel("Segment")
plt.title("Top 5 Segments with Highest Transactions")

# Show the plot
plt.show()
```
_**+ Result**_

![image](https://github.com/user-attachments/assets/a9df73ff-7a4d-43c4-a78b-b87b9ee3aba5)


_**📈 Strategic**_
•	Champions generate more than 4x the transactions of Loyal customers, confirming they are the engine of your business.

  o	Prioritize experience enhancement, loyalty rewards, and product recommendations.

•	At Risk and Potential Loyalists:

  o	Their strong activity levels indicate substantial upside if re-engaged or nurtured into loyalty.

  o	Segment-specific email flows or retargeting campaigns could help them graduate to "Loyal" or "Champions."

•	Hibernating customers are surprisingly active for a disengaged segment:

  o	Use this as a signal—they're not lost yet. Re-engage with behavior-driven campaigns.

## VI. Insights
_**1. Overview**_
| Segment                        | Total Customer | Avg. Recency (Days) | Avg. Purchase Frequency | Avg. Monetary Value | Actions                                                                                   |
|-------------------------------|----------------|----------------------|--------------------------|---------------------|--------------------------------------------------------------------------------------------|
| Champions (17%)               | 799            | **32.86**            | **286.87**               | 6824.64             | Most loyal, active, and valuable customers. Provide VIP perks, exclusive access, and reward loyalty. |
| Loyal (9%)                    | 418            | 59.69                | **121.65**               | 2453.42             | Strong segment, repeat buyers. Nurture with rewards, feedback loops, and upsell opportunities. |
| Potential Loyalist (12%)      | 512            | 50.52                | 61.94                    | 585.41              | Good frequency, moderate value. Build habit loops through loyalty nudges or referral programs. |
| New Customers (8%)            | 304            | 50.36                | 11.06                    | 209.11              | Early-stage customers. Optimize onboarding and trigger early repeat purchases.             |
| Promising (4%)                | 134            | 38.64                | 16.66                    | 2420.89             | New but promising spenders. Encourage frequent purchases with welcome bundles or progressive offers. |
| Need Attention (6%)           | 229            | 54.14                | 57.15                    | 1463.78             | Middle-tier customers. Strengthen relationship via content or targeted promotions.         |
| At Risk (11%)                 | 433            | 178.46               | 81.52                    | 1526.15             | Previously active, now less engaged. Try win-back emails, urgency-driven discounts, or feedback surveys. |
| About to Sleep (4%)           | 191            | 109.73               | 21.58                    | 284.32              | Near-dormant. Push re-engagement with time-limited offers or reminders.                    |
| Cannot Lose Them (3%)         | 87             | 240.97               | 49.62                    | 3380.06             | High value but disengaging. Launch reactivation campaigns with personalized offers or feedback. |
| Hibernating (18%)             | 808            | 171.08               | 23.24                    | 403.48              | Once-engaged but now cold. Offer “come back” incentives or ask why they dropped off.       |
| Lost Customers (9%)           | 424            | **300.58**           | 11.68                    | 170.59              | Highly inactive, least value. Low priority, but test win-back with strong reactivation offer. |

_**2. 🧩 Customer Count (Tree Map - RFM Segments by Count)**_

_- **Largest Segments**:_
  - `Hibernating`: **18.62%**
  - `Champions`: **18.41%**
_- **Followed by**:_
  - `Potential Loyalists`: **11.80%**
  - `At Risk`: **9.98%**
  - `Lost`: **9.77%**
_- **Insight**:  _
  The customer base is polarized — a large portion is highly engaged (`Champions`), but a similarly large portion is disengaged (`Hibernating` and `Lost`).

---

_**3. 💰 Revenue Contribution**_
- _**Champions**_ generate **~61%** of total revenue — your **most valuable segment**.
- `Loyal Customers`: contribute ~**$1M**
- `At Risk`: ~**$660K**
- _**Low contributors**_ despite higher count:
  - `New`
  - `Promising`
  - `Lost`
_**4. 🔁 Frequency** _ 
_- **Highest frequency**:_
  - `Champions`: **286.87**
  - `Loyal Customers`: **121.65**
- _**Strong engagement**_ from:
  - `At Risk`
  - `Potential Loyalists`
- _**Low frequency**_ from:
  - `New`
  - `Lost`
  - `Promising`
_**5. ⏱ Recency**_
_- **Most recently active**:_
  - `Champions`: **32 days**
  - `Promising`: **39 days**
_- **Long inactive**:_
  - `Lost`: **300+ days**
  - `Cannot Lose Them`
  - `At Risk`
_- **Opportunity**:  _
  - `Hibernating`: **171 days** – still recoverable with targeted reactivation campaigns.
_**6. 🔝 Top 5 Segments by Transactions**_

_- **Champions**_: ~**229K transactions** — lead by a large margin.
- Followed by:
  - `Loyal Customers`
  - `At Risk`
  - `Potential Loyalists`
- These four segments are the **core of your repeat business**.

## VII. Recomendations
**Customer Lifecycle Engagement Strategy**

_**🎯 Focus on Retention & Growth of Champions (Top Priority)**_
These customers are high frequency, recent, and drive most revenue.

_**Recommended Actions:**_
- Implement **VIP programs**, early access to sales, **exclusive perks**.
- Upsell and cross-sell via **personalized product recommendations**.

_**💎 Nurture Potential Loyalists & Loyal Customers**_

High potential segments that can become or support your Champions.

_**Recommended Actions:**_
- Offer **reward milestones** for continued loyalty.
- Send **celebratory emails** after milestones (e.g., 5th or 10th purchase).
- Run **referral campaigns** for Loyal customers.

_**💳 Recover *At Risk* & Cannot Lose Them Segments**_
These are **high-value but disengaging customers**.

_**Recommended Actions:**_
- Personalized **"We miss you" campaigns** with urgency.
- Offer **reactivation discounts** or loyalty reinstatement.
- Highlight **new arrivals or features** they haven’t seen.

_**🛫 Re-engage Hibernating & Need Attention**_
These groups are not high value yet, but worth low-cost automation.

_**Recommended Actions:**_
- Drip emails with **educational content**, **product reminders**.
- Incentivize next purchase with **free shipping** or **small discounts**.

_**⛔ Minimize Effort on Lost Customers**_
Very high recency (long inactive) and low value.

_**Recommended Actions:**_
- Use **automated low-touch final win-back campaigns**.
- Clean inactive lists to reduce email costs and improve deliverability.

_**🌱 Guide New & Promising Customers**_
Early in the lifecycle, with the right nudge can convert to loyal.

_**Recommended Actions:**_
- Triggered **welcome sequences**, tips for first purchase, testimonials.
- Highlight **value propositions** early.

## 🧩 Customer Segmentation Strategy Table

| Segment             | Priority      | Strategy                           |
|---------------------|---------------|------------------------------------|
| **Champions**        | ⭐ High        | Retain, reward, upsell             |
| **Loyal**            | High          | Reinforce loyalty, referrals       |
| **Potential Loyalists** | Medium-High | Convert to Champions               |
| **At Risk**          | High          | Win-back offers, urgency           |
| **Cannot Lose Them** | High          | Reactivation campaigns             |
| **Hibernating**      | Medium        | Nurture or automate                |
| **Need Attention**   | Medium        | Low-touch reminders                |
| **New/Promising**    | Medium        | Onboarding, early nudges           |
| **Lost**             | Low           | Final reactivation or sunset       |





  















 








