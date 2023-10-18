# Sales-Projects-SQL

In this project, I used SQL server to deal with the sales data from a FMCG brand. The work involved advanced join, union, subquery, update, case-when, rank over to answer the following question:

- Use the newly created April and June sales table to find a list of customers who bought in April but did not buy in June and vice versa.

- Compare each customer's purchase amount in April and June.

- Calculate the sum of the June and April net gains for each store.

- Take out the total transaction amount of each store in 2019 and see what percentage it represents compared to the total transaction amount of each store in the sales table.

- Get the largest transaction amount of each employee with each store in 2019 and see what proportion of the total transaction of that employee in the store in 2019

- Calculate the total transaction amount of each employee with each store in each month of 2019 to see what proportion of the store's total revenue in each of those months of 2019.

- Update table Discount column MucChietKhau = 0.01 if MucToiThieu from 0 > 5, MucChietKhau = 0.02 if MucToiThieu from 6 > 10, MucChietKhau = 0.03 if MucToiThieu from 11 > 30, the remaining discount is 0.05

- Get the transaction information with the largest amount by day and by store

- Get the customer with the largest amount of money by day and by store

Here are my SQL queries:

**1. Use the newly created April and June sales table to find a list of customers who bought in April but did not buy in June and vice versa.**

```SELECT *
INTO Task..BANHANG_201904
FROM Task..BANHANG
WHERE YEAR(TRANS_DATE) = 2019 AND MONTH(TRANS_DATE) = 04

SELECT *
INTO Task..BANHANG_201906
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019 AND MONTH(TRANS_DATE) = 06

SELECT A.CUS_ID
FROM Task..BANHANG_201904 A
LEFT JOIN Portfolio..BANHANG_201906 B
	ON A.CUS_ID = B.CUS_ID
WHERE B.CUS_ID IS NULL
UNION
SELECT A.CUS_ID
FROM Task..BANHANG_201906 A
LEFT JOIN Portfolio..BANHANG_201904 B
	ON A.CUS_ID = B.CUS_ID
WHERE B.CUS_ID IS NULL```

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/4d0905bb-1748-4502-ac31-1fc3b94ae333)

**2. Compare each customer's purchase amount in April and June.**

`SELECT
	ISNULL(A.CUS_ID, B.CUS_ID) CUS_ID,
	ISNULL(A.TOTAL_AMT_04,0) TOTAL_AMT_04,
	ISNULL(B.TOTAL_AMT_06,0) TOTAL_AMT_06,
	(ISNULL(B.TOTAL_AMT_06,0) - ISNULL(A.TOTAL_AMT_04,0)) [NET 06vs04]
FROM
(
SELECT 
	CUS_ID, 
	SUM(LCY_AMT) TOTAL_AMT_04
FROM Task..BANHANG_201904
GROUP BY CUS_ID
) A
FULL JOIN
(
SELECT 
	CUS_ID, 
	SUM(LCY_AMT) TOTAL_AMT_06
FROM Task..BANHANG_201906
GROUP BY CUS_ID
) B
ON A.CUS_ID = B.CUS_ID`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/5ad97fa1-00e1-48f5-ab19-16ae7bddc57f)

**3. Calculate the sum of the June and April net gains for each store.**

`SELECT
	ISNULL(A.STOREDID, B.STOREDID) STOREDID,
	ISNULL(A.TOTAL_AMT_04,0) TOTAL_AMT_04,
	ISNULL(B.TOTAL_AMT_06,0) TOTAL_AMT_06,
	(ISNULL(B.TOTAL_AMT_06,0) - ISNULL(A.TOTAL_AMT_04,0)) [NET 06vs04] 
FROM
(
SELECT 
	STOREDID, 
	SUM(LCY_AMT) TOTAL_AMT_04
FROM Task..BANHANG_201904
GROUP BY STOREDID
) A
FULL JOIN
(
SELECT 
	STOREDID,  
	SUM(LCY_AMT) TOTAL_AMT_06
FROM Task..BANHANG_201906
GROUP BY STOREDID
) B
ON A.STOREDID = B.STOREDID`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/22f9f376-1432-400f-bbcb-9be0f303feee)

**4. Take out the total transaction amount of each store in 2019 and see what percentage it represents compared to the total transaction amount of each store in the sales table.**

`SELECT 
	ISNULL(A.STOREDID, B.STOREDID) STOREDID,
	ISNULL(A.AMT_2019,0) AMT_2019,
	ISNULL(B.TOTAL_AMT,0) TOTAL_AMT,
	CAST((ISNULL(A.AMT_2019,0)/B.TOTAL_AMT)*100 AS DECIMAL(10,2)) [% AMT_2019]
FROM
(
SELECT 
	STOREDID,
	SUM(LCY_AMT) AMT_2019
FROM Task..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY STOREDID
) A
FULL JOIN 
(
SELECT 
	STOREDID,
	SUM(LCY_AMT) TOTAL_AMT
FROM Task..BANHANG
GROUP BY STOREDID
) B
ON A.STOREDID = B.STOREDID`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/1f46e025-1030-41fa-bc63-00006b9e923a)

