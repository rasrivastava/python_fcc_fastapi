https://pydantic-docs.helpmanual.io/

```
Data validation and settings management using python type annotations.

pydantic enforces type hints at runtime, and provides user friendly errors when data is invalid.

Define how data should be in pure, canonical python; validate it with pydantic.
```


```
class Post(BaseModel):
    title: str
    content: str
```

```
@apptest.post("/createposts")
def create_posts(new_post: Post):
    print(new_post)
    print(new_post.title)
    return {"data": "new post"}
# data stricture --> title str; content str

```

- complete code
```
from fastapi import FastAPI
from fastapi.params import Body
from pydantic import BaseModel

apptest = FastAPI()

"""
BaseModel class of the pydantic module will help to authenticate
or validate that the client inputs two things:
    1) title
    2) content
"""
class Post(BaseModel):
    title: str
    content: str

@apptest.get("/")
def printMessage():
    return "Welcome Family"


@apptest.get("/posts")
def printLogin():
    return {"data": "this is your post"}


@apptest.post("/createposts")
def create_posts(new_post: Post):
    print(new_post)
    print(new_post.title)
    return {"data": "new post"}
# data stricture --> title str; content str
```

- Now, sending the data from Postman
```
http://127.0.0.1:8000/createposts --> POST
```

- terminal

```
INFO:     Application startup complete.
title='top foods in varanasi' content='check out'
top foods in varanasi
INFO:     127.0.0.1:58925 - "POST /createposts HTTP/1.1" 200 OK
```
