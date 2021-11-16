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
    conn.commit()
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

## Get a single post

- normal

```
def find_post(id):
    for p in my_posts:
        if p["id"] == id:
            return p

@apptest.get("/posts/{id}")
def get_post(id: int):
    post = find_post(id)
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}
```

- using db

```
@apptest.get("/posts/{id}")
def get_post(id: int):
    #post = find_post(id)
    cusor.execute(""" SELECT * FROM posts WHERE id=%s """, (id))
    post = cusor.fetchone()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}
```

- it throws an error `TypeError: 'int' object does not support indexing`, we need to convert `id` to string as its currently int and sql query takes as string.

```
def find_post(id):
    for p in my_posts:
        if p["id"] == id:
            return p

@apptest.get("/posts/{id}")
def get_post(id: int):
    #post = find_post(id)
    cusor.execute(""" SELECT * FROM posts WHERE id=%s """, (str (id)))
    post = cusor.fetchone()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}
```

- output --> `http://127.0.0.1:8000/posts/1` GET

```
{
    "post_details": "Here is post RealDictRow([('id', 1), ('title', 'first post'), ('content', 'test for the post 1'), ('published', True), ('create_at', datetime.datetime(2021, 11, 12, 23, 12, 13, 607099, tzinfo=datetime.timezone(datetime.timedelta(seconds=19800))))])"
}
```

- there are some issue seen when we don't put "," after the argument provided -> `cusor.execute(""" SELECT * FROM posts WHERE id=%s """, (str (id),))`

```
def find_post(id):
    for p in my_posts:
        if p["id"] == id:
            return p

@apptest.get("/posts/{id}")
def get_post(id: int):
    #post = find_post(id)
    cusor.execute(""" SELECT * FROM posts WHERE id=%s """, (str (id),))
    post = cusor.fetchone()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}
```

## DELETION

```
def find_index_post(id):
    for i, p in enumerate(my_posts):
        if p["id"] == id:
            return i

@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int):
    index = find_index_post(id)
    if index == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    my_posts.pop(index)
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

- API code

```
@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int):
    cusor.execute(""" DELETE FROM posts WHERE id = %s RETURNING * """, (str(id),))
    delete_post = cusor.fetchone()
    conn.commit()

    if delete_post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

- Getting all the data

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

- Deleting the id:2 --> http://127.0.0.1:8000/posts/2 (**DELETE**)
  - Nothing will be returned 
- Getting the get the data: verify, we will the id: 2 is got deleted
```
{
    "data": [
        {
            "id": 1,
            "title": "first post",
            "content": "test for the post 1",
            "published": true,
            "create_at": "2021-11-12T23:12:13.607099+05:30"
        }
    ]
}
```

- If we again try to delete the id: 2, we will an error which we have handled --> http://127.0.0.1:8000/posts/2 (DELETE)

```
{
    "detail": "post with id: 2 was not found"
}
```

## UPDATE Operation

- Normal without using an API

```
def find_index_post(id):
    for i, p in enumerate(my_posts):
        if p["id"] == id:
            return i
 
 
@apptest.put("/posts/{id}") # user is going to 'PUT' a request on a particular ID to which update is required
def update_post(id: int, post: Post): # be sure the post comes in the right schema
    index = find_index_post(id)
    if index == None: # if it doesnot exits then throw an exception
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    # if its exits
    post_dict = post.dict() # get all the data from user related to update in a dict form
    post_dict['id'] = id # addind 'id' so that final dict has an id
    my_posts[index] = post_dict # replace with post_dict
    return {"data": post_dict}

```

- Using API

```
@apptest.put("/posts/{id}") # user is going to 'PUT' a request on a particular ID to which update is required
def update_post(id: int, post: Post): # be sure the post comes in the right schema
    cusor.execute(""" UPDATE posts SET title=%s, content=%s, published=%s WHERE id = %s RETURNING * """,
                   (post.title, post.content, post.published, str(id)))
    updated_post = cusor.fetchone()
    conn.commit()
    if update_post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"data": updated_post}

```

- before update: http://127.0.0.1:8000/posts (GET)

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
            "id": 6,
            "title": "updated title by PUT method",
            "content": "content of post",
            "published": true,
            "create_at": "2021-11-14T19:17:18.443175+05:30"
        }
    ]
}
```

-updating `http://127.0.0.1:8000/posts/6` (PUT)

```
{
    "title": "[UPDATED] updated title by PUT method",
    "content": "[UPDATED] content of post"
}
```

- verify it

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
            "id": 6,
            "title": "[UPDATED] updated title by PUT method",
            "content": "[UPDATED] content of post",
            "published": true,
            "create_at": "2021-11-14T19:17:18.443175+05:30"
        }
    ]
}
```

## Complete Code

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


@apptest.get("/")
def printMessage():
    return "Welcome Family"


@apptest.get("/posts")
def get_posts():
    cusor.execute(""" SELECT * FROM posts """)
    posts = cusor.fetchall()
    return {"data": posts}


@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post):
    cusor.execute(""" INSERT INTO posts (title, content, published) VALUES (%s, %s, %s) RETURNING * """, (post.title, post.content, post.published))
    new_post = cusor.fetchone()
    conn.commit()
    return {"data": new_post}

@apptest.get("/posts/{id}")
def get_post(id: int):
    cusor.execute(""" SELECT * FROM posts WHERE id=%s """, (str (id),))
    post = cusor.fetchone()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}


@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int):
    cusor.execute(""" DELETE FROM posts WHERE id = %s RETURNING * """, (str(id),))
    delete_post = cusor.fetchone()
    conn.commit()

    if delete_post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@apptest.put("/posts/{id}")
def update_post(id: int, post: Post):
    cusor.execute(""" UPDATE posts SET title=%s, content=%s, published=%s WHERE id = %s RETURNING * """,
                   (post.title, post.content, post.published, str(id)))
    updated_post = cusor.fetchone()
    conn.commit()
    if update_post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"data": updated_post}
```
