##Migrating from a Development Database to Production

If you are developing a Django project with a SQLite development database and want to migrate that data to your production database (either MYSQL or PostgresQL), there are many opportunities for things to go horribly wrong. Hopefully these steps will make the process a little easier.

1. First, dump the contents of the SQLite database using the `datadump` command. SQLite, Postgres, and MYSQL all use different flavors of SQL to create and populate tables. This makes it difficult to simply dump to a .sql file and load data from there. To help with this, we'll use `dumpdata` to dump structure and contents of the database to a JSON file first.

`python manage.py dumpdata --natural-foreign --exclude auth.permission --exclude contenttypes --indent 4 > dump.json`

2. Next, change the `settings.py` file to connect to the Postgres database and comment out the SQLite connection. 

3. If the Postgres database is new, it likely has no structure yet. Apply the structure from the `models.py` file by running either
`python manage.py syncdb` or `python manage.py migrate`

4. Finally, it's time to import the data from the JSON dump. We'll use Django's `loaddata` command for that.

`python manage.py loaddata dump.json`
