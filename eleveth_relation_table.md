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
