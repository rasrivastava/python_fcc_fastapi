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

- more better to handle the code error is using **status**

```
from fastapi import FastAPI, Response, status

...
...

@apptest.get("/posts/{id}")
def get_post(id: int, response: Response):
    post = find_post(id)
    if not post:
        response.status_code = status.HTTP_404_NOT_FOUND
    return {"post_details": f"Here is post {post}"}

```

- we can make much better user experience by returning a proper message

```
@apptest.get("/posts/{id}")
def get_post(id: int, response: Response):
    post = find_post(id)
    if not post:
        response.status_code = status.HTTP_404_NOT_FOUND
        return {"message": f"post with id: {id} was not found"}
    return {"post_details": f"Here is post {post}"}
```

`http://127.0.0.1:8000/posts/5` --> 404 error code
```
{
    "message": "post with id: 4 was not found"
}
```

- making it more clearner by using **HTTPException**

```
from fastapi import FastAPI, Response, status, HTTPException

...
...

@apptest.get("/posts/{id}")
def get_post(id: int):
    post = find_post(id)
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return {"post_details": f"Here is post {post}"}
```

## The default status code is always 200, we can customize as per oue requirement

- By using `status_code=status.HTTP_201_CREATED`

```
@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post):
    post_dict = post.dict()
    post_dict['id'] = randrange(0, 1000000)
    my_posts.append(post_dict)
    return {"data": post_dict}
```

## Delete operation

```
def find_index_post(id):
    for i, p in enumerate(my_posts):
        if p["id"] == id:
            return i

@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int):
    index = find_index_post(id)
    my_posts.pop(index)
    return {"message": "post was successfully deleted"}
```

- before deletion operation, get the data --> http://127.0.0.1:8000/posts (GET)

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

- applying the delete operation --> http://127.0.0.1:8000/posts/1 (DELETE)

```
{
    "message": "post was successfully deleted"
}
```

- checking the data after the delete operation

```
    "data": [
        {
            "title": "title of post 2",
            "content": "content of post 2",
            "id": 2
        }
    ]
}
```

- as 204 does not return anything, we need to update the return type

```
@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int):
    index = find_index_post(id)
    my_posts.pop(index)
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

- there is a glict we need to handle, when a id does not exists, we should inform

```
@apptest.delete("/posts/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int):
    index = find_index_post(id)
    if index == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    my_posts.pop(index)
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

## Update Operations

- Update --> PUT/PATCH --> /posts/:id --> @app.put("/posts/{id}")

```
@apptest.put("/posts/{id}")
def update_post(id: int, post: Post): # be sure the post comes in the right schema
    print(post)
    return {"message": "updated post"}
```

- postman --> http://127.0.0.1:8000/posts/1 (**PUT**)

```
{
    "title": "updated title",
    "content": "content of post 1"
}
```

- CLI
```
INFO:     Application startup complete.
title='updated title' content='content of post 1' published=True rating=None
INFO:     127.0.0.1:54687 - "PUT /posts/1 HTTP/1.1" 200 OK
```

- updating the code

```
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

- complete code

```
from typing import Optional
from fastapi import FastAPI, Response, status, HTTPException
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


@apptest.post("/posts", status_code=status.HTTP_201_CREATED)
def create_posts(post: Post):
    post_dict = post.dict()
    post_dict['id'] = randrange(0, 1000000)
    my_posts.append(post_dict)
    return {"data": post_dict}

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

- `http://127.0.0.1:8000/posts` (GET) before update

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

- `http://127.0.0.1:8000/posts/1` - (GET) before update

```
{
    "post_details": "Here is post {'title': 'title of post 1', 'content': 'content of post 1', 'id': 1}"
}
```

- applying `update` method `http://127.0.0.1:8000/posts/1`

```
{
    "title": "updated title by PUT method",
    "content": "content of post 1"
}
```

- `http://127.0.0.1:8000/posts/1` - (GET) After update

```
{
    "data": {
        "title": "updated title by PUT method",
        "content": "content of post 1",
        "published": true,
        "rating": null,
        "id": 1
    }
}
```

- `http://127.0.0.1:8000/posts` (GET) after update

```
{
    "data": [
        {
            "title": "updated title by PUT method",
            "content": "content of post 1",
            "published": true,
            "rating": null,
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

## Fast API uses SWAGGER API to create a documents

`http://127.0.0.1:8000/docs`
