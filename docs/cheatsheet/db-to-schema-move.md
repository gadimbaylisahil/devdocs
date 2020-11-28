# Move data from a DB's public schema into another DB's schema in PostgreSQL

This script is moving data from one DB's public schema into a schema inside of another DB. Both schemas database structure is equalent.

I used this when implementing a multitenancy solution using PostgreSQL schemas, on a legacy application where a separate DB was used for each tenant.

We created schemas for each tenant in the main DB, moved tenants data from their own separate DBs into their respective schemas inside the main DB.

```bash
#!/bin/bash

TENANT_DB=$1
TENANT_SCHEMA_NAME=$2
MAINDB=$3

echo "Backing up the main DB we are moving into in case something goes wrong"

/usr/bin/pg_dump "${MAINDB}" | /bin/gzip > /data/backup/${MAINDB}.dump.$(date +\%A-\%H).gz

echo "Dump and compress the TENANT_DB(data only), ignoring ar_internal_metadata and schema_migrations tables."

/usr/bin/pg_dump -a -b -d "${TENANT_DB}" -n public -T "ar_internal_metadata|schema_migrations" | /bin/gzip > /data/backup/${TENANT_DB}.gz

echo "Uncompress the dump, rename the `public.` to the schema name of the `tenant` inside the dumpfile."

zcat /data/backup/${TENANT_DB}.gz | sed "s/public\./${TENANT_SCHEMA_NAME}\./g" > /data/backup/${TENANT_DB}.import.sql

echo "Running the dumpfile inside the MAINDB"

/usr/bin/psql -d "${MAINDB}" -f /data/backup/${DB}.import.sql
```
