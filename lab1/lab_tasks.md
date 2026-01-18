OLTP (Online Transational Processing)
1. The following files contain a backup of a transactional database system:

init.sql
trans.sql
Restore the files into your PostgreSQL database.

OLAP (Online Analytical Processing)
2. Design an OLAP star model based on the given OLTP system. Create at least 3 dimensions. Mind that time granularity must be at least 1 month.

3. Implement OLAP as a relational database in a new schema, e.g. olap; when working with schemas, remember about setting the search_path variable, e.g.:

SET search_path TO olap, public;
4. Implement ETL with SQL loading your OLAP model with data from the OLTP tables. Use INSERT INTO ... SELECT. Provide queries for initial loading of all data. Make sure that all the records (especially last two records from trans.sql) are loaded properly.
5. Write an SQL query on the OLAP model which provides aggregated values for each month for a selected year. Pick the value depending on what your fact table looks like.