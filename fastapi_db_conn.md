- Created a DB with the below rows;
- Added two rows

```
INSERT INTO public.posts (
title, content) VALUES (
'second post'::character varying, 'test for the post 1'::character varying)
 returning id;
```

```
INSERT INTO public.posts (
title, content) VALUES (
'first post'::character varying, 'test for the post 1'::character varying)
 returning id;
```

## Psycopg 2.9.2 documentation

- Psycopg is the most popular PostgreSQL database adapter for the Python programming language.

- Install `pip install psycopg2-binary`

