# Food Manufacturer Analysis
## Abstract
Southwind is a manufacturer specializing in packaged food, including beverages, confectionery, meal products, and condiments. The dataset provided contains insights from customer data and past transactions, aimed at assisting Southwind Co. in improving their operations and client relations.

## ERD
<img width="512" alt="Northwind Schema" src="https://github.com/user-attachments/assets/59a4925a-abc3-431a-9e89-7f204bb41be5">


## Objectives
1. Address specific queries from Southwind's management
   - Determine total revenue for the second half of 1997
   - Identify the top five most ordered beverages
   - Analyze discount trends across product categories
2. Classify suppliers based on the popularity of their goods
3. Determine the top three revenue-generating product categories
4. Identify the highest-spending customer
## Analysis
The top-selling products for this period include **Guaraná Fantástica** with 51 orders, leading the list as the most popular item. Following closely are **Rhönbräu Klosterbier** with 46 orders and **Chang** with 44 orders. Both **Outback Lager** and **Lakkalikööri** also performed well, each receiving 39 orders. These figures highlight the strong demand across a diverse range of beverages.
![Top 5 Beverage](https://github.com/user-attachments/assets/38e5523a-a193-44e6-a044-29b6eaa2befb)

The average discounts across different product categories show a notable variation, with **Meat/Poultry** leading at an average discount of 6.45%. **Beverages** follow closely with an average discount of 6.19%, while **Seafood** receives a 6.02% discount on average. **Confections** and **Dairy Products** have slightly lower average discounts of 5.69% and 5.34%, respectively. This data indicates that Meat/Poultry and Beverages are the categories with the highest promotional offers.
![Average Discount for each category](https://github.com/user-attachments/assets/010b10d2-e700-492c-b040-d076947d1cd9)

The data indicates a strong correlation between supplier popularity and average sales revenue. Suppliers classified as **Not Popular** generate an average sales revenue of approximately $28,239.05. **Adequately Popular** suppliers see a significant increase, with an average revenue of $40,145.53. The highest average sales revenue is achieved by **Highly Popular** suppliers, with an impressive $107,211.36. This trend underscores the impact of supplier popularity on sales performance.
![Average Sales revenue vs supplier popularity](https://github.com/user-attachments/assets/2c00b9be-4333-4000-a8fc-e3aadd18e719)

The customer **QUICK** demonstrates a diverse purchasing pattern across various categories, contributing significantly to total sales. The highest revenue comes from the **Beverages** category, generating $36,216.43, which accounts for 13.52% of the total sales. **Confections** follow with $18,530.09, making up 11.07% of the total. Other notable contributions include **Condiments** at 8.69%, **Produce** at 8.08%, and **Seafood** at 7.14%. Categories like **Dairy Products** and **Meat/Poultry** contribute 5.89% and 5.98%, respectively, while **Grains/Cereals** account for 5.55% of the total sales. This distribution shows a balanced contribution across multiple product categories.
![percent of total sale for customer  QUICK](https://github.com/user-attachments/assets/7d8e95f0-c177-43af-bcb8-4472734d12ac)



