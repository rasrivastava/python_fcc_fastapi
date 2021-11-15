# Schema Model

- There can be time when we don't want to return all the proporties to the user and get need to show them specific properties basically the responce or return 

- So, to handle this, we will updating the 

- schemas.md
```
from pydantic import BaseModel

class PostBase(BaseModel):
    title: str
    content: str
    published: bool = True

class PostCreate(PostBase):
    pass

class Post(PostBase):
    title: str
    content: str
    published: bool
    
    class Config: # to support conveting into dict
        orm_mode = True
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

from . import models, schemas
from .database import engine, get_db
from sqlalchemy.orm import Session
from fastapi import Depends


models.Base.metadata.create_all(bind=engine)

apptest = FastAPI()

@apptest.get("/")
def printMessage():
    return "Welcome Family"


@apptest.get("/posts")
def get_posts(db: Session = Depends(get_db)):
    posts = db.query(models.Post).all()
    return posts


@apptest.post("/posts", status_code=status.HTTP_201_CREATED, response_model=schemas.Post)
def create_posts(post: schemas.PostCreate, db: Session = Depends(get_db)):
    new_post = models.Post(**post.dict())
    db.add(new_post)
    db.commit()
    db.refresh(new_post)
    return new_post
```
- `response_model=schemas.Post` adding above

- postmane `http://127.0.0.1:8000/posts` (POST)
- passing/input

```
{
    "title": "top foods in varanasi",
    "content": "check out"
}
```
- output 
```
{
    "title": "top foods in varanasi",
    "content": "check out",
    "published": true
}
```
- **NOTE:** here are only seeing three fields returning (responce) "title", "content" and "published" as mentioned in the schema

```
class Post(PostBase):
    title: str
    content: str
    published: bool
    
    class Config: # to support conveting into dict
        orm_mode = True
```

- Now, adding two parameter, id and created_at as it will be required to store this data aswell

```
from sqlalchemy import orm
from pydantic import BaseModel
from datetime import datetime

class PostBase(BaseModel):
    title: str
    content: str
    published: bool = True

class PostCreate(PostBase):
    pass

class Post(PostBase):
    id: int
    title: str
    content: str
    published: bool
    created_at: datetime
    
    class Config: # to support conveting into dict
        orm_mode = True
```
- Now let's again create it `http://127.0.0.1:8000/posts` (POST)
- input
```
{
    "title": "top foods in varanasi",
    "content": "check out"
}
```
- output
```
{
    "title": "top foods in varanasi",
    "content": "check out",
    "published": true,
    "id": 8,
    "created_at": "2021-11-15T23:32:47.592033+05:30"
}
```

- we can reduce the code as we have inherited the class and it has many data's already, we can remove the paramters which are already declated in the parent class

```class Post(PostBase):
    id: int
    created_at: datetime
```

```
from sqlalchemy import orm
from pydantic import BaseModel
from datetime import datetime

class PostBase(BaseModel):
    title: str
    content: str
    published: bool = True

class PostCreate(PostBase):
    pass

class Post(PostBase):
    id: int
    created_at: datetime
    
    class Config: # to support conveting into dict
        orm_mode = True

```
