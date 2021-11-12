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


## Connecting the database using fastapi

```
from fastapi import FastAPI
from pydantic import BaseModel

import psycopg2
from psycopg2.extras import RealDictCursor

class Post(BaseModel):
    title: str
    content: str
    published: bool = True


try:
    conn = psycopg2.connect(host="localhost",
                            database="fastapi",
                            user="postgres",
                            password="redhat",
                            cursor_factory=RealDictCursor)
    cusor = conn.cursor()
    print("Database connection is successfull")
except Exception as error:
    print("Connecting to database failed")
    print("Error: ", error)
```

- output

```
# uvicorn app.main:apptest --reload       ✔  fastapi   system   23:45:14  
INFO:     Will watch for changes in these directories: ['/Users/rasrivas/personal/python_freecodecamp_sanjeev/fastapi']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [2618] using watchgod
Database connection is successfull <---------------------
INFO:     Started server process [2624]
```

- much better ways

```
from typing import Optional
from fastapi import FastAPI, Response, status, HTTPException
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

import psycopg2
from psycopg2.extras import RealDictCursor

import time

apptest = FastAPI()


class Post(BaseModel):
    title: str
    content: str
    published: bool = True

while True:
    try:
        conn = psycopg2.connect(host="localhost",
                                database="fastapi",
                                user="postgres",
                                password="redhat",
                                cursor_factory=RealDictCursor)
        cusor = conn.cursor()
        print("Database connection is successfull")
        break
    except Exception as error:
        print("Connecting to database failed")
        print("Error: ", error)
        time.sleep(2) # retry every 2 sec
```
