
-- Step 1: Data Overview & Scope Validation
-- Description: Check the global analysis period, total unique users, and total sessions.

SELECT 
  MIN(Timestamp) AS project_start_date,
  MAX(Timestamp) AS project_end_date,
  COUNT(DISTINCT UserID) AS total_unique_users,
  COUNT(DISTINCT SessionID) AS total_sessions
FROM `project-2e3479d4-e6b2-4047-a87.Funnel_Project.Customer_journey` cj;

------------------

-- Step 2: Main Conversion Funnel & Progression Rates
-- Description: Aggregate multi-stage sessions and calculate step-by-step conversion rates.

WITH FunnelStages AS (
    SELECT
        COUNT(DISTINCT CASE WHEN PageType = 'home' THEN SessionID END) AS `stage_1_home`,
        COUNT(DISTINCT CASE WHEN PageType = 'product_page' THEN SessionID END) AS `stage_2_product`,
        COUNT(DISTINCT CASE WHEN PageType = 'cart' THEN SessionID END) AS `stage_3_cart`,
        COUNT(DISTINCT CASE WHEN PageType = 'checkout' THEN SessionID END) AS `stage_4_checkout`,
        COUNT(DISTINCT CASE WHEN PageType = 'confirmation' THEN SessionID END) AS `stage_5_confirmation`
    FROM `project-2e3479d4-e6b2-4047-a87.Funnel_Project.Customer_journey` cj     
)
SELECT
    stage_1_home AS `1_Home`,
    stage_2_product AS `2_Product_Page`,
    stage_3_cart AS `3_Cart`,
    stage_4_checkout AS `4_Checkout`,
    stage_5_confirmation AS `5_confirmation`,

    ROUND(stage_2_product / stage_1_home * 100, 2) AS `Home_to_Product_rate`,
    ROUND(stage_3_cart / stage_2_product * 100, 2) AS `Product_to_Cart_rate`,
    ROUND(stage_4_checkout / stage_3_cart * 100, 2) AS `Cart_to_Checkout_rate`,
    ROUND(stage_5_confirmation / stage_1_home * 100, 2) AS `Total_Conversion_rate`
FROM FunnelStages;

------------------


-- Step 3: Funnel Segmentation by Device Type
-- Description: Compare core conversion metric across Desktop, Mobile, and Tablet.

SELECT
    DeviceType,
    COUNT(DISTINCT CASE WHEN PageType = 'home' THEN SessionID END) AS home_cnt,
    COUNT(DISTINCT CASE WHEN PageType = 'cart' THEN SessionID END) AS cart_cnt,
    COUNT(DISTINCT CASE WHEN PageType = 'confirmation' THEN SessionID END) AS purchase_cnt,
    ROUND((COUNT(DISTINCT CASE WHEN PageType = 'confirmation' THEN SessionID END) / 
           COUNT(DISTINCT CASE WHEN PageType = 'home' THEN SessionID END)) * 100, 2) AS overall_conversion_rate
FROM `project-2e3479d4-e6b2-4047-a87.Funnel_Project.Customer_journey` cj 
GROUP BY DeviceType
ORDER BY overall_conversion_rate DESC;

------------------

-- Step 4: Purchase Lead Time Analysis (Time-to-Conversion)
-- Description: Calculate the average time (Seconds/Minutes) taken from first entry to final purchase.

WITH SessionTime AS (
    SELECT  
      SessionID,
      MIN(CASE WHEN PageType = 'home' THEN Timestamp END) AS first_visit,
      MAX(CASE WHEN PageType = 'confirmation' THEN Timestamp END) AS purchase_time
    FROM `project-2e3479d4-e6b2-4047-a87.Funnel_Project.Customer_journey` cj 
    GROUP BY SessionID
    HAVING purchase_time IS NOT NULL AND first_visit IS NOT NULL
)
SELECT  
    AVG(TIMESTAMP_DIFF(purchase_time, first_visit, SECOND)) AS avg_lead_time_seconds,
    AVG(TIMESTAMP_DIFF(purchase_time, first_visit, MINUTE)) AS avg_lead_time_minutes
FROM SessionTime;