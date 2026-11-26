# Quality checks of Bronze Layer (Before loading the Silver Layer)
Here we perform certain transformations in order to clean Bronze Layer and make ready for loading to Silver Layer 
- Removal of unwanted spaces (either leading spaces or lagging spaces)
- Creating derived columns for analytics
- Handling missing/null values
- Data Normalization and Standardization
- Datatype casting
- Data enrichment for advanced analytics
as applicable.
---

### Customer Table (crm_cust_info)

---

#### Checking for NULLs or Duplicates in Primary Key :-
```SQL
SELECT cst_id, COUNT(*) AS duplicate_count
FROM bronze.crm_cust_info
GROUP BY cst_id
HAVING COUNT(*) > 1 OR cst_id IS NULL;
```
<img width="197" height="177" alt="image" src="https://github.com/user-attachments/assets/a891e2b4-2af4-4859-80d3-aac270486e78" />

#### Checking for Unwanted Spaces :-
```SQL
SELECT cst_firstname AS fn, cst_lastname AS ln
FROM bronze.crm_cust_info
WHERE cst_firstname != TRIM(cst_firstname) OR cst_lastname != TRIM(cst_lastname);
```
<img width="197" height="177" alt="image" src="https://github.com/user-attachments/assets/1eead304-aacd-4fa5-9ec8-df65cec537e8" />

#### Checking Data Standardization & Consistency :-
```SQL
SELECT DISTINCT cst_gndr AS gndr, cst_marital_status AS ms
FROM bronze.crm_cust_info;
```
<img width="197" height="177" alt="image" src="https://github.com/user-attachments/assets/6175c1d1-66e2-495d-9dcf-bf6ba1e3c633" />

---

### Product table (crm_prd_info)

---

Similarly performed and fixed the above checks.

Additionally, Difference in prd_key column in ERP vs CRM (CO_RF vs CO-RF). 

```SQL
SELECT
	prd_id,
	prd_key,
	SUBSTRING(prd_key, 1, 5) AS cat_id,
	prd_nm,
	prd_cost,
	prd_line,
	prd_start_dt,
	prd_end_dt
FROM bronze.crm_prd_info
```
vs
```SQL
SELECT DISTINCT id FROM bronze.erp_px_cat_g1v2
```
Which gives :
<img width="197" height="177" alt="image" src="https://github.com/user-attachments/assets/addec021-5173-410e-bfc5-67cec7418dd5" /> vs <img width="197" height="177" alt="image" src="https://github.com/user-attachments/assets/df106118-de63-46c9-8474-8611990b2482" />

In order to join later, we need consistency in the first-half of prd_key. Therefore, replacing - with _ and verifying. Similarly, we need consistency in the second-half of prd_key.
```SQL
SELECT
	prd_id,
	REPLACE(SUBSTRING(prd_key, 1, 5), '-', '_') AS cat_id, -- first half
	SUBSTRING(prd_key, 7, LEN(prd_key)) as prd_key, -- second half
	prd_nm,
	prd_cost,
	prd_line,
	prd_start_dt,
	prd_end_dt
FROM bronze.crm_prd_info
```

#### Checking for NULLs or Negative Values in Cost :-
```SQL
SELECT prd_cost
FROM bronze.crm_prd_info
WHERE prd_cost < 0 OR prd_cost IS NULL;
```
<img width="197" height="177" alt="image" src="https://github.com/user-attachments/assets/7c6b84aa-3cca-47f4-a9c5-295373a9d59a" />

#### Checking for Invalid Date Orders (Start Date > End Date) :-
```SQL
SELECT *
FROM bronze.crm_prd_info
WHERE prd_end_dt < prd_start_dt;
```
<img width="827" height="287" alt="image" src="https://github.com/user-attachments/assets/f2e15161-7719-462e-b192-a85820725af3" />

Some additional inconsistency was observed especially with prd_keys 'AC-HE-HL-U509-R' and 'AC-HE-HL-U509'

<img width="671" height="287" alt="image" src="https://github.com/user-attachments/assets/c791308a-d722-4341-8df9-19361b805e0a" />

which was fixed using logic - [End Date = Start Date of the next record - 1]. Additionally, casting both prd_start_dt and prd_end_dt as DATE since no time info is provided in the source.
```SQL
SELECT
	prd_id,
	REPLACE(SUBSTRING(prd_key, 1, 5), '-', '_') AS cat_id,
	SUBSTRING(prd_key, 7, LEN(prd_key)) as prd_key,
	prd_nm,
	ISNULL(prd_cost, 0) AS prd_cost,
	CASE WHEN UPPER(TRIM(prd_line)) = 'M' THEN 'Mountain'
		 WHEN UPPER(TRIM(prd_line)) = 'R' THEN 'Road'
		 WHEN UPPER(TRIM(prd_line)) = 'S' THEN 'Other Sales'
		 WHEN UPPER(TRIM(prd_line)) = 'T' THEN 'Touring'
		 ELSE 'Not Specified'
	END AS prd_line,
	CAST (prd_start_dt AS DATE) AS prd_start_dt,
	CAST (LEAD(prd_start_dt) OVER(PARTITION BY prd_key ORDER BY prd_start_dt)-1 AS DATE) AS prd_end_dt
FROM bronze.crm_prd_info;
```
<img width="827" height="290" alt="image" src="https://github.com/user-attachments/assets/8e380bc8-92d5-4a3c-bad8-91ccc56d691d" />

---

### Sales table (crm_sales_details)

---

Similarly performed and fixed the above checks.

