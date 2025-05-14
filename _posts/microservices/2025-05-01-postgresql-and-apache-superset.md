---
title:  "Apache Superset and Postgresql : connecting your database to a powerful data visualisation engine"
teaser: "Unlock the power of your data with Apache Superset-effortlessly create stunning dashboards by connecting to PostgreSQL using Docker. Dive in and transform raw data into actionable insights in minutes!"
tags:
    - apache
    - superset
    - postgresql
    - docker
    - solution architect
categories:
    - microservcies
math: true
comments: true
author: kanmeugne
---

Data is only as valuable as the insights you can extract from it --- and in today’s world, those insights need to be fast, interactive, and visually compelling. 

[Apache Superset][1], a powerful open-source data visualization platform, is rapidly becoming the go-to tool for analysts and developers who want to turn raw data into actionable dashboards without wrestling with complex setup processes. Thanks to [Docker][4] and [Docker Compose][3], spinning up a full-featured Superset environment takes just a few minutes, letting you focus on what matters: connecting to your data and building beautiful, [shareable dashboards][2].

![Apache loves Postgresql Poster](/images/apachesupersetpostgresql.jpg){: width="500" }
_Kanmeugne's Blog : Apache Superset and Postgresql --- connecting your database to a powerful data visualisation engine_

In this hands-on guide, you’ll see how convenient it is to connect [Superset][1] to a [PostgreSQL database][5] using [Docker Compose][3]. Once you’re comfortable with Docker basics, Superset’s streamlined workflow makes dashboard creation not just possible, but enjoyable-even for complex data sources.

## Project files Walkthrough

I will assume you have installed Docker and Docker-compose on your computer --- follow this [installation guide][7] if you haven't, and come back to this stage of the tutorial after. 

## Build and run the application

