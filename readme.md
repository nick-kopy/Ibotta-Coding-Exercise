# Ibotta Coding Exercise

This coding exercise was completed by Nicholas Kopystynsky. Questions and data are from Ibotta.

## Part 1 - SQL

Some questions could be interpreted with some degree of flexibility, even if the code is very specific. As such I have included assumptions.

#### 1. Provide the state and average rebate_value for each state with an average rebate_value less than $0.50.

Assumptions: Rebate values and state information are in two separate tables with no explicit way to connect the data. I assumed connecting brand and week between the two tables could connect states to rebate values correctly. Campaigns that span multiple weeks or span multiple cities were counted multiple times and it was assumed that is acceptable.

>Code  

```sql
SELECT p.state, AVG(c.rebate_value) FROM campaign c  
JOIN pricing p ON c.brand = p.brand AND c.week = p.week  
WHERE p.state <> 0  
    AND state IS NOT NULL  
GROUP BY p.state  
HAVING AVG(c.rebate_value) < 0.5;  
```

>Output

|state | AVG(c.rebate_value)|
|----- | -------------------|
|AK  |   0.45787037037037   |
|AL |    0.452822966507177  |
|AR|     0.485393586005831  |
|AZ     |0.36613870246085   |
|CO    | 0.36406425102727   |
|CT   |  0.383367875647668  |
|GA  |   0.450027142303219  |
|IA |    0.495937142857143  |
|ID|     0.450104529616725  |
|IL     |0.47670928667564   |
|KS    | 0.498130841121495  |
|MD   |  0.487905660377359  |
|MN  |   0.377707607770299  |
|MO |    0.487270942867154  |
|MS|     0.283555555555556  |
|MT     |0.394512635379061  |
|ND    | 0.456708208955224  |
|NE   |  0.480843479107819  |
|NJ  |   0.331909699435621  |
|NM |    0.404751552795031  |
|NY    | 0.394574683544302  |
|SD   |  0.333920118343195  |
|TN  |   0.49073717948718   |
|TX |    0.418761974944729  |
|UT    | 0.408271428571429  |
|VT   |  0.4255             |
|WA  |   0.48418085106383   |
|WI |    0.475114953707326  |
|WY |    0.437444933920705 |

#### 2. Provide the city with the fourth highest median avg_retail_price without using the rank function.

Assumptions: Some versions of SQL do not support median or percentile functions. Assuming yours does, here is the code.

>Code

```sql
SELECT city, state, PERCENTILE(avg_retail_price, 0.5) AS median_price  
FROM pricing  
WHERE city <> 0  
    AND city IS NOT NULL  
GROUP BY city, state  
ORDER BY AVG(avg_retail_price) DESC  
LIMIT 1 OFFSET 3;  
```

>Output

|city    |  state | median_price|
|------ |   ----- | ------------|
|Mannford | OK  |   11.96 |

#### 3. Provide a list of brands that do not have a campaign running between weeks ‘2018-03-19’ and ‘2018-04-23’ and that also have more than 10,000 engagements.

Assumptions: I assume "have more than 10,000 engagements" here means in this dataset in total. If it were something like "in a single campaign or week" that would be a different question. I also assume it did not mean having 10,000+ engagements during the dates provided because any company that does not run a campaign during that time, by definition, will not have any engagements on Ibotta.

>Code

```sql
SELECT DISTINCT brand FROM campaign  
WHERE brand NOT IN (  
    SELECT DISTINCT brand FROM campaign  
    WHERE week BETWEEN '2018-03-19' AND '2018-04-23'  
    )  
AND brand IN (  
    SELECT brand FROM campaign  
    GROUP BY brand  
    HAVING SUM(engagements) > 10000  
    );  
```

>Output

|brand|
|---|
|Brand 17|
|Brand 14|


#### 4. 
>For the months of October 2018 through December 2018, provide the sum of total_quantity on each week and what percent that total is relative to the past four weeks.

Assumptions: Though there are a few ways to interpret "relative to the past for weeks," I assumed we wanted to take the total quantity from a specific week, sum the four previous weeks (but not the week in question), and then divide the week in question by that four week sum. 

>Code

