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


- Create a new server --> Create a new database `fastapi`

```
CREATE DATABASE fastapi
    WITH 
    OWNER = postgres
    ENCODING = 'UTF8'
    CONNECTION LIMIT = -1;
```

```
CREATE TABLE public.products
(
    name character varying NOT NULL,
    price integer NOT NULL,
    id serial,
    PRIMARY KEY (id)
);

ALTER TABLE IF EXISTS public.products
    OWNER to postgres;
```

![Screenshot 2024-06-06 at 00 20 10](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/69b8176a-b31c-4e89-9e05-c4c5fadcce71)



- selecting a table to get all the colums by **ascending** order, here table name is `products`

```
SELECT * FROM public.products
ORDER BY id ASC 
```

```
SELECT * FROM public.products
ORDER BY id ASC LIMIT 100
```

- to make changes permanent `COMMIT;`

- Adding a row to the table

```
INSERT INTO public.products (
name, price) VALUES (
'TV'::character varying, '200'::integer)
 returning id;
```

```
INSERT INTO public.products (
name, price) VALUES (
'remote'::character varying, '80'::integer)
 returning id;
```

```
INSERT INTO public.products (
name, price) VALUES (
'DVD Player'::character varying, '80'::integer)
 returning id;
```

- checking the updates

```
SELECT * FROM public.products
ORDER BY id ASC 
```


- Testing, if we don't provide a price while adding a new row --> it will throw an error

```
INSERT INTO public.products (
name) VALUES (
'microphone'::character varying)
 returning id;
```

## We can add new columns to the existing table

- adding a new column **is_sale** and adding a contraints to default false, so in this if user doesn't provide an entry to this, it will accept it without any error.
- Here is a twist, suppose we add a new column **inventory** and mark it for `Not Null` and save it, so while saving it, it will throw an error `ERROR: comun "inventory" of relation "product" contains null values`
- so, to avoid this for now, we add a constraint to default value as 0

![Screenshot 2024-06-06 at 00 29 41](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/b3ba95be-e53d-4f9e-975f-bde59a3e6b33)

![Screenshot 2024-06-06 at 00 31 00](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/757f8143-1b39-4c92-83aa-0cb6059c6edd)

## Handling the timestamp

- Suppose we hava usecase, we can add a timestamp for the entry created and it should be managed by postgres and the time should be current time with timezone.

- Here, we have to use a data type: `timestamp with time zone`
  - Add contraint `NOW()`: add the timezone of the current time when it was added
  - Enable as `NOT NULL`

```
ALTER TABLE IF EXISTS public.products
    ADD COLUMN create_at timestamp with time zone NOT NULL DEFAULT NOW();
```

- To add a column using postgres, follow the below paths
  - Table --> table name `products` --> Properties (rigth click) --> columns

![Screenshot 2024-06-06 at 00 36 07](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/f857af16-e0f4-4f2f-939c-77cc3d88906e)


## SQL queries

- `SELECT * from products;` -> To get all the columns of the table

  ```
  name         price id is_sale inventory create_at
  "DVD Player"	    80	2	false	  0	"2021-11-12 20:30:50.551514+05:30"
  "remote"	        80	3	false	  0	"2021-11-12 20:30:50.551514+05:30"
  "microphone"	    30	5	false	  0	"2021-11-12 20:30:50.551514+05:30"
  "pencil"	         2	6	false	  0	"2021-11-12 20:30:50.551514+05:30"
  "sharpner"	       4	7	false	  0	"2021-11-12 20:30:50.551514+05:30"
  "TV Blue"	       400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
  "TV Red"	       300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
  "TV"	           200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
  ```

- `SELECT name from products;` -> if we only want to `name` row

```
"DVD Player"
"remote"
"microphone"
"pencil"
"sharpner"
"TV Blue"
"TV Red"
"TV"
```

- `SELECT name, price from products;` --> it will follow that same order
```
"DVD Player"	80
"remote"	80
"microphone"	30
"pencil"	2
"sharpner"	4
"TV Blue"	400
"TV Red"	300
"TV"	200
```

- `SELECT id AS products_id FROM products;` --> Renaming a column
  ```
  2
  3
  5
  6
  7
  9
  8
  1
  ```
- `SELECT id AS products_id, is_sale AS on_sale FROM products;`
    ```
    2	false
    3	false
    5	false
    6	false
    7	false
    9	false
    8	false
    1	false
    ```

- Match a specific `id` which matches a row
  - `SELECT * FROM products WHERE id=5;`
  ```
  "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
  ```

- Get the rows where price is greater than `50`
  - `SELECT * FROM products WHERE price < 50;`
  ```
  "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
  "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
  "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
  ```

  - `SELECT * FROM products WHERE price < 50 AND price < 150;`
  ```
  "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
  "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
  "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
  ```

- Get the details for `TV` name
  - `SELECT * FROM products WHERE name = 'TV';`
  ```
  "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
  ```

- Get the details for the 1, 2, 3 ID
  - `SELECT * FROM products WHERE id IN (1, 2, 3);`
  ```
  "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
  "DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
  "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
  ```

