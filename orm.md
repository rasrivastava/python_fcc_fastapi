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
from .database import engine, SessionLocal
from sqlalchemy.orm import Session
from fastapi import Depends

models.Base.metadata.create_all(bind=engine)

apptest = FastAPI()

# Dependency
def get_db(): # get a session to the database
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@apptest.get("/sqlalchemy")
def test_posts(db: Session = Depends(get_db)):
    return {"status": "success"}
```

- Now, finally running `uvicorn app.main:apptest --reload` to test the below things
  - Connection is successfull
  - Table with the name `posts` is created 
