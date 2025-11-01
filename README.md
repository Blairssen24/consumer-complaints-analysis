# 🏦 CFPB Credit Card Complaints Analysis | Python + Tableau
Data-driven insights into consumer complaints, company responses, and dispute patterns

## 📊 Project Overview

This project analyzes consumer complaints data from the **Consumer Financial Protection Bureau (CFPB)** to uncover insights into:
- How financial companies handle consumer complaints
- Response efficiency and timeliness
- Dispute trends across different companies and products

As a **Data Analyst**, the goal was to clean, prepare, and analyze the dataset to answer key business questions and visualize insights using Python and Tableau.

## 💼 Business Problem Statement

The **CFPB** and partner companies want to understand how effectively consumer complaints are being managed and resolved.

They’ve asked for:
1. An overview of total complaints and complaint categories  
2. Identification of top issues and products with the most complaints  
3. Timeliness of company responses and average resolution time  
4. Insight into consumer disputes — how often customers are unsatisfied  
5. Complaint submission trends over time (daily, monthly, by channel)

The main question:  
> “Which companies are receiving the most complaints, how are they responding, and are customers satisfied?”

## 🧰 Tools & Technologies

- **Python (Pandas, Matplotlib, Seaborn, Plotly)** – Data cleaning, KPI computation, and visualizations  
- **Tableau** – Dashboard design and advanced data storytelling  
- **Excel** – Data inspection and initial exploration  
- **GitHub** – Version control and project documentation

## 🗂 Dataset Overview

**Source:** Consumer Financial Protection Bureau (CFPB) public dataset  
**Rows:** 2M+ complaints  
**Columns:** 23 fields including company, product, issue, state, complaint narrative, date received, and timely response flag.

**Key Columns:**
| Column | Description |
|--------|--------------|
| company | Name of the financial institution |
| complaint_id | Unique ID for each complaint |
| product | Product category (e.g., credit card, mortgage) |
| company_response_to_consumer | How company responded |
| timely_response? | Whether the company responded on time |
| consumer_disputed? | Whether consumer disputed the resolution |
| date_received | Date complaint was received |
| date_sent_to_company2 | Date complaint was sent to the company |


## 🧼 Data Cleaning Process

Steps performed in Python:
- Removed duplicate records  
- Standardized column names  
- Parsed and formatted date fields  
- Calculated response time (days between received and sent dates)  
- Created flags for “Timely Response” and “Disputed Complaint”  
- Filtered invalid or missing complaint entries

## 📈 Exploratory Data Analysis (EDA)

1. **Total Complaints Overview**  
   *Shows the total complaints received and rolling 12-month trend*  
   
#### Code Snippet (Total Complaints, Rolling 12 Months)
```python
# Total Complaints
total_complaints = credit_df.shape[0]
print("Total Complaints:", total_complaints)

# Rolling 12 Months
# Convert date column
credit_df['date_received'] = pd.to_datetime(credit_df['date_received'], errors='coerce')

# Define 12-month rolling window
max_date = credit_df['date_received'].max()
rolling_start = (max_date - pd.DateOffset(months=11)).replace(day=1)

rolling_df = credit_df[
    (credit_df['date_received'] >= rolling_start) &
    (credit_df['date_received'] <= max_date)
]

rolling_12m_complaints = rolling_df.shape[0]

print(f"Rolling 12-month window: {rolling_start.strftime('%b %Y')} → {max_date.strftime('%b %Y')}")
print(f"Rolling 12-month complaints: {rolling_12m_complaints:,}")
```
![Total Complaints](images/Total_complaint.png)
<img width="470" height="162" alt="Total_complaint" src="https://github.com/user-attachments/assets/96a52349-4898-43e7-b96d-2cb0b081d5b3" />

2. **Top 10 Companies by Complaint Volume**  
   *Highlights which companies received the most complaints*
   ```python
   company_complaints = (
    credit_df.groupby('company', as_index=False)['number_of_records']
    .sum()
    .sort_values(by='number_of_records', ascending=False)) company_complaints.head(10)
    ```

   ![Top Companies](images/top_10_companies_complaint_volume.png)

3. **Top Complaint Issues**  
   *Displays the most common issues raised by consumers*
   
   ```python
    top_issues = (
    credit_df.groupby('issue')['number_of_records']
    .sum()
    .reset_index()
    .sort_values(by='number_of_records', ascending=False)
    .head(10))


    fig = px.bar(
    top_issues,
    x='number_of_records',
    y='issue',
    orientation='h',
    title='Top 10 Consumer Complaint Issues',
    text='number_of_records',
    color='number_of_records',
    color_continuous_scale='Tealgrn')


    fig.update_traces(textposition='outside')
    fig.update_layout(
    yaxis=dict(title='Issue', autorange='reversed'),
    xaxis_title='Number of Complaints',
    showlegend=False)


    fig.show()

    ```
   
   ![Top Issues](images/top_10_consumer_issues.png)

