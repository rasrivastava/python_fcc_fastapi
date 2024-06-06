# Schema Model

![Screenshot 2024-06-06 at 16 04 03](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/62da1b83-d968-4d2e-be64-f079f1e95640)


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

- Now, updating other methods

```
from typing import Optional, List
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


@apptest.get("/posts", response_model=List[schemas.Post])
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

@apptest.get("/posts/{id}", response_model=schemas.Post)
def get_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id).first()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return post


@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id)

    if post.first() == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post.delete(synchronize_session=False)
    db.commit()
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@apptest.put("/posts/{id}", response_model=schemas.Post)
def update_post(id: int, updated_post: schemas.PostCreate, db: Session = Depends(get_db)):
    post_query = db.query(models.Post).filter(models.Post.id == id)
    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post_query.update(updated_post.dict(), synchronize_session=False)
    db.commit()
    return post_query.first()
```
- for the getting the all the post we have used the `List` datatype as we have multiple results.

## Next topic: user registration

- created a new model

- models.py
```
class User(Base):
    __tablename__ = "users"

    email = Column(String, nullable=False, unique=True)
    password = Column(String, nullable=False)
    id = Column(Integer, primary_key=True, nullable=False)
    created_at = Column(TIMESTAMP(timezone=True),
                        nullable=False,
                        server_default=text('now()'))
```

- schemas.py

```
class UserCreate(BaseModel):
    email: EmailStr
    password: str
```

- main.py

```
@apptest.post("/users", status_code=status.HTTP_201_CREATED)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    new_user = models.User(**user.dict())
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user
```

- testing it: `http://127.0.0.1:8000/users` (POST)

- input

```
{
    "email": "b@gmail.com",
    "password": "b@gmail.com"
}
```

- output

```
{
    "email": "b@gmail.com",
    "password": "b@gmail.com",
    "id": 1,
    "created_at": "2021-11-16T20:49:26.976379+05:30"
}
```

- **NOTE:** we just want to return id and username, so to handle this we need to add a class to the scheamas

- schemas.py

```
class UserOut(BaseModel):
    id: int
    email: EmailStr
    class Config: # to support conveting into dict
        orm_mode = True
```

- main.py add `response_model=schemas.UserOut`

```
@apptest.post("/users", status_code=status.HTTP_201_CREATED, response_model=schemas.UserOut)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    new_user = models.User(**user.dict())
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user
```

- Now, testing it:

- `http://127.0.0.1:8000/users` (POST)
```
{
    "email": "c@gmail.com",
    "password": "c@gmail.com"
}
```

- it only returns id and emailid

```
{
    "id": 2,
    "email": "c@gmail.com"
}
```


## Hashing the password

- Installed the package `pip install "passlib[bcrypt]"` (https://fastapi.tiangolo.com/es/tutorial/security/oauth2-jwt/#install-passlib)

- add below lines to the main.py 
```
    hashed_password = pwd_context.hash(user.password)
    user.password = hashed_password
```


```
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto") # what is hashing algo to be used

...
...
...

@apptest.post("/users", status_code=status.HTTP_201_CREATED, response_model=schemas.UserOut)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    # hash the password - user.password
    hashed_password = pwd_context.hash(user.password)
    user.password = hashed_password

    new_user = models.User(**user.dict())
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user
```

### code management

- we can create a new file `utils.py` and perform the operation related to the hash

```
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto") # what is hashing algo to be used

def hash(password: str):
    return pwd_context.hash(password)
```

- updating the main.py

```
...
...
from . import models, schemas, utils

...
...

@apptest.post("/users", status_code=status.HTTP_201_CREATED, response_model=schemas.UserOut)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    hashed_password = utils.hash(user.password)
    user.password = hashed_password

    new_user = models.User(**user.dict())
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user

```

- main.py (complete)

```
"""
from typing import Optional, List
from fastapi import FastAPI, Response, status, HTTPException
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

import psycopg2
from psycopg2.extras import RealDictCursor

import time

#from fastapi import app

from . import models, schemas, utils

from .database import engine, get_db
from sqlalchemy.orm import Session
from fastapi import Depends
"""

from typing import Optional, List
from fastapi import FastAPI, Response, status, HTTPException
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

import psycopg2
from psycopg2.extras import RealDictCursor

import time

from . import models, schemas, utils
from .database import engine, get_db
from sqlalchemy.orm import Session
from fastapi import Depends


models.Base.metadata.create_all(bind=engine)

apptest = FastAPI()

@apptest.get("/")
def printMessage():
    return "Welcome Family"


@apptest.get("/posts", response_model=List[schemas.Post])
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

@apptest.get("/posts/{id}", response_model=schemas.Post)
def get_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id).first()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return post


@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db)):
    post = db.query(models.Post).filter(models.Post.id == id)

    if post.first() == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post.delete(synchronize_session=False)
    db.commit()
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@apptest.put("/posts/{id}", response_model=schemas.Post)
def update_post(id: int, updated_post: schemas.PostCreate, db: Session = Depends(get_db)):
    post_query = db.query(models.Post).filter(models.Post.id == id)
    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")

    post_query.update(updated_post.dict(), synchronize_session=False)
    db.commit()
    return post_query.first()


@apptest.post("/users", status_code=status.HTTP_201_CREATED, response_model=schemas.UserOut)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    hashed_password = utils.hash(user.password)
    user.password = hashed_password

    new_user = models.User(**user.dict())
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user
```

## Get one user by its ID

```
@apptest.get("/users/{id}")
def get_user(id: int, db: Session = Depends(get_db)):
    user = db.query(models.User).filter(models.User.id == id).first()
    if not user:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"user with id: {id} was not found")
    return user
```

- `http://127.0.0.1:8000/users/1` (GET)

```
{
    "email": "b@gmail.com",
    "password": "b@gmail.com",
    "id": 1,
    "created_at": "2021-11-16T20:49:26.976379+05:30"
}
```

- here, we can use the responce model

```
@apptest.get("/users/{id}", response_model=schemas.UserOut)
def get_user(id: int, db: Session = Depends(get_db)):
    user = db.query(models.User).filter(models.User.id == id).first()
    if not user:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"user with id: {id} was not found")
    return user

```
- `http://127.0.0.1:8000/users/1` (GET) to remove password paramter

```
{
    "id": 1,
    "email": "b@gmail.com",
    "created_at": "2021-11-16T20:49:26.976379+05:30"
}
```
