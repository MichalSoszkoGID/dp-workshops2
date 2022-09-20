## GID DataOps CLI: Creating end-to-end data pipeline

Welcome to the DataOps CLI Labs workshop repository #2. By the end of this tutorial, you will know how to:
- store unused models in dbt
- configure dbt project and introduce layer structure
- transform custom SQL querries into data pipeline - best practices
- preview data with Data Studio

Target environment will be Google Cloud Platform's: BigQuery & Data Studio, Vertex AI Managed Notebook, VSCode as IDE. This tutorial was written with GID DataOps 1.0.9 as a current release.

# Excercise

## Storing unused resources

[To do]

## Business scenario

Create report on monthly revenue by new and regular users across the world (use continents)

### Write & execute prototype of the query

[To do: instructions, business logic etc. (create seed)]

````
with users_mapped as (
  select 
    u.id          as user_id,
    case 
      when u.country = 'Brasil' then 'Brazil'
      when u.country = 'United States' then 'US'
      when u.country = 'South Korea' then 'Korea, South'
      when u.country = 'España' then 'Spain'
      when u.country = 'Deutschland' then 'Germany'
      else u.country
    end           as user_country,
    c.Continent   as user_continent
  from `dataops-demo-342817.msoszko_US_private_working_schema.base_users` as u
  left join `dataops-demo-342817.msoszko_US_private_working_schema.seed_countries_continents` as c on
    case 
      when u.country = 'Brasil' then 'Brazil'
      when u.country = 'United States' then 'US'
      when u.country = 'South Korea' then 'Korea, South'
      when u.country = 'España' then 'Spain'
      when u.country = 'Deutschland' then 'Germany'
      else u.country
    end = c.Country
),
stg_order_items as (
  select 
    cast(date_trunc(created_at, month) as date)            as order_created_at,
    sum(o.sale_price)                                      as order_price,
    u.user_id,
    u.user_continent
  from `dataops-demo-342817.msoszko_US_private_working_schema.base_order_items` as o
  left join users_mapped as u on
  o.user_id = u.user_id
  where
    status = 'Complete'
  group by
    order_created_at, user_id, user_continent
),
stg_order_items_aggm as (
select 
  order_created_at,
  user_continent,
  count(distinct user_id) as users_count,
  sum(order_price)        as order_price
from stg_order_items 
group by order_created_at, user_continent
order by order_created_at, user_continent
)
select * from stg_order_items_aggm 
order by order_created_at, user_continent
````

### Create & launch data pipeline

[To do: instructions, add sample structure, show example of lineage graph (how could it look alike)

### Preview results in Data Studio

[To do: instructions: navigate to bigquery, choose table, enter Data Studio, decide on visualization type (graph vs table) create simple sketch
