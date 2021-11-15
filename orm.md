# Object Relational Mapper (ORM)

## What ORM can do for us:

- Instead of manually defining the tables in the postgres, we can define our tables as python models
- Queries can be made exclusively through python code, No SQL is necessary.


## sqlalchemy

- https://docs.sqlalchemy.org/en/14/
- https://docs.sqlalchemy.org/en/14/tutorial/index.html
- https://fastapi.tiangolo.com/tutorial/sql-databases/

`pip install sqlalchemy`

## Structure of the project

```
# ls app
__init__.py database.py main.py     models.py
```

- **models.py**: declare the table name and related colums here
- **datavase.py**: declare the database connections

- models.py
```
import sqlalchemy
from sqlalchemy import Column, Integer, String
from sqlalchemy.sql.sqltypes import Boolean
from .database import Base

class Post(Base):
    __tablename__ = "posts" # table name
    
    # below are the column names
    id = Column(Integer, primary_key=True, nullable=False)
    title = Column(String, nullable=False)
    content = Column(String, nullable=False)
    published = Column(Boolean, default=True)
```

- database.py

```
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker


#SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
#SQLALCHEMY_DATABASE_URL = "postgresql://<user_name>:<password>@<?ip_add/<hostname>/<db>"
SQLALCHEMY_DATABASE_URL = "postgresql://postgres:redhat@localhost/fastapi"

engine = create_engine(SQLALCHEMY_DATABASE_URL)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base() # base class

# Dependency
def get_db(): # get a session to the database
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

- main.py

```
from typing import Optional
from fastapi import FastAPI, Response, status, HTTPException
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

import psycopg2
from psycopg2.extras import RealDictCursor

import time

from . import models
from .database import engine, get_db
from sqlalchemy.orm import Session
from fastapi import Depends


models.Base.metadata.create_all(bind=engine)

apptest = FastAPI()

@apptest.get("/sqlalchemy")
def test_posts(db: Session = Depends(get_db)):
    return {"status": "success"}
```

- Now, finally running `uvicorn app.main:apptest --reload` to test the below things
  - Connection is successfull
  - Table with the name `posts` is created 

- `default=True` will not work, we need to set as `published = Column(Boolean, server_default='TRUE', default=True)`.
  - Also `sqlalchemy` has a limitation here, if we update the details of the existing table, it will not update the changes, we need to recreated it complettly.
  - For more details, please check https://fastapi.tiangolo.com/tutorial/sql-databases/#alembic-note

- Deleting the table and also updating for the timestamp

```
import sqlalchemy
from sqlalchemy import Column, Integer, String
from sqlalchemy.sql.expression import text
from sqlalchemy.sql.sqltypes import TIMESTAMP, Boolean
from .database import Base

class Post(Base):
    __tablename__ = "posts" # table name
    
    # below are the column names
    id = Column(Integer, primary_key=True, nullable=False)
    title = Column(String, nullable=False)
    content = Column(String, nullable=False)
    published = Column(Boolean, server_default='TRUE', nullable=False)
    created_at = Column(TIMESTAMP(timezone=True),
                        nullable=False,
                        server_default=text('now()'))

```
- now, it will be created correctly

## Get Method

```
main.py
-------
from typing import Optional
from fastapi import FastAPI, Response, status, HTTPException
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

import psycopg2
from psycopg2.extras import RealDictCursor

import time

from . import models
from .database import engine, get_db
from sqlalchemy.orm import Session
from fastapi import Depends


models.Base.metadata.create_all(bind=engine)

apptest = FastAPI()

@apptest.get("/sqlalchemy")
def test_posts(db: Session = Depends(get_db)):
    posts = db.query(models.Post).all() # explanation below
    return {"posts": posts}
    
database.py
-----------
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "postgresql://postgres:redhat@localhost/fastapi"

engine = create_engine(SQLALCHEMY_DATABASE_URL)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base() # base class

# Dependency
def get_db(): # get a session to the database query and close the connection once completed
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

