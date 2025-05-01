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
publish: false
author: kanmeugne
---

Data is only as valuable as the insights you can extract from it --- and in today’s world, those insights need to be fast, interactive, and visually compelling. [Apache Superset][1], a powerful open-source data visualization platform, is rapidly becoming the go-to tool for analysts and developers who want to turn raw data into actionable dashboards without wrestling with complex setup processes. Thanks to [Docker][4] and [Docker Compose][3], spinning up a full-featured Superset environment takes just a few minutes, letting you focus on what matters: connecting to your data and building beautiful, shareable dashboards[2].

In this hands-on guide, you’ll see just how convenient it is to connect Superset to a [PostgreSQL database][5] using [Docker Compose][3]. Once you’re comfortable with Docker basics, Superset’s streamlined workflow makes dashboard creation not just possible, but enjoyable-even for complex data sources. Whether you’re a data enthusiast or a business decision-maker, you’re about to discover how easy it is to go from database to dashboard, all with open-source tools and a few simple commands.

## The Dataset : Brazilian E-Commerce Public Dataset by Olist

For this article --- and several upcoming explorations of modern data tools --- I’ll be using the **Brazilian E-Commerce Public Dataset by Olist**, available on [Kaggle][6]. This rich, real-world dataset captures thousands of orders from a large Brazilian e-commerce platform, featuring detailed information on customers, sellers, products, payments, reviews, and more. Its comprehensive structure makes it perfect for showcasing data analysis, visualization, and dashboarding techniques across different platforms.

By working with this dataset, you’ll see practical, hands-on examples of connecting, exploring, and visualizing real e-commerce data-starting with Apache Superset and PostgreSQL in this article, and expanding to other tools in future posts.

## Infrastructure set-up

Starting from this section, I will assume you have installed Docker and Docker-compose on your computer --- which is fairly straitforward no matter what os you are running. I you haven't, pause the reading, and come back after the [installation][7]. 

## Project Folder

Our project directory tree looks as follows :

```tree
.
├── docker-compose.yaml
├── .env
└── source_data
    ├── init.sql
    ├── olist_customers_dataset.csv
    ├── olist_geolocation_dataset.csv
    ├── olist_order_items_dataset.csv
    ├── olist_order_payments_dataset.csv
    ├── olist_order_reviews_dataset.csv
    ├── olist_orders_dataset.csv
    ├── olist_products_dataset.csv
    ├── olist_sellers_dataset.csv
    └── product_category_name_translation.csv
```
> The full project can pulled from the [following github repository][8]
{: .prompt-tip }

### Project Folder Walkthrough

Our docker-compose file lists the 3 services we are going to use for our project :

#### docker-compose and .env files

```yaml
version: '3.8'

services:
  postgres:
    image: postgis/postgis:17-3.5
    container_name: postgis
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./source_data:/docker-entrypoint-initdb.d:ro
    networks:
      - geonetwork
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    

  pgadmin:
    image: dpage/pgadmin4:7.8
    container_name: pgadmin
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}
    ports:
      - "${PGADMIN_PORT}:80"
    networks:
      - geonetwork
      
  superset:
    image: apache/superset:latest-dev
    container_name: superset
    environment:
      - SUPERSET_SECRET_KEY=${SUPERSET_SECRET_KEY}
      - SUPERSET_ADMIN_USERNAME=${SUPERSET_ADMIN_USERNAME}
      - SUPERSET_ADMIN_PASSWORD=${SUPERSET_ADMIN_PASSWORD}
      - SUPERSET_ADMIN_EMAIL=${SUPERSET_ADMIN_EMAIL}
      - SUPERSET_ADMIN_FIRST_NAME=${SUPERSET_ADMIN_FIRST_NAME}
      - SUPERSET_ADMIN_LAST_NAME=${SUPERSET_ADMIN_LAST_NAME}
    ports:
      - "${SUPERSET_PORT}:8088"
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - geonetwork
    command: >
      /bin/bash -c "
        superset db upgrade &&
        superset fab create-admin --username ${SUPERSET_ADMIN_USERNAME} --firstname ${SUPERSET_ADMIN_FIRST_NAME} --lastname ${SUPERSET_ADMIN_LAST_NAME} --email ${SUPERSET_ADMIN_EMAIL} --password ${SUPERSET_ADMIN_PASSWORD} &&
        superset init &&
        superset run -h 0.0.0.0 -p 8088"

volumes:
  postgres_data:

networks:
  geonetwork:
    driver: bridge
```

