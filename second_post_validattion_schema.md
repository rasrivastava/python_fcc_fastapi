# Why we need a schema

![Screenshot 2024-06-05 at 17 40 27](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/592e5eba-c3cc-41a8-88c1-21e328174228)


# Validation from POST method

### Using the validattion we can be sure the data is verified and what is required to process the job

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

`http://127.0.0.1:8000/createposts --> POST method`

```
{
    "title": "top foods in varanasi",
    "content": "check out"
}
```

- terminal

```
INFO:     Application startup complete.
title='top foods in varanasi' content='check out'
top foods in varanasi
INFO:     127.0.0.1:58925 - "POST /createposts HTTP/1.1" 200 OK
```

- suppose if we only send the below data

```
"title": "top foods in varanasi",
```
- it will through an error on Postman as well on CLI --> **status 422** and it says that **"msg"** is missing

```
{
    "detail": [
        {
            "loc": [
                "body",
                "title"
            ],
            "msg": "field required",
            "type": "value_error.missing"
        }
    ]
}
```

- CMD line: `INFO:     127.0.0.1:58948 - "POST /createposts HTTP/1.1" 422 Unprocessable Entity`

## adding an additional data which is optinal and if not provided the the default value will be taken

`published: bool = True`

```
class Post(BaseModel):
    title: str
    content: str
    published: bool = True # if we don't provide this value this default will be "True"
```

```
{
    "title": "top foods in varanasi",
    "content": "check out",
    "published": true
}
```

```
INFO:     Application startup complete.
title='top foods in varanasi' content='check out' published=True
True
INFO:     127.0.0.1:59008 - "POST /createposts HTTP/1.1" 200 OK
```

- if we don't provide the `published` value then also it will return the True as it a default value set

```
{
    "title": "top foods in varanasi",
    "content": "check out"
}
```

```
INFO:     Application startup complete.
title='top foods in varanasi' content='check out' published=True
True
INFO:     127.0.0.1:59133 - "POST /createposts HTTP/1.1" 200 OK
```

## adding an optinal data and if not provided then the this value will be skipped

```
class Post(BaseModel):
    title: str
    content: str
    published: bool = True # if we don't provide this value this default will be "True"
    rating: Optional[int] = None # if nothing will be provided it will be take none --> `from typing import Optional`

...
...

@apptest.post("/createposts")
def create_posts(new_post: Post):
    print(new_post)
    print(new_post.rating) ## printing it
    return {"data": "new post"}

```
![Screenshot 2024-06-05 at 18 04 20](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/7db7226e-3707-40cb-a9ed-932e2613d333)


![Screenshot 2024-06-05 at 18 03 54](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/c9562191-7b5d-40fd-8fc5-48be627b2e5f)
![Screenshot 2024-06-05 at 18 04 06](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/8691cd14-d396-4884-8753-c5712f391c40)




![Screenshot 2024-06-05 at 18 07 07](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/55a60415-6b3a-421d-828c-db7acf2433cf)

![Screenshot 2024-06-05 at 18 07 20](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/9713fe59-6b05-4f51-bcb1-dbc3bdc2929f)

![Screenshot 2024-06-05 at 18 07 28](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/eb6b65cd-6eb1-485e-a9c0-892887838940)

![Screenshot 2024-06-05 at 18 08 06](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/4b259c2d-a874-46d8-8a93-a20445e0a003)


```
{
    "title": "top foods in varanasi",
    "content": "check out",
    "published": true
}
```

```
INFO:     Application startup complete.
title='top foods in varanasi' content='check out' published=True rating=None
None
INFO:     127.0.0.1:59193 - "POST /createposts HTTP/1.1" 200 OK
```

- if provided the **rating**

```
{
    "title": "top foods in varanasi",
    "content": "check out",
    "published": true,
    "rating": 5
}
```

```
title='top foods in varanasi' content='check out' published=True rating=5
5
INFO:     127.0.0.1:59210 - "POST /createposts HTTP/1.1" 200 OK
```

## converting to dict data type using pydantic model

```
@apptest.post("/createposts")
def create_posts(new_post: Post):
    print(new_post) # python dic
    print(new_post.dict()) # converting to dict data type using pydantic model
    return {"data": new_post} # python dic
# data stricture --> title str; content str

```

```
INFO:     Application startup complete.
title='top foods in varanasi' content='check out' published=True rating=5
{'title': 'top foods in varanasi', 'content': 'check out', 'published': True, 'rating': 5}
INFO:     127.0.0.1:59263 - "POST /createposts HTTP/1.1" 200 OK
```
![Screenshot 2024-06-05 at 17 58 41](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/81def120-f153-4676-a3e0-c8c02d7efb1c)



![Screenshot 2024-06-05 at 17 58 20](https://github.com/rasrivastava/python_fcc_fastapi/assets/11652564/065c3f33-7405-4a22-869a-b5e0d024c3ce)