```sql
SELECT week,  
    tq AS total_quantity,  
    CAST(tq AS FLOAT)  
        / (LAG(tq, 1) OVER (ORDER BY week)  
        + LAG(tq, 2) OVER (ORDER BY week)  
        + LAG(tq, 3) OVER (ORDER BY week)  
        + LAG(tq, 4) OVER (ORDER BY week)) * 100 AS '4 week percentage'  
FROM (  
    SELECT week, SUM(total_quantity) AS tq FROM market  
    WHERE week BETWEEN '2018-09-03' AND '2018-12-31'  
    GROUP BY week)  
LIMIT -1 OFFSET 4;  
```

>Output

|week | total_quantity | 4 week percentage|
|---|---|---|
|2018-10-01 | 12554       |    25.8706672711536|
|2018-10-08 | 11760       |    24.2494226327945 
|2018-10-15 | 11806       |    24.6749989549806 
|2018-10-22 | 12104      |     25.3912313824208 
|2018-10-29 | 11668     |      24.1954213669542 
|2018-11-05 | 12546    |       26.5030208289324 
|2018-11-12 | 10168   |        21.1287507272878 
|2018-11-19 | 7192    |        15.4713246999096 
|2018-11-26 | 15570  |         37.45129167268   
|2018-12-03 | 12714 |          27.9576040109068 
|2018-12-10 | 11480     |      25.1511699237578 
|2018-12-17 | 10972    |       23.3665559246955 
|2018-12-24 | 8776    |        17.2973825291706 
|2018-12-31 | 13726  |         31.236630103318 

## Part 2 - Python

Here I cleaned receipt data and explored potential data quality issues. For code and resulting tables, please see the associated [jupyter notebook](https://github.com/nick-kopy/Ibotta-Coding-Exercise/blob/main/ibotta.ipynb) for code and tables. It cannot be run without the associated data, but outputs should still be visible.

No major problems encountered on loading the data from .csv files.

### Missing data analysis

The primary data quality concern is missing data. Though not the only thing missing, item price and quantity were the biggest pieces of missing data. Only about 1,000 in 22,000 receipts had full price and quantity information, about 5%. Not very high. This tells us that there is a lot of information not being captured by the OCR system that scans the receipts. This is potentially a huge loss, depending on how important this information is. With this level of missing data it is unwise to try imputing something. If it had to be done, most receipt items would probably have a quantity of one. Some prices could be guessed by echoing item prices on other receipts (although 45% of products have no price on any receipt), some other database, a web scraped online price, or even imputed from items in the same product category. But again these could vary wildly (and as discussed below some prices are scanned incorrectly).

When we take the receipts with no missing information and try to reconstruct the total price, only 280 receipts matched exactly. Even if you give a 20% price tolerance to allow for possible taxes, discounts, surcharges, etc. you only have 656 receipts that match. That's 3% of the total receipts. Again not very high.

So even among the receipts we consider "complete," we have evidence that maybe half are still missing price related information. It could be an OCR scanning error like misreading numbers or missing items. Either way it is worth investigating some of the receipts that did and did not have price estimate matches to tease out systemic errors.

Ultimately the missing price and quantity data is potentially a big avenue that can't be explored, and it is not something that can really be reasonably fixed at this level. Quantity especially is important in campaign performance indicators (did the campaign make people buy more yogurt?), as well as for end users (if they bought two yogurts, they'll be wanting the rebates for both yogurts).

### Price analysis

The highest price for a single item was about $71 billion dollars and there were 72 additional items that were over $100,000 each. So either Jeff Bezos is scanning receipts or the data is wrong. 

It's very likely the wrong number is being mistaken for a price here and there. There are also products that have different prices on different receipts. This in of itself is not surprising, discounts happen all the time after all, but there are some clear outliers.

These can both be solved by removing errors, but until the receipt scans are improved we will have to fix this problem on the programming level. But where do we draw the line when we're defining a bad price? Do we remove anything priced above $10,000? $1,000? $500? There may not be a correct answer and you have to make a judgment call based on what's good for the business. A lower threshold means a higher percentage of correct prices but more "good data" that unfortunately gets thrown out.

### Misc analysis

Only a single receipt was missing a retailer ID. This was likely some sort of test case. The customer ID was 1, which was unusual in that all other IDs were at least 4 digits long. Probably safe to remove this receipt.

One particular receipt item id is repeated six times across the data, despite all other ids being unique. Since it looks like an isolated anomaly, it's probably best to remove these items from analysis completely. This receipt item is repeated on two separate receipts for whatever reason. It's probably worth looking up the two receipts where this anomaly came from and see if you can figure out why it happened. Both receipts were posted only a few minutes apart. That is, if it is allowed in terms of privacy and legality.
