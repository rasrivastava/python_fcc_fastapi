## The simplest FastAPI file could look like this:

```
from fastapi import FastAPI

app = FastAPI() # FastAPI instance name

@app.get("/") # decorator "@"; FastAPI instance name "app"; HTTP request method "get"; root path "/" 
def printMessage(): # function name
    return "Hello World" # return to the user

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

```
The command uvicorn main:app refers to:

main: the file main.py (the Python "module").
app: the object created inside of main.py with the line app = FastAPI().
--reload: make the server restart after code changes. Only use for development.
```

- http://127.0.0.1:8000/
```
{"message":"Hello World"}
```

```
from fastapi import FastAPI

app = FastAPI() # FastAPI instance name

@app.get("/") # decorator "@"; FastAPI instance name "app"; HTTP request method "get"; root path "/" 
def printMessage(): # function name
    return "Hello World" # return to the user


@app.get("/login") # # path "/login" -> is the path which has to gone in the URL
def printLogin(): # function name
    return "Logged In" # return to the user

```

- http://127.0.0.1:8000/login
```
"Logged In"
```

## changing the fastAPI instance name to "apptest"

```
from fastapi import FastAPI

apptest = FastAPI() # FastAPI instance name "apptest"

@apptest.get("/") # decorator "@"; FastAPI instance name "apptest"; HTTP request method "get"; root path "/" 
def printMessage(): # function name
    return "Hello World" # return to the user


@apptest.get("/login") # path "/login" -> is the path which has to gone in the URL
def printLogin(): # function name
    return "Logged In" # return to the user

```

```
 uvicorn main:apptest --reload                   ✔  fastapi   22:54:30  
INFO:     Will watch for changes in these directories: ['/Users/rasrivas/personal/python_freecodecamp_sanjeev/fastapi']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [14141] using watchgod
INFO:     Started server process [14143]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     127.0.0.1:56415 - "GET /login HTTP/1.1" 200 OK
INFO:     127.0.0.1:56416 - "GET / HTTP/1.1" 200 OK
```
