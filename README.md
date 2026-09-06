# Healthcare_data_analysis
Analysed public healthcare dataset (55,500 patient records) using BigQuery SQL and Tableau to answer business questions around billing trends, hospital performance and patient admissions.  
**Tools:** BigQuery SQL | Tableau
**Dataset:** [Kaggle Healthcare Dataset by Eduardo Licea](https://www.kaggle.com/datasets/eduardolicea/healthcare-dataset)
### Problem from stakeholders - "We think we're losing money on certain types of patients. Can you look into it?" 

Questions structured using 5W1H framework to explore the understanding of business problem

What - what medical conditions have the highest billing amount?
What is the average length of stay per medical condition?

When - how much money is billed yearly, Is there a trend in total billing amount over the years? Are there any monthly/seasonal trends in admissions or billing?

Who - Are certain hospitals associated with high billing amounts?


## Data cleaning checks

## Null check query 

 ```sql
select *
from `practise-sql-505810.Healthcare_data.Patient_details`
WHERE Name IS NULL 
   OR Age IS NULL 
   OR Gender IS NULL 
   OR Blood_Type IS NULL 
   OR Medical_Condition IS NULL 
   OR Date_of_Admission IS NULL 
   OR Doctor IS NULL 
   OR Hospital IS NULL 
   OR Insurance_Provider IS NULL 
   OR Billing_Amount IS NULL 
   OR Room_Number IS NULL
   OR Admission_Type IS NULL
   OR Discharge_Date IS NULL
   OR medication IS NULL
   OR `Test _Results` IS NULL
   OR Length_of_Stay IS NULL
```

Result - Nil field with null value was found

## Duplicate 

```sql
Select Name, Date_of_Admission, COUNT(*) AS occurrences
From `practise-sql-505810.Healthcare_data.Patient_details`
Group by Name, Date_of_Admission
Having count(*) > 1
```

Checked individual details of a duplicate value 

```sql
select *
from `practise-sql-505810.Healthcare_data.Patient_details`
where Name = "Emily King"
```

Result - Initial investigation using Name + Admission Date suggested ~5,509 potential duplicate records; however, deeper inspection revealed these were distinct patients sharing the same name and admission date — a known limitation of synthetic/generated datasets lacking unique patient identifiers.

## A different approach was then used to find out actual duplicates 

```sql
select *, count (*) as occurences
from `practise-sql-505810.Healthcare_data.Patient_details`
group by name, age, Gender, Blood_Type, Medical_Condition, Date_of_Admission, doctor, Hospital, Insurance_Provider, Billing_Amount,Room_Number, Admission_Type, Discharge_Date, Medication, `Test _Results`, Length_of_Stay
having count(*) >1
```

Results - 0 true duplicate values 

## Q1: Which Medical Conditions have the highest average Billing Amount?

```sql
select Medical_Condition,
avg (Billing_Amount) as avg_billingamount
from `practise-sql-505810.Healthcare_data.Patient_details`
group by Medical_Condition
order by avg(Billing_Amount) desc
```

Results: 
Cancer had the highest average billing amount of $64537 followed by Heart diseases at $44913 and Flu was the lowest billed condition with billing amount of $2744. 


## Q2: What is the average length of stay per medical condition?

```sql
select Medical_Condition,
avg (Length_of_Stay) As avg_lengthofstay
from `practise-sql-505810.Healthcare_data.Patient_details`
group by Medical_Condition
order by avg_lengthofstay desc
```

Results - Alzheimer's had the longest average length of stay of 54.4 days followed by Cancer (36.5 days) and Heart disease (26.8 days).


## Q3: how much money is billed yearly, Is there a trend in total billing amount over the years?
Built a Common Table Expression (CTE) to extract year from date of admission and then used this CTE to get total billed amount per year.

```sql
With year_date as (select
extract (year from date_of_admission) as Year, billing_amount
from `practise-sql-505810.Healthcare_data.Patient_details`)

select year,
sum (Billing_Amount) As total_billing_amount
from year_date
group by year 
order by total_billing_amount desc
```

Results - Total billed amount was consistent and within the range ($239M-$245M) from 2020 to 2023. However, the amount appeared to drop sharply in 2019 and 2024.

Further explored the reason for sharp drop in amount for those to years (2019 and 2024)

```sql
Select min(Date_of_Admission) as earliest_date, max(Date_of_Admission) as latest_date
From `practise-sql-505810.Healthcare_data.Patient_details`
```

Results - 2019 and 2024 only contain partial data as the data spans through May 2019 to May 2024. There was no meaningful trend observed in the total amount billed over the years.


## Q4. Are there any monthly/seasonal trends in admissions or billing?

Extracted month from date of admission data and then added this to CTE to calculate average billing amount and admission count in each month.

```sql
with monthly_admissions AS (select 
extract (month from date_of_admission) as Month, billing_amount
from `practise-sql-505810.Healthcare_data.Patient_details`)

select month,
avg (billing_amount) AS avgmonthly_billing_amount,
count (*) As total_admissions
from monthly_admissions
group by month
order by month
```

Results - Neither patient admission volume nor average billing amount showed meaningful monthly variation. Admission counts varied by ~13.6% between the highest (August, 4,832) and lowest (February, 4,255) months, while average billing per patient varied by only ~5% (January $22,178 vs October $21,091). Given the consistently narrow spread across both measures, no genuine seasonal trend is evident.

## Q5. Are certain hospitals associated with high billing amounts?

Calculated the total amount each hospital billed and observed that the spread across all hospitals was small. Calculated patient count and average billing amount per hospital to understand the trend observed in total billing amount from each hospital.

```sql
Select hospital,
sum (Billing_Amount) as hospital_total_billing_amount,
count (*) As patient_count, 
Avg(Billing_Amount) As Avg_billingamount
from `practise-sql-505810.Healthcare_data.Patient_details`
group by hospital 
order by avg_billingamount desc
```

Results - Patient count and average billing amount both showed a similarly narrow spread (~3-4%), consistent with the minimal variation observed across other dimensions (months, hospitals overall). This further supports the conclusion that this synthetic dataset was generated with fairly uniform distributions, rather than reflecting genuine differences in hospital scale or pricing.


## Key Findings
- Cancer and Heart Disease had the highest average billing amounts
- Alzheimer's patients had significantly longer stays (average 54 days)
- No meaningful seasonal or hospital-level billing trends identified
- Dataset limitations noted: synthetic generation, no unique patient ID

## Conclusion

Cancer and Alzheimer's patients consistently showed the highest billing amounts and longest lengths of stay — suggesting these conditions place the greatest financial and operational burden on the hospitals.
No significant trends were found across hospitals, months, or years — likely a reflection of this being a synthetically 
generated dataset rather than real-world data.

Note: This dataset only includes billing amounts, not actual treatment costs. To fully determine whether money is being lost, cost data would be needed. These findings highlight which conditions and areas are worth investigating further.
