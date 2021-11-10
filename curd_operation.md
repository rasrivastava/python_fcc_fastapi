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

