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
1. Inspect a csv file: [ISO_like_Countries-Continents.csv](https://gitlab.com/datamass-mdp-workshop/workshop-resources/-/blob/main/CSVs/ISO_like_Countries-Continents.csv) as a potential mapping table for countries and continents. Be warned! Some country names will require you attention!

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

>-> Hint: After each step you can double-check if pipeline works by performing control `dbt run` execution.
>-> Hint2: Or if you don't want to run the whole pipeline but the model you have just created - `dbt run --select name_of_model`

**Step 1.** Locate csv file: [ISO_like_Countries-Continents.csv](https://gitlab.com/datamass-mdp-workshop/workshop-resources/-/blob/main/CSVs/ISO_like_Countries-Continents.csv) and upload it to `seeds` folder inside of you dbt project.

>-> Hint: you can download the file into your local drive and then drag-drop it into VSCode

![image](https://user-images.githubusercontent.com/97670480/192153291-83f3a4b3-6d59-4f6b-a478-bf701ed0f23c.png)

*Step 2.** Load seed from you command line (VSCode terminal) with the command:

```
dbt seed
```

![image](https://user-images.githubusercontent.com/97670480/192153470-fe021884-8d02-45b9-bf82-4633e345add6.png)

**Step 3.** The seed file contains ISO-like country names and correspoinding continents. It can serve as a mapping table for our `user_address_country` column stored in `stg_ecommerce__users.sql` model. However, some country names in `users` table do no fit the counry names stored in csv file. Secondly, we will be perfoming JOIN statement on a `stg_` type of model. Thus it is (according to dbt best practices) recommended to include `base_` type of model before join is executed. In order to do that - create new `base_ecommerce__users.sql` model inside of `staging/ecommerce` layer:

```
{{
    config(
      materialized='table',
      persist_docs={"relation":true, "columns": true}
    )
}}


with source as (
    select * from {{ source('raw_ecommerce_eu', 'users') }}
), 
users as (

    select 
        id                              as user_id,
        first_name                      as user_first_name,
        last_name                       as user_last_name,
        email                           as user_email,
        age                             as user_age,
        gender                          as user_gender,
        state                           as user_address_state,
        street_address                  as user_street_address,
        postal_code                     as user_address_postal_code,
        city                            as user_address_city,
        country                         as user_address_country,
        latitude                        as user_address_latitude,
        longitude                       as user_address_longitude,
        traffic_source                  as user_traffict_source,
        created_at                      as user_account_created_at

    from source
)

select * from users
```

**Step 4.** Modify the existing `stg_ecommerce__events.sql` model by adding JOIN statement and formatting country names, so the JOIN will not return any null values. Note that we were asked to filter out sensitive data - in staging layer there will be no user name and their detailed addresses columns:

```
{{
    config(
      materialized='table',
      persist_docs={"relation":true, "columns": true}
    )
}}


with users as (
    select * from {{ ref('base_ecommerce__users') }}
), 
countries_continents_mapped as (
    select * from {{ ref('ISO_like_Countries-Continents')}}
),
user_with_countries_reforged as (

    select 
        *,
        case 
            when user_address_country = 'Brasil' then 'Brazil'
            when user_address_country = 'United States' then 'US'
            when user_address_country = 'South Korea' then 'Korea, South'
            when user_address_country = 'España' then'Spain'
            when user_address_country = 'Deutschland' then'Germany'
            else user_address_country
        end as country_reforged            
    
    from users
),
users_with_continents as (

    select 
        user_id,
        split(user_email, '@')[ORDINAL(2)]   as user_email_domain,
        user_age,
        user_gender,
        user_address_postal_code,
        user_address_city,
        country_reforged                     as user_address_country,
        c.Continent                          as user_address_continent,  -- new continent column
        user_traffict_source,
        user_account_created_at

    from user_with_countries_reforged as u
    left join countries_continents_mapped as c 
    on u.country_reforged = c.Country
)

select * from users_with_continents
```

The corresponding yaml file (`stg_ecommerce__users.yml`) will now contain the following code:

```
version: 2

models:
  - name: stg_ecommerce__users
    description: ""
    columns:
      - name: user_id
        description: ""

      - name: user_email_domain
        description: ""

      - name: user_age
        description: ""

      - name: user_gender
        description: ""

      - name: user_address_postal_code
        description: ""

      - name: user_address_city
        description: ""

      - name: user_address_country
        description: ""

      - name: user_address_continent
        description: ""

      - name: user_traffict_source
        description: ""

      - name: user_account_created_at
        description: ""
```

**Step.5** Attach the `user_address_continent` in dim_users is performed by simply adding line of code in the final CTE:

```
{{
    config(
      materialized='table',
      persist_docs={"relation":true, "columns": true}
    )
}}

with users as (
    select * from {{ ref('stg_ecommerce__users') }}
),
events as (
    select * from {{ ref('int_events_traffic_sources_pivoted')}}
),
users_and_events as (

    select 

        u.user_id,
        u.user_age,
        u.user_gender,
        u.user_email_domain,
        u.user_address_postal_code,
        u.user_address_city,
        u.user_address_country,
        u.user_address_continent,   -- This is new continent column
        e.total_traffic,
        e.first_event,
        e.most_recent_event,
        e.adwords_traffic,
        e.email_traffic,
        e.facebook_traffic,
        e.organic_traffic,
        e.youtube_traffic

    from users as u
    left join events as e
    on u.user_id = e.user_id
)

select * from users_and_events
```
**Step 6.** Inspect the DAG (lineage graph) using dbt docs function (we use a `DP` command here):
```
dp docs-serve
```
and clicking on the DBT-Docs icon (Notebook Launcher). The DAG should now look like this:

![image](https://user-images.githubusercontent.com/97670480/192155662-9b1b13e1-70f3-4846-8e32-a9a39bf9fa9b.png)

### Adding new `CLV` & `orders` related columns to `dim_users`

[To do]