- **pull the application** from [the github repository](https://github.com/kanmeugne/modern-data-architectures.git)

```bash
$ git clone https://github.com/kanmeugne/modern-data-architectures.git
$ cd modern-data-architecture/handzon-apache-superset
```
- **create a `.env`** with the required environment variables for the project :

```bash
# postgres variables
POSTGRES_USER=***
POSTGRES_PASSWORD=***
POSTGRES_DB=***
POSTGRES_PORT=***

# pgadmin variables
PGADMIN_DEFAULT_EMAIL=***
PGADMIN_DEFAULT_PASSWORD=***
PGADMIN_PORT=***

# superset variable
SUPERSET_SECRET_KEY=***
SUPERSET_PORT=***
SUPERSET_ADMIN_USERNAME=***
SUPERSET_ADMIN_PASSWORD=***
SUPERSET_ADMIN_EMAIL=***
SUPERSET_ADMIN_FIRST_NAME=***
SUPERSET_ADMIN_LAST_NAME=***
```
- **download the dataset** and save the file into `source_data` (you need to do it before building the app) :
  
```bash
handzon-apache-superset/source_data$ curl -L -o brazilian-ecommerce.zip \
        https://www.kaggle.com/api/v1/datasets/download/olistbr/brazilian-ecommerce 
handzon-apache-superset/source_data$ unzip brazilian-ecommerce.zip
...
handzon-apache-superset/source_data$ rm brazilian-ecommerce.zip
```

{% plantuml style="width:100%" %}
@startuml
header Fig. Olist Brazilian Database

entity customer {
  * **customer_id : str**
  .. fkeys ..
  * customer_zip_code_prefix : varchar
  ..
  customer_unique_id : varchar
  customer_city : varchar
  customer_state : varchar
}

entity geolocation {
  * **zip_code_prefix : varchar**
  ..
  geolocation_lat : double
  geolocation_lng : double
  geolocation_city : varchar
  geolocation_state : varchar

}

entity order {
    * **order_id : varchar**
    .. fkeys ..
    * customer_id : varchar
    ..
    order_status : varchar 
    order_purchase_timestamp : timestamp
    order_approved_at : timestamp
    order_delivered_carrier_date : timestamp
    order_delivered_customer_date : timestamp
    order_estimated_delivery_date : timestamp

}

entity order_payment {
    .. fkeys ..
    * order_id : varchar
    ..
    order_payment : varchar    
    payment_sequential : integer
    payment_type : varchar
    payment_installments : varchar
    payment_value : double
}

entity seller {
    * **seller_id : varchar**
    .. fkeys ..    
    * seller_zip_code_prefix : varchar
    * ..
    seller_city : varchar
    seller_state : varchar
}

entity product {
    * **product_id : varchar**
    .. fkeys ..
    * production_category_name : varchar
    ..
    product_name_length : integer
    product_description_length : integer
    product_photos_qty : integer
    product_weight_g : integer
    product_length_cm : integer
    product_height_cm : integer
    product_width_cm : integer
}

entity order_review {
    .. fkeys ..
    * order_id varchar
    ..
    review_id : varchar
    review_score: integer
    review_comment_title: varchar
    review_comment_message: text
    review_creation_date: date
    review_answer_timestamp: timestamp
}

entity order_item {
    .. fkeys ..
    * order_id : varchar
    * product_id : varchar
    * seller_id : varchar
    ..
    order_item_id integer
    shipping_limit_date timestamp
    price : double
    freight_value : double
}

entity product_category_name_translation {
    * **product_category_name : varchar**
    ..
    product_category_name_english : varchar
}

order_review -r-> order : order_id
order_payment -d-> order : order_id
order -d-> customer : customer_id
order_item -l-> order : order_id
order_item -r-> seller : seller_id
seller -d-> geolocation : seller_zip_code_prefix
customer -r--> geolocation : customer_zip_code_prefix
product -r-> product_category_name_translation : product_category_name
product <-u- order_item : product_id

hide empty members
{% endplantuml %} 
_Data model of the [**Brazilian E-Commerce Public Dataset by Olist**][6]_

- **build the application** with `docker compose` from the project folder :

```shell
handzon-apache-superset$ docker compose up -d
...
```
The build process might be long at first run, so be patient... 

## Database walkthrough

When everything is up, use your favorite browser and do the following checkups :

- **check the database content** by opening the **pgadmin** web endpoint in your browser : `http://localhost:<PGADMIN_PORT>`. You will have to log in **pgadmin** with the username and the password that you have defined in the `.env` file.

```shell
# pgadmin variables
PGADMIN_DEFAULT_EMAIL=***
PGADMIN_DEFAULT_PASSWORD=***
PGADMIN_PORT=***
```
![img-description](/images/login-page-pgadmin.png){: width="500"}
_PGAdmin Login Page : use the credentials defined in `.env`_

- **navigate** in your database by setting a connection with the proper variables (the server address is the name of corresponding service container : `postgis`)

```shell
# postgres variables
POSTGRES_USER=***
POSTGRES_PASSWORD=***
POSTGRES_DB=***
POSTGRES_PORT=***
```

![img-description](/images/create-db-connexion.gif){: width="100%"}
_PGAdmin Connexion + Navigation into your sql tables_

## Create analytics within Superset

To be able to create analytics, you should at least :
- set a database connexion 
- create datasets
- create charts and add them to dashboards

### Create a database connexion

Follow these steps in order to create a connexion to your postgresql database :

- **open the superset endpoint in your browser** : `http://localhost:<SUPERSET_PORT>` and use the following credentials to sign in :
```shell
SUPERSET_PORT=***
SUPERSET_ADMIN_USERNAME=***
SUPERSET_ADMIN_PASSWORD=***
```
![img-description](/images/login-superset.png){: width="50%"}
_login into Superset_

- **when signed in, go to `(1) + > (2) Data > (3) Connect database`** to configure a new database connexion
  
![img-description](/images/create-connexion-01.png){: width="50%"}
_add connexion to the database - select data source_

- **click on the `Postgresql` button** since we are using a Postgresql Database

![img-description](/images/create-connexion-02.png){: width="50%"}
_add connexion to the database - select database type_

- **use the postgresql credentials** from the `.env` and click *Connexion*, then *Finish*. 

![img-description](/images/create-connexion-03.png){: width="50%"}
_add connexion to the database - set credential and save the connexion_

Within the tool, you can now query data from the database and build the analytics upon it.

### Create a dataset

Datasets can be created either directly from sql table or from sql queries. In this tutorial, we are going to create one from a sql query : 

- Go to the Menu `(1) SQL>SQL Lab`, to open the sql dataset creation wizard. 
- Use the `(2) DATABASE` field to select the database connexion you just created
- Use `(3) SCHEMA` field to pick the right schema. 

![img-description](/images/create-query-dataset-01.png){: width="50%"}
_set credentials for the dataset_

- In the right panel, copy the following sql code to collect relevant data about orders, products and sellers
```sql
SELECT
    oi.order_id, oi.product_id,
    tr.product_category_name_english as product_name,
    oi.price, oc.customer_city
from olist_order_items_dataset  oi 
     inner join olist_orders_dataset oo on oo.order_id = oi.order_id
     inner join olist_customers_dataset oc on oo.customer_id = oc.customer_id
     inner join olist_products_dataset op on oi.product_id = op.product_id
     left join  product_category_name_translation tr on op.product_category_name = tr.product_category_name
```

- save the dataset as **orders, products, sellers**

Now that we have created an extended dataset on orders and products, we can edit charts and dashboards to explore it.

### Create charts

Let's create 3 charts that we are going to add to a dashboard later. 

*$1^{st}$ chart : total sales per city*

- on the dataset tab, click on **orders, products, sellers**
- drag `price` attribute from the left panel and drop it in the *metrics* cell - confirm `SUM(price)` as the agregation operation. 
- drag `customer_city` from the left panel to the *Dimensions* cell. You should see the following chart (see screenshot). Save it as *total sales per city*

![img-description](/images/chart-01-06.png){: width="100%"}
_add connexion to the database - 3_

*$2^{nd}$ chart : number of different products per city*

- on the dataset tab, click on **orders, products, sellers**
- use `product_id` as the metric (confirm `COUNT_DISTINCT(product_id)` as the agregation operation), and `product_name` as the dimension. You should see this chart (save it as *number of different products per city*)

![img-description](/images/chart-01-07.png){: width="100%"}
_add connexion to the database - 3_

*$3^{rd}$ chart : top ten  sales*

- on the dataset tab, click on **orders, products, sellers**
- use `product_id` as the metric (confirm `COUNT_DISTINCT(product_id)` as the agregation operation), and `product_name` as the dimension. 
- use the `ROW LIMIT` option to limit the number of items to `10`. You should see this chart (save it as *top ten sales*)

![img-description](/images/chart-01-08.png){: width="100%"}
_add connexion to the database - 3_

It is now possible to agregate charts into a dashboard to have an overview of your data

### Create a dashboard

Dashboard creation is quite simple : 

- on the dashboard tab, add a new dashboard

![img-description](/images/dashboard-01.png){: width="50%"}
_create a new dashboard_

- check on the right pane to see the charts you can use to build your dashboard (you should see the charts you have created above)

![img-description](/images/dashboard-02.png){: width="50%"}
_explore charts on the right panel_

- drag all your charts from the right pane and drop them anywhere on the dashboard canva

![img-description](/images/dashboard-03.png){: width="100%"}
_drag and drop charts into the dashboard_

- save the dashboard as *Product Sales View*.
  
![img-description](/images/dashboard-04.png){: width="100%"}
_set the dashboard title and save_

You should now see *Product Sales View* under the dashboard tab. Interested readers can check the [Apache Superset][1] website to see how you can add CSS template in order to build even more compeling visuals. 

## Conclusion

I hope this tutorial will be helpful for those who want to play with Apache SuperSet and PostgreSQL. Feel free to send me your comments and remarks.

[1]: https://superset.apache.org/docs/installation/docker-compose/
[2]: https://github.com/user-attachments/assets/b37388f7-a971-409c-96a7-90c4e31322e6
[3]: https://docs.docker.com/compose/
[4]: https://docker-curriculum.com/
[5]: https://www.postgresql.org/
[6]: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
[7]: https://docs.docker.com/compose/install/
[8]: https://github.com/kanmeugne/modern-data-architectures/tree/master/handzon-apache-superset