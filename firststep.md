## The simplest FastAPI file could look like this:

```
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

```
uvicorn main:app --reload                           ✔  fastapi   22:37:57  
INFO:     Will watch for changes in these directories: ['/Users/rasrivas/personal/python_freecodecamp_sanjeev/fastapi']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [13923] using watchgod
INFO:     Started server process [13927]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     127.0.0.1:56259 - "GET / HTTP/1.1" 200 OK
```

- http://127.0.0.1:8000/
```
{"message":"Hello World"}
```