#### Checking for Invalid Dates :-
Some dates contain 0 or are not the right length so replacing them with NULL
```SQL
SELECT
NULLIF(sls_order_dt, 0) AS sls_order_dt
FROM bronze.crm_sales_details
WHERE sls_order_dt <= 0
OR LEN(sls_order_dt) != 8
OR sls_order_dt > 20500101
OR sls_order_dt < 19000101;
```
and similarly for other dates (sls_ship_dt and sls_due_dt). Further transforming them
```SQL
SELECT
sls_ord_num,
sls_prd_key,
sls_cust_id,
CASE WHEN sls_order_dt = 0 OR LEN(sls_order_dt) != 8 THEN NULL
	 ELSE CAST(CAST(sls_order_dt AS VARCHAR) AS DATE)
END AS sls_order_dt,
CASE WHEN sls_ship_dt = 0 OR LEN(sls_ship_dt) != 8 THEN NULL
	 ELSE CAST(CAST(sls_ship_dt AS VARCHAR) AS DATE)
END AS sls_ship_dt,
CASE WHEN sls_due_dt = 0 OR LEN(sls_due_dt) != 8 THEN NULL
	 ELSE CAST(CAST(sls_due_dt AS VARCHAR) AS DATE)
END AS sls_due_dt,
sls_sales,
sls_quantity,
sls_price
FROM bronze.crm_sales_details
```

#### Checking for Invalid Date Orders (Order Dates > Shipping/Due Dates) :-
```SQL
SELECT *
FROM bronze.crm_sales_details
WHERE sls_order_dt > sls_ship_dt OR sls_order_dt > sls_due_dt;
```

#### Checking Data Consistency: Sales = Quantity * Price :-
```SQL
SELECT DISTINCT
sls_sales, sls_quantity, sls_price
FROM bronze.crm_sales_details
WHERE sls_sales != sls_quantity * sls_price
   OR sls_sales IS NULL OR sls_quantity IS NULL OR sls_price IS NULL
   OR sls_sales <= 0 OR sls_quantity <= 0 OR sls_price <= 0
ORDER BY sls_sales, sls_quantity, sls_price;
```
This gives us wrong results as well as calculations. Therefore, we fix this
```SQL
SELECT
sls_ord_num,
sls_prd_key,
sls_cust_id,
CASE WHEN sls_order_dt = 0 OR LEN(sls_order_dt) != 8 THEN NULL
	 ELSE CAST(CAST(sls_order_dt AS VARCHAR) AS DATE)
END AS sls_order_dt,
CASE WHEN sls_ship_dt = 0 OR LEN(sls_ship_dt) != 8 THEN NULL
	 ELSE CAST(CAST(sls_ship_dt AS VARCHAR) AS DATE)
END AS sls_ship_dt,
CASE WHEN sls_due_dt = 0 OR LEN(sls_due_dt) != 8 THEN NULL
	 ELSE CAST(CAST(sls_due_dt AS VARCHAR) AS DATE)
END AS sls_due_dt,
CASE WHEN sls_sales IS NULL OR sls_sales <= 0 OR sls_sales != sls_quantity * ABS(sls_price)
		THEN sls_quantity * ABS(sls_price)
		ELSE sls_sales
END AS sls_sales,
sls_quantity,
CASE WHEN sls_price IS NULL OR sls_price <= 0
		THEN sls_sales / NULLIF(sls_quantity,0)
		ELSE sls_price
END AS sls_price
FROM bronze.crm_sales_details
```

---

### Extra Customer information table (erp_cust_az12)

---

Old data has 'NAS' prefix therefore need to take care of it because new data (post join) does not have that prefix
```SQL
SELECT
CASE WHEN cid LIKE 'NAS%' THEN SUBSTRING(cid, 4, LEN(cid))
     ELSE cid
END AS cid,
bdate,
gen
FROM bronze.erp_cust_az12
```

#### Identifying Out-of-Range Dates (Birthdates between 1924-01-01 and Today) :-
```SQL
SELECT DISTINCT bdate
FROM bronze.erp_cust_az12
WHERE bdate < '1924-01-01'
   OR bdate > GETDATE();
```
and fix it. Also taking care of standardization and normalization
```SQL
SELECT
CASE WHEN cid LIKE 'NAS%' THEN SUBSTRING(cid, 4, LEN(cid))
     ELSE cid
END AS cid,
CASE WHEN bdate > GETDATE() THEN NULL
     ELSE bdate
END AS bdate,
CASE WHEN UPPER(TRIM(gen)) IN ('F','FEMALE') THEN 'Female'
     WHEN UPPER(TRIM(gen)) IN ('M','MALE') THEN 'Male'
     ELSE 'Not Specified'
END AS gen
FROM bronze.erp_cust_az12
```

---

### Customer location table (erp_loc_a101)

---

Similarly replacing '-' with '_' and standardizing and normalizing
```SQL
SELECT
REPLACE(cid, '-', '') AS cid,
CASE WHEN TRIM(cntry) = 'DE' THEN 'Germany'
     WHEN TRIM(cntry) IN ('US', 'USA') THEN 'United States'
     WHEN TRIM(cntry) = '' OR cntry IS NULL THEN 'n/a'
     ELSE TRIM(cntry)
END AS cntry
FROM bronze.erp_loc_a101;
```

---

### Product categories table (erp_px_cat_g1v2)

---
We had already performed relevant changes therefore nothing required here
```SQL
SELECT
id,
cat,
subcat,
maintenance
FROM bronze.erp_px_cat_g1v2;
```

# NOTE - Update the DDL scripts accordingly 