- query - `http://127.0.0.1:8000/sqlalchemy` (GET)

```
{
    "posts": [
        {
            "title": "first post",
            "published": true,
            "content": "first post",
            "created_at": "2021-11-14T23:28:03.429537+05:30",
            "id": 2
        }
    ]
}
```

- If we want to more know about the `posts = db.query(models.Post).all()`, we can print the `posts = db.query(models.Post)` this data and we will acutally see behind the uses the SQL

```
@apptest.get("/sqlalchemy")
def test_posts(db: Session = Depends(get_db)):
    #posts = db.query(models.Post).all()
    posts = db.query(models.Post)
    print(posts)
    return {"posts": "okay"}
```

- cli output

```
INFO:     127.0.0.1:56630 - "GET /posts HTTP/1.1" 200 OK
SELECT posts.id AS posts_id, posts.title AS posts_title, posts.content AS posts_content, posts.published AS posts_published, posts.created_at AS posts_created_at 
FROM posts
INFO:     127.0.0.1:56646 - "GET /sqlalchemy HTTP/1.1" 200 OK
```

`SELECT posts.id AS posts_id, posts.title AS posts_title, posts.content AS posts_content, posts.published AS posts_published, posts.created_at AS posts_created_at 
FROM posts`

## Fetching the GET posts table data

- before

```
@apptest.get("/posts")
def get_posts():
    cusor.execute(""" SELECT * FROM posts """)
    posts = cusor.fetchall()
    return {"data": posts}
```
- Now
```
@apptest.get("/posts")
def get_posts(db: Session = Depends(get_db)):
    posts = db.query(models.Post).all()
    return {"data": posts}
```

- verifying `http://127.0.0.1:8000/posts` (GET)
```
{
    "data": [
        {
            "title": "first post",
            "published": true,
            "created_at": "2021-11-14T23:28:03.429537+05:30",
            "id": 2,
            "content": "first post"
        }
    ]
}
```

## POST operation

- before
```
@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post):
    cusor.execute(""" INSERT INTO posts (title, content, published) VALUES (%s, %s, %s) RETURNING * """, (post.title, post.content, post.published))
    new_post = cusor.fetchone()
    conn.commit()
    return {"data": new_post}
```

- new

```
@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post, db: Session = Depends(get_db)):   
    new_post = models.Post(title=post.title, content=post.content, published=post.published)
    db.add(new_post) # add the post
    db.commit() # commit it
    db.refresh(new_post) ## retrive the new post
    return {"data": new_post}
```

- create post `http://127.0.0.1:8000/posts` (POST)
- data input
```
{
    "title": "[ORM] top foods in varanasi",
    "content": "check out",
    "published": true,
    "rating": 5
}
```

- output

```
{
    "data": {
        "title": "[ORM] top foods in varanasi",
        "published": true,
        "content": "check out",
        "id": 3,
        "created_at": "2021-11-15T19:57:22.514406+05:30"
    }
}
```

- verify the result `http://127.0.0.1:8000/posts` (GET)

```
{
    "data": [
        {
            "title": "first post",
            "published": true,
            "content": "first post",
            "id": 2,
            "created_at": "2021-11-14T23:28:03.429537+05:30"
        },
        {
            "title": "[ORM] top foods in varanasi",
            "published": true,
            "content": "check out",
            "id": 3,
            "created_at": "2021-11-15T19:57:22.514406+05:30"
        }
    ]
}
```

- here, `new_post = models.Post(title=post.title, content=post.content, published=post.published)` can be pain sometime, we can use `new_post = models.Post(**post.dict())`

```
@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post, db: Session = Depends(get_db)):
    new_post = models.Post(**post.dict())
    db.add(new_post) # add the post
    db.commit() # commit it
    db.refresh(new_post) ## retrive the new post
    return {"data": new_post}
```

## Get a single post

