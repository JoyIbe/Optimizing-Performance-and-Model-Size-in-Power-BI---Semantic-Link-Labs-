# Optimizing Performance and Model Size in PowerBI Using Semantic Link Labs

## DOCUMENTATION

## Project Title: Van Arsdel Dashboard Performance Optimization

## Objective: 
To enhance the performance and reduce the size of the Van Arsdel quarterly executive dashboard.

**Initial State:**
The dashboard suffered from slow load times (10-15 seconds), a large model size (192 MB), and visual timeouts, leading to lost productivity and delayed decision-making.

[Here](https://github.com/DonFrancis1/FabricSaturdayChallenges/blob/main/Challenge%202%20-%20%20Optimizing%20Performance%20and%20Size%20of%20Power%C2%A0BI%C2%A0model/PROJECT_DETAILS.md)

**Key Actions:**

- Streamlined data model and query steps.
- Optimized data relationships.
- Improved DAX measure efficiency.
- Implemented Power BI best practices.

**Results:**

- Dashboard load time reduced to under 2 seconds.
- Data model size reduced from 192 MB to 20 MB, a 90% reduction.
- Performance is now stable, ensuring a seamless user experience for stakeholders.


## Data Model and Query Optimization

**Challenge:** A large model size was the primary cause of slow performance.

**Initial State:**

<img width="776" height="338" alt="Initial Model" src="https://github.com/user-attachments/assets/b6286c2b-9ca7-4d80-9415-b10793db510f" />

**- Unnecessary columns:** The model contained heavy, unreferenced columns that inflated its size.
**- Inefficient joins:** The relationships were not configured for optimal performance, including a problematic many-to-many relationship with bi-directional filtering.
**- Automatic features:** Unused features like "Auto Date/Time" added hidden complexity.

**Actions Taken:**

**- Rectified data sources:** Disaabled unused tables and eliminated unreferenced and redundant columns directly in Power Query to reduce memory footprint.

**- Optimized relationships:** Replaced the problematic many-to-many relationship between the fact table and the product table with an efficient one-to-many relationship using a bridge table.

**- Disabled bi-directional filters:** Switched from bi-directional to single-directional filtering on key relationships, which improves query performance by reducing context propagation.

**- Disabled Auto Date/Time:** Turned off this feature to prevent the creation of hidden, resource-intensive calendar tables for every date column.

**Updated Model**

<img width="704" height="337" alt="Updated Model" src="https://github.com/user-attachments/assets/ce82b283-1c17-4202-81a9-a149d4fc51b9" />


## DAX and Measure Refinement

**Challenge:** Inefficient DAX measures caused visuals to time out.

**Initial State:**

DAX measures were not optimized, leading to slow rendering times. For example, a measure for "Revenue Last Year" was using AVERAGEX and EARLIER which is highly inefficient for time intelligence calculations or nested SUMX + FILTER + EARLIER, this are  row-by-row evaluation which forces the storage engine to scan the fact table multiple times, which explains the 5,500 ms runtime.

**Actions Taken:**

**- Replaced inefficient DAX:** Refactored DAX measures using standard time intelligence functions such as DATEADD and replacing AUTOCALENDAR for significant performance gains.

**Disabled unused tables:** Deactivated unused tables like InternationalSales, USASales, and Transform sample, which reduced the model size and improved query processing by preventing the engine from scanning unnecessary data.


## Performance Validation

<img width="918" height="587" alt="Dashboard Layouts (19)" src="https://github.com/user-attachments/assets/92a75da9-5dd3-418f-8b91-619dbd3306cc" />

**Results: Performance is now stable, and the dashboard is highly responsive.**

**- Model Size:**

    - Before: 192 MB
    
    - After: 20 MB



<img width="918" height="587" alt="Dashboard Layouts (18)" src="https://github.com/user-attachments/assets/04edd983-156d-421a-b4c5-cc2c72e1b6b7" />

**- Query Performance:**

    - Before: 10-15 seconds load time
    
    - After: Visuals load in under 2 seconds.

Tooling: The analysis and optimization were guided by Power BI's built-in Performance Analyzer, along with best practice recommendations from tools like DAX Studio and Bravo for Power BI.



## Recommendation from Model Best Practice Analyzer - Semantic Lab Link

Highlights of few unique recommendations:

- Remove Inactive Relationships: Deleting inactive relationships from your model simplifies the data flow and can improve performance by reducing the number of potential join paths for the DAX engine to evaluate.


- Turn Off Auto Date/Time: This feature automatically creates a hidden date table for every date column, which can significantly increase your model's size. By turning it off, you can use a single, explicit date table, which is a best practice for time intelligence.


- Remove Unreferenced Columns: Columns that are not used in your visuals, measures, or filters should be removed from the data model. This directly reduces the model size and the amount of data the engine needs to process, leading to faster load times and better performance.


- Avoid Bi-directional Cross-filtering: Bi-directional filters can cause ambiguity and degrade performance. They are best avoided and should be replaced with single-directional filters or a bridge table.


- Use KEEPFILTERS in CALCULATE: This recommendation is for specific DAX measures where you want to add new filters to an expression without removing the existing ones. Using KEEPFILTERS prevents the filter context from being cleared, which can lead to more predictable and potentially more performant results.


- Use Variables to Store Calculated Values: When a calculation is used multiple times within a single measure, storing the value in a VAR variable can prevent redundant calculations and improve performance.

- Hide Columns from Client Tools: Hiding columns that are not used in visuals or measures reduces the clutter in the Fields pane and can improve query performance by making the data model leaner. This is a good practice for user experience and efficiency.



### Corrected DAX Measures

**Total Sales YTD**

From
``Total Sales YTD = 
CALCULATE (
    SUM ( 'All_Sales'[Revenue] ),
    FILTER (
        ALL ( 'Calendar'[Date] ),
        'Calendar'[Date] <= MAX ( 'Calendar'[Date] )
            && YEAR ( 'Calendar'[Date] ) = YEAR ( MAX ( 'Calendar'[Date] ) )
    )
)
`` 

To

``Total Sales YTD =
TOTALYTD (SUM ( 'All_Sales'[Revenue] ), 'Calendar'[Date])``


**Revenue Last Year**

``Revenue Last Year = 
AVERAGEX (
    VALUES ( 'Calendar'[Year] ),
    CALCULATE (
        SUM ( 'All_Sales'[Revenue] ),
        FILTER ( ALL ( 'Calendar' ), 'Calendar'[Year] = EARLIER ( 'Calendar'[Year] ) - 1 )
    )
)
``

To

``
Revenue Last Year =
CALCULATE (
    [TotalSales],
    FILTER ( ALL ( 'Calendar' ),'Calendar'[Year] = MAX ( 'Calendar'[Year] ) - 1
     && 'Calendar'[Date] <= MAX ( 'Calendar'[Date] )
    )
)
``

**Revenue Last 60 Day**

``
Revenue Last 60 Days = 
SUMX( 
FILTER( 
ALL(All_Sales), 
All_Sales[Date] >= MAX(All_Sales[Date]) - 60 &&
All_Sales[Date] <= MAX(All_Sales[Date]) ), All_Sales[Revenue] )
``

To

``
Revenue Last 60 Days =
CALCULATE (
    SUM ( All_Sales[Revenue] ),
    DATESINPERIOD ('Calendar'[Date],MAX ( 'Calendar'[Date] ),-60, DAY)
)
``

**Best Product Last 60 Days**

``
Best Product Last 60 Days =
VAR MaxDate = MAX(All_Sales[Date])
VAR DateRange = FILTER(
    ALL(All_Sales[Date]),
    All_Sales[Date] >= MaxDate - 60 &&
    All_Sales[Date] <= MaxDate
)
VAR ProductSales = SUMMARIZE(
    FILTER(
        All_Sales,
        All_Sales[Date] IN DateRange
    ),
    All_Sales[ProductID],
    "TotalUnits", SUM(All_Sales[Units])
)
VAR MaxUnits = MAXX(ProductSales, [TotalUnits])
VAR TopProductID = MAXX(
    FILTER(ProductSales, [TotalUnits] = MaxUnits),
    All_Sales[ProductID]
)
RETURN
CALCULATE(
    VALUES(Product[Product]),
    FILTER(Product, Product[ProductID] = TopProductID)
)
``

To

``
VAR DateRange =
    DATESINPERIOD (
        'Calendar'[Date],MAX ( 'Calendar'[Date] ),-60, DAY
    )
VAR TopProduct =
    TOPN ( 1, SUMMARIZECOLUMNS ( Product[Product], "TotalUnits",
         CALCULATE ( SUM ( All_Sales[Units] ), DateRange)
        ), [TotalUnits], DESC, Product[Product], ASC
    )
RETURN
MAXX ( TopProduct, Product[Product] )
``

## Semantic Modeling

**Model Best Practice Analyzer**

Code Snippet

```
import sempy_labs as labs

dataset = '' # Enter the name or ID of your semantic model
workspace = None # Enter the name or ID of the workspace in which the semantic model resides

labs.run_model_bpa(dataset=dataset, workspace=workspace) #Standard BPA
labs.run_model_bpa(dataset=dataset, workspace=workspace, extended=True) # Setting extended=True will fetch Vertipaq Analyzer statistics and use them to run advanced BPA rules against your model
labs.run_model_bpa(dataset=dataset, workspace=workspace, export=True) # Setting export=True will export the results to a delta table in the lakehouse attached to the notebook
labs.run_model_bpa(dataset=dataset, workspace=workspace, language="French") # Setting the 'language' parameter will dynamically translate the rules, categories and descriptions to the specified language.

```
