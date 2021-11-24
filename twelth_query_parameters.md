# Query Paramaters

## Get only one post

`db.query(models.Post).limit(limit).all()`

```
@router.get("/", response_model=List[schemas.Post])
def get_posts(db: Session = Depends(get_db),
              current_user: int = Depends(oauth2.get_curent_user),
              limit: int = 100):
    # this will return all the posts from all the users
    #posts = db.query(models.Post).all()
    # if we want to get the posts from the logged in user
    #posts = db.query(models.Post).filter(models.Post.owner_id == current_user.id).all()
    
    posts = db.query(models.Post).limit(limit).all()
    
    return posts
```
<img width="1017" alt="Screenshot 2021-11-24 at 23 38 13" src="https://user-images.githubusercontent.com/11652564/143292160-85b838a2-d321-4e3d-8dc8-fe3d13807b77.png">

## skip parameter

```
@router.get("/", response_model=List[schemas.Post])
def get_posts(db: Session = Depends(get_db),
              current_user: int = Depends(oauth2.get_curent_user),
              limit: int = 100, # default is 100
              skip: int = 0): # don't skip any by default

    posts = db.query(models.Post).limit(limit).offset(skip).all()
    
    return posts
```

<img width="1020" alt="Screenshot 2021-11-24 at 23 43 18" src="https://user-images.githubusercontent.com/11652564/143292740-8263b306-063f-4a43-9bda-85154742e999.png">


## search a word in the title

```
@router.get("/", response_model=List[schemas.Post])
def get_posts(db: Session = Depends(get_db),
              current_user: int = Depends(oauth2.get_curent_user),
              limit: int = 100, # default is 100
              skip: int = 0, # don't skip any by default
              search: Optional[str] = ""):
    
    posts = db.query(models.Post).filter(models.Post.title.contains(search)).limit(limit).offset(skip).all()
    
    return posts
```
<img width="1016" alt="Screenshot 2021-11-24 at 23 46 54" src="https://user-images.githubusercontent.com/11652564/143293223-25b515b6-aca8-4d34-b4ad-9b1b844ed76e.png">

