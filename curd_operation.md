# CURD Operations

- ***Create*** --> **POST** -->  ***/posts*** --> ***@app.post("/posts")***
- ***Read*** --> **GET** --> ***/posts/:id*** OR ***/posts*** --> ***@app.get("/posts/{id}")*** OR ***@app.get("/posts")***
- ***Update*** --> **PUT/PATCH** --> ***/posts/:id*** --> ***@app.put("/posts/{id}")***
- ***Delete*** --> **DELETE** --> ***/posts/:id*** --> ***@app.delete("/posts/{id}")***


- Below example uses the best practice

```
@apptest.get("/posts")
def printLogin():
    return {"data": "this is your post"}


@apptest.post("/posts")
def create_posts(post: Post):
    print(post)
    print(post.dict()) # converting to dict data type using pydantic model
    return {"data": post} # python dic

```

### Get method

```
my_posts = [{"title": "title of post 1", "content": "content of post 1", "id": 1},
            {"title": "title of post 2", "content": "content of post 2", "id": 2}
           ] # post objects (static data later we will store in database)
```

```
@apptest.get("/posts")
def printLogin():
    return {"data": my_posts} # prints the static data from the my_posts variable
```

- postman output `http://127.0.0.1:8000/posts` --> GET method

```
{
    "data": [
        {
            "title": "title of post 1",
            "content": "content of post 1",
            "id": 1
        },
        {
            "title": "title of post 2",
            "content": "content of post 2",
            "id": 2
        }
    ]
}
```

## Creating a POST method operation

```
my_posts = [{"title": "title of post 1", "content": "content of post 1", "id": 1},
            {"title": "title of post 2", "content": "content of post 2", "id": 2}
           ] # post objects (static data later we will store in database)
```

```
@apptest.post("/posts")
def create_posts(post: Post):
    post_dict = post.dict()
    post_dict['id'] = randrange(0, 1000000)
    my_posts.append(post_dict)
    return {"data": post_dict}
```

- Complete code

```
from typing import Optional
from fastapi import FastAPI
from fastapi.params import Body
from pydantic import BaseModel
from random import randrange

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
    published: bool = True # if we don't provide this value this default will be "True"
    rating: Optional[int] = None # if nothing will be provided it will be take none

my_posts = [{"title": "title of post 1", "content": "content of post 1", "id": 1},
            {"title": "title of post 2", "content": "content of post 2", "id": 2}
           ] # post objects (static data later we will store in database)

@apptest.get("/")
def printMessage():
    return "Welcome Family"


@apptest.get("/posts")
def printLogin():
    return {"data": my_posts} # prints the static data from the my_posts variable


@apptest.post("/posts")
def create_posts(post: Post):
    post_dict = post.dict()
    post_dict['id'] = randrange(0, 1000000)
    my_posts.append(post_dict)
    return {"data": post_dict}
```

- **Postman console**
  - `http://127.0.0.1:8000/posts` ==> POST
  - Sending the request
    ```
    {
        "data": {
            "title": "top foods in varanasi",
            "content": "check out",
            "published": true,
            "rating": 5,
            "id": 224461
        }
    }
    ```
   - Now, getting request from `http://127.0.0.1:8000/posts` ==> GET
   - Before
       ```
    {
        "data": [
            {
                "title": "title of post 1",
                "content": "content of post 1",
                "id": 1
            },
            {
                "title": "title of post 2",
                "content": "content of post 2",
                "id": 2
            }
        ]
    }
    ```
   - After, we will see `"title": "top foods in varanasi",` related block will be added:
    ```
    {
        "data": [
            {
                "title": "title of post 1",
                "content": "content of post 1",
                "id": 1
            },
            {
                "title": "title of post 2",
                "content": "content of post 2",
                "id": 2
            },
            {
                "title": "top foods in varanasi",
                "content": "check out",
                "published": true,
                "rating": 5,
                "id": 224461
            }
        ]
    }
    ```






