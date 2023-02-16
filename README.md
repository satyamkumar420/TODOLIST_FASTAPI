## TODOLIST FASTAPI

### First, make sure to install the required packages using pip:

```   
pip install fastapi uvicorn pymongo python-dotenv
```
`Then, create a new file called "main.py" and add the following code:`

```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from pymongo import MongoClient
from bson.objectid import ObjectId
from dotenv import load_dotenv
import os

load_dotenv()

app = FastAPI()
client = MongoClient(os.getenv("MONGO_URI"))

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def get_db():
    db = client["todo"]
    yield db
    client.close()

@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    if form_data.username != os.getenv("USERNAME") or form_data.password != os.getenv("PASSWORD"):
        raise HTTPException(status_code=400, detail="Incorrect username or password")
    return {"access_token": "secret_token"}

@app.get("/todos")
def read_todos(db = Depends(get_db), token: str = Depends(oauth2_scheme)):
    todos = []
    for todo in db.todos.find():
        todo["_id"] = str(todo["_id"])
        todos.append(todo)
    return todos

@app.post("/todos")
def create_todo(todo: dict, db = Depends(get_db), token: str = Depends(oauth2_scheme)):
    result = db.todos.insert_one(todo)
    todo["_id"] = str(result.inserted_id)
    return todo

@app.get("/todos/{id}")
def read_todo(id: str, db = Depends(get_db), token: str = Depends(oauth2_scheme)):
    todo = db.todos.find_one({"_id": ObjectId(id)})
    if todo:
        todo["_id"] = str(todo["_id"])
        return todo
    else:
        raise HTTPException(status_code=404, detail="Todo not found")

@app.put("/todos/{id}")
def update_todo(id: str, todo: dict, db = Depends(get_db), token: str = Depends(oauth2_scheme)):
    result = db.todos.replace_one({"_id": ObjectId(id)}, todo)
    if result.modified_count == 1:
        todo["_id"] = id
        return todo
    else:
        raise HTTPException(status_code=404, detail="Todo not found")

@app.delete("/todos/{id}")
def delete_todo(id: str, db = Depends(get_db), token: str = Depends(oauth2_scheme)):
    result = db.todos.delete_one({"_id": ObjectId(id)})
    if result.deleted_count == 1:
        return {"message": "Todo deleted"}
    else:
        raise HTTPException(status_code=404, detail="Todo not found")






```



### This code defines a FastAPI app that has the following endpoints:

* /token: A POST endpoint that allows a user to authenticate and receive an access token.
* /todos: A GET endpoint that returns a list of all todos, and a POST endpoint that creates a new todo.
* /todos/{id}: A GET endpoint that retrieves a specific todo by ID, a PUT endpoint that updates a todo by ID, and a DELETE endpoint that deletes a todo by ID.

#### To run the app, use the following command:
```
   uvicorn main:app --reload
```

#### This will start the app on http://localhost:8000. You can use tools like Postman or curl to interact with the API.

