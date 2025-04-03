# Class 7 Participation Kilpatrick, Sarah

## Question 1

What's the SQL statement to run on the `cloudfront_logs` table to find the average amount of bytes transmitted by a given location? Sort the results from the highest average to the lowest average. Only return the top 5 results.

```sql
SELECT AVG(bytes) AS avgbytes, location
FROM cloudfront_logs
GROUP BY location
ORDER BY avgbytes DESC
LIMIT 5;
```

```text
#	avgbytes	location
1	3666.3576642335765	LAX1
2	3631.4068241469818	FRA2
3	3581.9139784946237	SEA4
4	3571.174025974026	MIA3
5	3464.700808625337	DFW3

```

## Question 2

Using the `financials_raw` table, what's the absolute value difference between the two trading volumes figures?

```sql
SELECT
  symbol, 
  financials[1].reportdate AS one_report_date, 
  financials[1].volume AS one_trading_volume,
  financials[2].reportdate AS another_report_date,
  financials[2].volume AS another_trading_volume,
  ABS(financials[1].volume - financials[2].volume) AS volume_difference
FROM
  financials_raw
ORDER BY
  symbol;
```

```text
#	symbol	            one_report_date one_trading_volume	another_report_date another_trading_volume	volume_difference
1	WILDRYDESCHIPS	    2011-06-10	    9974659	            2011-06-13	        12722991	              2748332
2	WILDRYDESFENSTERS	  2011-06-10	    40780545	          2011-06-13	        38823894	              1956651
3	WILDRYDESFRUITS	    2011-06-10	    397722153	          2011-06-13	        273023123	              124699030
4	WILDRYDESRIVALCHIPS	2011-06-10	    12773081	          2011-06-13	        11011184	              1761897
```