```
@apptest.get("/posts/{id}")
def get_post(id: int):
    cusor.execute(""" SELECT * FROM posts WHERE id=%s """, (str (id),))
    post = cusor.fetchone()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}
```

```
@apptest.get("/posts/{id}")
def get_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id).first() # get the first one not all which check for all the post
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": post}
```

- `http://127.0.0.1:8000/posts/4` (get)

```
{
    "post_details": {
        "published": true,
        "title": "[UPDATED ORM] top foods in varanasi",
        "created_at": "2021-11-15T20:03:14.032177+05:30",
        "id": 4,
        "content": "check out"
    }
}
```

## Delete operation

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

```
@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id)

    if post.first() == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post.delete(synchronize_session=False)
    db.commit()
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

- `http://127.0.0.1:8000/posts/4` (DELETE)
- `http://127.0.0.1:8000/posts` (Get), we will see id 4 related row is deleted
```
{
    "data": [
        {
            "title": "first post",
            "published": true,
            "content": "first post",
            "id": 2,
            "created_at": "2021-11-14T23:28:03.429537+05:30"
        },
        {
            "title": "[ORM] top foods in varanasi",
            "published": true,
            "content": "check out",
            "id": 3,
            "created_at": "2021-11-15T19:57:22.514406+05:30"
        }
    ]
}
```


## Updated POST

```
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

- now

```
@apptest.put("/posts/{id}")
def update_post(id: int, updated_post: Post, db: Session = Depends(get_db)):
    post_query = db.query(models.Post).filter(models.Post.id == id)
    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post_query.update(updated_post.dict(), synchronize_session=False)
    db.commit()
    return {"data": post_query.first()}
```

- query `http://127.0.0.1:8000/posts/3` (put)

```
{
    "data": {
        "published": true,
        "title": "rasrivas.....updated title by PUT method",
        "id": 3,
        "created_at": "2021-11-15T19:57:22.514406+05:30",
        "content": "content of post"
    }
}
```

- verify  `http://127.0.0.1:8000/posts` (GET), it gets updated

```
{
    "data": [
        {
            "published": true,
            "title": "first post",
            "id": 2,
            "created_at": "2021-11-14T23:28:03.429537+05:30",
            "content": "first post"
        },
        {
            "published": true,
            "title": "rasrivas.....updated title by PUT method",
            "id": 3,
            "created_at": "2021-11-15T19:57:22.514406+05:30",
            "content": "content of post"
        }
    ]
}
```

## Complete code

```
from typing import Optional
from fastapi import FastAPI, Response, status, HTTPException
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

import psycopg2
from psycopg2.extras import RealDictCursor

import time

from . import models
from .database import engine, get_db
from sqlalchemy.orm import Session
from fastapi import Depends


models.Base.metadata.create_all(bind=engine)

apptest = FastAPI()

@apptest.get("/sqlalchemy")
def test_posts(db: Session = Depends(get_db)):
    posts = db.query(models.Post).all()
    print(posts)
    return {"posts": "okay"}

class Post(BaseModel):
    title: str
    content: str
    published: bool = True


@apptest.get("/")
def printMessage():
    return "Welcome Family"


@apptest.get("/posts")
def get_posts(db: Session = Depends(get_db)):
    posts = db.query(models.Post).all()
    return {"data": posts}


@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post, db: Session = Depends(get_db)):
    new_post = models.Post(**post.dict())
    db.add(new_post) # add the post
    db.commit() # commit it
    db.refresh(new_post) ## retrive the new post
    return {"data": new_post}

@apptest.get("/posts/{id}")
def get_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id).first() # get the first one not all which check for all the post
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": post}


@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id)

    if post.first() == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post.delete(synchronize_session=False)
    db.commit()
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@apptest.put("/posts/{id}")
def update_post(id: int, updated_post: Post, db: Session = Depends(get_db)):
    post_query = db.query(models.Post).filter(models.Post.id == id)
    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post_query.update(updated_post.dict(), synchronize_session=False)
    db.commit()
    return {"data": post_query.first()}
```
