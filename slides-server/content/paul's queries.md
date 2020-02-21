Number of customers with loans:
```
SELECT count(distinct loans.account_id)
FROM loans JOIN dispositions
     ON loans.account_id = dispositions.account_id
```
Number of customers with cards:
```
SELECT count(distinct credit_cards.disp_id)
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
```
This just to make sure that there is a disp_id per account_id:
```
SELECT count(distinct credit_cards.disp_id),
       count(distinct dispositions.account_id)
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
```
This is a histogram of user counts by how many transactions they make
```
SELECT bucket(num_transactions BY 10),
       count(*) AS num_users
FROM (
    SELECT dispositions.client_id, count(*) AS num_transactions
    FROM transactions JOIN dispositions
         ON transactions.account_id = dispositions.account_id
    GROUP BY 1
) t
GROUP BY 1
ORDER BY 1
```
This tells us the types of cards
```
SELECT credit_cards.type, count(*)
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
GROUP BY 1
```
This tells us the average loan by credit card type. It is interesting to see that JUNIOR loans are substantially more than GOLD loans.
```
SELECT credit_cards.type, avg(loans.amount) AS avg_loan, count(*) AS num_cust
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
     JOIN loans
     ON loans.account_id = dispositions.account_id
GROUP BY 1
ORDER BY 2
```
This tells us the average total transaction volume by credit card type. Here we see that GOLD card holders have more volume than JUNIOR
```
SELECT credit_cards.type,
       sum(transactions.amount)/count(DISTINCT credit_cards.disp_id) AS avg_volume,
       count(DISTINCT credit_cards.disp_id) AS num_cust
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
     JOIN transactions
     ON transactions.account_id = dispositions.account_id
GROUP BY 1
ORDER BY 2
```
This tells us the average transaction amount by credit card type.
```
SELECT credit_cards.type,
       avg(transactions.amount) AS avg_amount,
       count(DISTINCT credit_cards.disp_id) AS num_cust
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
     JOIN transactions
     ON transactions.account_id = dispositions.account_id
GROUP BY 1
ORDER BY 2
```
This gives a histogram of how many transactions on average per user for different transaction amounts. We see that GOLD holders have more transactions across the board. JUNIOR holders have more smaller transactions than CLASSIC, while CLASSIC has more larger transactions than JUNIOR.
```
SELECT credit_cards.type,
       bucket(transactions.amount by 10000),
       count(*)/count(DISTINCT credit_cards.disp_id) AS number_transactions_per_cust
FROM credit_cards JOIN dispositions
     ON credit_cards.disp_id = dispositions.disp_id
     JOIN transactions
     ON transactions.account_id = dispositions.account_id
GROUP BY 1,2
ORDER BY 1,2
```
This looks at three histograms of user counts by number of transactions, one per credit card type. Here we see that more GOLD users have 200 or 300 transactions than 100 transactions, whereas there are more JUNIOR and CLASSIC users with 100 transactions than 200 or 300.
```
SELECT cc_type,
       bucket(num_transactions BY 100),
       count(*) AS num_users
FROM (
    SELECT dispositions.client_id,
           credit_cards.type AS cc_type,
           count(*) AS num_transactions
    FROM credit_cards JOIN dispositions
         ON credit_cards.disp_id = dispositions.disp_id
         JOIN transactions
         ON transactions.account_id = dispositions.account_id
    GROUP BY 1,2
) t
GROUP BY 1,2
ORDER BY 1,2
```





17:20
on that last query, it is interesting to note that if you substitute dispositions.account_id for dispositions.client_id, the result is messed up (the data gets lost).