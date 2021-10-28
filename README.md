# Timescale NFT Starter Kit
This project intends to help you get started with NFT sales analysis.

Project components:

* Data ingestion script, which collects historical data from OpenSea and ingests it into TimescaleDB
* Sample dataset, to get started quickly, if you don't want to wait too much time ingesting data
* Schema for storing NFT sales, assets, collections, and accounts
* Local TimescaleDB database, pre-loaded with sample NFT data
* Pre-built dashboards and charts in Apache Superset and Grafana for visualizing your data analysis
* Queries to use as a starting point for your own analysis


## Get started
Clone the nft-starter-kit repository:
```bash
git clone https://github.com/timescale/nft-starter-kit.git
cd nft-starter-kit
```

## Setting up the pre-built Superset dashboards
This part of the project is fully Dockerized. TimescaleDB and the Superset dashboard 
is built out automatically using docker-compose. After completing the steps below, you 
will have a local TimescaleDB and Superset instance running in 
containers - containing 500K+ NFT transactions from OpenSea.

The Docker service uses port 8088 (for Superset) and 6543 (for TimescaleDB) so make sure 
there's no other services using those ports before starting the installation process.

### Prerequisites

* [Docker](https://docs.docker.com/get-docker/)
* [Docker compose](https://docs.docker.com/compose/install/)
    Verify that both are installed:
    ```bash
    docker --version && docker-compose --version
    ```

### Instructions

1. Run `docker-compose up --build` in the /pre-built-dashboards folder:
    ```bash
    cd pre-built-dashboards
    docker-compose up --build
    ```
    See when the process is done (it could take a couple of minutes):
    ```bash
    timescaledb_1      | PostgreSQL init process complete; ready for start up.
    ```
1. Go to http://0.0.0.0:8088/ in your browser and login with these credentials:
    ```txt
    user: admin
    password: admin
    ```
1. Open the `Databases` page inside Superset (http://0.0.0.0:8088/databaseview/list/). You will see exactly one item there
    called `NFT Starter Kit`.
1. Click the edit button (pencil icon) on the right side of the table (under "Actions").
1. Don't change anything in the popup window, just click `Finish`. This will make sure the database can be 
   reached from Superset.
1. Go check out your NFT dashboards! 

    Collections dashboard: http://0.0.0.0:8088/superset/dashboard/1

    Assets dashboard: http://0.0.0.0:8088/superset/dashboard/2 

## Running the data ingestion script
If you'd like to ingest data into your database (be it locally, or in Timescale Cloud) 
straight from the OpenSea API, follow these steps to configure the ingestion script:

### Prerequisites
* [TimescaleDB installed][install-ts]
* Schema has been set up using the [`schema.sql`][schema] script.

1. Go to the root folder of the project:
    ```python
    bash cd nft-starter-kit
    ```
1.  Create a new Python virtual environment and install the requirements:
    ```bash
    virtualenv env && source env/bin/activate
    pip install -r requirements.txt
    ```
1.  Replace the parameters in the `config.py` file:
    ```python
    DB_NAME="tsdb"
    HOST="YOUR_HOST_URL"
    USER="tsdbadmin"
    PASS="YOUR_PASSWORD_HERE"
    PORT="PORT_NUMBER"
    OPENSEA_START_DATE="2021-10-01T00:00:00" # example start date (UTC)
    OPENSEA_END_DATE="2021-10-06T23:59:59" # example end date (UTC)
    ```
1.  Run the Python script:
    ```python
    python opensea_ingest.py
    ```
    This will start ingesting data in batches, 300 rows at a time:
    ```bash
    Start ingesting data between 2021-10-01 00:00:00+00:00 and 2021-10-06 23:59:59+00:00
    ---
    Fetching transactions from OpenSea...
    Data loaded into temp table!
    Data ingested!
    Data has been backfilled until this time: 2021-10-06 23:51:31.140126+00:00
    ---
    ```
    You can stop the ingesting process anytime (Ctrl+C), otherwise the script will run until all 
    the transactions have been ingested from the given time period.


## Ingest the sample data
If you don't want to spend waiting until a decent amount of data is ingested, 
you can just use our sample dataset which contains 500K+ sale transactions from 
OpenSea (this sample was used for the Superset dashboard as well) 

### Prerequisites
* [TimescaleDB installed][install-ts]
* PSQL ([installation guide](https://blog.timescale.com/blog/how-to-install-psql-on-mac-ubuntu-debian-windows/))

### Instructions
1.  Go to the folder with the sample CSV files:
    ```bash
    cd pre-built-dashboards/database/data
    ```
1.  Connect to your database with PSQL:
    ```bash
    psql -x "postgres://host:port/tsdb?sslmode=require"
    ```
    If you're using Timescale Cloud, the instructions under `How to Connect` provide a 
    customized command to run to connect directly to your database.
1.  Import the CSV files in this order (it can take a few minutes in total):
    ```bash
    \copy accounts FROM accounts.csv CSV HEADER;
    \copy collections FROM collections.csv CSV HEADER;
    \copy assets FROM assets.csv CSV HEADER;
    \copy nft_sales FROM nft_sales.csv CSV HEADER;
    ```  


[schema]: https://github.com/timescale/nft-starter-kit/blob/master/schema.sql
[install-ts]: https://docs.timescale.com/timescaledb/latest/how-to-guides/install-timescaledb/#install-timescaledb