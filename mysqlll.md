mysqldump -u username -p'password' --no-create-info --no-data --no-create-db --routines --skip-triggers --single-transaction --skip-lock-tables dbname > dbname_routines.sql