4. **Disputed vs Non-Disputed Complaints**  
   *Shows how many consumers disputed the company’s response*
   ```python
    dispute_summary = (
    credit_df.groupby('consumer_disputed?')['number_of_records']
    .sum()
    .reset_index()
    .sort_values(by='number_of_records', ascending=False))


    dispute_summary['percentage'] = (
    dispute_summary['number_of_records'] / dispute_summary['number_of_records'].sum() * 100 ).round(2)



    print(dispute_summary)

    fig = px.pie(
    dispute_summary,
    names='consumer_disputed?',
    values='number_of_records',
    color='consumer_disputed?',
    color_discrete_sequence=px.colors.sequential.RdBu,
    title='Consumer Dispute Status',
    hole=0.4)

    fig.update_traces(
    textinfo='label+percent+value',
    pull=[0.05 if x == 'Yes' else 0 for x in dispute_summary['consumer_disputed?']])

    fig.show()
    ```
   
   ![Dispute Chart](images/consumer_dispute.png)

5. **Timely vs Untimely Responses**  
   *Analyzes how quickly companies respond to complaints*
    ```python
    timely_summary = (
    credit_df.groupby('timely_response?')['number_of_records']
    .sum()
    .reset_index()
    .sort_values(by='number_of_records', ascending=False))


    timely_summary['percentage'] = (
    timely_summary['number_of_records'] / timely_summary['number_of_records'].sum() * 100 ).round(2)


    # Display the summary table
    print(timely_summary)

    ```
   ![Timely Response](images/timely_vs_untimely.png)

6. **Complaints by Submission Channel**  
   *Visualizes the percentage of complaints submitted via each channel (Web, Phone, Email, etc.)
    
    ```python 
    submitted_via = (
    credit_df.groupby('submitted_via')['number_of_records']
    .sum()
    .reset_index()
    .sort_values(by='number_of_records', ascending=False))

  ![Submitted Via](images/Submitted_via_visual.png)

7. **Complaint Trend Over Time**  
   *Daily complaint trend line showing peaks and drops over time*
   ```python 
   daily_trend = (
    credit_df.groupby('date_received')['number_of_records']
    .sum()
    .reset_index())
   ```
     
   ![Daily Complaint](images/daily_complaint.png)

8. **Top 10 State by complaint volume**
   ```python 

    state_complaints = (
    credit_df.groupby('state')['number_of_records']
    .sum()
    .reset_index()
    .sort_values(by='number_of_records', ascending=False))

    # Display top 10 states
    print(state_complaints.head(10))
    ```

   ![State_volume_10](https://github.com/user-attachments/assets/98ce16fd-a799-407e-a323-63543e604e7e)

9. **Companies responses to consumers**
```python
    # Company response type
response_summary = (
    filtered_df.groupby('company_response_to_consumer')['number_of_records']
    .sum()
    .reset_index()
    .sort_values(by='number_of_records', ascending=False))

# percentage of total
response_summary['percentage'] = (
    response_summary['number_of_records'] / response_summary['number_of_records'].sum() * 100
).round(2)


print(response_summary)
```
![Response_to_consumer](https://github.com/user-attachments/assets/933d4674-f32f-43f3-bb98-55c62468bcd5)

10. **In Progress**
  ```python
in_progress_count = credit_df[
    credit_df['company_response_to_consumer'].str.lower() == 'in progress'
].shape[0]

in_progress_percent = (in_progress_count / total_complaints) * 100

print(f" In-Progress Complaints: {in_progress_count:,}")
print(f" In-Progress Percentage: {in_progress_percent:.2f}%")

```
<img width="482" height="160" alt="In_progress" src="https://github.com/user-attachments/assets/49b8338b-d521-469c-9ebe-5267f9b846dd" />

## 🧾 KPIs & Definitions

| KPI | Definition | Formula | Example Use |
|------|-------------|----------|--------------|
| Total Complaints | Total number of consumer complaints recorded | Count of `complaint_id` | Overall complaint volume |
| In-Progress Complaints | Complaints still marked as “In Progress” | Count of `company_response_to_consumer = "In Progress"` | Monitor open cases |
| Timely Response Rate | % of complaints responded to on time | (`Timely="Yes"` ÷ Total Complaints) × 100 | Measure response efficiency |
| Disputed Complaints Rate | % of complaints consumers marked as “Disputed” | (`Disputed="Yes"` ÷ Total Complaints) × 100 | Measure dissatisfaction |
| Avg. Days to Respond | Average days between received and sent dates | Mean(`avg_no_of_days`) | Measure operational efficiency |
| Top Complaint Issue | Most frequent issue raised | Mode(`issue`) | Identify recurring problems |


## 💡 Key Insights

1. **High Complaint Volume:** Top 3 companies account for over 40% of total complaints.  
2. **Response Timeliness:** ~88% of complaints were responded to on time.  
3. **Disputed Rate:** About 7–10% of consumers disputed company resolutions.  
4. **Submission Channels:** Most complaints were submitted via Web, showing customer preference for online engagement.  
5. **Daily Trends:** Noticeable spikes during certain months — may align with product releases or financial events.

## 🧭 Recommendations

- Focus on **improving dispute handling** — analyze reasons for disputes in narratives.  
- Enhance **timely response tracking** through automated alerts.  
- Prioritize customer experience for **top complaint products** (like credit cards and mortgages).  
- Use complaint channel insights to **optimize support staffing** (e.g., more agents online).  

## 📊 Tableau Dashboard Preview

The Tableau dashboard includes:
- Total complaints by company and issue  
- Disputed vs Timely complaint rates  
- Submission channel and trend analysis  

![Dashboard](images/Final_dashboard.png)

## 🔗 Project Links
- **Tableau Public Dashboard:** []

 ## ✨ Developed by: Blessing Igwe

---




