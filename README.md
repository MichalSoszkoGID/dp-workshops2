## GID DataOps CLI: Modifying an end-to-end data pipeline

Welcome to the DataOps CLI Labs workshop repository #2. By the end of this tutorial, you will know how to:
- store unused models in dbt
- configure dbt project and introduce layer structure
- transform custom SQL querries into data pipeline - best practices

Target environment will be Google Cloud Platform's: BigQuery & Data Studio, Vertex AI Managed Notebook, VSCode as IDE. This tutorial was written with GID DataOps 1.0.9 as a current release.

# Excercise

## Storing unused resources

Task: move (cut & paste) all models and singular tests (if present) created during Session 1 excercises to `analyses` folder. 
All models stored in `analyses` forlder will be noticed by, dbt but skipped during the pipeline execution. But stay warned: if your other models still have references to the deprecated models, the dbt pipeline will probably fail. Alternatively - you can delete unwanted models from your project or remove their extensions. Without having the ".sql" / ".yml" extention, the file will be ignored by dbt.

## Business task

During the demo session we have created a pipeline ending with `dim_users` model. `Dim_users` is a data mart model, where we store basic information about our customers (with confidential data filtered off) - like his / her `user_id`, `gender`, `age`, `postal code`, `country` etc. This data was taken from the `raw users table` and then enriched with information about his/her web activity in the fictional ecommerce platform (`total events`, date of the `first` and `most recent web activity`, splitted between different traffic sources). Now, our data team has been asked for upgrading the model so it can also present information regarding `customer lifetime value` so the analysts can search for patterns and insights reflecting web activity and purchases made by the customers. Moreover, there has been a request for extending the users localisation data with continents - "a small upgrade but a necessary one". 

### Steps to perform:
Your task is to:
1. Inspect a csv: <link to the csv> as a potential mapping table for countries and continents. Be warned! Some country names will require you attention!

    1a. Include the csv in the pipeline.
    
    1b. Create corresponding dbt resources (models). At this stage adding the `.yml` configs is optional.

    1c. Include the information about user continent as a column `user_address_continent` in the `dim_users` table.

2. Locate in the DWH a table storing information about user orders. This table should allow you to extract information on order prices.

    2a. Create dbt resources (models). At this stage adding the `.yml` configs is optional.
    
    2b. Add created models to the pipeline. `Dim_users` table should be upgraded with the following columns:
    
     - `CLV`: a customers lifetime value (here it is a sum of prices for completed orders, filter out orders that are in progress, returned, cancelled or shipped etc.)
     
     - `order_cnt`: count of all completed orders placed by the user
        
     - `first_order_date`: date of the first order for a given customer (ignore order status)
        
     - `most_recent_order_date`: date of the most recent order (ignore order status)
     
3. Inspect the pipeline and execute it locally.

4. Preview the results in BI tools
   
The task given above, although simplified, represents a real-world scenario for analytics engineer everyday work. That includes familiarizing ourself with the business logic, raw data structure, dbt project shape and internal rules regarding building models for the pipeline we are going to work with. We encourage you to try the excercise on your own but example of how the updated pipeline could look alike (with more detailed instruction how to get there) is provided underneath.

In case you need to catch up with the dbt project we created during demo session - You can find it in this repository: [dbt project - Session 2](https://gitlab.com/datamass-mdp-workshop/msoszko-datamass-project/-/tree/session-2-updated)

>-> Tip: you can delete / comment / move to `analyses` all models you've been working with and copy paste the models folder from the sample repository. 
 

## Solution

### Adding new `users_address_continents` column to `dim_users`

[To do]

### Adding new `CLV` & `orders` related columns to `dim_users`

[To do]