**5. Get the largest transaction amount of each employee with each store in 2019 and see what proportion of the total transaction of that employee in the store in 2019**

`SELECT 
	ISNULL(A.SALE_ID, B.SALE_ID) SALE_ID,
	ISNULL(A.STOREDID, B.STOREDID) STOREDID,
	ISNULL(A.MAX_AMT,0) MAX_AMT,
	ISNULL(B.TOTAL_AMT,0) TOTAL_AMT,
	CAST((ISNULL(A.MAX_AMT,0)/B.TOTAL_AMT)*100 AS DECIMAL(10,2)) [% MAX_AMT]
FROM
(
SELECT
	STOREDID,
	SALE_ID,
	MAX(LCY_AMT) MAX_AMT
FROM Task..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY STOREDID, SALE_ID
) A
FULL JOIN 
(
SELECT
	STOREDID,
	SALE_ID,
	SUM(LCY_AMT) TOTAL_AMT
FROM Task..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY STOREDID, SALE_ID
) B
ON 
	A.SALE_ID = B.SALE_ID AND
	A.STOREDID = B.STOREDID`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/f2bfaa88-4f69-4064-8f21-966f726c2896)

**6. Calculate the total transaction amount of each employee with each store in each month of 2019 to see what proportion of the store's total revenue in each of those months of 2019.**

`SELECT
	ISNULL(A.STOREDID,B.STOREDID) STOREDID,
	A.SALE_ID,
	ISNULL(A.[MONTH],B.[MONTH]) [MONTH],
	ISNULL(A.MONTH_AMT,0) MONTH_AMT,
	ISNULL(B.TOTAL_MONTH_AMT,0) TOTAL_MONTH_AMT,
	CAST((ISNULL(A.MONTH_AMT,0)/ISNULL(B.TOTAL_MONTH_AMT,0))*100 AS DECIMAL(10,4)) [% SALE_AMT]
FROM
(
SELECT
	STOREDID,
	SALE_ID,
	MONTH(TRANS_DATE) [MONTH],
	SUM(LCY_AMT) MONTH_AMT
FROM Task..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY
	STOREDID,
	SALE_ID,
	MONTH(TRANS_DATE)
) A
FULL JOIN
(
SELECT 
	STOREDID,
	MONTH(TRANS_DATE) [MONTH],
	SUM(LCY_AMT) TOTAL_MONTH_AMT
FROM Task..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY
	STOREDID,
	MONTH(TRANS_DATE)
) B
ON 
	A.STOREDID = B.STOREDID AND
	A.[MONTH] = B.[MONTH]`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/a85c4168-e69c-4dd2-9dd2-81b6a7905c08)

**7. Update table Discount column MucChietKhau = 0.01 if MucToiThieu from 0 > 5, MucChietKhau = 0.02 if MucToiThieu from 6 > 10, MucChietKhau = 0.03 if MucToiThieu from 11 > 30, the remaining discount is 0.05**

`UPDATE Task..DISCOUNT 
SET MUC_CHIET_KHAU = 
	CASE
		WHEN MUC_TOI_THIEU BETWEEN 0 AND 5 THEN 0.01
		WHEN MUC_TOI_THIEU BETWEEN 6 AND 10 THEN 0.02
		WHEN MUC_TOI_THIEU BETWEEN 11 AND 30 THEN 0.03
		ELSE 0.05
	END`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/fd7b7391-4f08-413e-b9ba-4fd27bd1d4fe)

**8. Get the transaction information with the largest amount by day and by store**

`WITH T AS
(
SELECT 
	*,
	RANK () OVER (PARTITION BY STOREDID, TRANS_DATE ORDER BY LCY_AMT DESC) AS RNK
FROM Task..BANHANG
)
SELECT 
	STOREDID,
	TRANS_DATE,
	CUS_ID,
	RETAIL_BILL,
	LCY_AMT
FROM T
WHERE RNK = 1
ORDER BY CAST(RIGHT(STOREDID,(LEN(STOREDID) - LEN('STORE '))) AS INT), TRANS_DATE`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/473936a0-0e7a-4862-a274-26c8a84e5130)

**9. Get the customer with the largest amount of money by day and by store**

`WITH T AS 
(
SELECT 
	STOREDID,
	TRANS_DATE,
	CUS_ID,
	SUM(LCY_AMT) [TOTAL AMOUNT],
	RANK () OVER (PARTITION BY STOREDID, TRANS_DATE ORDER BY SUM(LCY_AMT) DESC) AS RNK
FROM Task..BANHANG
GROUP BY
	STOREDID,
	TRANS_DATE,
	CUS_ID
)
SELECT *
FROM T 
WHERE RNK = 1
ORDER BY CAST(RIGHT(STOREDID,(LEN(STOREDID) - LEN('STORE '))) AS INT), TRANS_DATE`

![image](https://github.com/truongnc17/Sales-Projects-SQL/assets/131191379/8a181bf8-d8d2-4dd9-bce5-89371b8bd305)

