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

## Testing POST method

```
@apptest.get("/posts")
def get_posts():
    cusor.execute(""" SELECT * FROM posts """)
    posts = cusor.fetchall()
    print(posts)
    return {"data": my_posts}
```

- hit the `get` method from postmane --> `http://127.0.0.1:8000/posts` (GET)

- cmd output

```
INFO:     Application startup complete.
[RealDictRow([('id', 1), ('title', 'first post'), ('content', 'test for the post 1'), ('published', True), ('create_at', datetime.datetime(2021, 11, 12, 23, 12, 13, 607099, tzinfo=datetime.timezone(datetime.timedelta(seconds=19800))))]), RealDictRow([('id', 2), ('title', 'second post'), ('content', 'test for the post 1'), ('published', True), ('create_at', datetime.datetime(2021, 11, 12, 23, 12, 13, 607099, tzinfo=datetime.timezone(datetime.timedelta(seconds=19800))))])]
INFO:     127.0.0.1:51851 - "GET /posts HTTP/1.1" 200 OK
```

- updating the return value

```
@apptest.get("/posts")
def get_posts():
    cusor.execute(""" SELECT * FROM posts """)
    posts = cusor.fetchall()
    return {"data": posts}
```

- hit the `get` method from postmane --> `http://127.0.0.1:8000/posts` (GET)
- output on postman

```
{
    "data": [
        {
            "id": 1,
            "title": "first post",
            "content": "test for the post 1",
            "published": true,
            "create_at": "2021-11-12T23:12:13.607099+05:30"
        },
        {
            "id": 2,
            "title": "second post",
            "content": "test for the post 1",
            "published": true,
            "create_at": "2021-11-12T23:12:13.607099+05:30"
        }
    ]
}
```

## Testing POST method

- normal variable method

```
@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post):
    post_dict = post.dict()
    post_dict['id'] = randrange(0, 1000000)
    my_posts.append(post_dict)
    return {"data": my_posts}
```

```
@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post):
    cusor.execute(""" INSERT INTO posts (title, content, published) VALUES (%s, %s, %s) RETURNING * """, (post.title, post.content, post.published))
    new_post = cusor.fetchone()
    return {"data": new_post}
```

- postman `http://127.0.0.1:8000/posts` (POST)

- data given
```
{
    "title": "this is my new post using db conn",
    "content": "check out"
}
```

- output on postman

```
{
    "data": {
        "id": 5,
        "title": "this is my new post using db conn",
        "content": "check out",
        "published": true,
        "create_at": "2021-11-13T00:08:49.871733+05:30"
    }
}
```

- get all the posts: `get` method from postmane --> `http://127.0.0.1:8000/posts` (GET)

```
{
    "data": [
        {
            "id": 1,
            "title": "first post",
            "content": "test for the post 1",
            "published": true,
            "create_at": "2021-11-12T23:12:13.607099+05:30"
        },
        {
            "id": 2,
            "title": "second post",
            "content": "test for the post 1",
            "published": true,
            "create_at": "2021-11-12T23:12:13.607099+05:30"
        },
        {
            "id": 5,
            "title": "this is my new post using db conn",
            "content": "check out",
            "published": true,
            "create_at": "2021-11-13T00:08:49.871733+05:30"
        }
    ]
}
```









