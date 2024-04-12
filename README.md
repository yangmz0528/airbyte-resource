# Airbyte

## Running Airbyte in Docker
In terminal, 
```sh 
docker compose -f airbyte-docker-compose.yml -d
```

## Tutorial (Google Sheets to Google BigQery)

### Source
#### in airbyte
1. Go to new connection and select google sheets as source
2. Select authentication method as service account key authentication
3. Under Optional fields, enable `Convert Column Names to SQL-Compliant Format`

#### in Google cloud
1. Set up Google Cloud account (free trial available)
2. Once set up, create a new project name 'Bank' and go to `APIs & Services`
3. Search for `Google Sheets API` and enable API
4. Under Google Sheets API, create credentials -> select Application data and enter service account details `airbyte-sheets`
5. Grant `owner` access to this service account
6. Create Key for service account -> Go to `IAM & Admin` -> service accounts
7. Select the service account that you have just created and add key -> create new key -> json
8. The key will be downloaded to local machine, copy the entire json and paste it in airbyte in the service account information under authentication

#### in Google Sheets
1. go to `transactions.xlsx` and click on share
2. paste the service account email (can access this from service account detail page) into `add people and groups`, make sure you have editor access
3. copy share link and paste in airbyte spreadsheet link

### Destination
1. go to Google Cloud BigQuery Studio and create dataset, name the dataset ID `customers`, multi-region and deafult location as US. 
2. Enter the respective information in airbyte
3. For loading method, there are 2 ways, one is GCS staging whereby the data is loaded to Google Cloud Storage (GCS) before copying the data to load in BigQuery (recommended for production setting). Another way is to use Standard Inserts (not recommended) where it direct loading using SQL INSERT statements, this method is only for quick testing.

## Tutorial (Postgre -> BigQuery using CDC)

### Create Source
1. Create a new source for Postgres
2. host: host.docker.internal, db name: airbyte, username & password: docker (stated in .env file), update mode: scan changes with user defined cursor

### Add data to Postgres
1. Go to pgAdmin (localhost:8888)
2. login with docker@docker.com, password: docker
3. Add new server, general-name (can be anything - Airbyte Postgres), connection-hostname: airbyte-db, username: docker, password: docker

### Sync Mode
```sh
Full refresh | Overwrite
```
cursor: value to track whether a record should be replicated (i.e: timestamp, a date)
cursor field: field or column in the data where the cursor can be found (i.e: updated_at, created_at)
Definition on the left: how airbyte reads data from the source
Definition on the right: how airbyte writes data into the destination
- Full Refresh | Overwrite: retrieves all data from the source, regardless of whether it has been synced before, and replaces existing data in the destination with new data.
- Full Refresh | Append: retrieves all data from the sources, regardless of whether it has been synced before, and appends the data to the destination table.