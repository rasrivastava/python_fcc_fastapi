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

## Retriving one individual post

```
@apptest.get("/posts/{id}")  # id : path parameter
def get_post(id):
    print(id)
    return {"post_details": f"Here is post {id}"}
```

- postman --> `http://127.0.0.1:8000/posts/2` (GET)
  - Get the request
  ```
  {
    "post_details": "Here is post 2"
  }
  ```

- creating a function to handle it better

```
def find_post(id):
    for p in my_posts:
        if p["id"] == id:
            return p

@apptest.get("/posts/{id}")  # id : path parameter
def get_post(id):
    post = find_post(int(id))
    return {"post_details": f"Here is post {post}"}
```

- postman --> `http://127.0.0.1:8000/posts/2` (GET)
```
{
    "post_details": "Here is post {'title': 'title of post 2', 'content': 'content of post 2', 'id': 2}"
}
```
- but it will create a issue and return internal server 500 error when we pass a string, this can be handled with the fastapi

```
@apptest.get("/posts/{id}")  # id : path parameter
def get_post(id: int):
    post = find_post(id)
    return {"post_details": f"Here is post {post}"}
```

- Now, if user provides a string this, it will generate a exception which can be handled.

## Here we have a problem, suppose a user searches for id: 5 which does not exit, with the current logic it will return null everywhere it is used with status code "200" which might confuse

`http://127.0.0.1:8000/posts/5` --> 200 error code

```
{
    "post_details": "Here is post None"
}
```

- Handling to code to return **404** error

```
from fastapi import FastAPI, Response

...
...


@apptest.get("/posts/{id}")
def get_post(id: int, response: Response):
    post = find_post(id)
    if not post:
        response.status_code = 404
    return {"post_details": f"Here is post {post}"}
```

`http://127.0.0.1:8000/posts/5` --> 404 error code
