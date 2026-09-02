Problem - "We think we're losing money on certain types of patients. Can you look into it?" 



Questions 5W1H



What -  what medical conditions have the highest billing amount?

What is the average length of stay per medical condition?



When - What is yearly total revenue, Is there a trend in total billing amount over the years? Are there any monthly/seasonal trends in admissions or billing?



Who - Are certain hospitals associated with high billing amounts?





\## Data cleaning checks



Nulls



SELECT \*

FROM `practise-sql-505810.healthcare\_dataset.patient\_details`

select \*

from `practise-sql-505810.Healthcare\_data.Patient\_details`

WHERE Name IS NULL 

&#x20;  OR Age IS NULL 

&#x20;  OR Gender IS NULL 

&#x20;  OR Blood\_Type IS NULL 

&#x20;  OR Medical\_Condition IS NULL 

&#x20;  OR Date\_of\_Admission IS NULL 

&#x20;  OR Doctor IS NULL 

&#x20;  OR Hospital IS NULL 

&#x20;  OR Insurance\_Provider IS NULL 

&#x20;  OR Billing\_Amount IS NULL 

&#x20;  OR Room\_Number IS NULL

&#x20;  OR Admission\_Type IS NULL

&#x20;  OR Discharge\_Date IS NULL

&#x20;  OR medication IS NULL

&#x20;  OR `Test \_Results` IS NULL

&#x20;  OR Length\_of\_Stay IS NULL



result - Nil null found



Duplicate 



syntax: SELECT Name, Date\_of\_Admission, COUNT(\*) AS occurrences

FROM `practise-sql-505810.Healthcare\_data.Patient\_details`

GROUP BY Name, Date\_of\_Admission

HAVING COUNT(\*) > 1



checked individual details of a duplicate value 



syntax: select \*

from `practise-sql-505810.Healthcare\_data.Patient\_details`

where Name = "Emily King"



Initial investigation using Name + Admission Date suggested \~5,509 potential duplicate records; however, deeper inspection revealed these were distinct patients coincidentally sharing a name and admission date — a known limitation of synthetic/generated datasets lacking unique patient identifiers.



different approach to find out actual duplicates 



select \*, count (\*) as occurences

from `practise-sql-505810.Healthcare\_data.Patient\_details`

group by name, age, Gender, Blood\_Type, Medical\_Condition, Date\_of\_Admission, doctor, Hospital, Insurance\_Provider, Billing\_Amount,Room\_Number, Admission\_Type, Discharge\_Date, Medication, `Test \_Results`, Length\_of\_Stay

having count(\*) >1



results - 0 true duplicate values 



Q1: Which Medical Conditions have the highest average Billing Amount?



syntax: select Medical\_Condition,

avg (Billing\_Amount) as avg\_billingamount

from `practise-sql-505810.Healthcare\_data.Patient\_details`

group by Medical\_Condition

order by avg(Billing\_Amount) desc



Results: 

Row	Medical\_Condition	avg\_billingamount

1	Cancer	64537.088666791176

2	Heart Disease	44913.427048720165

3	Alzheimer’s	32543.542291466209

4	Diabetes	12503.18846986499

5	Obesity	10055.563054532904

6	Asthma	5025.3482901469406

7	Infections	2747.5674928981239

8	Flu	2744.1520501681034







Q2: What is the average length of stay per medical condition?





syntax: select Medical\_Condition,

avg (Length\_of\_Stay) As avg\_lengthofstay

from `practise-sql-505810.Healthcare\_data.Patient\_details`

group by Medical\_Condition



Results: Row	Medical\_Condition	avg\_lengthofstay

1	Infections	5.5219106047327058

2	Flu	2.5049673573658877

3	Asthma	3.5036189924725036

4	Obesity	5.96639977123251

5	Diabetes	8.0630977872948151

6	Heart Disease	26.860579710144933

7	Cancer	36.543804034582159

8	Alzheimer’s	54.41728610989648



Alzheimer's had the highest average length of stay.





Q3: What is yearly total revenue, Is there a trend in total billing amount over the years?



Built a CTE to extract year from date and then used this CTE to get total revenue per year 



syntax: 

With year\_date as (select

extract (year from date\_of\_admission) as Year, billing\_amount

from `practise-sql-505810.Healthcare\_data.Patient\_details`)



select year,

sum (Billing\_Amount) As total\_revenue

from year\_date

group by year 

order by total\_revenue desc



Results - revenue appeared to drop sharply in 2019 and 2024



Verified if the low revenue is a genuine low revenue or incomplete data 



Syntax:

SELECT MIN(Date\_of\_Admission) AS earliest\_date, MAX(Date\_of\_Admission) AS latest\_date

FROM `practise-sql-505810.Healthcare\_data.Patient\_details`



Results - 2019 and 2024 only contain partial data as the data spans through May 2019 to May 2024. There is no meaningful trend in yearly total revenue.





Q4. Are there any monthly/seasonal trends in admissions or billing?



extracted Month from date data and then added this to CTE to calculate total monthly revenue from all data and admission count in each month.



syntax: with monthly\_admissions AS (select 

extract (month from date\_of\_admission) as Month, billing\_amount

from `practise-sql-505810.Healthcare\_data.Patient\_details`)



select month,

Sum (billing\_amount) AS total\_revenue,

count (\*) As total\_admissions

from monthly\_admissions

group by month

order by month



results: August, July, and June showed the highest admissions and highest total revenue, while February was the lowest for both. However, the spread was modest for both measures — approximately 13.6% for admissions and 14% for revenue between the highest and lowest months. Given this consistent, relatively narrow spread across two independent measures, and the synthetic nature of this dataset, this is more likely natural random variation than a genuine seasonal healthcare pattern.



Q5. Are certain hospitals associated with high billing amounts?



calculated total revenue from each hospital and observed that the spread across all hospitals revenue is small. Calculated patient count and average billing amount per hospital to understand whether similar totals were driven by similar patient volume, similar per-patient billing, or a mix of both.



syntax: Select hospital,

sum (Billing\_Amount) as hospital\_totalrevenue,

count (\*) As patient\_count, 

Avg(Billing\_Amount) As Avg\_billingamount

from `practise-sql-505810.Healthcare\_data.Patient\_details`

group by hospital 

order by avg\_billingamount desc



Result: Patient count and average billing amount both showed a similarly narrow spread (\~3-4%) between the highest and lowest hospitals — consistent with the minimal variation observed across other dimensions (months, hospitals overall). This further supports the conclusion that this synthetic dataset was generated with fairly uniform distributions, rather than reflecting genuine differences in hospital scale or pricing.