We have 3 different containers that communicate on the same network:
- `postgres` : a postgis-enabled postgresql database service -- since we will need to manipulate spatial data (who doesn't !).
- `pgadmin` --- a powerfull opensource database management system for PostgreSQL
- `superset` --- Apache Superset service for the analytics
> In the example, we are using a dedicated volume --- `postgres_data` --- for postgres, but the reader can also bind that volume to a local folder if more convenient. 
{: .prompt-tip }

Of course, an `.env` file should be provided at the root of the folder with at least the following definitions :

```bash
# postgres variables
POSTGRES_USER=<your-username>
POSTGRES_PASSWORD=<your-password>
POSTGRES_DB=<your-db-name>
POSTGRES_PORT=<your-db-port>

# pgadmin variables
PGADMIN_DEFAULT_EMAIL=<pg-admin-email>
PGADMIN_DEFAULT_PASSWORD=<pgadmin-password>
PGADMIN_PORT=<pgadmin-port>

# superset variable
SUPERSET_SECRET_KEY=<a-random-secret-key>
SUPERSET_PORT=<superset-port>
SUPERSET_ADMIN_USERNAME=<superset-admin>
SUPERSET_ADMIN_PASSWORD=<superset-admin-password>
SUPERSET_ADMIN_EMAIL=<superset-admin-email>
SUPERSET_ADMIN_FIRST_NAME=<superset-admin-first-name>
SUPERSET_ADMIN_LAST_NAME=<superset-admin-last-name>
```
> The template for the .env file at the root of the project. Make sure the chosen port numbers are available on your system.
{: .prompt-tip }

#### source_data and init.sql

You have probably noticed the `source_data` folder, which is bound to postgres `docker-entrypoint-initdb.d` folder. Litterally, any *.sql files inside that folder will be run by docker at build --- this is where we define our database initilisation script `init.sql`. 

For convenience, we also upload our raw csv dataset files in `source_data` -- by typing the following command at the right place : 

```shell
path/to/source_data$ curl -L -o brazilian-ecommerce.zip https://www.kaggle.com/api/v1/datasets/download/olistbr/brazilian-ecommerce && unzip brazilian-ecommerce.zip && rm brazilian-ecommerce.zip 
```

Our `init.sql` file therefore is as follows :

```sql
CREATE SCHEMA SOURCE_DATA;

-- 1. Creating Table Definitions

CREATE TABLE SOURCE_DATA.product_category_name_translation (
    product_category_name VARCHAR(64) PRIMARY KEY,
    product_category_name_english VARCHAR(64)
);
CREATE TABLE SOURCE_DATA.olist_customers_dataset (
    customer_id VARCHAR(32) PRIMARY KEY,
    customer_unique_id VARCHAR(32),
    customer_zip_code_prefix VARCHAR(8),
    customer_city VARCHAR(64),
    customer_state CHAR(2)
);

CREATE TABLE SOURCE_DATA.olist_geolocation_dataset (
    geolocation_zip_code_prefix VARCHAR(8),
    geolocation_lat DOUBLE PRECISION,
    geolocation_lng DOUBLE PRECISION,
    geolocation_city VARCHAR(64),
    geolocation_state CHAR(2)
);

CREATE TABLE SOURCE_DATA.olist_orders_dataset (
    order_id VARCHAR(32) PRIMARY KEY,
    customer_id VARCHAR(32) REFERENCES SOURCE_DATA.olist_customers_dataset(customer_id),
    order_status VARCHAR(20),
    order_purchase_timestamp TIMESTAMP,
    order_approved_at TIMESTAMP,
    order_delivered_carrier_date TIMESTAMP,
    order_delivered_customer_date TIMESTAMP,
    order_estimated_delivery_date TIMESTAMP
);

CREATE TABLE SOURCE_DATA.olist_order_payments_dataset (
    order_id VARCHAR(32) REFERENCES SOURCE_DATA.olist_orders_dataset(order_id),
    payment_sequential INTEGER,
    payment_type VARCHAR(20),
    payment_installments INTEGER,
    payment_value NUMERIC(10,2)
);

CREATE TABLE SOURCE_DATA.olist_sellers_dataset (
    seller_id VARCHAR(32) PRIMARY KEY,
    seller_zip_code_prefix VARCHAR(8),
    seller_city VARCHAR(64),
    seller_state CHAR(2)
);

CREATE TABLE SOURCE_DATA.olist_products_dataset (
    product_id VARCHAR(32) PRIMARY KEY,
    product_category_name VARCHAR(64),
    product_name_lenght INTEGER,
    product_description_lenght INTEGER,
    product_photos_qty INTEGER,
    product_weight_g INTEGER,
    product_length_cm INTEGER,
    product_height_cm INTEGER,
    product_width_cm INTEGER
);

CREATE TABLE SOURCE_DATA.olist_order_reviews_dataset (
    review_id VARCHAR(32),
    order_id VARCHAR(32) REFERENCES SOURCE_DATA.olist_orders_dataset(order_id),
    review_score INTEGER,
    review_comment_title VARCHAR(255),
    review_comment_message TEXT,
    review_creation_date DATE,
    review_answer_timestamp TIMESTAMP
);

CREATE TABLE SOURCE_DATA.olist_order_items_dataset (
    order_id VARCHAR(32) REFERENCES SOURCE_DATA.olist_orders_dataset(order_id),
    order_item_id INT,
    product_id VARCHAR(32) REFERENCES SOURCE_DATA.olist_products_dataset(product_id),
    seller_id VARCHAR(32)  REFERENCES SOURCE_DATA.olist_sellers_dataset(seller_id),
    shipping_limit_date TIMESTAMP,
    price NUMERIC(10,2),
    freight_value NUMERIC(10,2)
);

-- 2. Populating the database 

COPY SOURCE_DATA.product_category_name_translation
FROM '/docker-entrypoint-initdb.d/product_category_name_translation.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_customers_dataset
FROM '/docker-entrypoint-initdb.d/olist_customers_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_geolocation_dataset
FROM '/docker-entrypoint-initdb.d/olist_geolocation_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_orders_dataset
FROM '/docker-entrypoint-initdb.d/olist_orders_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_sellers_dataset
FROM '/docker-entrypoint-initdb.d/olist_sellers_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_products_dataset
FROM '/docker-entrypoint-initdb.d/olist_products_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_order_items_dataset
FROM '/docker-entrypoint-initdb.d/olist_order_items_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_order_payments_dataset
FROM '/docker-entrypoint-initdb.d/olist_order_payments_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);

COPY SOURCE_DATA.olist_order_reviews_dataset
FROM '/docker-entrypoint-initdb.d/olist_order_reviews_dataset.csv'
WITH (FORMAT csv, ON_ERROR 'ignore', HEADER);
```
> In init.sql, we first create the table definitions, and then, populate the database. Note that, additionnal data clean-up might be necessary in order to run the initialization without blocking errors.
{: .prompt-tip }

### The build process

Technically, you should be able to build your app get your services ready to run. Go to the project folder an type the following command:

```shell
path/to/project/folder$  docker compose up # Optionally, you can use the `-d` for the silent mode (i.e. no log)
```

Notice that the compose process might be long at first run, so be patient... When everything is up, use your favorite browser and do the following checkups

### pgadmin

- First, check if the database is correctly set up by opening the pgadmin web endpoint in your browser : https://localhost:<PGADMIN_PORT>. You will have to log in with the username and the password that you have defined in the .env file.
- create a server and set the localhost to `postgres` since `pgadmin` and `postgres` container are on the same network
- set the port to 5432 (docker internal port)
- set the username and the password with `$POSTGRES_USER` and `$POSTGRES_PASSWORD` respectively. Voilà, you be able to explore your table.

![img-description](/images/pgadmin-superset.png){: w="500"}
_PGAdmin Dashboard : Navigating through your tables_





[1]: https://superset.apache.org/docs/installation/docker-compose/
[2]: https://github.com/user-attachments/assets/b37388f7-a971-409c-96a7-90c4e31322e6
[3]: https://docs.docker.com/compose/
[4]: https://docker-curriculum.com/
[5]: https://www.postgresql.org/
[6]: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
[7]: https://docs.docker.com/compose/install/
[8]: https://github.com