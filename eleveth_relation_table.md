# Relationship between tables

- current we don't have any relation between the `User` and `Posts` table

![Screenshot 2021-11-20 at 13 37 13](https://user-images.githubusercontent.com/11652564/142719310-e58957df-94c8-4235-974a-ea7b4dc1e455.png)

![Screenshot 2021-11-20 at 13 37 46](https://user-images.githubusercontent.com/11652564/142719328-dfda83df-50c4-4a6d-b3b9-e2921111b41d.png)


7:55:04


<img width="1320" alt="Screenshot 2021-11-20 at 17 09 28" src="https://user-images.githubusercontent.com/11652564/142724973-b1b23cf5-b105-4231-a84b-b023f017a736.png">

<img width="1328" alt="Screenshot 2021-11-20 at 17 09 42" src="https://user-images.githubusercontent.com/11652564/142724983-c262bb88-b975-4bbb-9b18-1e553cc79b87.png">

<img width="934" alt="Screenshot 2021-11-20 at 17 12 33" src="https://user-images.githubusercontent.com/11652564/142725073-800a1fff-f320-4175-96ba-d6cd23946ac8.png">

<img width="1335" alt="Screenshot 2021-11-20 at 17 12 47" src="https://user-images.githubusercontent.com/11652564/142725083-166335e4-7d2d-42be-9b1e-c94314359905.png">

<img width="970" alt="Screenshot 2021-11-20 at 17 13 08" src="https://user-images.githubusercontent.com/11652564/142725093-063b3554-760f-471c-99c4-b49bf433dc9a.png">

- **CASCADE** --> delete the primary key table row as well when the for
<img width="932" alt="Screenshot 2021-11-20 at 17 14 42" src="https://user-images.githubusercontent.com/11652564/142725151-2153a974-f586-471a-934a-9c5b2e361de2.png">
<img width="1069" alt="Screenshot 2021-11-20 at 17 16 00" src="https://user-images.githubusercontent.com/11652564/142725181-32be614d-77b8-488b-bbd8-9142e540f7a1.png">

- adding a row in the post able by following the foreign key rule
 
<img width="1117" alt="Screenshot 2021-11-20 at 17 18 48" src="https://user-images.githubusercontent.com/11652564/142725246-217f4bdc-1925-4ef4-bb08-bf26937f04c9.png">

## now we will be how we can do in pgadmin
- deleting the foreign column
 
<img width="1106" alt="Screenshot 2021-11-20 at 17 21 05" src="https://user-images.githubusercontent.com/11652564/142725304-5a455b9c-60c0-475d-98dd-44c071e614e0.png">

```
    owner_id = Column(Integer, ForeignKey("users.id", ondelete="CASCADE"), nullable=False)
    # (tabl_name.column_name, ondelete="")
```
- complete code
```
import sqlalchemy
from sqlalchemy import Column, Integer, String
from sqlalchemy.sql.expression import text
from sqlalchemy.sql.schema import ForeignKey
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
    owner_id = Column(Integer, ForeignKey("users.id", ondelete="CASCADE"), nullable=False)
    # (tabl_name.column_name, ondelete="")

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, nullable=False)
    email = Column(String, nullable=False, unique=True)
    password = Column(String, nullable=False)
    created_at = Column(TIMESTAMP(timezone=True),
                        nullable=False,
                        server_default=text('now()'))
```

- now we need to restart the app, but still we will not see the table has added that row, as we have earlier this issue, we need to entirely delete the complete row and re-run the app

<img width="1132" alt="Screenshot 2021-11-20 at 17 28 36" src="https://user-images.githubusercontent.com/11652564/142725474-6e60e434-40ad-462f-ba87-2464e4a8c3ae.png">


``8:14``

# As we have a relation between the two table

- Now, **owner_id** will be required while posting a post
- this is can be done as, the user will be only able to post when its logged in, so we can use the owner id here

- updating the schemas `schemas.py`

```
class Post(PostBase):
    id: int
    created_at: datetime
    owner_id: int
    
    class Config: # to support conveting into dict
        orm_mode = True
```

- updating the post create `new_post = models.Post(owner_id=current_user.id, **post.dict())`

- `post.py`

```
@router.post("/", status_code=status.HTTP_201_CREATED, response_model=schemas.Post)
def create_posts(post: schemas.PostCreate, db: Session = Depends(get_db),
                 current_user: int = Depends(oauth2.get_curent_user)):
    new_post = models.Post(owner_id=current_user.id, **post.dict())
    db.add(new_post)
    db.commit()
    db.refresh(new_post)
    return new_post
```
<img width="1038" alt="Screenshot 2021-11-23 at 20 25 12" src="https://user-images.githubusercontent.com/11652564/143047453-544cbe55-503c-426b-b6f1-e3f52e59ab30.png">


# updating delete and update option so that only logged user can perform the actions

```
@router.delete("/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db),
                current_user: int = Depends(oauth2.get_curent_user)):
    post_query = db.query(models.Post).filter(models.Post.id == id)

    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    
    if post.owner_id != current_user.id:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN)

    post_query.delete(synchronize_session=False)
    db.commit()
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```


```
@router.put("/{id}", response_model=schemas.Post)
def update_post(id: int, updated_post: schemas.PostCreate, db: Session = Depends(get_db),
                current_user: int = Depends(oauth2.get_curent_user)):
    post_query = db.query(models.Post).filter(models.Post.id == id)
    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    
    if post.owner_id != current_user.id:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN)

    post_query.update(updated_post.dict(), synchronize_session=False)
    db.commit()
    return post_query.first()
```

- **post.py**

```
from fastapi import FastAPI, Response, status, HTTPException, Depends, APIRouter
from sqlalchemy.orm import Session
from typing import List
from .. import models, schemas, oauth2
from ..database import get_db

router = APIRouter(
    prefix="/posts",
    tags=["Posts"]
)

@router.get("/", response_model=List[schemas.Post])
def get_posts(db: Session = Depends(get_db), current_user: int = Depends(oauth2.get_curent_user)):
    posts = db.query(models.Post).all()
    return posts


@router.post("/", status_code=status.HTTP_201_CREATED, response_model=schemas.Post)
def create_posts(post: schemas.PostCreate, db: Session = Depends(get_db),
                 current_user: int = Depends(oauth2.get_curent_user)):
                 # this will force the user is logged in
    print(current_user.email)
    print(current_user.id)
    print("========================================================")
    #new_post = models.Post(**post.dict())
    new_post = models.Post(owner_id=current_user.id, **post.dict())
    db.add(new_post)
    db.commit()
    db.refresh(new_post)
    return new_post

@router.get("/{id}", response_model=schemas.Post)
def get_post(id: int, db: Session = Depends(get_db), current_user: int = Depends(oauth2.get_curent_user)):
    post = db.query(models.Post).filter(models.Post.id == id).first()
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    return post


@router.delete("/{id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_post(id: int, db: Session = Depends(get_db),
                current_user: int = Depends(oauth2.get_curent_user)):
    post_query = db.query(models.Post).filter(models.Post.id == id)

    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    
    if post.owner_id != current_user.id:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN)

    post_query.delete(synchronize_session=False)
    db.commit()
    return Response(status_code=status.HTTP_204_NO_CONTENT)


@router.put("/{id}", response_model=schemas.Post)
def update_post(id: int, updated_post: schemas.PostCreate, db: Session = Depends(get_db),
                current_user: int = Depends(oauth2.get_curent_user)):
    post_query = db.query(models.Post).filter(models.Post.id == id)
    post = post_query.first()

    if post == None:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"post with id: {id} was not found")
    
    if post.owner_id != current_user.id:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN)

    post_query.update(updated_post.dict(), synchronize_session=False)
    db.commit()
    return post_query.first()
```

## Get the posts only created by logged user

```
@router.get("/", response_model=List[schemas.Post])
def get_posts(db: Session = Depends(get_db), current_user: int = Depends(oauth2.get_curent_user)):
    # this will return all the posts from all the users
    #posts = db.query(models.Post).all()
    # if we want to get the posts from the logged in user
    posts = db.query(models.Post).filter(models.Post.owner_id == current_user.id).all()
    return posts
```
