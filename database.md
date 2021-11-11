## postgres installation

https://content-www.enterprisedb.com/downloads/postgres-postgresql-downloads

- **Tables**: Represents a subject or event in an application
- **Column**: Represents a different attributes (ID, name, age....)
- **Row**:    Respresents a different entry in the table (111, rasrivas, 50...; 222, ram, 65...;)


- Primary Key
- **Unique Contraints**:
  - A UNIQUE contraint can be applied to any column to make sure every record has a unique value for that column.
  - For example: if we apply for **Unique Contraints** for the `Name` column then the duplicate name (with the same name) will not be allowed
- **NULL Contraints**:
  -  By default, when adding a new entry to a database, any column can be left blank. When a column is left blank, it has a null value.
  -  If you need column to be a properly filled in to create a new record, a `NOT NULL` contraints can be added to the column to ensure that the column is never left blank.