- `SELECT * FROM products;`
```
"DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
"remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
"microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
"pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
"sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
"TV Blue"	400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
"TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
"TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
```

- Get all the data where name starts with `TV`
  - `SELECT * FROM products WHERE name LIKE 'TV%';`
  ```
  "TV Blue"	400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
  "TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
  "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
  ```

- Reorder any column using **ORDER** keyword
  - `SELECT * FROM products ORDER BY price;`
    ```
    "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
    "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
    "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
    "DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
    "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
    "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
    "TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
    "TV Blue"	400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
    ```
  - `SELECT * FROM products ORDER BY price DESC;`
    ```
    "TV Blue"	400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
    "TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
    "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
    "DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
    "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
    "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
    "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
    "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
    ```

- Multiple re-ordering:
  - `SELECT * FROM products ORDER BY price DESC, price DESC;`
  ```
  "TV Blue"	400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
  "TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
  "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
  "DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
  "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
  "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
  "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
  "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
  ```
 
- Limiting the result
  - `SELECT * FROM products LIMIT 5;`
    ```
    "DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
    "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
    "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
    "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
    "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
    ```
  - `SELECT * FROM products WHERE price > 100 LIMIT 5;`
    ```
    "TV Blue"	400	9	false	100	"2021-11-12 22:31:44.872235+05:30"
    "TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
    "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
    ```

- Limiting but removing some result
  - `SELECT * FROM products ORDER BY id LIMIT 5;`
    ```
    "TV"	200	1	false	300	"2021-11-12 20:30:50.551514+05:30"
    "DVD Player"	80	2	false	0	"2021-11-12 20:30:50.551514+05:30"
    "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
    "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
    "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
    ```
  - `SELECT * FROM products ORDER BY id LIMIT 5 OFFSET 2;` 
      ```
      "remote"	80	3	false	0	"2021-11-12 20:30:50.551514+05:30"
      "microphone"	30	5	false	0	"2021-11-12 20:30:50.551514+05:30"
      "pencil"	2	6	false	0	"2021-11-12 20:30:50.551514+05:30"
      "sharpner"	4	7	false	0	"2021-11-12 20:30:50.551514+05:30"
      "TV Red"	300	8	false	200	"2021-11-12 22:31:27.299919+05:30"
      ```

  ![Screenshot 2024-06-06 at 00 47 53](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/3188ffd3-a505-4a8d-8d6c-c8eb87d12344)

![Screenshot 2024-06-06 at 00 48 14](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/cb2b1952-1d34-4ef4-aa90-4b4d4b5d564a)

![Screenshot 2024-06-06 at 00 48 32](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/68e2070a-73b9-4427-ba4c-1153b7b83503)



![Screenshot 2024-06-27 at 20 13 12](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/520ecebb-0ac1-487f-b3dd-f16a59d7ff74)

![Screenshot 2024-06-27 at 20 13 32](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/1e6e8862-c301-497d-bf07-b03f85fe6124)

![Screenshot 2024-06-27 at 20 13 51](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/0ce0a5d6-612d-47e0-a31c-b4c3b1d23116)

![Screenshot 2024-06-27 at 20 14 42](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/dc7e2e9c-3bfe-44e2-96aa-3a846c5ea1fa)
![Screenshot 2024-06-27 at 20 15 01](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/9d6c2762-8bed-4112-9fea-857b874342bb)

![Screenshot 2024-06-27 at 20 18 33](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/686dbb6f-486e-4370-92f4-df475c662cda)

![Screenshot 2024-06-27 at 20 18 44](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/1acd90e4-024f-407a-8734-1245e5139069)


 
 ## Adding a record
 
 Standard `INSERT INTO products () VALUES ()`
 
 - Adding a new row `INSERT INTO products (name, price, inventory) VALUES ('mobile', 100, 50);`
 - Verify it `SELECT * FROM products WHERE name='mobile';`
   ```
   "mobile"	100	10	false	50	"2021-11-12 22:52:27.478787+05:30"
   ```
- Above we following two steps, but if we want to insert and print as well

  ```
  INSERT INTO products (name, price, inventory) VALUES ('car', 1000, 50) RETURNING *;
  ```
  ```
  "car"	1000	11	false	50	"2021-11-12 22:56:11.583032+05:30"
  ```

- Inserting multiple rows at a time

`INSERT INTO products (name, price, inventory) VALUES ('car', 1000, 50), ('bat', 150, 50) RETURNING *;`
 
 
 ## DELETION
 
 - Deleting a id=12
   `DELETE FROM products WHERE id=12;`
   
 - `DELETE FROM products WHERE id=13 RETURNING *;`


## UPDATION

- `UPDATE products SET name='airpod', price=60 WHERE id = 10 RETURNING *;`
  `"airpod"	60	10	false	50	"2021-11-12 22:52:27.478787+05:30"`
  
- `UPDATE products SET is_sale =true WHERE id > 8 RETURNING *;`
   ```
   "TV Blue"	400	9	true	100	"2021-11-12 22:31:44.872235+05:30"
    "airpod"	60	10	true	50	"2021-11-12 22:52:27.478787+05:30"
    "car"	1000	11	true	50	"2021-11-12 22:56:11.583032+05:30"
   ```











