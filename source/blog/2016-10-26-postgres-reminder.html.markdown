---
title: Postgres Reminder
date: 2016-10-26 18:56 UTC
tags: Postgres
---
Finally moving this out of `notes.txt` into the blog for future reference.

To kill a connection:

``` sql
SELECT pid FROM pg_stat_activity where pid <> pg_backend_pid();
SELECT pg_terminate_backend($1);
```

To dump a database into a SQL-script file:

``` sql
pg_dump db_name > outfile.sql
```

To dump a database into a compressed file:

``` sql
pg_dump -Fc db_name > outfile.dump
```

To restore a database:

``` sql
psql db_name < infile.sql
```

To grant all privileges to a user:

``` sql
GRANT ALL PRIVILEGES ON DATABASE db_name TO db_user;
```

log in as admin :

``` sql
psql -U postgres
```

change owner of database:

``` sql
ALTER DATABASE name OWNER TO new_owner;
```

Change a user's password:

``` sql
ALTER USER "user_name" WITH PASSWORD 'new_password';
```

To view all users:

``` sql
\du
```

To connect to a database as a specific user:

``` sql
psql -d database_name -U user
```

To change a database's owner:

``` sql
ALTER DATABASE name OWNER TO new_owner;
```
