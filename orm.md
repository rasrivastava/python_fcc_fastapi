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
    posts = db.query(models.Post).all()
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
