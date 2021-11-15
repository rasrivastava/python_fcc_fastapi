# Schema Model

- There can be time when we don't want to return all the proporties to the user and get need to show them specific properties basically the responce or return 

- So, to handle this, we will updating the 

- schemas.md
```
from pydantic import BaseModel

class PostBase(BaseModel):
    title: str
    content: str
    published: bool = True

class PostCreate(PostBase):
    pass

```
