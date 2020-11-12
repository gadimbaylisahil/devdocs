# Script to move data from a DB into a schema in another DB

This script is moving data from DB into a schema inside of another DB. Prerequisite is that the
database structure of DB and the schema within another DB should be same.

I used this when implementing a multitenancy solution using PostgreSQL schemas, on an application where a separate DB was used for each tenant.

Thus, I needed to move the data from separate DBs into newly created schemas.

```bash

#!/bin/bash
echo "Backing up the DB we are moving into"

DB=$1
MOVING_INTO_DB_SCHEMA_NAME=$2

/usr/bin/pg_dump upg | /bin/gzip > /data/backup/upg.dump.$(date +\%A-\%H).gz

echo "Dump the DB(data only)"

/usr/bin/pg_dump -a -b -d "${DB}" -n public -T "ar_internal_metadata|schema_migrations" | /bin/gzip > /data/backup/${DB}.gz

zcat /data/backup/${DB}.gz | sed "s/public\./${MOVING_INTO_DB_SCHEMA_NAME}\./g" > /data/backup/${DB}.import.sql

echo "Running the dump inside the MOVING_INTO_DB"

/usr/bin/psql -d upg -f /data/backup/${DB}.import.sql
```
