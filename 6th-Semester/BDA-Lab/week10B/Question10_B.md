Week 10 B: Create a data file for below schemas: 
Order: CustomerId, ItemId, ItemName, OrderDte, DeliveryDate 
Customers: CustomerId, CustomerName, Address, City, State, Country 
Load Order and Customer Data. 
Write a Pig Latin Script to determine number of items bought by each customer. 

*Order.txt*
```
CustomerId,ItemId,ItemName,OrderDate,DeliveryDate
1,101,Widget A,2024-04-01,2024-04-05
1,102,Widget B,2024-04-01,2024-04-06
2,103,Widget C,2024-04-02,2024-04-07
3,104,Widget D,2024-04-03,2024-04-08
```


*customer.txt*
```
CustomerId,CustomerName,Address,City,State,Country
1,John Doe,123 Elm St.,Anytown,State,Country
2,Jane Smith,456 Oak St.,Othertown,State,Country
3,Bob Johnson,789 Pine St.,Anycity,State,Country
```

PigScript.pig

```
-- Load Orders and Customer data
orders = LOAD 'orders.txt' USING PigStorage(',') AS (CustomerId:int, ItemId:int, ItemName:chararray, OrderDate:chararray, DeliveryDate:chararray);
customers = LOAD 'customers.txt' USING PigStorage(',') AS (CustomerId:int, CustomerName:chararray, Address:chararray, City:chararray, State:chararray, Country:chararray);

-- Remove the headers from the data
orders_data = FILTER orders BY CustomerId IS NOT NULL AND CustomerId > 0;
customers_data = FILTER customers BY CustomerId IS NOT NULL AND CustomerId > 0;

-- Group Orders by CustomerId
grouped_orders = GROUP orders_data BY CustomerId;

-- Count the number of items for each customer
items_per_customer = FOREACH grouped_orders GENERATE group AS CustomerId, COUNT(orders_data) AS NumItems;

-- Join the counts with the Customers data to get the names
customer_items = JOIN items_per_customer BY CustomerId, customers_data BY CustomerId;

-- Project the desired output: CustomerName and NumItems
customer_item_counts = FOREACH customer_items GENERATE customers_data::CustomerName AS CustomerName, items_per_customer::NumItems AS NumItems;

-- Store the result
STORE customer_item_counts INTO 'output' USING PigStorage(',');

```
